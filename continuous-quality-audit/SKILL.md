---
name: continuous-quality-audit
description: Use when continuously auditing model outputs to verify that they remain normal, stable, and policy-compliant over time, including sampling, drift detection, LLM-as-judge review, and storage of flagged outputs for follow-up.
---

# Continuous Quality Audit

## Overview

Use this skill when the goal is to continuously inspect a model's outputs and verify that they remain normal, stable, and acceptable over time.

This skill is for ongoing audit workflows, not one-off offline evaluation.

Start with a short interview before changing code or launching an audit. Ask only the minimum needed to determine:

- which model or pipeline is being audited
- what the output unit is:
  - label
  - score
  - JSON object
  - text generation
  - ranking
  - another structured output
- where the outputs come from:
  - logs
  - DB table
  - API traces
  - batch files
  - warehouse query
- whether the audit is:
  - retrospective
  - streaming
  - periodic batch
  - deployment-gated
- how the user defines normal behavior
- whether a baseline or reference distribution already exists
- whether drift detection is part of the goal
- whether LLM-as-judge is allowed
- where audited or flagged outputs should be saved

## Defaults

Use these defaults unless the user overrides them:

- start with a small reproducible audit slice before scaling up
- define or load a reference baseline before judging normality
- support both random and risk-based sampling
- use strong LLMs as judges only when the user allows them
- save flagged outputs to `Argilla` when human review is expected
- otherwise save flagged outputs to a DB table or structured artifact
- log audit windows, sampling policy, and baseline version explicitly
- treat bounded recurring audits as the default rather than open-ended monitoring

## Audit Modes

Choose and log the audit mode explicitly.

Common modes include:

- distribution audit
- rule-based audit
- sample-based audit
- LLM-as-judge audit
- disagreement audit
- drift detection

Use distribution audit when normality is defined by expected output frequencies or score ranges.

Use rule-based audit when outputs must satisfy schema, formatting, or business constraints.

Use sample-based audit when the user wants a direct inspection subset.

Use LLM-as-judge audit when a strong model should evaluate quality, normality, or policy adherence.

Use disagreement audit when outputs should be compared against another model, heuristic, or label source.

Use drift detection when the goal is to identify changes over time relative to a trusted baseline or comparison window.

## Reference and Baseline

If a trusted baseline already exists, load and reference it explicitly.

If no baseline exists, create one from a trusted historical slice and log:

- the source window
- the sampling rule
- the model version
- the data segment
- the summary statistics or reference artifact path

Supported reference types may include:

- expected label distribution
- expected score or confidence distribution
- schema or field frequency profile
- lexical or structural output profile
- judge-score distribution
- baseline model comparison
- historical time-window comparison

Always version or timestamp the reference used for the audit.

## Drift Detection

Drift detection is a first-class audit objective.

The agent may actively search for drift, not only passively report obvious failures.

Possible drift checks include:

- changes in label distribution
- changes in confidence or score distribution
- changes in embedding clusters
- changes in schema validity rate
- changes in field frequency
- changes in lexical or structural patterns
- changes in LLM judge score distribution
- changes across slices such as:
  - time window
  - deployment version
  - user cohort
  - geography
  - language
  - product surface

If drift is detected, log:

- the comparison window
- the baseline used
- the metric or heuristic that detected the shift
- the affected segment
- whether the drift is believed to be benign traffic shift or a model-quality issue

Do not assume every shift is a failure. Some drift reflects legitimate traffic change rather than degraded behavior.

## Sampling

Choose and log the sampling policy explicitly.

Sampling may be:

- random
- stratified
- risk-based
- uncertainty-based
- trigger-based
- drift-targeted

When appropriate, oversample:

- rare outputs
- low-confidence cases
- schema failures
- newly shifted segments
- disagreement cases

Always record:

- sample size
- sample fraction
- selection rule
- audit window

## LLM as Judge

A stronger LLM may be used as a judge when the user allows it.

Possible uses:

- scoring output quality
- checking whether outputs look normal
- detecting semantic mismatch
- categorizing failure type
- triaging examples for human review
- supporting weak interim evaluation

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

Keep judge-assisted metrics separate from human-verified metrics.

## Storage

Decide and log where audited outputs and flagged examples will be stored.

Common destinations include:

- `Argilla` dataset
- DB table
- warehouse table
- `jsonl` or `parquet` artifact
- review queue

Use `Argilla` by default when humans will review or annotate flagged outputs.

For each stored record, preserve at least:

- example identifier or source reference
- model output
- model version
- timestamp
- score or confidence if available
- audit result
- audit method
- judge result if applicable
- anomaly reason or drift flag

## Thresholds and Cadence

Define what constitutes a warning or failure before scaling up the audit.

Examples:

- invalid schema rate above threshold
- drift metric above threshold
- judge score below threshold
- policy-failure count above threshold
- confidence distribution shift above threshold

Also define cadence:

- one-time audit
- hourly
- daily
- per deployment
- post-release canary

Default to a bounded periodic audit if the user has not specified cadence.

## Outputs and Logging

Keep structured outputs for every audit run.

At minimum log:

- model or pipeline audited
- audit window
- audit mode
- baseline or reference used
- sample size
- sampling policy
- drift checks performed
- judge model if used
- anomaly count
- flagged output location
- notes on suspected failure modes

Useful files may include:

- `audit_results.tsv`
- `flagged_outputs.jsonl`
- `baseline_reference.json`
- `notes.md`

Use Weights & Biases when the project already uses it or the user wants centralized tracking.

## Failure Modes

Watch for:

- assuming normality without a reference baseline
- confusing legitimate traffic-mix change with model-quality degradation
- auditing only easy or common cases
- using LLM judge outputs without a clear rubric
- failing to version the baseline
- storing flagged outputs without enough metadata to reproduce them
- mixing human and judge outcomes into one unlabeled score
- detecting drift but not tracing it to a segment or deployment boundary

If the audit is noisy or inconclusive, inspect:

- the reference slice
- the sampling policy
- the flagged segments
- the judge rubric
- the output storage records
- whether the observed shift is operational, behavioral, or data-distribution related
