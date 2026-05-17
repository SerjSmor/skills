---
name: active-learning
description: Use when building an active learning loop for low-label predictive tasks, where a model is trained on a small labeled set, used to score an unlabeled pool, and then used to select the next batch for annotation or review.
---

# Active Learning

## Overview

Use this skill when the goal is to improve a classifier or similar predictor with very limited labeled data by iterating between training, scoring, selecting examples, and annotating new batches.

This skill is for agent-assisted active learning workflows, not one-off supervised training.

Start with a short interview before changing code. Ask only the minimum needed to determine:

- how to pull the data
- whether the data is labeled, partially labeled, or unlabeled
- what the task type is:
  - classification
  - multilabel classification
  - ranking
  - extraction
  - another predictive task
- what the prediction unit is:
  - row
  - document
  - sentence
  - chunk
  - message
- what the label schema is
- what metric matters most
- whether there is a held-out validation or test set
- what the annotation budget is:
  - examples
  - rounds
  - time
  - money
- whether LLM-as-judge is allowed for weak evaluation or triage

## Defaults

Use these defaults unless the user overrides them:

- use existing labeled data if available
- if labels are missing or too small, create a small seed set first
- use `Argilla` as the default annotation tool
- start with a simple baseline model before adding more complex selection logic
- start with uncertainty-based sampling for standard single-label classification
- add diversity filtering when duplicates or near-duplicates are likely
- evaluate after every annotation round
- keep final evaluation separate from the active learning pool
- use bounded runs rather than open-ended loops

## Loop

A standard active learning loop should look like this:

1. build or confirm the initial labeled set
2. train a baseline model
3. score the unlabeled pool
4. select the next annotation batch using a defined heuristic
5. send the batch for annotation
6. merge newly labeled examples back into the labeled set
7. retrain and re-evaluate
8. repeat until stopping criteria are met

Always log what changed between rounds.

## Seed Set

When labeled data is very small:

- use existing labels if they exist
- otherwise create a small seed set first
- prefer a diverse seed set over a purely random one when possible
- avoid a seed set made entirely of majority-class examples when class imbalance is known

## Sampling Strategies

Choose and log the selection strategy explicitly.

Common strategies include:

- uncertainty sampling
- margin sampling
- entropy sampling
- diversity-aware sampling
- class-balance-aware sampling
- hybrid sampling:
  - uncertainty + diversity
  - uncertainty + heuristics
  - uncertainty + LLM triage

For standard single-label classification, uncertainty sampling is a good default.

Be careful:

- pure lowest-confidence sampling can over-select duplicates
- pure uncertainty can over-select outliers or noisy examples
- add deduplication or diversity controls when possible

## Annotation

Use `Argilla` by default when no annotation tool is specified.

When setting up annotation:

- define the dataset fields clearly
- define the label schema clearly
- preserve review metadata:
  - model prediction
  - confidence or uncertainty score
  - selection reason
  - iteration number
- keep annotation provenance by round

If the user already has another annotation system, adapt to that system instead of forcing Argilla.

## Evaluation

Prefer human-labeled validation for actual model evaluation.

Track at least:

- primary metric
- labeled set size
- unlabeled pool size
- selected batch size
- iteration number

If possible, keep a stable held-out validation set that is never reused as annotation input.

Do not report pool-selection performance as final model quality.

## LLM as Judge

A stronger LLM may be used as a judge when the user allows it.

Possible uses:

- weak interim evaluation when gold labels are unavailable
- triage for borderline examples
- disagreement analysis
- ranking or scoring candidate predictions
- adjudication support

Do not treat LLM judgment as default ground truth unless the user explicitly wants that.

If using LLM-as-judge, log:

- judge model
- judge prompt or rubric version
- whether the judge output is:
  - label
  - score
  - ranking
  - rationale
- whether it is used for:
  - evaluation
  - prioritization
  - adjudication
  - pseudo-labeling

Keep human-gold metrics and judge-assisted metrics clearly separate.

## Stopping Criteria

Define stopping criteria before running many rounds.

Examples:

- fixed number of rounds
- fixed annotation budget
- validation metric plateau
- annotator capacity limit
- time or cost cap

Default to a bounded run if the user has not specified a stopping rule.

## Outputs and Logging

Keep structured outputs for every round.

At minimum log:

- iteration number
- model or program version
- labeled set size
- unlabeled pool size
- selected batch size
- sampling strategy
- evaluation metric
- annotation destination
- notes on observed failure modes

Useful files may include:

- `results.tsv`
- `selection_round_XX.jsonl`
- `annotation_round_XX.jsonl`
- `notes.md`

Use Weights & Biases when the project already uses it or the user wants centralized tracking.

## Failure Modes

Watch for:

- duplicate-heavy candidate pools
- uncertain but low-value outliers
- class imbalance dominating selection
- label schema confusion
- evaluation leakage
- using LLM-judged metrics as if they were human-gold metrics
- repeatedly selecting near-identical examples

If performance is not improving, inspect:

- the selected examples
- annotation quality
- class distribution
- whether the selection heuristic is exploring useful boundaries
