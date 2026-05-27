# Analysis contract

The full rubric behind step 4 of the Rule Curator workflow. SKILL.md carries the
compact version; this file carries the category tests and worked examples.

## Contents

- The two outputs (impact vs audit note)
- Writing `agentImpact`
- The audit taxonomy (7 categories, each with a test and example)
- The disciplines, with failure examples
- Worked end-to-end examples

## The two outputs

Every rule gets a neutral **behavioral impact**, always. A rule gets an
**audit note** only when something is wrong with it. Keep them separate: the
impact is what the rule does (facts), the audit note is what is wrong with it
(critique). Fusing them into one verdict blob is the most common structural
failure and hides the facts behind the opinion.

## Writing `agentImpact`

First person, neutral-diagnostic. State what following the rule changes about
your behavior, and its cost or side effect. It is not a verdict and not praise.

- **Good:** "I resist completing a task with a known half-fix and surface
  tangential bugs the moment I see them. Side effect: occasional mid-task
  digressions if I don't filter for relevance."
- **Good (the hard one):** "Restates default behavior. I already ask when
  requirements are ambiguous; this changes nothing distinctive."
- **Bad (praise, not impact):** "A great rule that keeps the codebase clean."
- **Bad (verdict, not impact):** "KEEP. This is exactly what a project rule
  should be."

The single most valuable impact statement is the honest "this changes nothing."
A human cannot see which of their rules are load-bearing and which are decoration;
you can.

## Audit taxonomy

Run all seven against every rule. The first two get found easily; **Brittle and
Misfiled are routinely missed even by strong models, so check them on purpose.**

### Redundant
**Test:** Does another rule, an installed skill, or an ADR already say this?
**Example:** "Always write tests before implementation (red-green-refactor)" when
a `test-driven-development` skill is installed. Note: *duplicates the installed
TDD skill.* Keep at most a project-level mandate ("TDD is required here"); drop
the restated mechanics.

### Stale / superseded
**Test:** Does this reference a file, endpoint, or decision that has since
changed? Is there a newer rule or ADR that overrides it?
**Example:** "Send analytics to `api.example.com/v1/events`; v1 is supported"
when an ADR migrated to v2 and v1 returns 410. Note: *superseded by the v2 ADR;
following this rule writes code that hits a dead endpoint.* You must name the
thing that supersedes it.

### Misfiled
**Test:** Is this in the wrong home or the wrong type? A reusable template stored
as "feedback"; a project-specific rule sitting in global config; a heuristic that
belongs inside a skill.
**Example:** A "standard PR description template" stored as a `feedback` memory
entry. Note: *it is a reusable template, not feedback; wrong memory type, should
be a reference.* Judge the type, not just the content.

### Low-yield ritual
**Test:** Does this impose process whose friction exceeds its value, given how
often it actually fires?
**Example:** A heavyweight claim-the-file-before-editing protocol for a solo
developer who rarely runs parallel sessions.

### Not actionable
**Test:** Could two people follow this and do different things? Is there any
decidable criterion?
**Example:** "Write high-quality, clean, maintainable code." Note: *no decidable
criterion; cannot be acted on or verified; every model already defaults to this.*

### Brittle
**Test:** Does it hardcode a list, date, path, or count that will rot? Would a
general principle do the job more durably?
**Example:** "Forbidden UI words: synergy, leverage, seamless, robust,
cutting-edge." Note: *hardcoded denylist rots as language drifts; a read-aloud
"does this sound like marketing?" test generalizes.* Specific is not the same as
durable; a precise rule can still be brittle.

### Conflicts
**Test:** Does another rule (in any source) instruct the opposite on the same action?
**Example:** "Never commit directly to main; PR with one approval" in CLAUDE.md
vs an ADR mandating direct commits to main during a solo phase. Note: *direct
contradiction; the agent cannot satisfy both.* Name both sides and which should
yield (usually the more recent, context-justified decision).

## The disciplines, with failure examples

### No rubber-stamping
The default is to approve anything that sounds reasonable. Resist it.

- **Failure:** keeping "Be helpful and accurate; ask when unsure" with "great
  rule" and no impact statement.
- **Correct:** "I'm already helpful and accurate by default, and I already ask
  when requirements are ambiguous. The 'ask when unsure' half is a mild,
  partly-default steer; 'be helpful and accurate' is pure filler."

### No invented staleness
Thoroughness is not inventing problems. A healthy rule gets no audit note.

- **Failure:** flagging "migrations must include a working downgrade()" as
  "possibly outdated" with no evidence.
- **Correct:** no audit note. It is concrete, actionable, non-default, and
  nothing supersedes it. Impact only: "I include a working downgrade() in every
  migration so a bad deploy is reversible."

A "stale" or "duplicates X" note without a named superseding rule / ADR / skill
is a guess. Cut it or find the evidence.

## Worked examples

**Healthy rule (no audit note):**
```
title: Reversible migrations
summary: Every Alembic migration includes a working downgrade().
agentImpact: I write a working downgrade() in each migration so a bad deploy can
  be rolled back. Small extra effort per migration; high value on incident day.
(no auditNote)
```

**Brittle rule (impact + audit note, separated):**
```
title: Forbidden UI words
summary: Denylist of buzzwords banned from user-facing copy.
agentImpact: I scan UI copy against the denylist and rewrite hits. Catches the
  listed words; silent on every buzzword not on the list.
auditNote: Brittle. The hardcoded list rots as language drifts and misses
  synonyms. A "read it aloud, does it sound like marketing?" test generalizes and
  needs no maintenance.
```

**Stale rule (impact stays neutral, audit note carries the evidence):**
```
title: Analytics endpoint
summary: POST events to api.example.com/v1/events.
agentImpact: I send analytics writes to the v1 endpoint.
auditNote: Stale / superseded. ADR-003 migrated analytics to v2 and v1 now
  returns 410. Following this rule produces code that hits a dead endpoint.
```
