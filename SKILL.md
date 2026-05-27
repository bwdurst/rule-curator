---
name: rule-curator
description: Use when reviewing, auditing, pruning, or cleaning up the behavioral rules an agent operates under (global/project CLAUDE.md, memory entries, project rule docs like AGENTS.md, ADRs, COORDINATION) to decide which rules to keep, drop, or rewrite, and to apply those decisions.
---

# Rule Curator

## Overview

Agents accumulate behavioral rules across many files. Nobody prunes them, so they
go stale, duplicate each other, duplicate installed skills, restate default
behavior, or impose rituals whose friction exceeds their value.

**Core principle: every rule is a permanent tax.** It costs context tokens every
session and dilutes instruction-following attention across more directives. A
rule earns its place only if its behavioral payoff beats that ongoing cost. This
skill audits the rule set against that bar and acts on the result.

This skill exists because a capable model, left to its own devices, finds the
*loud* defects (conflicts, staleness, duplication) but skips the *quiet* ones
(brittle rules, misfiled entries), never separates a rule's neutral effect from
the critique of it, and produces prose instead of a usable deliverable. The
workflow below forces all four.

## When to use

- "Audit / clean up / prune my CLAUDE.md / rules / memory"
- After a big refactor or decision, to find rules that went stale
- The rule set has grown and instruction-following feels diluted
- Periodic hygiene on the directives an agent operates under

**Not for:** authoring new rules (this curates existing ones), or auditing PRDs
and feature specs (those are bounded deliverable specs, not standing rules).

## Workflow

Copy this checklist and track progress:

```
Rule Curator progress:
- [ ] 1. Discover rule sources (+ enumerate installed skills and ADRs)
- [ ] 2. SCOPE GATE: confirm/narrow/expand sources + emphasis (default: all, full)
- [ ] 3. Extract rules into rules.json
- [ ] 4. Analyze each rule against the whole corpus (impact + conditional audit note)
- [ ] 5. PRE-CURATE CHECKPOINT: report "N rules, K flagged"; proceed
- [ ] 6. Build the curation UI (run build_curator.py)
- [ ] 7. Human curates in browser, exports decisions.json
- [ ] 8. Build the apply-plan from decisions.json
- [ ] 9. APPLY GATE: confirm, then execute the edits
```

### 1. Discover

A **rule** is a *standing directive that shapes behavior across tasks* (do /
never / prefer / always / use X not Y). That is the inclusion test, not a file
list. A bounded deliverable spec (PRD, feature spec) is not a rule.

Auto-detect the sources that exist and contain standing directives:
- Global `~/.claude/CLAUDE.md` and project `CLAUDE.md` (including nested)
- Memory: `MEMORY.md` plus every entry it points to
- Project rule docs: `AGENTS.md`, `COORDINATION.md`, primers, conventions
- **ADR-type decision records, however named** (`docs/adr/*`, `docs/decisions/*`,
  RFCs). Extract only the *binding constraint*, not the narrative.

Also enumerate **installed skills** (names + descriptions) and the ADRs found,
so "duplicates a skill / duplicates an ADR" findings are grounded, not guessed.
PRDs and feature specs are excluded by default; a user may add sources at the
scope gate. If nothing is found, report and stop. Never fabricate a rule set.

### 2. Scope gate (required, defaults to proceed)

Present the discovered sources and a rough rule count. Offer to confirm, narrow,
or expand the source set, and optionally to focus ("general health audit" vs
"hunting duplicates / staleness / bloat"). **If the user just says go: all
sources, full audit.** Running unattended: take that default.

### 3. Extract

Normalize each rule into the schema `build_curator.py` consumes:

```json
{
  "meta": { "title": "...", "generated": "YYYY-MM-DD", "sources": ["..."] },
  "categories": [ { "id": "global-claudemd", "label": "Global · CLAUDE.md", "color": "#c0392b" } ],
  "rules": [
    {
      "id": "G1", "category": "global-claudemd",
      "title": "short title", "summary": "one-line plain-English statement",
      "source": "path:line", "why": "optional origin",
      "agentImpact": "see step 4", "auditNote": "see step 4 (omit if healthy)"
    }
  ]
}
```

