# GREEN verification — findings (with skill)

Three runs against the same baseline corpus, agents operating under the skill
(SKILL.md + references read as instructions). Scored against
`baseline-corpus/ANSWER-KEY.md`.

## Run 1 — analysis, strong model (Opus general-purpose)

Near-perfect. Every planted issue caught in the right category, including the two
the RED baseline missed:

- **Brittle** (forbidden-words denylist): caught.
- **Misfiled** (PR template stored as `type: feedback`): caught, named the wrong type.
- **Impact vs audit note separated** on every rule (the big baseline structural gap): present.
- Redundant, Stale (named ADR-003), both Conflicts (PR vs ADR-007; npm vs pnpm), Not-actionable: caught.
- 5 healthy rules got no audit note; no invented staleness.
- Produced a valid `rules.json` in schema + checkpoint summary.

The agent explicitly noted it "checked Brittle and Misfiled on purpose since the
contract flags them as routinely missed" — the skill drove the behavior the
baseline lacked.

## Run 2 — analysis, weak model (Haiku)

Carried the headline value: **caught Brittle and Misfiled**, held the
impact/audit separation, kept the healthy controls clean. This is the key
evidence that the skill, not the model, does the work.

Weaknesses (model-capability, not skill-clarity):
- Missed the cross-source npm/pnpm conflict (listed pnpm as healthy without
  connecting it to the npm command rule).
- Fuzzy rule count and did not inline the full rules.json as asked.

Decision: do NOT overfit the skill to force the cross-source conflict (the
general "conflicts are cross-source, analyze the whole corpus at once" guidance
is already present). Recorded as a known weak-model limitation.

## Run 3 — apply phase (strong model)

Produced a correct apply-plan from a crafted `apply-test/decisions.json`:

- **Memory drop did both halves** (delete entry file AND remove the MEMORY.md
  pointer line), citing "a dangling pointer is a broken state." This was the main
  thing to verify; passed.
- **Stale decision** (`Z9`, a rule absent from the sources) flagged and skipped.
- **keep** = no-op; **modify** = correct before/after.
- Reasoned correctly about paraphrase-vs-verbatim `original` (not a false drift-skip).

New, general finding (drove the one REFACTOR below): dropping the only rule under
a section heading **orphans the heading**. The agent removed it via general
surgical-change discipline, but the apply guidance did not name this case.

## REFACTOR applied

- `references/apply-plan-format.md`: added handling for structure orphaned by a
  drop (remove a heading left empty when its last rule is dropped). General, not
  fixture-specific. The Run-3 agent already exhibited this behavior via general
  discipline, so the edit codifies an observed-correct behavior.

## Verdict

Skill closes all RED baseline gaps (Brittle, Misfiled, separated structure,
deliverable) and holds the disciplines (no rubber-stamping, no invented
staleness) on both a strong and a weak model. Apply phase correct, including the
both-halves memory drop and stale-decision skip. Bulletproof for the tested
scenarios.
