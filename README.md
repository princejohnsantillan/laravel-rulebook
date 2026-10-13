# Laravel Rulebook

Versioned Laravel rules for code-reviewing agents, covering application code (codebase) and infrastructure/DevOps (infra).

## Layout

- `versions/<YYYY-MM-DD_HHmm>/`: one folder per rulebook version, named by its generation time (Asia/Manila). Versions are never edited after they are added.
- `latest/`: an identical copy of the newest version.

Each version contains:

- `laravel-codebase-rules.md` / `laravel-codebase-rules.cited.md`: codebase rules, without and with citations
- `laravel-infra-rules.md` / `laravel-infra-rules.cited.md`: infra rules, without and with citations
- `conflicts.md`: conflicting guidance left out pending a decision
- `outdated.md`: patterns excluded as outdated for Laravel 13

Each new version arrives through a pull request.
