# Optimizer Choice

Use this file when the user has not already chosen a DSPy optimizer.

## Interview Questions

Ask only what changes the optimizer decision:

- Is this classification, extraction, ranking, generation, or agent/tool use?
- Is the goal a cheap baseline, a strong prompt optimizer, or a broader adaptive search?
- Is the metric accuracy-like, `macro_f1`, cost-sensitive, or latency-sensitive?
- Are teacher or reflection models allowed?
- How much search budget is acceptable?

## Default Order

1. `LabeledFewShot`
2. `MIPROv2`
3. `GEPA`
4. `BootstrapFewShotWithRandomSearch`

## When To Start With LabeledFewShot

Use `LabeledFewShot` first when:

- the user wants a fast baseline
- the task is straightforward classification or extraction
- you need to verify the split, signature, or label set
- you want a cheap sanity check before heavier search

This should usually be the first DSPy baseline.

## When To Prefer MIPROv2

Use `MIPROv2` when:

- the task benefits from instruction optimization and demo selection
- you want a structured, repeatable optimizer
- you can afford a moderate compile budget
- the experiment should remain comparable across a fixed run plan

Good default for plain DSPy benchmarking.

## When To Prefer GEPA

Use `GEPA` when:

- prompt wording and decision rules matter a lot
- error analysis suggests the system needs better task instructions, not just better demos
- reflection is allowed
- you want agentic DSPy iteration rather than a fixed planned sweep

If using `GEPA`, consider:

- stronger reflection model than student model
- smaller search tracking set than final eval set
- metric aligned with the real target, for example a macro-oriented surrogate when optimizing for `macro_f1`

## When To Prefer BootstrapFewShotWithRandomSearch

Use `BootstrapFewShotWithRandomSearch` when:

- the main search space is demos rather than instruction rewriting
- you want a broader but simpler search than `GEPA`
- the task is likely very example-sensitive

## Anti-Patterns

Do not:

- start heavy search before a baseline exists
- optimize exact match when the real goal is clearly `macro_f1` without discussing the mismatch
- compare optimizers fairly if they use different splits
- bury optimizer identity in vague run names
