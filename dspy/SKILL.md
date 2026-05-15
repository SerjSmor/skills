---
name: dspy
description: Use when setting up, refactoring, or iterating on a DSPy experiment workflow, especially when you need to choose an optimizer, structure an experiment folder, name iterations consistently, or log compile/eval tokens, cost, and runtime separately. This skill is for agent-assisted DSPy experimentation rather than plain prompt-only work.
---

# DSPy

## Overview

Use this skill for DSPy experiment loops where your coding agent is expected to behave like an experiment engineer: clarify the task, choose an optimizer, structure the experiment folder, run bounded iterations, and keep metrics and accounting clean.

Start with a short interview before changing code. Ask only the minimum needed to determine:

- task type: classification, extraction, ranking, generation, tool use
- optimization target: accuracy, `macro_f1`, latency, cost, or another metric
- budget: cheap baseline, moderate tuning, or broad search
- allowed models: student, teacher, reflection
- evaluation protocol: train / val / test split and whether test is locked

If the user has not chosen an optimizer yet, use [references/optimizer-choice.md](references/optimizer-choice.md).

If the user needs token or cost accounting, use [references/compile-accounting.md](references/compile-accounting.md).

## Defaults

Use these defaults unless the user overrides them:

- start with a cheap baseline using `LabeledFewShot`
- keep the main DSPy experiment script close to the experiment it belongs to
- keep experiment-local outputs close to that script:
  - `results.tsv`
  - `runs/`
  - `programs/`
- log each run to Weights & Biases when available
- encode iteration identity in `run_name` and artifact prefixes, not by copying the script into many variants
- the agent may subsample train or validation examples to control what enters compilation, but it must log the policy clearly and keep final evaluation on the intended held-out split

Recommended run-name style:

- `lfs-baseline-iter-01`
- `miprov2-iter-02`
- `gepa-iter-03`

If a repo already has a shared or root-level `train_dspy.py`, prefer creating an experiment-local owned surface for the active workflow instead of editing many unrelated entrypoints.

## Folder Rules

For a new DSPy experiment surface:

1. Create or confirm a dedicated place for the experiment implementation.
2. Keep `train_dspy.py` with that experiment surface.
3. Keep outputs local to that surface.
4. Keep shared data and shared metrics helpers outside the experiment-owned surface.

Preferred layout:

```text
<experiment-surface>/
  train_dspy.py
  results.tsv
  runs/
  programs/
```

Do not create a new script per iteration. The script is the evolving implementation; iteration identity belongs in:

- `run_name`
- `wandb_job_type`
- output filenames
- result rows

## Optimizer Workflow

Follow this order unless the user asks otherwise:

1. Interview briefly.
2. Establish a `LabeledFewShot` baseline.
3. Confirm the evaluation metric and locked split.
4. Pick the next optimizer based on problem shape and budget.
5. Run one bounded change at a time.
6. Keep notes on what changed between iterations.

Do not jump straight to a heavy optimizer unless there is a clear reason.

## Naming And Logging

Every run should log:

- `run_name`
- `optimizer_name`
- student model
- teacher model if any
- reflection model if any
- metric objective
- compile runtime
- eval runtime
- compile tokens
- eval tokens
- compile cost estimate
- eval cost estimate
- total cost estimate

Primary logging surfaces:

- local `results.tsv` for structured run history
- Weights & Biases for dashboards, grouped comparisons, and artifact links
- local `runs/` and `programs/` directories for per-run outputs

Preferred columns:

- `compile_prompt_tokens`
- `compile_completion_tokens`
- `compile_total_tokens`
- `eval_prompt_tokens`
- `eval_completion_tokens`
- `eval_total_tokens`
- `estimated_compile_cost_usd`
- `estimated_eval_cost_usd`
- `estimated_total_run_cost_usd`

If a project already has a single ambiguous `estimated_cost_usd` field, split it instead of continuing to overload it.

## Iteration Discipline

For agentic DSPy work, each iteration should change one of:

- optimizer
- optimizer budget
- signature shape
- instruction prior
- demo selection strategy
- compilation subset or example-selection policy
- metric objective
- model allocation across student / teacher / reflection

Avoid bundling multiple unrelated changes unless the user explicitly wants a bigger jump.

## Example Selection

The agent is allowed to choose or subsample examples to improve compilation efficiency or search quality.

Allowed uses:

- smaller compilation subsets for optimizer search
- rare-class prioritization
- boundary-example prioritization
- deterministic demo candidate ordering
- separate tracking subsets for cheap optimizer guidance

Requirements:

- describe the subset policy in code or logs
- keep the split boundaries intact
- do not tune on the locked test set
- distinguish between compilation subsets and final evaluation sets

## Validation

Before presenting results:

- verify the script parses
- verify the run writes outputs to the experiment-local folder
- verify the run appends a row to `results.tsv`
- verify the run logs to Weights & Biases if that is part of the project setup
- verify compile and eval accounting are both present
- verify the run name reflects the iteration number
- state clearly whether the result is baseline, planned sweep, or adaptive iteration
