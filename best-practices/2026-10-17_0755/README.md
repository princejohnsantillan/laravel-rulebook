# Laravel best practices: 2026-10-17_0755

Built by the **`laravel-best-practices-compile`** skill (first run). This version is separate from the rulebook in `latest/` and `versions/`, and does not touch them. Target: **Laravel 13**. Finalised on 2026-10-18 (Asia/Manila).

## Laravel 13 filter
Every active and aspirational rule was tagged against the Laravel 13 docs and framework source. **These docs include only `L13-current` and `L13-updated` rules.** The `L13-obsolete` rules are left out here, but kept (and marked) in Notion for review.

| Tag | Rules |
|---|---|
| L13-current | 6,892 |
| L13-updated (rule and example rewritten to the L13 form) | 134 |
| L13-obsolete (excluded from these docs) | 2 |

## Counts (L13-current and L13-updated only)
| Level | Rules |
|---|---|
| Must (7+ points) | 413 |
| Should (4-6) | 2,715 |
| Consider (2-3) | 3,727 |
| Aspirational (under 30% adoption; in aspirational.md) | 162 (21 Must-scored, 38 Should-scored, 103 Consider-scored) |
| Open conflicts (3 pairs, 6 rules; in conflicts.md) | 6 |
| Rejected after Claude verification | 3 |

## Files
- `laravel-best-practices.md`: the rules, uncited
- `laravel-best-practices.cited.md`: the rules with their sources
- `enforcement.md`: which rules a tool (Pint, Larastan/PHPStan, Rector, etc.) can enforce, with config snippets
- `adoption-report.md`: how often 44 pinned public repos follow each rule
- `aspirational.md`: well-sourced rules that most code does not follow yet
- `conflicts.md`: opposing rules that both scored 4 or more, with Claude's assessment

## How it was built
- Sources were collected separately by Codex (gpt-6.1-sol) and Claude (Opus), then merged and deduplicated. Claude collection ran as 37 small chunks: official docs and source (o01-o10), Laracon and teaching (t01-t18), linters (l01-l04), community and blogs (c01-c05).
- Scoring: official docs, first-party code and Boost 3 points; packages and linters 2; Laracon 2 (3 for a Laravel-employee speaker); teaching/blog and community 1; Laravel News 0. Adoption adds +2 if first-party code follows a rule at least 80% of the time and +1 if the whole sample follows it at least 70%. Adoption points need at least 5 applicable repos (3 for first-party).
- Practices found by only one model were checked against their sources by Claude, and every Must rule was rechecked by Claude at xhigh effort.
- Adoption method v2: non-applicable repos are excluded, and equivalent implementations count as followed. A Claude recheck of a fresh sample agreed with **85% (51/60)** of the v2 results, up from 67% (120/178) under v1.

## Caveats
- The first Claude collection round ran at max effort, before the effort policy changed. The first 975 single-model checks ran at xhigh and the rest at medium.
- Three Claude chunks found nothing, for legitimate reasons: t01 (there was no Laracon US 2022), t05 (the Laracon EU 2023 talks are not on YouTube) and t12 (Laravel Live UK has only highlight reels).
- Claude's recheck demoted many Musts (691 of 1,171 checked) as overstated.
- Rules found only by Claude were source-checked by Claude in a separate pass, not by Codex.
- Deduplication is conservative, so some near-duplicate rules remain.
- 3,840 rules use the v1 adoption measurement because the v2 re-run covered only the rules most sensitive to the v1 errors. All new rules use v2.
