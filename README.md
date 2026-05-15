# Skills

Local Codex-compatible skills.

This repository collects reusable skills that were developed during real project work and then extracted into a cleaner standalone form.

## Skills

### `dspy`

A skill for agentic DSPy experimentation.

It helps with:
- choosing a reasonable optimizer for the problem
- starting with a cheap baseline such as `LabeledFewShot`
- deciding when to use teacher or reflection models
- structuring a local DSPy experiment surface
- naming iterations consistently
- tracking compilation and evaluation tokens, cost, and runtime separately
- keeping `results.tsv` and Weights & Biases logging disciplined

This skill was born out of the ATIS comparison work in:
- https://github.com/SerjSmor/agentic_atis

That project compared prompt-only agentic iteration, plain DSPy optimization, and agentic-on-DSPy workflows. The `dspy` skill captures the parts of that process that were reusable beyond that one repository.

## Structure

Each skill lives in its own folder and is self-contained:

- `SKILL.md`: primary instructions
- `agents/openai.yaml`: display metadata and default prompt hook
- `references/`: optional supporting notes
- `scripts/`: optional helper scripts

## Install / use

You can use this repo in two common ways.

1. Copy a skill folder into a project-local `skills/` directory.
2. Copy a skill folder into your Codex skills home so it is available across sessions.

Example:

```bash
cp -R dspy /path/to/project/skills/
```

Or:

```bash
cp -R dspy ~/.codex/skills/
```

## Repository layout

```text
README.md
 dspy/
   SKILL.md
   agents/
     openai.yaml
   references/
     optimizer-choice.md
     compile-accounting.md
```

## Notes

- Keep each skill narrowly scoped.
- Prefer reusable references over project-specific assumptions.
- Put durable workflow guidance in `SKILL.md` and keep repo-specific conventions out unless the skill is intentionally project-bound.