Group `categories` by source/kind (global vs project, CLAUDE.md vs memory vs ADR
vs convention). One rule per standing directive.

### 4. Analyze (against the whole corpus)

Two outputs per rule, **explicitly separated** so the human gets facts and
critique as distinct things. This separation is mandatory; do not fuse verdict
and reasoning into one blob.

- **`agentImpact` — always.** Neutral-diagnostic, first person. What following
  this rule actually changes about your behavior, plus its side effect / cost.
  When a rule merely restates default behavior, **say so plainly** ("restates
  default behavior; changes nothing distinctive"). That is the finding the human
  cannot see for themselves.
- **`auditNote` — only when something is wrong.** The opinionated health/cost
  judgment, drawn from the fixed taxonomy below. **Omit it when the rule is
  healthy.** Forcing a note onto every rule manufactures fake problems.

| Audit category | Fires when |
|----------------|-----------|
| Redundant | Duplicates another rule, or an installed skill / ADR |
| Stale / superseded | References changed files or decisions; a newer rule overrides it |
| Misfiled | Wrong home (a template stored as "feedback"; a project rule in global config) |
| Low-yield ritual | Friction exceeds value |
| Not actionable | Too vague to change behavior |
| Brittle | Will rot (hardcoded word lists, dates, paths) |
| Conflicts | Contradicts another rule |

Run the **whole taxonomy** against every rule. The easy categories (Conflicts,
Stale) get found on their own; **Brittle and Misfiled are the ones routinely
missed**, so check them explicitly. Redundancy, Conflicts, and Supersession are
only visible across the corpus, so analyze with all rules and the skill/ADR list
in view, never each rule in isolation.

**Full rubric, category tests, and worked good/bad examples:** see
[references/analysis-contract.md](references/analysis-contract.md).

### 5. Pre-curate checkpoint (informational)

Report "analyzed N rules, flagged K" with the per-category breakdown, then
proceed. Offer a last chance to re-scope; do not block waiting for input.

### 6. Build the curation UI

Run the bundled script (standard library only, no install):

```bash
python build_curator.py rules.json -o rule-curator.html
```

It injects the data into `curator-template.html` and writes one self-contained
file. The user double-clicks it; decisions persist locally; it exports
`decisions.json` and a markdown summary.

### 7. Curate (human)

The human sets keep / drop / modify per rule in the browser, with optional
replacement text and notes, then exports `decisions.json`.

### 8-9. Apply-plan and apply gate

Read `decisions.json`, compute the exact edits against **current** file contents,
and emit an auditable apply-plan. Then confirm once ("apply these N edits?") and
execute. Per-verdict edits, drift handling, and the plan format: see
[references/apply-plan-format.md](references/apply-plan-format.md).

## Disciplines (hold these)

**No rubber-stamping.** The default failure is "good rule, keep it." Every rule
gets its honest cost named in `agentImpact`. A rule that restates default
behavior is dead weight; say so even when it sounds wise.

**No invented staleness.** An `auditNote` of "stale" / "superseded" /
"duplicates X" must be tied to actual corpus evidence (the conflicting rule, the
superseding ADR, the duplicated skill). Never guess a problem onto a healthy,
current rule to look thorough. A healthy rule gets no audit note.

## Red flags — stop and recheck

- You gave a rule a verdict without writing its `agentImpact`.
- You wrote an `auditNote` on every rule (manufactured problems).
- You praised a rule's specificity without checking if its list/date/path will rot (Brittle).
- You judged a memory entry's content but not whether its *type* fits (Misfiled).
- You called something stale without naming what supersedes it.
- You fused "what it does" and "what's wrong with it" into one verdict blob.

## Common mistakes

- **Analyzing rules one at a time.** Cross-rule findings need the whole corpus loaded at once.
- **Skipping the skill/ADR enumeration**, then guessing at duplication.
- **Treating PRDs as rule sources** and flooding the audit with feature requirements.
- **Writing back blind.** Recompute the apply-plan against current files; skip rules whose text drifted since curation.
