# Conflicts needing a decision

**Version:** 2026-10-14 12:13. A conflict is two opposing positions that both score 4 or more. These rows are left out of the rules until you decide.

## New conflicts in this version

None. None of the 775 new findings opposes an existing rule.

## Pending from the previous version (still undecided)

### 1. Codebase — Persistence coupling

- Side A: Keep entities independent of the framework and persistence behind replaceable adapters. — score 6: [@tightenco](https://tighten.com/insights/bol-ep19-jack-mcdade-statamic) (Tier 2, 3); [@maryperry](https://www.youtube.com/watch?v=vsxoaTgtyjw) (Laracon speaker, 2); [@matthiasnoback](https://matthiasnoback.nl/2022/04/ddd-entities-and-orm-entities/) (Tier 3, 1)
- Side B: Use Eloquent-backed domains and introduce abstractions only when their value is demonstrated. — score 8: [@mattstauffer](https://www.youtube.com/watch?v=f4QShF42c6E&t=13999s) (Laracon speaker, 3); [@maryperry](https://www.youtube.com/watch?v=vsxoaTgtyjw) (Laracon speaker, 2); [@sileence](https://www.youtube.com/watch?v=1sixoOQliZI) (Laracon speaker, 2); [@matthiasnoback](https://matthiasnoback.nl/2022/04/ddd-entities-and-orm-entities/) (Tier 3, 1)
- Status: pending. Both sides still score 4 or more.

### 2. Codebase — Test isolation

- Side A: Design and test object interactions using substituted collaborators. — score 22: [@christophrumpel](https://x.com/christophrumpel/status/1985307348659732858) (Tier 1, 3); [@freekmurze](https://spatie.be/docs/laravel-event-sourcing/v7/using-aggregates/testing-aggregates) (Tier 2, 3); [@lukedowning19](https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=1500s) (Laracon speaker, 3); [@mateusjatenee](https://laracasts.com/episodes/3047) (Tier 2, 3); [Laravel (official)](https://github.com/laravel/boost/tree/main/.ai) (Official, 3); [@jasonmccreary](https://www.youtube.com/watch?v=LaKEFjA25r4) (Laracon speaker, 2); [@maryperry](https://www.youtube.com/watch?v=vsxoaTgtyjw) (Laracon speaker, 2); [@ryutahamasaki](https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=4011s) (Laracon speaker, 2); [@jwmcpeak](https://laracasts.com/episodes/4124) (Tier 3, 1)
- Side B: Prioritize request-boundary behavior tests with real persistence. — score 15: [@ashleyhindle](https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=31692s) (Laracon speaker (employee), 3); [@enunomaduro](https://www.youtube.com/watch?v=IH4-bkDSqyk) (Laracon speaker (employee), 3); [@mattstauffer](https://www.youtube.com/watch?v=KBigS5vLwZk) (Laracon speaker, 3); [Laravel (official)](https://github.com/laravel/boost/tree/main/.ai) (Official, 3); [Kath Young](https://www.youtube.com/watch?v=pRaV4BG8zvg) (Laracon speaker, 2); [Instructor Team](https://laracasts.com/episodes/3705) (Tier 3, 1)
- Status: pending. Both sides still score 4 or more.

### 3. Codebase — Event payload semantics

- Side A: Carry immutable IDs and event-time values, avoiding mutable entities. — score 5: [Laravel (official)](https://github.com/hirethunk/verbs/blob/13ee038c0ad76e1df4e63c182135cea367893d8c/docs/event-lifecycle.md) (Official, 3); [@coulbourne](https://www.youtube.com/watch?v=OpCT3s4wTI0) (Laracon speaker, 2)
- Side B: Carry explicit participating Eloquent models in events. — score 0 (no 2022+ evidence)
- Status: pending. New evidence leans to side A (5 vs 0); the other side no longer reaches 4, so this may be settled — your call.

### 4. Codebase — Validation location

- Side A: Center all validation and business rules on the data model. — score 0 (no 2022+ evidence)
- Side B: Separate operation-specific validation from persistence. — score 3: [Laravel (official)](https://laravel.com/docs/13.x/precognition#handling-file-uploads) (Official, 3)
- Status: pending. New evidence leans to side B (3 vs 0); the other side no longer reaches 4, so this may be settled — your call.

### 5. Codebase — Universal TDD obligation

- Side A: Advocate TDD for everyone. — score 6: [@christophrumpel](https://laracasts.com/episodes/2864) (Tier 1, 3); [@lukedowning19](https://laracasts.com/episodes/3008) (Tier 2, 3)
- Side B: Judge testing automation by behavioral risk and permit exemptions for low-value work. — score 21: [@bobbybouwmann](https://www.youtube.com/watch?v=jX3nWXyLank) (Laracon speaker, 3); [@christophrumpel](https://www.youtube.com/watch?v=f4QShF42c6E&t=24315s) (Laracon speaker (employee), 3); [@enunomaduro](https://x.com/enunomaduro/status/1649035225924616192) (Tier 1, 3); [@jeffrey_way](https://x.com/jeffrey_way/status/1985327265954009101) (Tier 2, 3); [Laravel (official)](https://github.com/laravel/boost/tree/main/.ai) (Official, 3); [@jasonmccreary](https://www.youtube.com/watch?v=LaKEFjA25r4) (Laracon speaker, 2); [@samsehnert](https://www.youtube.com/watch?v=IPbxscS0xiY) (Laracon speaker, 2); [Kath Young](https://www.youtube.com/watch?v=pRaV4BG8zvg) (Laracon speaker, 2)
- Status: pending. Both sides still score 4 or more.

### 6. Codebase — Form Requests

- Side A: Avoid Form Requests because of authorization boilerplate. — score 0 (no 2022+ evidence)
- Side B: Use Form Requests for substantial validation and custom messages. — score 18: [@enunomaduro](https://github.com/pestphp/pest/blob/5.x/src/ArchPresets/Laravel.php) (Tier 1, 3); [@freekmurze](https://freek.dev/2441-returning-native-types-from-the-laravel-request-class) (Tier 2, 3); [@jeffrey_way](https://laracasts.com/episodes/3881) (Tier 2, 3); [Laravel (official)](https://github.com/laravel/boost/blob/5c0a822e35cce497a595f5ee3eb6418ae2111aeb/.ai/laravel/skill/laravel-best-practices/rules/validation.md#L3-L16) (Official, 3); [@dunglas](https://www.youtube.com/watch?v=DJJt_2V26V8) (Laracon speaker, 2); [@tightenco](https://tighten.com/insights/request-level-validation/) (Tier 2, 2); [@jasonmccreary](https://laracasts.com/episodes/3638) (Tier 3, 1); [@jwmcpeak](https://laracasts.com/episodes/4128) (Tier 3, 1)
- Status: pending. New evidence leans to side B (18 vs 0); the other side no longer reaches 4, so this may be settled — your call.

### 7. Codebase — Client validation timing

- Side A: Give early browser validation feedback and avoid obviously invalid submissions. — score 6: [@timacdonald87](https://www.youtube.com/watch?v=MMc2TzBY6l4) (Laracon speaker (employee), 3); [Laravel (official)](https://docs.vapor.build/resources/storage) (Official, 3)
- Side B: Submit and display backend errors rather than duplicating validation rules. — score 3: [@timacdonald87](https://www.youtube.com/watch?v=MMc2TzBY6l4) (Laracon speaker (employee), 3)
- Status: pending. New evidence leans to side A (6 vs 3); the other side no longer reaches 4, so this may be settled — your call.

### 8. Codebase — Interactive frontend preference

- Side A: Prefer JavaScript page components. — score 2: [@maryperry](https://www.youtube.com/watch?v=vsxoaTgtyjw) (Laracon speaker, 2)
- Side B: Prefer Blade with selective Livewire and Alpine. — score 5: [@ashleyhindle](https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=31692s) (Laracon speaker (employee), 3); [@dunglas](https://www.youtube.com/watch?v=DJJt_2V26V8) (Laracon speaker, 2)
- Status: pending. New evidence leans to side B (5 vs 2); the other side no longer reaches 4, so this may be settled — your call.

### 9. Codebase — Compromised-password policy

- Side A: Allow either a warning or rejection for a known compromised password. — score 3: [@valorin](https://securinglaravel.com/security-tip-should-you-block-compromised) (Tier 2, 3)
- Side B: Reject known compromised passwords. — score 3: [@valorin](https://securinglaravel.com/security-tip-should-you-block-compromised) (Tier 2, 3)
- Status: pending. Tied at 3 vs 3; neither side reaches 4.

### 10. Codebase — Reusable-action validation

- Side A: Validate registration input inside the reusable action. — score 0 (no 2022+ evidence)
- Side B: Pass already-validated attributes into actions. — score 3: [@enunomaduro](https://x.com/enunomaduro/status/2071213127840960842) (Tier 1, 3)
- Status: pending. New evidence leans to side B (3 vs 0); the other side no longer reaches 4, so this may be settled — your call.

### 11. Codebase — Workflow status

- Side A: Persist explicit current status and resolve state objects from it. — score 0 (no 2022+ evidence)
- Side B: Reject status-column-driven designs in favor of event history. — score 0 (no 2022+ evidence)
- Status: pending. Tied at 0 vs 0; neither side reaches 4.

### 12. Infra — Staging before release

- Side A: Validate on staging before automated production deployment. — score 3: [@mattstauffer](https://laracasts.com/episodes/3304) (Tier 2, 3)
- Side B: Omit staging when tests and controlled feature flags support direct production delivery. — score 0 (no 2022+ evidence)
- Status: pending. New evidence leans to side A (3 vs 0); the other side no longer reaches 4, so this may be settled — your call.
