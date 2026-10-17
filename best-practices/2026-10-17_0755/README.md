# Laravel best practices: 2026-10-17_0755 (INTERIM)

> **INTERIM: Claude collection chunks and the stricter adoption re-run are still pending.**
> This is a snapshot published on 2026-10-17 at about 22:50 (Asia/Manila) while waiting for Claude usage resets. Later updates will be pushed to this same branch and PR. Do not merge until the PR is marked final.

Built by the **`laravel-best-practices-compile`** skill (first run). This version is separate from the rulebook in `latest/` and `versions/`, and does not touch them. Target: **Laravel 13**.

## Laravel 13 filter
Every active and aspirational rule was tagged against the Laravel 13 docs and framework source. **These docs include only `L13-current` and `L13-updated` rules.** `L13-obsolete` rules are left out here, but kept (and marked) in Notion for review.

| Tag | Rules |
|---|---|
| L13-current | 6,042 |
| L13-updated (rule and example rewritten to the L13 form) | 123 |
| L13-obsolete (excluded from these docs) | 2 |

## Counts in this snapshot (L13-current and L13-updated only)
| Level | Rules |
|---|---|
| Must (7+ points) | 279 |
| Should (4-6) | 1,930 |
| Consider (2-3) | 3,594 |
| Aspirational (under 30% adoption; in aspirational.md) | 354 (59 Must-scored, 105 Should-scored, 190 Consider-scored) |
| Open conflicts (6 rules, 3 pairs; in conflicts.md) | 6 |
| Rejected after Claude verification | 2 |

## Files
- `laravel-best-practices.md`: the rules, uncited
- `laravel-best-practices.cited.md`: the rules with their sources
- `enforcement.md`: which rules a tool (Pint, Larastan/PHPStan, Rector, etc.) can enforce, with config snippets
- `adoption-report.md`: how often 44 pinned public repos follow each rule
- `aspirational.md`: well-sourced rules that most code does not follow yet
- `conflicts.md`: opposing rules that both scored 4 or more, with Claude's assessment

## What is still pending
- **Claude collection:** 37 small chunks of Claude source collection (official docs, Laracon/teaching, linters, community/blog) are queued to run after the Claude usage reset. Their findings will be merged, scored, verified and L13-tagged, and may add rules or change levels.
- **Stricter adoption re-run (v2):** the adoption method was fixed after a Claude recheck agreed with only 120/178 (67%) of results. Fixes: rules need at least 5 applicable repos (3 first-party) to earn adoption points, non-applicable repos no longer count as "not followed", and equivalent implementations count as followed. In this snapshot, 326 rules use the v2 result, 2,087 use v1 marked **provisional** (v2 still running), and 3,754 use v1 (not in the re-run set). A fresh Claude sample will confirm the fix.
- New Must rules from those steps will get a Claude recheck before the final version.

## How it was built
- Sources were collected by Codex (gpt-6.1-sol) and Claude (Opus) separately, merged, and deduplicated. Practices found by only one model were checked by the other model.
- Scoring: official docs, first-party code and Boost 3 points; packages and linters 2; Laracon 2 (3 for a Laravel-employee speaker); teaching/blog and community 1; Laravel News 0. Adoption adds +2 if first-party code follows a rule at least 80% of the time and +1 if the whole sample follows it at least 70%.
- Every Must rule was rechecked by Claude: 252 confirmed, 60 reworded, 460 demoted to Should, 1 rejected.

## Caveats
- The first Claude collection round ran at max effort before the effort policy changed. The first 975 single-model checks ran at xhigh and the remaining 2,929 at medium.
- Deduplication is conservative, so some near-duplicate rules remain.
- Adoption numbers marked provisional may move.
