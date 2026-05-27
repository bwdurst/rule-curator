# Rule Curator

A skill that audits the behavioral rules an agent operates under and helps you
keep, drop, or rewrite them.

Agents accumulate rules across many files (global and project `CLAUDE.md`, a
memory system, project docs like `AGENTS.md`, ADRs, conventions). Nobody prunes
them, so they go stale, duplicate each other, duplicate installed skills, restate
default behavior, or impose rituals whose friction exceeds their value. Every
rule is a permanent tax: it costs context tokens every session and dilutes
instruction-following attention across more directives.

Rule Curator gathers those rules, analyzes each one against the whole set,
produces a self-contained browser UI where you decide keep / drop / modify, then
turns your decisions into an auditable apply-plan and (on confirmation) edits the
source files.

## What it does

For every rule it produces two things, kept separate:

1. A neutral, first-person **behavioral impact** statement (what following the
   rule actually changes, and its cost), always present.
2. An **audit note**, only when something is wrong, drawn from a fixed taxonomy:
   redundant, stale or superseded, misfiled, low-yield ritual, not actionable,
   brittle, or conflicting.

It deliberately checks the categories that capable models skip on their own
(brittle rules, misfiled entries), and it refuses to rubber-stamp or to invent
staleness on healthy rules.

## Repo contents

- `SKILL.md` the workflow, the analysis contract, and the disciplines
- `references/analysis-contract.md` the full rubric with worked examples
- `references/apply-plan-format.md` the apply-plan shape and per-verdict edit rules
- `build_curator.py` standard-library script that builds the self-contained UI
- `curator-template.html` the UI template (data is injected at build time)
- `evals/` the planted-issue test corpus, answer key, and RED/GREEN findings
- `docs/specs/` the design spec

## Install

Copy this directory into your Claude Code skills location, for example
`~/.claude/skills/rule-curator/`, or add it through a plugin marketplace. Once
installed, invoke it by asking to audit, prune, or clean up your rules.

## Build the curation UI directly

You can also run the build step yourself once you have a `rules.json`:

```bash
python build_curator.py rules.json -o rule-curator.html
```

This injects the data into `curator-template.html` and writes one self-contained
file you can open offline. It uses the standard library only, so there is nothing
to install.

## How it was built

This skill was developed test-first. The `evals/` directory holds a synthetic
rule corpus with planted issues across every audit category, an answer key, and
the recorded baseline (without the skill) and verification (with the skill) runs
on both a strong and a weak model. See `evals/RED-baseline-findings.md` and
`evals/GREEN-verification-findings.md`.
