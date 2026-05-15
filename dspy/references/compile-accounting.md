# Compile And Eval Accounting

Use this file when implementing DSPy cost and token accounting.

## Goal

Track compile and eval separately.

Do not report one blended `estimated_cost_usd` if it only covers eval.

## Recommended Approach

Use DSPy's usage tracker around compilation:

```python
from dspy import track_usage

with track_usage() as usage_tracker:
    optimized_program = optimizer.compile(...)

compile_usage_by_lm = usage_tracker.get_total_tokens()
```

Continue collecting eval usage from predictions, for example through `prediction.get_lm_usage()`.

## What To Save

Save these separately:

- compile prompt tokens
- compile completion tokens
- compile total tokens
- eval prompt tokens
- eval completion tokens
- eval total tokens
- compile runtime
- eval runtime

Then compute:

- `estimated_compile_cost_usd`
- `estimated_eval_cost_usd`
- `estimated_total_run_cost_usd`

## Important Nuance

Compile-time usage may involve multiple LMs:

- student LM
- prompt LM
- teacher LM
- reflection LM

So compile cost should be summed by actual LM name, not charged entirely to the student model.

## Pricing Pattern

Use the repo pricing map keyed by the actual model names that appear in usage tracking.

For each LM:

1. read `prompt_tokens`
2. read `completion_tokens`
3. look up input and output pricing
4. sum across all LMs

## Result Columns

Preferred TSV columns:

- `compile_prompt_tokens`
- `compile_completion_tokens`
- `compile_total_tokens`
- `eval_prompt_tokens`
- `eval_completion_tokens`
- `eval_total_tokens`
- `estimated_compile_cost_usd`
- `estimated_eval_cost_usd`
- `estimated_total_run_cost_usd`

## Reporting Rule

In summaries and READMEs, call the number exactly what it is:

- `Estimated Eval Cost` if it excludes compile
- `Estimated Compile Cost` if it covers only optimization
- `Estimated Total Run Cost` if it includes both
