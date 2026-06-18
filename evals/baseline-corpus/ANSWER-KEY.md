# Baseline corpus — answer key

Synthetic "Habitsmith" rule corpus with planted issues across every audit-note
category, plus a healthy control and a rubber-stamp-bait rule. Used to score
both the RED baseline (no skill) and GREEN verification (with skill).

Assume an installed `tdd` / `test-driven-development` skill exists in the
environment (so the "tests before code" rule is a skill duplicate).

## Planted findings

| # | Rule (source) | Expected audit category | What a good audit must say |
|---|---------------|-------------------------|----------------------------|
| 1 | "Always write tests before implementation / red-green-refactor" (CLAUDE.md) | **Redundant** | Duplicates the installed TDD skill; drop the inline copy or defer to the skill. |
| 2 | "Never commit directly to main; PR with one approval" (CLAUDE.md) | **Conflicts** | Directly contradicts ADR-007 (solo trunk-based, PR waived). One must yield. |
| 3 | "Send analytics to api...v1/events; v1 is supported" (CLAUDE.md) | **Stale / superseded** | ADR-003 migrated to v2 and v1 returns 410. Rule is wrong now. |
| 4 | "Write high-quality, clean, maintainable code" (CLAUDE.md) | **Not actionable** | Too vague to change behavior; cannot be acted on or verified. |
| 5 | "Forbidden words in UI copy: synergy, leverage, ..." (CLAUDE.md) | **Brittle** | Hardcoded denylist; rots as language drifts. A read-aloud test generalizes. |
| 6 | "Be helpful and accurate; ask when unsure" (CLAUDE.md) | **Low-yield / restates default** | Restates default model behavior; does nothing distinctive. Impact = none. |
| 7 | "Migrations must be reversible; working downgrade()" (CLAUDE.md) | **HEALTHY — no audit note** | Concrete, actionable, non-default. Must NOT be flagged. False-positive trap. |
| 8 | PR description template stored as `type: feedback` (memory) | **Misfiled** | It is a reusable template; wrong memory type (should be reference). |
| 9 | "Prefer pnpm for all package operations" (memory) | **Conflicts** | CLAUDE.md mandates `npm run lint` / `npm test`. Cross-source inconsistency. |
| 10 | "Streak rollover in local tz" (memory) | **HEALTHY — no audit note** | Concrete domain rule; must NOT be flagged. Second false-positive trap. |
| 11 | "Search revamp v2 status" stored as `type: project` (memory) | **Work-status, not a standing rule (+ extract buried directive)** | Mostly a ship log (date, commit, migration, p95, backfill count, Phase-3 follow-ups) that ages into noise — `isStandingRule` is false for the entry as a whole. It buries ONE durable directive ("never query the legacy `search_index` table directly; go through `SearchService.query()`"). Correct verdict: extract the directive to a clean entry and archive/trim the status log. Failing either way: keeping the whole entry as a healthy rule (weak-model rubber-stamp), or dropping it wholesale and losing the directive. |

## Corpus-level pattern (expected at the pre-curate checkpoint)

Beyond per-rule notes, the checkpoint must name recurring cross-cutting patterns
(same defect/structure across 3+ rules):

- **Systemic over-specification.** `feedback_destructive_migrations`,
  `domain_checkin_dedup`, and `feedback_rate_limit_budget` each pair a sound
  durable directive with a thick layer of rot-prone provenance (commit hashes,
  migration numbers, file:line refs, dates, row counts). The `project_search_revamp`
  status log shares the habit. A correct audit names this as one habit spanning
  those rules, with a single corpus-wide fix (keep the directive; move
  commit/line/date/count detail to a provenance footer or drop it) — not just 3-4
  disconnected Brittle notes.

**No invented patterns:** the healthy controls (#7 reversible migrations, #10
streak timezone) and the standalone rules must NOT be swept into a fake pattern.
A pattern claim must name 3+ real rules; promoting a one-off is a discipline
failure mirroring "no invented staleness."

## Discipline checks (the two bulletproofed rules)

- **No rubber-stamping:** Rule #6 is bait. A rubber-stamper says "good rule,
  keep." A correct audit names that it restates default behavior and carries no
  distinctive cost/benefit.
- **No invented staleness:** Rules #7 and #10 are healthy and current. An agent
  that invents a problem for them (e.g. "this may be outdated") fails the
  discipline. Staleness claims must be tied to actual corpus evidence (only #3
  has it, via ADR-003).

## Structural expectations

- Per rule: a neutral behavioral-impact statement (what it does + cost) AND a
  separate audit note only when warranted.
- Cross-rule findings (#1 skill-dup, #2 and #9 conflicts, #3 supersession)
  require viewing the whole corpus, not each rule in isolation.
