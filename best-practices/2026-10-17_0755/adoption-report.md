# Adoption report

> **INTERIM snapshot (2026-10-17 22:50, Asia/Manila).** Claude collection chunks and the stricter adoption re-run are still pending. Levels and counts will change when they are folded in. See README.md.

Each candidate rule was measured against 44 public repos pinned to a commit: first-party 15, package 20, app 9. Methods: GitHub-style text signatures (rg), Codex reading the shallow clones, and linter rule mapping. Rules that cannot be checked from code (process, infra, docs-only) carry no adoption adjustment.

Scoring: +2 if first-party code follows the rule at least 80% of the time, +1 if the whole sample follows it at least 70%. Rules below 30% overall go to aspirational.md.

- Scored rules (Must/Should/Consider): 6159
- Code-checkable with data: 2025
- Followed by >=70% of the sample: 1326
- First-party >=80%: 957
- Aspirational (<30%): 354

## Claude recheck
Claude (Opus, medium effort) independently re-measured a random 5% sample (178 rules). Agreement (within 15 points and same side of the 30%/70% thresholds): 120/178 (67%).

| Rule | Codex % | Claude % | Note |
|---|---|---|---|
| Provide a valid guest login destination | 33 | 100 | Applicable: 12 app-shaped repos with guest-protected web routes (9 apps minus krayin, which uses its own Bouncer middleware; plus react/vue/livewire starter kits; laravel/laravel skeleton has no auth  |
| Document collection and array element types | 23 | 36 | All 44 applicable. Measured per repo the share of native array/Collection-returning functions with generic @return docs (plus docblock-only @param/@return array generic rate for repos without native t |
| Enable asset preload response headers in the web middleware stack | 100 | 33 | Applicable: 9 app-shaped repos serving Vite assets (coolify, firefly, monica, invoiceninja, krayin, laravel/laravel, react/vue/livewire starter kits). Only react-starter-kit, vue-starter-kit and monic |
| Do not assume magic Data factories run the normal transformation pipeline | 0 | 100 | Applicable: spatie/laravel-data and coolify (only repo depending on laravel-data). Coolify's Data::fromSentinel factories explicitly map snake_case keys, cast each value and supply defaults via new se |
| Use conventional CRUD ability names | 100 | 75 | Applicable: repos defining app policies - coolify (28 policies, all viewAny/view/create/update/delete), monica, pixelfed (conventional), invoiceninja. Invoiceninja's EntityPolicy/CompanyPolicy/UserPol |
| Escape untrusted HTML output | 43 | 86 | Applicable: 14 repos rendering user-facing Blade (9 apps + filament, framework, livewire, livewire-starter-kit, pulse). Escaped {{ }} vastly outnumbers {!! !!} everywhere; most raw echoes are trusted/ |
| Assert every supported browser and JSON response contract | 0 | 18 | Applicable: 9 apps (all have wantsJson/expectsJson/ajax response branches) + laravel/fortify and lorisleiva/laravel-actions (both ship dual-format endpoints); starter kits only consume Fortify in brow |
| Describe console arguments and options | 56 | 81 | Applicable: 37 repos with Artisan commands taking inputs (parsed $signature tokens for ' : ' descriptions, plus Symfony getOptions/InputOption descriptions for filament, ide-helper, Laravel-Excel). Fo |
| Prevent browser MIME sniffing | 31 | 23 | Applicable: 13 app-shaped repos (9 apps + laravel/laravel + 3 starter kits). Followed: firefly (SecureHeaders middleware), akaunting (AddXHeader middleware + nginx example), BookStack (nosniff on Down |
| Use strict equality or timing-safe comparison for digest strings | 0 | 93 | Applicable = 15 repos that compare hash/digest/signature strings (framework, fortify, sanctum, socialite, reverb, livewire, filament, debugbar, coolify, firefly, invoiceninja, pixelfed, akaunting, kra |
| Invalidate verification when the email changes | 67 | 80 | Applicable = app-shaped repos with an email-change path plus email verification: 7 apps (BookStack, coolify, firefly, invoiceninja, monica, pixelfed, linkstack) + react/vue/livewire starter kits. Foll |
| Associate model factories explicitly for static analysis | 80 | 47 | Applicable = 15 repos with own Eloquent models using HasFactory (BookStack, akaunting, coolify, firefly, invoiceninja, krayin, linkstack, monica, pixelfed, cashier, telescope, laravel/laravel, 3 start |
| Set a deterministic testing timezone | 46 | 91 | Applicable = 34 repos with date-sensitive tests (excluded repos with no date tests: debugbar, boost, reverb, socialite, livewire, ziggy, laravel/laravel, linkstack; plus rector-laravel/pest whose date |
| Control lottery outcomes in tests | 0 | 100 | Applicable = repos whose tests exercise Lottery-dependent branches: pulse (Sampling/Pulse.php use Lottery::odds; recorder tests call Lottery::alwaysWin()/determineResultNormally() in 7 test files), pe |
| Commit application Composer and frontend package-manager lockfiles | 62 | 89 | Applicable = 9 deployable apps. 8 track composer.lock + their JS lockfile (BookStack, akaunting, firefly, invoiceninja, pixelfed, linkstack: package-lock; monica: yarn.lock; coolify: bun.lock+package- |
| Keep internal-only enums pure | 75 | 95 | Applicable: 19 repos with non-test enums in src/app. Nearly all backed enums have values consumed (DB/config/CSS/API/serialization, e.g. coolify statuses, filament CSS sizes, framework WorkerStopReaso |
| Expose frontend quality checks through shared package scripts | 0 | 35 | Applicable: 20 repos with package.json + frontend code (excluded spatie/laravel-data, postinstall only). Followed: react- and vue-starter-kit (check/check:fix/types:check scripts reused by composer ci |
| Sanitize or isolate uploaded SVG files | 100 | 75 | Applicable: repos that accept user SVG uploads: krayin (enshrined svg-sanitizer trait, followed), firefly-iii (image/svg+xml attachments served inline only with strict CSP script-src none, followed),  |
| Advance the fake clock with simulated sleeping | 0 | 100 | Applicable: repos whose Sleep::fake tests involve elapsed time: laravel/framework (ConcurrencyLimiterTest, FileCacheLockTest, Cloud QueueTest use Sleep::fake(syncWithCarbon: true)), laravel/pulse (Slo |
| Create dedicated failure exceptions with descriptive named constructors | 29 | 39 | Applicable: 38 repos with failure sites (excluded laravel/laravel, react/vue/livewire starter kits, fortify, ziggy - no/trivial throws). Followed (dedicated exception classes dominate, mostly with sta |
| Prefer automatic data-loading styling on Livewire 4 | 0 | 100 | Livewire 4 repos: livewire-starter-kit (^4.1) and livewire/livewire itself (v4 main; Islands). Starter kit's single wire:loading.class is on recovery-code list items, not on the triggering button - da |
| Show Inertia upload progress | 0 | 100 | Only monica has an Inertia useForm file upload (profile photo). Its createInertiaApp enables the global Inertia progress bar (resources/js/app.js progress:{color}), which tracks upload progress; other |
| Pin local service versions to the required environment | 100 | 71 | Applicable = 7 repos with local compose files: BookStack (mysql:8.4), coolify (postgres:15/redis:7), framework (mysql 5.7/redis 7.0/memcached 1.6), pixelfed (mysql:9/redis:8), tenancy (mysql:8/postgre |
| Select a related key with a subquery and eager-load its belongsTo relation when only one child model is needed | 100 | 50 | Rare situation. Applicable: coolify (ScheduledVolumeBackup latest execution via hasOne()->latestOfMany(), avoiding full child load; counted followed as equivalent) and monica (JournalShowViewHelper::p |
| Use the base query builder when model hydration and model value conversion are unnecessary | 50 | 89 | Applicable = 9 apps (packages rarely do large app reads). Most apps predominantly use query-level pluck()/DB::table()/toBase() for scalar reads (BookStack SettingService/SearchRunner toBase, coolify u |
| Transfer lock ownership explicitly across queue boundaries | 33 | 100 | Only laravel/framework hands a lock across a queue boundary: Bus/UniqueLock stores $lock->owner() on the job and CallQueuedHandler releases via restoreLock(key, owner) -> followed. App Cache::lock usa |
| Scope static-analysis suppressions by error identifier and path, with a reason | 29 | 52 | Applicable = 29 repos with phpstan ignoreErrors or inline ignores. Followed (identifier+path scoped, incl. generated baselines, or inline @phpstan-ignore <identifier>): ide-helper, rector-laravel, lar |
| Use blank and filled with intentional value semantics | 100 | 13 | Applicable = 30 repos with >=5 emptiness checks (excluded tiny/none: skeletons, Inertia kits, pennant, pulse, ziggy, sanctum, pest, inertia, actions, excel, debugbar, livewire-kit, query-builder). Fol |
| Use an atomic Redis operation for dependent multi-command writes | 100 | 67 | Applicable = repos issuing dependent multi-key Redis writes directly: framework (RedisQueue/LuaScripts eval for push/pop/migrate) and horizon (repositories use transaction/pipeline + Lua) -> followed. |
| In nested routes, check that the user has access to the parent resource as well as the child | 36 | 83 | Applicable: 6 apps with permissioned parent/child nested resource routes (coolify, BookStack, firefly-iii, monica, pixelfed, invoiceninja); akaunting/krayin/linkstack nested params are not tenant pare |
| Exclude passwords and secrets from flashed input | 8 | 82 | Applicable: 11 repos that redirect with flashed input on password-bearing forms (9 apps + framework + fortify). Followed: framework (Handler default dontFlash password/current_password/password_confir |
| Expose standard checks through Composer scripts | 74 | 59 | Applicable: all 44. Followed = composer scripts for tests plus formatting and/or static analysis (26): BookStack, tenancy, debugbar, ide-helper, rector-laravel, filament, inertia, invoiceninja, larast |
| Use consistent Pest declarations | 59 | 82 | Applicable: 17 repos with Pest it()/test() test files. Per-file mixing rate measured: 0% tenancy/reverb/activitylog/ziggy; 2-7% coolify(26/1106), boost, pixelfed, permission, data, filament, pulse, la |
| Use upsert for atomic bulk insert-or-update | 100 | 62 | Applicable: 8 repos with bulk insert-or-update synchronization. Followed: pulse (DatabaseStorage upsert), pennant (DatabaseDriver upsert), Laravel-Excel (ModelManager upsert for WithUpserts), krayin ( |
| Run project checks through the intended container environment | 0 | 60 | Applicable = repos with a containerized PHP dev runtime: BookStack, archtechx/tenancy, coolify, monica (.devcontainer), pixelfed (app compose). Followed: BookStack (dev docs run 'docker-compose run ap |
| Allowlist external redirect destinations or map configured destination identifiers to URLs | 0 | 67 | Applicable = apps with input-influenced external redirects: pixelfed, linkstack, invoiceninja. Followed: pixelfed (RemoteAuth domain checked by isDomainCompatible plus optional default/custom domain a |
| Use explicit NULL predicates | 80 | 100 | Applicable = 14 repos with query NULL predicates (BookStack, akaunting, tenancy, coolify, filament, firefly, invoiceninja, krayin, cashier, framework, linkstack, monica, pixelfed, permission). whereNu |
| Run large data backfills outside schema migrations | 56 | 86 | Applicable = 7 apps whose migrations transform data: BookStack, akaunting, coolify, firefly, invoiceninja, monica, pixelfed (krayin/linkstack have no data-transform migrations). Followed: firefly (bac |
| Test the serialized field set at sensitive client boundaries | 82 | 36 | Applicable = BookStack, coolify, firefly, monica, pixelfed, invoiceninja, akaunting, krayin, linkstack, react-starter-kit, vue-starter-kit. Followed (field-contract tests at API boundary): BookStack ( |
| Isolate multiple forms with named validation error bags | 0 | 100 | Scanned app Blade views for pages with 2+ forms sharing an errored field name. The only hit (pixelfed auth/login) puts the forms in mutually exclusive @if step branches, so they never share a page. Ap |
| Configure an ordered failover mailer when delivery must survive a primary provider outage | 100 | 0 | Counted the 9 apps (production mail senders) as applicable. Codex counted 11 configs that merely contain the skeleton 'failover' mailer stub (smtp->log; pixelfed log only) as followed. In every case t |
| Type query and tap closure parameters with their actual argument types | 19 | 32 | Scripted count of where/orWhere/whereHas/whereNot/tap closures in app/src/packages (excl. tests) across 22 repos that have them; followed if >50% typed: BookStack 17/26, firefly 121/122, filament 22/2 |
| Declare arrays with short [] syntax | 0 | 100 | All 44 repos contain PHP arrays; per-repo scan of app/src/packages/config shows long array() syntax is negligible everywhere (0 in 30 repos; max invoiceninja ~85 mostly a pasted var_export block in Pa |
| Test custom validation rules directly | 17 | 44 | Applicable = 9 repos defining Laravel validation rule classes (excluded larastan and laravel-actions whose 'Rule' hits are PHPStan rules, and livewire whose hits are test fixtures). Checked whether ru |
| Use withoutTimestamps() for a bounded write that must preserve updated_at | 100 | 67 | Applicable: monica (9 withoutTimestamps uses in DAV import/export), BookStack (BookSorter withoutTimestamps), and laravel/framework, whose EloquentUserProvider::updateRememberToken manually sets $user |
| Rehash outdated passwords after successful verification | 10 | 67 | Applicable = 15 repos with password login: 9 apps, framework, fortify, filament, 3 starter kits. Laravel 11+ SessionGuard::attempt and Fortify's default pipeline rehash automatically (hashing.rehash_o |
| Run local background workers with the development stack | 0 | 86 | Applicable: apps/skeletons whose local default queue is non-sync (laravel/laravel, 3 starter kits, coolify, monica sail, pixelfed). Followed: the 4 skeletons via composer dev -> artisan dev (DevComman |
| Offer TOTP or security keys rather than relying on SMS alone for high-assurance MFA | 83 | 100 | Applicable: repos implementing MFA: fortify, 3 starter kits (Fortify TOTP + passkeys), coolify, filament (app TOTP/email), BookStack (TOTP+backup codes), firefly-iii, pixelfed, invoiceninja (Google2FA |
| Set activity log retention | 50 | 100 | Applicable: coolify (only app using spatie/laravel-activitylog) and the activitylog package itself. Coolify follows: app/Console/Kernel.php schedules 'cleanup:database --yes' daily, and CleanupDatabas |
| Match anonymous authentication routes to Fortify's configured guard | 100 | 83 | Applicable: Fortify itself plus the Fortify-using apps (3 starter kits, coolify, monica), all with guard=web. Fortify routes use guest:config('fortify.guard'); the starter kits and monica rely on thos |
| Serialize mutually exclusive work with an atomic lock | 100 | 83 | Applicable: repos with concurrent critical sections. Followed (10): coolify, invoiceninja, pixelfed (Cache::lock/WithoutOverlapping/ShouldBeUnique, many uses), filament (export lock), scout (MakeSearc |
| Preserve active authenticator secrets on repeated enable requests | 33 | 82 | Applicable (11): fortify, 3 starter kits, coolify and monica (Fortify EnableTwoFactorAuthentication only regenerates when the secret is empty or force=true; no consumer passes force), filament, BookSt |
| Delay released throttled jobs | 0 | 100 | Applicable: apps with throttled or rate-limited jobs: pixelfed, invoiceninja, coolify. Pixelfed: StoryFetch release(3600) when rate limited; MovePipeline jobs use ThrottlesExceptions with backoff(1-5  |
| Reject known compromised passwords | 29 | 36 | Applicable: 9 apps + react/vue/livewire starter kits + fortify + filament (14). Followed: coolify, firefly-iii (custom PwndVerifier secure_password rule), react/vue/livewire starter kits (Password::de |
| Decorate or configure an existing container service with extend() and return the resulting service | 100 | 67 | Applicable (non-test container ->extend usage): framework, debugbar, laravel-actions, tenancy, coolify, pixelfed. Followed: framework (url, LostConnectionDetector decorated), debugbar (view factory de |
| Enforce domain invariants at every state-change boundary | 100 | 78 | Applicable: the 9 apps (packages/skeletons have no business-domain write paths). Followed (invariants live in shared mutation layer): monica (BaseService validates rules per service), akaunting (Jobs  |
| Test the migration schema consumers actually install | 80 | 100 | Applicable = packages shipping migrations whose Testbench tests touch those tables: cashier-stripe, fortify, pennant, pulse, sanctum, telescope (all via testbench.yaml migrations: database/migrations  |
| Represent trusted generated HTML explicitly | 100 | 61 | Applicable: 18 repos that emit raw HTML in Blade or build HtmlString. Followed: filament (~1100 HtmlString uses), framework, livewire, boost, horizon, reverb, telescope, pulse, medialibrary, coolify,  |

## Sample

| Group | Repo | Commit |
|---|---|---|
| first-party | [laravel/framework](https://github.com/laravel/framework/tree/e51b5fe16d6f0e547137087e52c6106118c319f1) | `e51b5fe16d` |
| first-party | [laravel/laravel](https://github.com/laravel/laravel/tree/9ffef3bc4241e3cc333ef267258a404d0d95f194) | `9ffef3bc42` |
| first-party | [laravel/react-starter-kit](https://github.com/laravel/react-starter-kit/tree/717b8f55aefd82d25d4119eaebdc8e3a72b8d7e5) | `717b8f55ae` |
| first-party | [laravel/vue-starter-kit](https://github.com/laravel/vue-starter-kit/tree/d282e817c6c2fa1bd475f7c42ea785ccfc67d0ab) | `d282e817c6` |
| first-party | [laravel/livewire-starter-kit](https://github.com/laravel/livewire-starter-kit/tree/0f62a26c4e4b401c1300930f47d72a497add8cce) | `0f62a26c4e` |
| first-party | [laravel/pulse](https://github.com/laravel/pulse/tree/1e7a62a8ac93e186cf9d34331125c0b4fc8b5c9f) | `1e7a62a8ac` |
| first-party | [laravel/horizon](https://github.com/laravel/horizon/tree/9de5ceb25c0a8288cf09f6c31dee8ccb91c0738e) | `9de5ceb25c` |
| first-party | [laravel/telescope](https://github.com/laravel/telescope/tree/dcc6674c825f2614c3f18f55860f36e5d145bb1e) | `dcc6674c82` |
| first-party | [laravel/reverb](https://github.com/laravel/reverb/tree/2fb770fbe7d20eed29a4bf7f9dc15172197f4c54) | `2fb770fbe7` |
| first-party | [laravel/cashier-stripe](https://github.com/laravel/cashier-stripe/tree/fb29fd5244d9ec7a77396df9fcb0559d38273f10) | `fb29fd5244` |
| first-party | [laravel/fortify](https://github.com/laravel/fortify/tree/7aa4ac434062c2f4754985ab74adb1dcb11f1123) | `7aa4ac4340` |
| first-party | [laravel/sanctum](https://github.com/laravel/sanctum/tree/c33d50f6f1dd9fa0b1672c24611ca83a939b0583) | `c33d50f6f1` |
| first-party | [laravel/boost](https://github.com/laravel/boost/tree/f1411f90d54cb01ffe30606b7ca92789aad737d9) | `f1411f90d5` |
| first-party | [laravel/pennant](https://github.com/laravel/pennant/tree/d9ea5939eb9e9c0dfadcf9e0ccdf621340fac21a) | `d9ea5939eb` |
| first-party | [laravel/scout](https://github.com/laravel/scout/tree/d6d1774c4fa55035ac30b6312961edcd4155b9a7) | `d6d1774c4f` |
| package | [livewire/livewire](https://github.com/livewire/livewire/tree/5383ac73a06e61fdde68d002e8eb775061dc67a1) | `5383ac73a0` |
| package | [filamentphp/filament](https://github.com/filamentphp/filament/tree/7e268d74423d641d0935bac223246c889456487e) | `7e268d7442` |
| package | [pestphp/pest](https://github.com/pestphp/pest/tree/e9fa0f50c5c6f11be77bf8d69ba121c9f0b54d8c) | `e9fa0f50c5` |
| package | [inertiajs/inertia-laravel](https://github.com/inertiajs/inertia-laravel/tree/8355911931b82df0804b793e3fc400fc19b11d9c) | `8355911931` |
| package | [spatie/laravel-permission](https://github.com/spatie/laravel-permission/tree/6615eefac655efcd652fe394a20cbdf4f72c609f) | `6615eefac6` |
| package | [spatie/laravel-medialibrary](https://github.com/spatie/laravel-medialibrary/tree/1f73cad3497b9b26228ef78b46f900dc49c07286) | `1f73cad349` |
| package | [spatie/laravel-data](https://github.com/spatie/laravel-data/tree/ce296f22861dc1237ce468754cc7f46d3ac34ad5) | `ce296f2286` |
| package | [spatie/laravel-query-builder](https://github.com/spatie/laravel-query-builder/tree/bdd59bdc0c3c1afcb3f4694c393177afb92fa83b) | `bdd59bdc0c` |
| package | [spatie/laravel-backup](https://github.com/spatie/laravel-backup/tree/2b39b6b0fb98eefb95f1ef6b6311983396823c72) | `2b39b6b0fb` |
| package | [spatie/laravel-activitylog](https://github.com/spatie/laravel-activitylog/tree/c37c43b32ea1ff15dc00491c07dafdfb8fd57340) | `c37c43b32e` |
| package | [barryvdh/laravel-debugbar](https://github.com/barryvdh/laravel-debugbar/tree/cde47957014b00a09c24eb1f8a296a6c10211e36) | `cde4795701` |
| package | [barryvdh/laravel-ide-helper](https://github.com/barryvdh/laravel-ide-helper/tree/f9521af2a0ed5ff1d70386cac3fae547a96be947) | `f9521af2a0` |
| package | [larastan/larastan](https://github.com/larastan/larastan/tree/043139b9829917363086d6b10f916772420a6102) | `043139b982` |
| package | [driftingly/rector-laravel](https://github.com/driftingly/rector-laravel/tree/17d2417b4080f646b1252613f6396724cd9b6cac) | `17d2417b40` |
| package | [tighten/ziggy](https://github.com/tighten/ziggy/tree/766315d44ae1930b862df583d1094704c4e2626f) | `766315d44a` |
| package | [lorisleiva/laravel-actions](https://github.com/lorisleiva/laravel-actions/tree/c28e8fb0573324d28fdfdf780425329646bf28a1) | `c28e8fb057` |
| package | [nunomaduro/collision](https://github.com/nunomaduro/collision/tree/9716022d2d48a14fa72bcc40239259051b27a1aa) | `9716022d2d` |
| package | [archtechx/tenancy](https://github.com/archtechx/tenancy/tree/42a5e97a4daae94ccbdb51d95c83aef6e158f0b7) | `42a5e97a4d` |
| package | [maatwebsite/Laravel-Excel](https://github.com/maatwebsite/Laravel-Excel/tree/30f17acbb89bf260173387349722eec2cc817016) | `30f17acbb8` |
| package | [laravel/socialite](https://github.com/laravel/socialite/tree/7a3b628fa10692f614dea6569bb52c0db26fb8db) | `7a3b628fa1` |
| app | [coollabsio/coolify](https://github.com/coollabsio/coolify/tree/6964714b5da26d600621eb137c9845e6240f7d52) | `6964714b5d` |
| app | [BookStackApp/BookStack](https://github.com/BookStackApp/BookStack/tree/d3db80004b51a395a9dabdd406f5fbda3d4d76e4) | `d3db80004b` |
| app | [firefly-iii/firefly-iii](https://github.com/firefly-iii/firefly-iii/tree/866e79fa02cbcf752dc11a2ff7c3437d5c07bc9f) | `866e79fa02` |
| app | [monicahq/monica](https://github.com/monicahq/monica/tree/6e6ec217457e946e330ba7cd3b28107f4332c7b1) | `6e6ec21745` |
| app | [pixelfed/pixelfed](https://github.com/pixelfed/pixelfed/tree/6cea5a4e9688471e8cb4d8ff744dc56518496669) | `6cea5a4e96` |
| app | [invoiceninja/invoiceninja](https://github.com/invoiceninja/invoiceninja/tree/f1ffa5f9989afbed4e7dc80b5c0cc18944335ffc) | `f1ffa5f998` |
| app | [akaunting/akaunting](https://github.com/akaunting/akaunting/tree/06dfa473e0d71a5262a58f6f0759c6badf34d5d2) | `06dfa473e0` |
| app | [krayin/laravel-crm](https://github.com/krayin/laravel-crm/tree/999b596d0f53cb11c9b029f733a1d67e01fccda6) | `999b596d0f` |
| app | [linkstackorg/linkstack](https://github.com/linkstackorg/linkstack/tree/8ccdee471c2875377c3f61309c97f29d63158ad5) | `8ccdee471c` |

## Listed rules by adoption

| Level | Rule | All | First-party |
|---|---|---|---|
| Must | Use Unicode-aware string operations for text | 100% | 100% |
| Must | Replace legacy Laravel global helpers with Arr and Str | 100% | 100% |
| Must | Fail clearly when optional package features need a newer framework | 100% | 100% |
| Must | Keep authoritative lookup data outside disposable caches | 100% |  |
| Must | Preserve required Laravel runtime directories without versioning their contents | 100% | 100% |
| Must | Register application service providers in bootstrap/providers.php when using the streamlined skeleton | 100% | 100% |
| Must | Configure trusted proxies and forwarded headers when HTTPS or host routing terminates at a reverse proxy | 100% |  |
| Must | Merge package configuration defaults in the provider’s register() method | 100% | 100% |
| Must | Register intentionally shared package services as container singletons | 100% | 100% |
| Must | Use a StatefulGuard implementation for Fortify’s configured authentication guard | 100% | 100% |
| Must | Require a valid two-factor code before presenting setup as complete, and explain the confirmation step | 100% | 100% |
| Must | Select authentication guards by names defined in config/auth.php | 100% | 100% |
| Must | Match Fortify login form field names to the configured username field | 100% | 100% |
| Must | Respect cached configuration and skip unnecessary package-config file reads or merges when configuration is cached | 100% | 100% |
| Must | Lock rows inside the transaction that changes them | 100% |  |
| Must | Allowlist dynamic SQL identifiers | 100% | 100% |
| Must | Keep implicit-commit statements out of managed transactions | 100% | 100% |
| Must | Cascade deletion only for genuinely owned children | 100% | 100% |
| Must | Use supported LIKE helpers with deliberate case semantics | 100% | 100% |
| Must | Keep SQLite foreign-key enforcement enabled | 100% | 100% |
| Must | Create one sessions table before using database sessions | 100% | 100% |
| Must | Give conventional model tables identity and timestamps | 100% | 100% |
| Must | Select constrained eager-load columns inside the callback | 100% |  |
| Must | Pass explicit validated fields to mass-assignment operations | 100% | 100% |
| Must | Choose compatible primary-key strategies in models and their schema | 100% |  |
| Must | Keep reusable scopes composable and let callers choose query execution | 100% | 100% |
| Must | List only defined computed accessors in model appends | 100% |  |
| Must | Replace the removed model dates property with date or datetime casts | 100% | 100% |
| Must | Match model primary-key and timestamp metadata to its table | 100% | 100% |
| Must | Include soft-deleted rows only in queries that intentionally require them | 100% |  |
| Must | Specify explicit relationship keys when conventions cannot infer the intended association | 100% | 100% |
| Must | Expose custom Alpine control state through x-modelable and forwarded attributes | 100% | 100% |
| Must | Discover conventional event listeners automatically | 100% |  |
| Must | Use immutable dates for calculations | 100% | 100% |
| Must | Bind PHP component expressions explicitly | 100% | 100% |
| Must | Keep Vite entry declarations and template references aligned | 100% | 100% |
| Must | Keep absolute asset references in public | 100% | 100% |
| Must | Render one root element per Livewire component | 100% | 100% |
| Must | Load one Alpine instance with Livewire | 100% |  |
| Must | Define new Tailwind 4 theme tokens in CSS | 100% | 100% |
| Must | Preserve client template expressions when composing Blade and JavaScript | 100% | 100% |
| Must | Configure middleware, exception handling and route loading in bootstrap/app.php in streamlined applications | 100% | 100% |
| Must | Customize packages through supported overrides rather than vendor edits | 100% | 100% |
| Must | Leave SESSION_DOMAIN null unless you need cross-subdomain cookies, and never set it to an IP or mismatched domain | 100% | 100% |
| Must | Type-hint the Request in controller methods rather than injecting it into the controller constructor | 100% | 100% |
| Must | Review browser-facing SPA and component payloads for sensitive fields as well as public API responses | 100% | 100% |
| Must | Use the appropriate browser or API middleware group | 100% | 100% |
| Must | Have users confirm their two-factor setup by submitting a valid TOTP code to POST /user/confirmed-two-factor-authentication | 100% | 100% |
| Must | Name the email verification link handler route verification.verify and assign it both the auth and signed middleware | 100% | 100% |
| Must | Resolve current request/container/config dependencies at use time rather than retaining stale instances in Octane singletons | 100% | 100% |
| Must | Load package views under a namespace with loadViewsFrom() so applications can override them in resources/views/vendor/<package> | 100% | 100% |
| Must | Return the continuing middleware or pipeline result | 100% | 100% |
| Must | Deny web access to sensitive dotfiles | 100% | 100% |
| Must | Include the S3 filesystem adapter in production dependencies | 100% |  |
| Must | Prevent script execution in upload storage | 100% |  |
| Must | Serialize mutually exclusive work with an atomic lock | 100% |  |
| Must | Inject job services at execution time | 100% | 100% |
| Must | Do not rely on batch or bulk dispatch uniqueness | 100% |  |
| Must | Enforce mutable invariants atomically | 100% | 100% |
| Must | Use Livewire 3 event dispatch APIs | 100% |  |
| Must | Give Eloquent models one identifying primary key | 100% | 100% |
| Must | Ignore only the trusted current record in unique validation | 100% | 100% |
| Must | Replace obsolete framework event-name strings with event classes | 100% | 100% |
| Must | Create required queue tables without duplicating skeleton migrations | 100% |  |
| Must | Bound ordinary job and listener attempts | 100% | 100% |
| Must | Bound job and queued-listener execution time | 100% | 100% |
| Must | Scope mutable services to the request or job | 100% | 100% |
| Must | Use the current framework password-strength rule builder | 100% | 100% |
| Must | Throttle abuse-prone endpoints | 100% | 100% |
| Must | Choose limiter identities deliberately | 100% | 100% |
| Must | Use framework password hashing | 100% | 100% |
| Must | Use established cryptographic implementations | 100% | 100% |
| Must | Require recent password confirmation for sensitive account operations | 100% | 100% |
| Must | Default authentication cookies to SameSite=Lax | 100% | 100% |
| Must | Limit cookie-encryption exceptions to named required cookies | 100% |  |
| Must | Use session-cookie authentication for the first-party browser application | 100% |  |
| Must | Forward Authorization headers through the deployed web server | 100% | 100% |
| Must | Keep session-based browser routes in the web middleware group | 100% |  |
| Must | Authenticate protected API and MCP endpoints | 100% |  |
| Must | Remove debug calls before committing | 100% | 100% |
| Must | Use valid delimited regular expressions | 100% | 100% |
| Must | Use blank and filled with intentional value semantics | 100% | 100% |
| Must | Set mock expectations before acting | 100% | 100% |
| Must | Follow PHPUnit discovery and naming conventions | 100% | 100% |
| Must | Declare PHPUnit providers public and static | 100% | 100% |
| Must | Reset database state between database-backed tests | 100% | 100% |
| Must | Map only fields intended for the current operation | 100% |  |
| Must | Preserve automatic failure behavior for manual validators | 100% | 100% |
| Must | Keep classes compatible with PSR-4 autoloading | 98% | 93% |
| Must | Avoid request-dependent URL helpers in configuration | 97% | 100% |
| Must | Set real test configuration instead of mocking Config | 97% | 92% |
| Must | Use framework fakes at framework boundaries | 97% | 100% |
| Must | Exclude plaintext secrets from version control | 96% | 100% |
| Must | Define a bounded freshness policy for cached data | 96% | 100% |
| Must | Do not pass complex or multi-line expressions to Blade directives such as @json or {{ }}; they are regex-based and can break | 95% | 100% |
| Must | Use Laravel request, server and session APIs instead of PHP request/session superglobals | 95% | 100% |
| Must | Use explicit nullable parameter types | 95% | 100% |
| Must | Mock external boundaries and use real in-process logic | 95% | 100% |
| Must | Declare nullable parameters explicitly | 95% | 100% |
| Must | Use cache-aside reads that preserve falsy values | 95% | 100% |
| Must | Read environment variables in configuration files and read those settings through config() in application code | 94% | 100% |
| Must | Use class-based factories and named state methods | 94% | 86% |
| Must | Detect inherited and nested trait use | 94% | 83% |
| Must | Check engine-specific SQL against the deployed database | 94% | 100% |
| Must | Make exception expectations explicit | 94% | 100% |
| Must | Keep provider register methods focused on bindings | 94% | 100% |
| Must | Queue slow or retryable work durably | 94% | 100% |
| Must | Use random storage names with collision protection | 94% | 100% |
| Must | Bind dynamic values in raw SQL instead of interpolating them | 93% |  |
| Must | Validate external input on the server before use | 93% | 100% |
| Must | Serve Laravel from the public directory | 93% | 100% |
| Must | Normalize login rate-limit identifiers like database matching | 93% | 100% |
| Must | Use cryptographic randomness for secrets | 93% | 100% |
| Must | Generate security tokens from cryptographically secure randomness | 93% | 100% |
| Must | Restrict CORS origins to trusted clients | 93% | 83% |
| Must | Store application timestamps in UTC and convert them at presentation boundaries | 92% | 100% |
| Must | Name model trait boot hooks boot{TraitName}() | 92% |  |
| Must | Exclude authentication secrets from model serialization | 92% | 100% |
| Must | Return anonymous migration classes | 92% | 100% |
| Must | Use $request->input() instead of $request->get(); input() supports dot notation for nested values | 92% |  |
| Must | Provide recovery-code management for two-factor authentication | 91% | 100% |
| Must | Declare development tools in Composer require-dev | 91% | 100% |
| Must | Keep authentication factors independently enforced | 91% | 100% |
| Must | Choose cryptographic hashes suited to their purpose | 90% | 100% |
| Must | Declare PHPUnit metadata with attributes | 90% | 100% |
| Must | Restrict register() to bindings and defer service-dependent bootstrapping to boot() | 90% | 100% |
| Must | Keep HTTP tests to one principal request | 89% | 93% |
| Must | Exclude Vite hot markers from production | 89% | 100% |
| Must | Use the Tailwind 4 CSS import | 89% | 83% |
| Must | Use the application base case for framework tests | 89% | 86% |
| Must | Keep route names unique | 88% | 100% |
| Must | Hide sensitive model attributes from serialization | 88% | 100% |
| Must | Namespace package and module views | 88% | 100% |
| Must | Serialize model identifiers rather than live dependencies | 88% | 100% |
| Must | Use standard resource controller actions | 87% | 100% |
| Must | Bind variable boundary contracts to concrete implementations in a service provider’s register() method | 86% | 100% |
| Must | Check expected HTTP status codes or throw before consuming an outbound success payload | 86% |  |
| Must | Read persisted state in failure handlers | 86% |  |
| Must | Enforce permission checks on protected actions | 86% | 100% |
| Must | Send CSRF credentials from browser clients | 86% | 100% |
| Must | Authenticate and authorize private-channel subscription signing | 86% |  |
| Must | Use application environment predicates for environment checks | 85% | 88% |
| Must | Keep browser request-forgery protection enabled | 85% | 100% |
| Must | Prefer non-mutating collection transforms when shared state is unnecessary | 84% | 100% |
| Must | Confirm a second-factor code before enabling MFA and require step-up for management | 83% | 75% |
| Must | Provide usable MFA recovery codes | 82% | 100% |
| Must | Declare Blade component data inputs | 80% | 100% |
| Must | Never trust user-supplied fields like user_id, role or is_admin from the request | 80% |  |
| Must | Register macros, gates and view extensions during provider boot | 80% | 100% |
| Must | Store intended post-login redirects in the session instead of trusting arbitrary redirect parameters | 75% |  |
| Must | Centralize password validation defaults | 75% | 100% |
| Must | Generate storage names for untrusted uploads | 75% |  |
| Must | Bind collaborator mocks through the container | 75% | 75% |
| Must | Sanitize intentional rich HTML | 70% |  |
| Must | Implement truthful reversible down methods | 69% | 100% |
| Must | Prepare PHP data before rendering and use context-safe encoding when passing it to JavaScript | 67% | 100% |
| Must | Encode server data safely for JavaScript | 67% | 100% |
| Must | Include CSRF tokens in state-changing forms | 67% |  |
| Must | Compare shared secrets with timing-safe equality | 67% | 75% |
| Must | Reset mutable static state after tests | 67% |  |
| Must | Translate user-facing text in localized applications | 64% |  |
| Must | Use cursor() only for streaming reads that do not require eager-loaded relationships | 62% |  |
| Must | Explicitly render API exceptions as JSON when the API contract requires it regardless of Accept headers | 62% | 100% |
| Must | Declare Attribute accessors and attributed local scopes protected | 60% |  |
| Must | Authenticate signed API and webhook payloads before processing them | 60% |  |
| Must | Scope dispatch uniqueness to the logical operation | 60% |  |
| Must | Load package HTTP route definitions through loadRoutesFrom | 58% | 100% |
| Must | Allowlist fields for each write operation | 58% | 75% |
| Must | Authenticate CSRF-exempt webhooks | 57% |  |
| Must | Restrict who can frame the application | 56% | 0% |
| Must | Guard request-derived mass assignment | 56% | 67% |
| Must | Use the Laravel Vite asset pipeline | 54% | 100% |
| Must | Pass validated input to persistence and downstream actions | 54% | 100% |
| Must | Use a dedicated test database | 51% | 50% |
| Must | Pin GitHub Actions to immutable revisions | 51% | 100% |
| Must | Declare foreign keys with deliberate targets and actions | 50% | 75% |
| Must | Encrypt recoverable sensitive attributes | 50% | 100% |
| Must | Revoke persistent remember-me credentials during password reset | 50% | 0% |
| Must | Make nonconventional model factories discoverable | 50% | 67% |
| Must | Implement reusable custom rules with ValidationRule | 50% |  |
| Must | Commit dependent writes atomically | 47% | 50% |
| Must | Restrict classes restored from persisted cache payloads | 46% |  |
| Must | Use JSON session serialization when objects are unnecessary | 46% | 100% |
| Must | Honor batch cancellation before doing work | 43% |  |
| Must | Escape untrusted HTML output | 43% | 67% |
| Must | Load relationship counts with withCount() instead of hydrating children | 40% | 0% |
| Must | Queue external notification delivery | 40% |  |
| Must | Prohibit destructive database reset commands in production | 38% | 75% |
| Must | Check upload contents rather than trusting names or client metadata | 38% |  |
| Must | Reset application-owned state between worker lifecycles | 36% |  |
| Must | Invalidate sessions and regenerate CSRF tokens on logout or account deletion | 36% | 100% |
| Must | Protect authenticated application pages with auth and verified middleware | 33% | 100% |
| Must | Namespace resources for parallel tests | 31% | 20% |
| Should | Mark intentionally empty bodies under the placeholder convention | 100% | 100% |
| Should | Represent a disabled driver with a null implementation | 100% | 100% |
| Should | Combine sorted trait imports under the comma-separated convention | 100% | 100% |
| Should | Use invoke as the entry point for invokable actions | 100% |  |
| Should | Extract shared libraries only around standalone responsibilities | 100% | 100% |
| Should | Separate domain subtyping from cross-cutting capabilities | 100% | 100% |
| Should | Reuse suitable Laravel and application capabilities | 100% | 100% |
| Should | Use Laravel localization facilities | 100% | 100% |
| Should | Remove unused dependencies after checking usage | 100% | 100% |
| Should | Require a concrete reason for an abstraction | 100% | 100% |
| Should | Respect class visibility boundaries | 100% | 100% |
| Should | Define one unconditional type per application file | 100% | 100% |
| Should | Expose narrow overridable steps in extensible actions | 100% |  |
| Should | Give classes one cohesive responsibility | 100% | 100% |
| Should | Remove no-op value helper calls | 100% |  |
| Should | Use lookup defaults only when semantics match | 100% | 100% |
| Should | Separate class members consistently | 100% | 100% |
| Should | Remove obsolete package PHPDoc tags | 100% | 100% |
| Should | Use inheritdoc only for inherited contracts | 100% | 100% |
| Should | Avoid duplicate trait members | 100% | 100% |
| Should | Import resolvable traits | 100% | 100% |
| Should | Reference existing interfaces in inheritance declarations | 100% | 100% |
| Should | Keep magic method annotations valid | 100% | 100% |
| Should | Keep magic property annotations valid | 100% | 100% |
| Should | Use classes for PHP attribute types | 100% | 100% |
| Should | Remove trait conditions proven constant in consumers | 100% | 100% |
| Should | Catch valid throwable types | 100% | 100% |
| Should | Match generic interface annotations to real contracts | 100% | 100% |
| Should | Match generic trait annotations to imported traits | 100% | 100% |
| Should | Declare interface methods public | 100% | 100% |
| Should | Implement every required method in concrete types | 100% | 100% |
| Should | Use early returns for invalid or empty input | 100% | 100% |
| Should | Preserve inherited exception contracts | 100% | 100% |
| Should | Avoid PHP 8.5 global helper name collisions | 100% | 100% |
| Should | Replace obsolete Laravel type names | 100% | 100% |
| Should | Bound generics to required capabilities | 100% | 100% |
| Should | Keep generic parameter counts minimal | 100% | 100% |
| Should | Handle exhausted rate-limit attempts | 100% |  |
| Should | Guard web-only package bootstrapping | 100% | 100% |
| Should | Customize Fortify registration through its action | 100% | 100% |
| Should | Register Fortify action implementations centrally | 100% | 100% |
| Should | Handle noninteractive input in custom commands | 100% | 100% |
| Should | Keep package scaffolding proportional to shipped features | 100% | 100% |
| Should | Keep feature frontend modules separate from shared infrastructure | 100% | 100% |
| Should | Split optional capabilities into small interfaces | 100% | 100% |
| Should | Keep one application task in each action | 100% | 100% |
| Should | Reserve primary styling for the screen main action | 100% | 100% |
| Should | Preserve behavioral substitutability | 100% | 100% |
| Should | Depend on abstractions across high and low level modules | 100% | 100% |
| Should | Extend an existing behavior through its implementation contract | 100% | 100% |
| Should | Expose clear extension points instead of accumulating tiny options | 100% | 100% |
| Should | Use an atomic write when a key must be absent | 100% | 100% |
| Should | Use explicit methods on contract-typed cache repositories | 100% | 100% |
| Should | Initialize expiring counters atomically before updating them | 100% | 100% |
| Should | Configure TOTP drift tolerance deliberately | 100% | 100% |
| Should | Keep environment fallbacks compatible with the intended setting type | 100% | 100% |
| Should | Parse simple environment-backed lists once in configuration | 100% | 100% |
| Should | Keep exception configuration in the registered exception handler when retaining the Laravel 10 application structure | 100% |  |
| Should | Let the container resolve concrete classes until an abstraction has a real substitution boundary | 100% | 100% |
| Should | Match CI PHP versions, extensions, and container services to the project’s supported development and runtime requirements | 100% | 100% |
| Should | Keep third-party service credentials and connection settings in config/services.php | 100% |  |
| Should | Keep user- or deployment-dependent computations in application code rather than database generated columns or views | 100% | 100% |
| Should | Declare auto-discoverable package providers in Composer extra.laravel.providers | 100% | 100% |
| Should | Offer publishable package configuration and assets through named publish groups | 100% | 100% |
| Should | Register package Artisan commands through the service provider | 100% | 100% |
| Should | Read environment-backed settings through env() inside configuration files instead of direct $_ENV access | 100% | 100% |
| Should | Use self for enum and final-class member references when late static binding cannot change the target | 100% | 100% |
| Should | Use array-unpacking key types supported by the configured PHP runtime | 100% | 100% |
| Should | Pass container-resolved class names to pipelines whose stages need injected dependencies | 100% | 100% |
| Should | Configure an ordered failover mailer when delivery must survive a primary provider outage | 100% | 100% |
| Should | Set shared sender and reply-to defaults in config/mail.php instead of repeating them in mailables | 100% | 100% |
| Should | Use Laravel filesystem disks for application file operations that must move between storage backends | 100% |  |
| Should | Use filesystem paths relative to the configured disk root | 100% |  |
| Should | Configure an intentional fallback locale for missing translations | 100% | 100% |
| Should | Put application bootstrapping and container bindings in service providers | 100% | 100% |
| Should | Rely on automatic resolution for concrete classes that need no custom construction or lifetime | 100% | 100% |
| Should | Register package service boundaries in the container so consumers can resolve and replace them | 100% | 100% |
| Should | Keep class responsibilities focused as dependency and facade usage grows | 100% | 100% |
| Should | Keep closures out of configuration files that must support config:cache | 100% | 100% |
| Should | Provide complete nested overrides or an explicit merge strategy for package configuration merged with mergeConfigFrom() | 100% | 100% |
| Should | Configure application exception policy through withExceptions() in the streamlined bootstrap | 100% | 100% |
| Should | Route unexpected application exceptions to configured logs or an error tracker | 100% | 100% |
| Should | Capture external values explicitly in manager extend() closures instead of relying on a provider-bound $this | 100% |  |
| Should | Use a round-robin mail transport when delivery load should be distributed across providers | 100% | 100% |
| Should | Keep Larastan’s noEnvCallsOutsideOfConfig rule enabled when enforcing config-only environment access | 100% | 100% |
| Should | Make package cache-store selection configurable and default to the application’s chosen store | 100% |  |
| Should | Configure optional framework services lazily through resolution callbacks | 100% | 100% |
| Should | Remove unused authentication guards | 100% | 100% |
| Should | Commit an explicit PHPStan analysis level | 100% | 100% |
| Should | Allowlist required Composer plugins explicitly | 100% | 100% |
| Should | Wrap third-party library initialization behind a small application-owned module surface | 100% |  |
| Should | Expose package lifecycle events for open-ended integration requirements instead of adding one configuration switch per custom behavior | 100% | 100% |
| Should | Document development conventions or encode them in committed repository tooling | 100% | 100% |
| Should | Declare Sail as a development-only Composer dependency | 100% | 100% |
| Should | Generate standards-compliant UUIDs for native UUID columns | 100% |  |
| Should | Apply the schema required by enabled Fortify authentication features | 100% | 100% |
| Should | Use SQLite for simple local development when its behavior is sufficient | 100% | 100% |
| Should | Use full Unicode storage with a deliberate MySQL collation | 100% | 100% |
| Should | Disable Apache content negotiation and directory listing for Laravel | 100% | 100% |
| Should | Keep mutable local SQLite databases out of source control | 100% | 100% |
| Should | Choose compact column types that faithfully fit domain values | 100% | 100% |
| Should | Keep InnoDB primary keys compact when identity requirements allow | 100% | 100% |
| Should | Make database nullability reflect actual absence semantics | 100% | 100% |
| Should | Render Boost skill Artisan commands for the host execution environment | 100% | 100% |
| Should | Use conventional foreign and pivot key names | 100% | 100% |
| Should | Configure cache failover where cache availability is required | 100% | 100% |
| Should | Keep each migration focused on one deployable intent | 100% | 100% |
| Should | Use snake_case database columns | 100% | 100% |
| Should | Make required query ordering explicit | 100% | 100% |
| Should | Use id for conventional primary keys | 100% | 100% |
| Should | Perform bulk relational changes inside the database | 100% | 100% |
| Should | Reserve JSON attributes for sparse flexible data | 100% | 100% |
| Should | Keep database JSON shapes shallow | 100% | 100% |
| Should | Serialize API responses through explicit resources | 100% | 100% |
| Should | Memoize repeated Livewire lookups with computed properties | 100% | 100% |
| Should | Register application seeders in DatabaseSeeder | 100% | 100% |
| Should | Use schema-builder callbacks for portable table definitions | 100% | 100% |
| Should | Allow IPv6-sized nullable session IP addresses | 100% | 100% |
| Should | Publish package migrations with timestamp-aware publication | 100% | 100% |
| Should | Import the Schema facade explicitly | 100% | 100% |
| Should | Rename frontend environment variables during Vite migration | 100% | 100% |
| Should | Preserve associative keys when merging collections | 100% | 100% |
| Should | Reindex transformed collections when returning a list | 100% | 100% |
| Should | Use join predicates that distinguish columns from values | 100% | 100% |
| Should | Check database support for the JSON query forms used | 100% |  |
| Should | Keep database connection details in environment-driven configuration | 100% | 100% |
| Should | Configure an unambiguous SQLite database file path | 100% | 100% |
| Should | Apply foreign-key column modifiers before constrained | 100% | 100% |
| Should | Split large seeders into focused ordered classes | 100% |  |
| Should | Avoid total counts when pagination needs only next/previous | 100% |  |
| Should | Put public static assets in the public directory | 100% | 100% |
| Should | Keep seeders in the generated Laravel namespace and directory | 100% | 100% |
| Should | Track ownership of database-backed locks | 100% | 100% |
| Should | Organize package JavaScript in predictable modules | 100% | 100% |
| Should | Keep service-provider registration free of host-state reads | 100% | 100% |
| Should | Centralize package resource registration with package tools | 100% |  |
| Should | Use explicit NULL predicates | 100% |  |
| Should | Namespace package cache keys | 100% | 100% |
| Should | Use exact queries for exact-count requirements | 100% |  |
| Should | Clamp keyboard selection to available items | 100% | 100% |
| Should | Isolate component-internal z-index layers | 100% | 100% |
| Should | Size encrypted columns for ciphertext expansion | 100% | 100% |
| Should | Equip the configured user model for each enabled Fortify authentication feature | 100% | 100% |
| Should | Persist significant domain-event history when traceability and replay are required | 100% |  |
| Should | Dispatch optional driver behavior through capability contracts | 100% | 100% |
| Should | Order parents by a single related value without multiplying parent rows | 100% |  |
| Should | Allow lazy loading only when a relationship is genuinely optional or a bounded single-model read | 100% |  |
| Should | Cast date attributes in the model instead of parsing database strings in consumers | 100% |  |
| Should | Prefer Eloquent relationships for model-backed queries that need model behavior | 100% | 22% |
| Should | Resolve model-configured table names when lower-level application queries must follow the model | 100% |  |
| Should | Keep explicit action routes when resource modeling would obscure domain meaning | 100% |  |
| Should | Name Eloquent models in singular StudlyCase | 100% | 100% |
| Should | Name conventional pivot tables with alphabetical singular model names in snake_case | 100% |  |
| Should | Use typed request accessors when their coercion matches the input contract | 100% |  |
| Should | Keep HTML data attributes small and avoid embedding full model payloads | 100% | 100% |
| Should | Use built-in casts for simple conversions and a consistent custom-cast boundary for richer values | 100% | 100% |
| Should | Keep model lifecycle handlers in the established observer, callback or event-class pattern | 100% |  |
| Should | Choose explicit per-query or model-default eager loading based on repeated access needs | 100% |  |
| Should | Use conventional foreign-key declarations when their inferred table, type and actions match the schema | 100% | 100% |
| Should | Choose pagination methods per endpoint contract and query characteristics | 100% |  |
| Should | Use wire:model.live when a Livewire input must update the server as the user edits | 100% |  |
| Should | Centralize data-driven native UI colors in one enum or model method | 100% |  |
| Should | Name columns in snake_case without repeating the owning model name | 100% | 100% |
| Should | Follow Eloquent schema conventions for new models unless domain or legacy constraints require overrides | 100% |  |
| Should | Use model casts and Attribute objects for domain value conversion | 100% | 100% |
| Should | Avoid repositories or wrappers that only forward Eloquent calls | 100% | 100% |
| Should | Use when() for optional query constraints when it keeps composition clear | 100% | 100% |
| Should | Use SoftDeletes for recoverable deletion that should receive standard query scoping | 100% |  |
| Should | Use loadMissing() when an already-loaded relation should be retained | 100% |  |
| Should | Install API and broadcasting scaffolding only when the application needs those features | 100% |  |
| Should | Validate and normalize external input before security-sensitive comparisons | 100% | 100% |
| Should | Check existence on a query rather than on a possibly null fetched model | 100% |  |
| Should | Push large-data filtering into SQL rather than filtering hydrated models in PHP | 100% |  |
| Should | Use model creation for seed data that requires casts, observers or lifecycle behavior | 100% |  |
| Should | Use descriptive singular variables for individual model instances | 100% | 100% |
| Should | Use AsArrayObject when JSON attributes must support direct nested-offset mutation | 100% |  |
| Should | Keep ordinary application models in App\Models unless the project uses an established modular layout | 100% | 100% |
| Should | Call parent::boot() when overriding model boot() | 100% |  |
| Should | Write the PHP scope-resolution operator without surrounding spaces | 100% | 100% |
| Should | Replace the removed unset cast with explicit null or unset as appropriate | 100% | 100% |
| Should | Pass only string-convertible values to output and native string-conversion operations | 100% | 100% |
| Should | Use conversions supported by the source and required target types | 100% | 100% |
| Should | Prefer composition for reusable model/controller behavior over deep custom base-class hierarchies | 100% | 100% |
| Should | Use a model-aware exists rule when validation must follow model table and connection configuration | 100% |  |
| Should | Group custom OR predicates before chunkById() or lazyById() adds its key constraint | 100% |  |
| Should | Use fail-on-missing retrieval when an absent required record should return 404 | 100% |  |
| Should | Use per-model writes when lifecycle events or cleanup hooks are required | 100% | 100% |
| Should | Use addSelect() when a global scope augments selected columns | 100% |  |
| Should | Group OR predicates so they cannot escape relationship or global-scope constraints | 100% | 100% |
| Should | Compare model identity with is() or isNot() when table and connection matter | 100% |  |
| Should | Place lifecycle work before or after persistence according to its data requirements | 100% |  |
| Should | Group substantial sets of model lifecycle listeners in an observer | 100% |  |
| Should | Represent a true many-to-many association with an intermediate table | 100% |  |
| Should | Keep relationship existence subqueries within one supported database query boundary | 100% |  |
| Should | Keep attribute/cast names distinct from relationship names | 100% | 100% |
| Should | Configure primary-key types through key metadata rather than an ordinary attribute cast | 100% | 100% |
| Should | Keep chunk progression and relationship keys stable during batch writes | 100% |  |
| Should | Match polymorphic ID column types to the related models' key strategy | 100% |  |
| Should | Use class-aware route can() authorization where it improves policy readability | 100% | 100% |
| Should | Use authenticated private or presence channels for nonpublic broadcast data | 100% |  |
| Should | Allowlist broadcast event payload fields | 100% |  |
| Should | Extract channel authorization into channel classes when route closures become substantial | 100% |  |
| Should | Use model broadcasting for events whose sole purpose is exposing model lifecycle changes | 100% |  |
| Should | Avoid instantiating a model while its own boot cycle is running | 100% |  |
| Should | Name event classes after completed domain events | 100% | 100% |
| Should | Name model observers with the model name and Observer suffix | 100% |  |
| Should | Use nested eager-load arrays for deep repeated relationship prefixes when they improve clarity | 100% |  |
| Should | Declare a relation's column projection once per eager-load plan | 100% |  |
| Should | Add withCount() after custom select() calls | 100% |  |
| Should | Normalize query values to the intended column type before MySQL/MariaDB comparisons | 100% |  |
| Should | Name boolean state and reversible setters clearly | 100% |  |
| Should | Register package Blade extensions after the compiler resolves | 100% | 100% |
| Should | Pass route-key-aware model objects to Ziggy route generation | 100% | 100% |
| Should | Define and bind explicit contracts for swappable package services and model strategies | 100% | 100% |
| Should | Use Str::slug() for one-off unpersisted slugs | 100% | 100% |
| Should | Map domain exceptions to intentional HTTP responses at the application boundary | 100% |  |
| Should | Declare a precise array return contract for model casts() | 100% | 100% |
| Should | Defer package console command resolution to Artisan startup | 100% | 100% |
| Should | Observe framework activity through existing lifecycle events | 100% | 100% |
| Should | Name events according to when the action occurs | 100% | 100% |
| Should | Persist email verification state in the schema | 100% | 100% |
| Should | Set the application name used in branded notifications | 100% |  |
| Should | Register Fortify view callbacks during application provider boot | 100% | 100% |
| Should | Apply a Blade-aware formatter only to genuine Blade templates | 100% | 100% |
| Should | Use concise named-slot tags under the adopted Blade convention | 100% | 100% |
| Should | Avoid Blade wrappers with no reusable interface | 100% | 100% |
| Should | Reuse existing UI components | 100% | 100% |
| Should | Prefer direct view data when it is sufficient | 100% | 100% |
| Should | Forward declared wrapper props explicitly | 100% | 100% |
| Should | Reuse shared Blade layout markup | 100% | 100% |
| Should | Use Blade for Laravel server templates | 100% | 100% |
| Should | Keep view-data types stable | 100% | 100% |
| Should | Do not locate Blade sources with PHP file constants | 100% | 100% |
| Should | Render stacks after their producers | 100% |  |
| Should | Use Blade comments for private template notes | 100% | 100% |
| Should | Keep Blade directives out of component tag attributes | 100% | 100% |
| Should | Avoid reserved Blade component member names | 100% |  |
| Should | Provide fallbacks for optional layout slots | 100% |  |
| Should | Avoid periods in view directory names | 100% | 100% |
| Should | Share global view data during provider boot | 100% |  |
| Should | Use build-managed relative asset references | 100% | 100% |
| Should | Configure fonts through the fonts option | 100% | 100% |
| Should | Keep authoritative Livewire state on the server | 100% | 100% |
| Should | Avoid the reserved upload method name | 100% | 100% |
| Should | Match the existing Tailwind styling system | 100% | 100% |
| Should | Extract repeated utility patterns into UI components | 100% | 100% |
| Should | Remove redundant and conflicting utility classes | 100% | 100% |
| Should | Use utilities supported by the installed Tailwind major | 100% | 100% |
| Should | Replace removed opacity utilities in Tailwind 4 | 100% | 100% |
| Should | Prefer utilities over repeated inline styles | 100% | 100% |
| Should | Write complete literal utility names | 100% | 100% |
| Should | Use automatic command discovery in standard directories | 100% | 100% |
| Should | Use a fixed executable with separate argument values | 100% | 100% |
| Should | Replace obsolete view expression types | 100% | 100% |
| Should | Version shared project convention files | 100% | 100% |
| Should | Enforce agreed formatting with automated tooling | 100% | 100% |
| Should | Render paginator-generated links | 100% |  |
| Should | Use pagination templates matching the frontend CSS framework | 100% |  |
| Should | Explain verification and provide a resend action | 100% | 100% |
| Should | Show authenticator enrollment and recovery information | 100% | 100% |
| Should | Treat non-class merged attributes as overridable defaults | 100% |  |
| Should | Match component parameter and HTML attribute naming | 100% |  |
| Should | Normalize package view directories before registration | 100% | 100% |
| Should | Encapsulate repeated Vue DOM initialization in a component | 100% | 100% |
| Should | Name Livewire components for the user action they implement | 100% | 100% |
| Should | Integrate package services and resources through its service provider | 100% | 100% |
| Should | Skip verification mail for users whose email is already verified | 100% |  |
| Should | Keep reusable package controllers independent of the host base controller | 100% | 100% |
| Should | Select an explicit supported version for versioned external APIs | 100% |  |
| Should | Reuse organization-maintained CI workflows instead of copying their logic | 100% | 100% |
| Should | Keep streamlined application base controllers minimal | 100% | 100% |
| Should | Use lowercase Composer package identifiers | 100% | 100% |
| Should | Prefer container-backed facades in controllers under the facade-first convention | 100% |  |
| Should | Prepare response data explicitly for JSON and streamed responses rather than relying on view composers | 100% | 100% |
| Should | Define explicit routes for operations that do not fit resource-controller actions | 100% |  |
| Should | Centralize repeated outbound client settings in a dedicated client or macro | 100% |  |
| Should | Do not return HTTP 200 with a generic error message payload for failures | 100% | 100% |
| Should | Do not add a permanent /clear-cache web route, even behind authentication | 100% | 100% |
| Should | Default to explicit route registration; scope automatic discovery to deliberately selected areas | 100% | 100% |
| Should | Avoid terminating requests while debugging session writes | 100% | 100% |
| Should | Never run untrusted code from forks in a pull_request_target workflow | 100% | 100% |
| Should | Remove obsolete translation roots after migration | 100% | 100% |
| Should | Run session-dependent middleware after session startup | 100% |  |
| Should | Prefer anonymous Blade components so template-specific code lives under resources/views | 100% | 100% |
| Should | Memoize repeated expensive calculations for the current request only | 100% |  |
| Should | Do not expose phpinfo() through application routes, including administrator-only routes | 100% | 100% |
| Should | Prefer explicit resource controllers over an ever-growing universal CRUD base controller | 100% | 100% |
| Should | Do not mutate application state through GET routes | 100% | 100% |
| Should | Use PSR-4 mappings for application or package namespaces in composer.json | 100% | 100% |
| Should | Declare arrays with short [] syntax | 100% | 100% |
| Should | Place PHP code on a line after the opening tag and separate it with a blank line | 100% | 100% |
| Should | Write nullable type declarations without spaces between ? and the type | 100% | 100% |
| Should | Write true, false, and null in lowercase | 100% | 100% |
| Should | Save PHP files as UTF-8 without a byte-order mark | 100% | 100% |
| Should | Use <?php or <?= instead of other PHP opening-tag variants | 100% | 100% |
| Should | Normalize legacy @inheritDocs tags to @inheritDoc with an explicit PHPDoc tag replacement map | 100% | 100% |
| Should | Use nowdoc instead of heredoc when the string needs no interpolation | 100% | 100% |
| Should | Indent PHP code with spaces instead of tabs | 100% | 100% |
| Should | Use lowercase numeric base prefixes and uppercase hexadecimal digits in integer literals | 100% | 100% |
| Should | Use LF line endings consistently in PHP files | 100% | 100% |
| Should | Put code after a PHP opening tag on a new line | 100% | 100% |
| Should | Write PHP keywords in lowercase | 100% | 100% |
| Should | Write self, static, and parent in lowercase | 100% | 100% |
| Should | Use the canonical uppercase spelling of PHP magic constants | 100% | 100% |
| Should | Remove spaces before argument commas and put one space after commas without forcing multiline argument lists to expand | 100% | 100% |
| Should | Call native PHP functions using their canonical casing | 100% | 100% |
| Should | Call canonical PHP functions instead of their aliases | 100% | 100% |
| Should | Remove redundant b or B prefixes from string literals | 100% | 100% |
| Should | Do not put blank lines immediately after a class opening brace | 100% | 100% |
| Should | Place PHPDoc blocks directly above the documented element without an intervening blank line | 100% | 100% |
| Should | Remove empty PHPDoc blocks | 100% | 100% |
| Should | Remove redundant empty semicolon statements | 100% | 100% |
| Should | Do not split an array key and its => operator across lines | 100% | 100% |
| Should | Put each PHP statement on its own line | 100% | 100% |
| Should | Remove spaces immediately before statement-ending semicolons | 100% | 100% |
| Should | Do not put spaces immediately inside or outside array-offset brackets | 100% | 100% |
| Should | Do not add trailing commas to lists written entirely on one line | 100% | 100% |
| Should | Remove trailing whitespace from non-blank code lines | 100% | 100% |
| Should | Remove unnecessary parentheses around break, clone, continue, echo, return, case, and yield expressions | 100% | 100% |
| Should | Remove superfluous braces while retaining control-structure body braces | 100% | 100% |
| Should | Do not declare optional parameters before required parameters | 100% | 100% |
| Should | Remove empty return statements at the end of functions | 100% | 100% |
| Should | Do not put whitespace before array-item commas | 100% | 100% |
| Should | Do not put spaces around -> or ?-> | 100% | 100% |
| Should | Indent PHPDoc blocks to the same level as the documented element | 100% | 100% |
| Should | Normalize the braces and syntax of inline PHPDoc tags | 100% | 100% |
| Should | Remove obsolete @access PHPDoc tags | 100% | 100% |
| Should | Order @param, @return, and @throws PHPDoc tags in that sequence | 100% | 100% |
| Should | Use int, bool, and float instead of integer, boolean, real, or double in PHPDoc | 100% | 100% |
| Should | Write inheritdoc as an inline PHPDoc tag | 100% | 100% |
| Should | Use the canonical casing of built-in PHPDoc types | 100% | 100% |
| Should | Put no space before and one space after the return-type colon | 100% | 100% |
| Should | Declare each class property and constant in its own statement | 100% | 100% |
| Should | Use a separate use statement for every import | 100% | 100% |
| Should | Put each import on its own line and one blank line after the import block | 100% | 100% |
| Should | Use // instead of # for single-line comments | 100% | 100% |
| Should | Put one space after semicolons when another expression follows on the same line | 100% | 100% |
| Should | Remove spaces immediately inside parentheses | 100% | 100% |
| Should | Use != instead of <> | 100% | 100% |
| Should | Put one space around ternary operators | 100% | 100% |
| Should | Remove spaces immediately inside single-line array brackets | 100% | 100% |
| Should | Put one space between a type declaration and its variable | 100% | 100% |
| Should | Put one space after array-item commas when the next item is on the same line | 100% | 100% |
| Should | Read and write only accessible declared properties or properties supported by a documented dynamic API | 100% | 100% |
| Should | Use the injected request object in controllers and middleware instead of equivalent request facade or helper calls | 100% | 100% |
| Should | Report optional failures and return a defined fallback | 100% | 100% |
| Should | Put injected dependencies before route parameters | 100% | 100% |
| Should | Design each middleware as a single layer that examines the request and may reject it entirely before it reaches the application | 100% | 100% |
| Should | Keep page presentation in view templates rather than constructing full HTML documents in controllers or routes | 100% | 100% |
| Should | Register HTTP-verb-specific routes before any, match, and redirect routes sharing the same URI | 100% | 100% |
| Should | Apply session middleware to back redirects that depend on flashed input, errors or session fallback state | 100% |  |
| Should | Return a JSON 503 maintenance response to JSON clients instead of a browser maintenance page or redirect | 100% | 100% |
| Should | Use descriptive variable names | 100% | 100% |
| Should | Submit Livewire forms through wire:submit | 100% |  |
| Should | Give application view overrides precedence | 100% | 100% |
| Should | Enable prefer-stable when a package intentionally permits development-stability dependencies | 100% | 100% |
| Should | Declare package service providers and supported facade aliases in Composer Laravel auto-discovery metadata | 100% | 100% |
| Should | Make package route prefix/path, domain and middleware configurable through the package config instead of hard-coding them | 100% | 100% |
| Should | Do not evaluate code supplied in HTTP requests | 100% | 100% |
| Should | Rate-limit abuse-prone and resource-intensive endpoints with workload-appropriate keys and quotas | 100% | 100% |
| Should | Scope input-normalization exceptions to the fields or request types that require them | 100% |  |
| Should | Aggregate intended log destinations through a stack | 100% | 100% |
| Should | Keep an explicit CLI interface for automatable operations | 100% | 100% |
| Should | Set protected form values on the server | 100% |  |
| Should | Avoid OAuth implicit grants | 100% |  |
| Should | Rate-limit authentication endpoints | 100% | 100% |
| Should | Let applications opt into package schedules | 100% | 100% |
| Should | Register console behavior in the structure the app retains | 100% |  |
| Should | Keep bundled skill descriptions within client limits | 100% | 100% |
| Should | Provide intentional runtime extension points with Macroable | 100% | 100% |
| Should | Generate and bind passkey challenges on the server | 100% | 100% |
| Should | Let responsive flex rows wrap from a deliberate item basis | 100% | 100% |
| Should | Use the project exception customization location | 100% | 100% |
| Should | Exclude deliberately non-reportable exceptions | 100% |  |
| Should | Prefer readable framework and language operations | 100% | 100% |
| Should | Keep comments aligned with behavior | 100% | 100% |
| Should | Reserve raw SQL for necessary query behavior | 100% | 100% |
| Should | Exclude generated builds and runtime files from version control | 100% | 100% |
| Should | Mark intentionally unresolved types explicitly | 100% | 100% |
| Should | Use Laravel authentication components | 100% | 100% |
| Should | Render untrusted Markdown with safe options | 100% |  |
| Should | Introduce abstractions for concrete variation | 100% | 100% |
| Should | Split unrelated method responsibilities | 100% | 100% |
| Should | Prefer native declarations over redundant PHPDoc | 100% | 100% |
| Should | Generate HTTPS URLs behind trusted TLS termination | 100% |  |
| Should | Avoid security questions for account recovery | 100% | 100% |
| Should | Start with the default Laravel structure | 100% | 100% |
| Should | Choose collection extraction semantics deliberately | 100% | 100% |
| Should | Sort structured collection values by an explicit field | 100% |  |
| Should | Use interoperable agent skill packaging | 100% | 100% |
| Should | Avoid reinstalling authentication already scaffolded | 100% | 100% |
| Should | Use constrained choices for finite CLI inputs | 100% | 100% |
| Should | Record application-specific agent conventions | 100% |  |
| Should | Prompt for missing inputs in interactive commands | 100% |  |
| Should | Offer deliberate macro extension points | 100% | 100% |
| Should | Reject unsupported state transitions | 100% | 100% |
| Should | Avoid duplicate per-job failure reporting | 100% |  |
| Should | Use batches for tracked groups of jobs | 100% |  |
| Should | Dispatch a job for one deferred action | 100% | 100% |
| Should | Name jobs for the action they perform | 100% | 100% |
| Should | Distinguish command classes from jobs | 100% | 100% |
| Should | Treat unique conflicts as already-completed creation | 100% |  |
| Should | Bound deadlock retries | 100% |  |
| Should | Publish package lifecycle events as extension points | 100% | 100% |
| Should | Keep backed enums valid | 100% | 100% |
| Should | Use unique parameter names | 100% | 100% |
| Should | Declare valid PHPDoc generic contracts | 100% | 100% |
| Should | Replace legacy job retry configuration names | 100% |  |
| Should | Return the model explicitly after updating it | 100% | 100% |
| Should | Choose connection and queue separately | 100% | 100% |
| Should | Keep application jobs in the conventional job directory | 100% |  |
| Should | Implement reusable queue concerns as middleware | 100% |  |
| Should | Chain jobs that require successful predecessors | 100% |  |
| Should | Avoid capturing this in serialized queue callbacks | 100% |  |
| Should | Avoid implicit-commit statements in batched jobs | 100% |  |
| Should | Use one destination for all jobs in a batch | 100% |  |
| Should | Manage indexes explicitly when changing columns | 100% |  |
| Should | Rescue noncritical broadcast failures | 100% |  |
| Should | Persist the generated stored file path | 100% |  |
| Should | Keep Redis retry configuration cacheable | 100% | 100% |
| Should | Set descriptive distinct page titles | 100% |  |
| Should | Use public events between independent modules | 100% | 100% |
| Should | Keep form instruction IDs unique | 100% |  |
| Should | Allowlist the fields passed to credential authentication | 100% |  |
| Should | Confirm passwords without performing a new login | 100% |  |
| Should | Match anonymous authentication routes to Fortify's configured guard | 100% | 100% |
| Should | Condition compatibility tests on the framework behavior they exercise | 100% | 100% |
| Should | Select the intended database driver explicitly | 100% | 100% |
| Should | Mark intentionally unreported exceptions | 100% |  |
| Should | Build custom permissions on Laravel authorization | 100% |  |
| Should | Use precise validation and throttle statuses | 100% |  |
| Should | Limit CSRF exceptions to specific externally authenticated endpoints | 100% |  |
| Should | Assert image behavior without platform-dependent pixel hashes | 100% | 100% |
| Should | Memoize repeated request-local lookups before adding cross-request cache | 100% | 100% |
| Should | Explain compromised-password validation failures clearly | 100% | 100% |
| Should | Preserve MFA assurance during account recovery | 100% | 100% |
| Should | Use the hashed cast for password writes under a cast-based convention | 100% | 100% |
| Should | Use state-changing HTTP methods for mutations | 100% | 100% |
| Should | Customize API authentication errors through exception rendering | 100% |  |
| Should | Complete generated Form Request authorization stubs | 100% |  |
| Should | Use validation failures for invalid input and authorization failures for denied actors | 100% |  |
| Should | Key two-factor challenge limits to the pending login identity | 100% | 100% |
| Should | Keep authentication session cookies HttpOnly | 100% | 100% |
| Should | Keep encryption and signing on cookies trusted by the application | 100% |  |
| Should | Apply relationship aggregates after custom select clauses | 100% |  |
| Should | Model permissions with authorization rather than extra authentication guards | 100% | 100% |
| Should | Size password columns for the selected hash algorithm | 100% | 100% |
| Should | Use request-carried API credentials for nonbrowser service clients | 100% |  |
| Should | Pass the submitted plaintext password to Auth::attempt | 100% |  |
| Should | Use gates for nonresource abilities and policies for resource abilities | 100% |  |
| Should | Keep supported centralized hasher costs unless measured requirements justify tuning | 100% |  |
| Should | Use per-response CSP nonces consistently in headers and generated assets | 100% | 100% |
| Should | Quote environment-file values containing spaces | 100% | 100% |
| Should | Name middleware after the condition it enforces | 100% | 100% |
| Should | Count only the relevant outcomes in response-based abuse limits | 100% |  |
| Should | Return generic internal errors outside debug environments | 100% | 100% |
| Should | Fail explicitly when a configured security control cannot be applied | 100% | 100% |
| Should | Name behavioral interfaces by their capability | 100% | 100% |
| Should | Use null coalescing for equivalent isset fallbacks | 100% | 100% |
| Should | Do not declare a class constant named class | 100% | 100% |
| Should | Omit final on private constants and ordinary private methods | 100% | 100% |
| Should | Avoid reserved parameter variable names | 100% | 100% |
| Should | Leave constructor return types undeclared | 100% | 100% |
| Should | Keep class identity changes out of incidental formatting | 100% | 100% |
| Should | Put literals first under an adopted Yoda comparison policy | 100% | 100% |
| Should | Expose distinct builder contexts through descriptive named factories | 100% | 100% |
| Should | Use descriptive domain names | 100% | 100% |
| Should | Keep Laravel directories recognizable | 100% | 100% |
| Should | Follow the project action invocation convention | 100% | 100% |
| Should | Leave incidental one-off strings inline | 100% | 100% |
| Should | Use plural names for collections | 100% | 100% |
| Should | Use singular names for individual objects | 100% | 100% |
| Should | Prefer clear expressions over shorter expressions | 100% | 100% |
| Should | Use Arr for dot paths and standard array operations | 100% | 100% |
| Should | Extract meaningful behavior instead of narrating code | 100% | 100% |
| Should | Apply one deliberate namespace layout | 100% | 100% |
| Should | Pass date objects across application boundaries | 100% | 100% |
| Should | Format dates in the presentation layer | 100% | 100% |
| Should | Preserve intended package extension points | 100% | 100% |
| Should | Back enums that cross scalar boundaries | 100% | 100% |
| Should | Move complex state behavior out of enums | 100% | 100% |
| Should | Separate enum labels from stored values | 100% |  |
| Should | Keep shared traits focused and discoverable | 100% | 100% |
| Should | Define extension contracts as interfaces | 100% | 100% |
| Should | Keep Larastan in development dependencies | 100% | 100% |
| Should | Baseline existing analysis debt during adoption | 100% |  |
| Should | Enable stricter and deprecation analysis where adopted | 100% |  |
| Should | Leave one blank line after namespaces | 100% | 100% |
| Should | Separate the namespace from the file preamble | 100% | 100% |
| Should | Keep namespace names clean and unindented | 100% | 100% |
| Should | Join continuation keywords to closing braces | 100% | 100% |
| Should | Use canonical PHP magic method casing | 100% | 100% |
| Should | Keep semicolons with their expression | 100% | 100% |
| Should | Use canonical lowercase native types | 100% | 100% |
| Should | Omit leading separators from imports | 100% | 100% |
| Should | Attach call parentheses to the callee | 100% | 100% |
| Should | Remove redundant import aliases | 100% | 100% |
| Should | Use canonical internal class casing | 100% | 100% |
| Should | Give intentional empty loops braces | 100% | 100% |
| Should | Remove redundant empty-string concatenation | 100% | 100% |
| Should | Collapse extra PHPDoc blank lines | 100% | 100% |
| Should | Put annotation types before variable names | 100% | 100% |
| Should | Use supported braced string interpolation | 100% | 100% |
| Should | Separate return and control statements under the custom policy | 100% |  |
| Should | Destructure only offset-accessible values | 100% | 100% |
| Should | Remove loops over provably empty input | 100% | 100% |
| Should | Avoid duplicate array literal keys | 100% | 100% |
| Should | Use supported array key types | 100% | 100% |
| Should | Iterate only iterable values | 100% | 100% |
| Should | Read existing supported offsets | 100% | 100% |
| Should | Write offsets only on writable receivers | 100% | 100% |
| Should | Preserve array and ArrayAccess element types | 100% | 100% |
| Should | Unpack only iterable values into arrays | 100% | 100% |
| Should | Validate attribute targets and arguments | 100% | 100% |
| Should | Reference existing accessible constants | 100% | 100% |
| Should | Declare each symbol once in its scope | 100% | 100% |
| Should | Extend existing extensible classes | 100% | 100% |
| Should | Use real object types in instanceof | 100% | 100% |
| Should | Make custom attributes constructible | 100% | 100% |
| Should | Remove genuinely unused parameters | 100% | 100% |
| Should | Remove statically impossible or redundant branches | 100% | 100% |
| Should | Keep match arms exhaustive and reachable | 100% | 100% |
| Should | Make unbounded loops intentional | 100% | 100% |
| Should | Preserve inherited and composed constant contracts | 100% | 100% |
| Should | Use results of pure operations | 100% | 100% |
| Should | Remove unreachable statements after exits | 100% | 100% |
| Should | Remove unconsumed private members | 100% | 100% |
| Should | Keep catch clauses reachable | 100% | 100% |
| Should | Do not override exits in finally | 100% | 100% |
| Should | Remove no-op callback-free array filters | 100% | 100% |
| Should | Avoid reindexing known lists | 100% | 100% |
| Should | Return values matching native contracts | 100% | 100% |
| Should | Do not consume void expression results | 100% | 100% |
| Should | Call accessible symbols with compatible arguments | 100% | 100% |
| Should | Create first-class callables from real callable symbols | 100% | 100% |
| Should | Use resolvable declaration types | 100% | 100% |
| Should | Match defaults to parameter types | 100% | 100% |
| Should | Choose one filter failure mode | 100% | 100% |
| Should | Match formatted-output placeholders and arguments | 100% | 100% |
| Should | Validate random integer bounds | 100% | 100% |
| Should | Control active content in public uploads | 100% |  |
| Should | Gate newer PHP syntax on the supported runtime floor | 100% | 100% |
| Should | Throw exception objects | 100% | 100% |
| Should | Use declared function-name casing | 100% | 100% |
| Should | Keep enum declarations nongeneric | 100% | 100% |
| Should | Keep PHPStan ignore annotations valid | 100% | 100% |
| Should | Resolve statically included files | 100% | 100% |
| Should | Avoid unstable named arguments across inheritance | 100% | 100% |
| Should | Preserve inherited method contracts | 100% | 100% |
| Should | Import real functions and constants with correct names | 100% | 100% |
| Should | Use operators with compatible operand types | 100% | 100% |
| Should | Keep PHPDoc compatible with native declarations | 100% | 100% |
| Should | Use valid recognized PHPStan PHPDoc tags | 100% | 100% |
| Should | Document throwable exception types | 100% | 100% |
| Should | Do not overwrite inferred types with contradictory assertions | 100% | 100% |
| Should | Access declared accessible static properties | 100% | 100% |
| Should | Assign values matching writable property types | 100% | 100% |
| Should | Represent stored callbacks with Closure or callable PHPDoc | 100% | 100% |
| Should | Preserve inherited property contracts | 100% | 100% |
| Should | Remove unneeded return union members | 100% | 100% |
| Should | Keep closed property types precise | 100% | 100% |
| Should | Define local variables on every read path | 100% | 100% |
| Should | Unset only valid mutable targets | 100% | 100% |
| Should | Clone only cloneable objects | 100% | 100% |
| Should | Narrow unions before member-specific operations | 100% | 100% |
| Should | Handle nullable receivers before dereferencing | 100% | 100% |
| Should | Narrow mixed values before typed operations | 100% | 100% |
| Should | Remove impossible strict membership candidates | 100% | 100% |
| Should | Remove writes whose values are never read | 100% | 100% |
| Should | Remove impossible throws annotations | 100% | 100% |
| Should | Initialize typed properties before use | 100% | 100% |
| Should | Give AI skills discoverable frontmatter | 100% | 100% |
| Should | Match Laravel contribution formatting | 100% | 100% |
| Should | Keep commands in discoverable directories | 100% | 100% |
| Should | Name exceptions for the failure they represent | 100% | 100% |
| Should | Implement documented additions to Laravel contracts | 100% | 100% |
| Should | Alias imports that share a short name | 100% | 100% |
| Should | Remove identity-only with helper calls | 100% | 100% |
| Should | Remove provably redundant sorts | 100% | 100% |
| Should | Surface unexpected exceptions in successful-path HTTP tests | 100% | 100% |
| Should | Make factory defaults a common valid scenario | 100% | 100% |
| Should | Use the model returned by creation | 100% | 100% |
| Should | Use focused request builders for complex validation scenarios | 100% | 100% |
| Should | Load the framework host schema when package integration tests depend on it | 100% | 100% |
| Should | Keep exhaustive matrices at the lowest useful layer | 100% |  |
| Should | Group assertions by subject | 100% | 100% |
| Should | Calculate expected results independently | 100% | 100% |
| Should | Parameterize equivalent test cases | 100% | 100% |
| Should | Enforce architectural conventions with architecture tests | 100% | 100% |
| Should | Assert meaningful mock argument constraints | 100% | 100% |
| Should | Name tests by result and condition | 100% | 100% |
| Should | Use groups for test selection | 100% |  |
| Should | Choose database reset strategies intentionally | 100% | 100% |
| Should | Evaluate Laravel-dependent skip conditions after boot | 100% | 100% |
| Should | Extract repeated test conditions into helpers | 100% | 100% |
| Should | Scope test support code by reuse | 100% | 100% |
| Should | Return real relationships for queryable associations | 100% | 100% |
| Should | Respect color preferences without relying only on color | 100% | 100% |
| Should | Use deliberate Faker locales | 100% | 100% |
| Should | Use container-backed facades for terse service access | 100% | 100% |
| Should | Supply request input through HTTP test helpers | 100% | 100% |
| Should | Keep test-double behavior method-specific | 100% | 100% |
| Should | Open the Laravel server when testing application pages | 100% |  |
| Should | Install test runners as development dependencies | 100% | 100% |
| Should | Avoid validating a Form Request twice | 100% | 100% |
| Should | Run cross-field and application-state checks after base validation | 100% |  |
| Should | Use built-in rules before inventing custom validators | 100% |  |
| Should | Keep one-off custom checks as closure rules | 100% |  |
| Should | Keep custom rule objects in the conventional rules directory | 100% |  |
| Should | Choose consistent ownership for validation messages | 100% |  |
| Should | Normalize accepted input before applying validation rules | 100% | 100% |
| Should | Give accepted fields substantive validation rules | 100% | 100% |
| Should | Use an explicit consistent request-access convention | 100% | 100% |
| Should | Express file limits with the correct size units | 100% |  |
| Should | Order field rules to fail cheaply and deliberately | 100% | 100% |
| Should | Allow null explicitly on nullable typed fields | 100% |  |
| Should | Validate optional update fields only when present | 100% |  |
| Should | Use valid preg_match patterns in regex rules | 100% |  |
| Should | Keep non-configuration PHP files outside config | 100% | 100% |
| Should | Import the Validator facade explicitly | 100% | 100% |
| Should | Install the Laravel 13 email validation security fix | 100% |  |
| Should | Make validation messages actionable | 100% |  |
| Should | Keep HTTP boundary classes in conventional directories | 100% |  |
| Should | Inject rule-building dependencies through rules method parameters | 100% | 100% |
| Should | Create a manual validator only when the flow needs custom control | 100% |  |
| Should | Extract reusable after-validation checks into composable classes | 100% |  |
| Should | Reject conditionally forbidden nonempty fields | 100% |  |
| Should | Keep console prompt text usable in narrow terminals | 100% | 100% |
| Should | Keep unvalidated array keys excluded | 100% | 100% |
| Should | Return after a custom rule failure when later logic must stop | 100% |  |
| Should | Keep client validation semantics aligned with server rules | 100% |  |
| Should | Indent each nested array element by one indentation level | 98% | 100% |
| Should | Separate groups of constant, class, and function imports with blank lines | 98% | 100% |
| Should | Enclose every control-structure body in braces | 98% | 100% |
| Should | Normalize function declaration spacing, including one space before a closure use clause | 98% | 100% |
| Should | Remove trailing whitespace from comment and PHPDoc lines | 98% | 100% |
| Should | Remove whitespace from blank lines | 98% | 100% |
| Should | Use consistent spaces in single-line @var PHPDoc comments | 98% | 100% |
| Should | End PHP files without closing tags with exactly one newline | 98% | 100% |
| Should | Indent statements according to their nesting level | 98% | 100% |
| Should | Add a trailing comma to multiline arrays | 98% | 100% |
| Should | Remove unused imports | 98% | 100% |
| Should | Use braces for every control-structure body | 98% | 100% |
| Should | Remove unused variables from closure use lists | 98% | 100% |
| Should | Capture only needed valid closure variables | 98% | 100% |
| Should | Test application behavior rather than declarations | 98% | 100% |
| Should | Extract repeated setup into scoped hooks | 98% | 93% |
| Should | Keep test namespaces in development autoloading | 98% | 100% |
| Should | Extract large coherent code blocks into small named functions | 98% | 100% |
| Should | Bound framework dependency constraints to supported majors | 98% | 100% |
| Should | Use canonical PHP function names | 98% | 100% |
| Should | Declare application object properties | 97% | 90% |
| Should | Use kebab-case for Artisan command words | 97% | 100% |
| Should | Keep PHPUnit classes focused on one subject | 97% | 100% |
| Should | Use the result of non-mutating collection operations | 97% | 100% |
| Should | Read only declared console inputs | 97% | 100% |
| Should | Omit redundant property names from type tags | 97% | 100% |
| Should | Keep variables on the left of comparisons | 97% | 100% |
| Should | Share Faker generators across fixture creation | 97% | 100% |
| Should | Use a predictable package filesystem layout | 97% | 100% |
| Should | Name repeated or constrained domain values | 96% | 100% |
| Should | Exclude generated and runtime directories from style fixing | 96% | 100% |
| Should | Resolve package publication destinations through application path helpers | 96% | 100% |
| Should | Preserve inline type annotations when formatting comments | 96% | 100% |
| Should | Remove comments that only restate the code | 96% | 100% |
| Should | Declare publishable package configuration in provider boot() | 96% | 100% |
| Should | Configure package test environments through Testbench lifecycle hooks | 96% | 89% |
| Should | Follow the existing Faker convention | 96% | 100% |
| Should | Read a cached value once instead of checking then fetching | 96% | 100% |
| Should | Use the concise equality where form | 96% | 100% |
| Should | Rely on Eloquent’s default timestamp names | 95% | 88% |
| Should | Import Blade View and Vite facades explicitly | 95% | 86% |
| Should | Go through Laravel's Cache API (e.g. Cache::remember) rather than raw Redis commands for cached data | 95% | 100% |
| Should | Remove empty content lines at the start and end of PHPDoc blocks | 95% | 100% |
| Should | Prefer Laravel abstractions for infrastructure operations | 95% | 100% |
| Should | Name application methods in camelCase | 95% | 100% |
| Should | Use lowercase canonical PHP casts with one space before the operand | 95% | 100% |
| Should | Follow established test conventions | 95% | 93% |
| Should | Choose one primary test framework convention | 95% | 93% |
| Should | Process large inputs lazily from the source | 95% | 100% |
| Should | Define model casts in casts(), preserving an established project convention | 95% | 100% |
| Should | Separate include and require from their paths with one space and remove unnecessary path parentheses | 95% | 100% |
| Should | Assert the exact access-control outcome in tests | 95% | 86% |
| Should | Prefer a shared Pint preset to ad hoc tuning | 95% | 83% |
| Should | Use generic templates to preserve linked argument and return types in reusable APIs | 95% | 100% |
| Should | Use declarative tests for facts | 95% | 100% |
| Should | Use middleware for shared HTTP concerns | 95% | 89% |
| Should | Import class and facade symbols explicitly instead of relying on global aliases or repeated fully qualified references | 95% | 100% |
| Should | Name conventional Eloquent tables with plural snake_case model names | 95% | 100% |
| Should | Use elseif instead of else if | 95% | 100% |
| Should | Name event timestamps consistently | 95% | 100% |
| Should | Use snake_case for application configuration and translation keys | 95% | 100% |
| Should | Name traits by the capability they provide | 95% | 100% |
| Should | Keep console commands thin and delegate application work to services | 95% | 100% |
| Should | Name traits for their capability | 95% | 100% |
| Should | Preserve parent test lifecycle hooks | 95% | 92% |
| Should | Make automated Artisan calls noninteractive | 94% | 100% |
| Should | Prefer feature coverage for framework behavior | 94% | 100% |
| Should | Do not define overlapping routes for the same URI and verb, because the winning route differs with and without route caching | 94% | 100% |
| Should | Namespace package environment variables | 94% | 100% |
| Should | Name Pest behaviors as verb phrases | 94% | 100% |
| Should | Keep relational references out of JSON | 94% | 100% |
| Should | Name conventional foreign keys with a singular snake_case model name and _id suffix | 94% | 100% |
| Should | Name to-one relationship methods in singular camelCase | 94% | 100% |
| Should | Use match expressions for clear value mappings | 94% | 100% |
| Should | Persist factory records only when needed | 94% | 100% |
| Should | Use post-increment and post-decrement when the expression result is unused | 94% | 100% |
| Should | Version Laravel packages with Git tags instead of a hard-coded version field in composer.json | 94% | 100% |
| Should | Sort globally before database pagination | 93% | 100% |
| Should | Order paginated results in the database | 93% | 100% |
| Should | Bootstrap package integration tests with Testbench | 93% | 100% |
| Should | Create only required host tables in isolated package tests | 93% | 100% |
| Should | Commit .editorconfig with the project’s indentation and line-ending conventions | 93% | 100% |
| Should | Declare the actual supported PHP version range in composer.json | 93% | 100% |
| Should | Use single spaces in class declarations and keep single-item inheritance lists on one line | 93% | 100% |
| Should | Use single quotes for strings that need no interpolation or escaping available only in double quotes | 93% | 100% |
| Should | Name PHP variables in camelCase | 93% | 100% |
| Should | Structure tests as arrange, act, assert | 93% | 100% |
| Should | Let Eloquent resolve polymorphic query types | 93% |  |
| Should | Expose a configured application health endpoint for deployment and uptime probes | 93% | 100% |
| Should | Name Blade view files in kebab case | 93% | 100% |
| Should | Format comments consistently | 93% | 100% |
| Should | Group business domains inside standard Laravel folders | 92% | 100% |
| Should | Stream uploaded files into storage | 92% |  |
| Should | Keep application files in application storage | 92% |  |
| Should | Give optional environment-backed configuration settings explicit safe defaults | 92% | 100% |
| Should | Suffix application service-provider class names with ServiceProvider | 92% | 100% |
| Should | Normalize persisted instants to UTC | 92% | 100% |
| Should | Make package installation commands repeat-safe and explicit about optional setup | 92% | 100% |
| Should | Keep HTML markup in views | 92% | 100% |
| Should | Mark application npm manifests private | 92% | 100% |
| Should | Remove the obsolete fideloper/proxy package and use Laravel's built-in Illuminate\Http\Middleware\TrustProxies for trusted proxies | 92% | 100% |
| Should | Handle decryption failures as invalid data | 92% | 100% |
| Should | Escape regex data with the actual delimiter | 92% | 100% |
| Should | Use canonical collection method names | 92% | 86% |
| Should | Keep the package test application provider set intentional | 92% | 100% |
| Should | Verify and close Mockery during teardown | 92% | 83% |
| Should | Prepare database data before rendering Blade | 92% | 100% |
| Should | Keep historical migrations independent of mutable application definitions | 92% | 100% |
| Should | Compose interchangeable action variants through contracts | 92% | 100% |
| Should | Model meaningful domain values with dedicated types | 92% | 100% |
| Should | Enforce domain invariants at every state-change boundary | 92% | 100% |
| Should | Use owned atomic locks for exclusive critical sections | 92% | 80% |
| Should | Use consistent scoped cache key names | 92% | 100% |
| Should | Namespace Redis keys by application and environment | 92% | 100% |
| Should | Let applications own publishable package migrations | 92% | 100% |
| Should | Make package migration publication repeat-safe | 92% | 100% |
| Should | Keep lazy side effects lazy | 92% | 100% |
| Should | Namespace package route names | 92% | 88% |
| Should | Avoid nested named functions | 92% | 100% |
| Should | Accept real-world personal names | 92% | 100% |
| Should | Call shared application operations directly from HTTP handlers | 92% | 100% |
| Should | Register installed PHPStan extensions | 91% | 100% |
| Should | Use the selected brace placement | 91% | 100% |
| Should | Order member modifiers consistently | 91% | 92% |
| Should | Keep installed dependency trees out of Git | 91% | 73% |
| Should | Remove obsolete code | 91% | 100% |
| Should | Use lazy pipelines for large collection transformations | 91% |  |
| Should | Declare value-taking command options as value-taking | 91% | 100% |
| Should | Combine related aggregate queries | 91% |  |
| Should | Keep new library helper functions in their package namespace | 91% |  |
| Should | Remove accidental output around PHP files | 91% | 100% |
| Should | Register package providers for analysis bootstrap | 91% | 100% |
| Should | Treat assertion-free tests as risky by default | 91% | 87% |
| Should | Commit an explicit Larastan configuration | 90% | 100% |
| Should | Keep constant annotations precise and compatible | 90% | 100% |
| Should | Use a standard opaque identifier generator | 90% | 100% |
| Should | Use syncWithoutDetaching() for additive associations and sync() only for replacement | 90% |  |
| Should | Return an intentional response from every HTTP route action | 90% | 75% |
| Should | Test enum inputs wherever the public API promises them | 90% | 100% |
| Should | Test package configuration override precedence | 90% | 100% |
| Should | Create related records through the owning relationship when it defines their foreign key | 90% | 100% |
| Should | Use canonical language constructs such as exit instead of aliases such as die | 89% | 100% |
| Should | Build response metadata fluently | 89% | 100% |
| Should | Split scenarios requiring different assertions | 89% | 90% |
| Should | Derive document language and default branding from application settings | 89% | 75% |
| Should | Rate-limit abuse-sensitive actions with a deliberate cache-backed limit | 89% | 71% |
| Should | Tag package resources for selective publication | 89% | 90% |
| Should | Use Attribute-returning methods for new model accessors and mutators unless a legacy style is established | 89% |  |
| Should | Instantiate unsaved models with new when no factory behavior is needed | 89% |  |
| Should | Make include dependencies explicit | 89% |  |
| Should | Centralize recurring view data in targeted composers | 89% |  |
| Should | Reload Blade changes through the Vite plugin | 89% | 100% |
| Should | Declare the Laravel majors a package actually supports | 89% | 90% |
| Should | Validate upload presence and transport success | 89% |  |
| Should | Guard package global helper declarations | 89% |  |
| Should | Require only the Illuminate components a package uses | 89% | 90% |
| Should | Start route definitions with the HTTP verb | 89% | 100% |
| Should | Group OR predicates inside relationship constraints | 89% |  |
| Should | Separate rule tests from endpoint wiring tests | 89% |  |
| Should | Keep ordinary test cache and session state in memory | 89% | 93% |
| Should | Sort imports alphabetically within groups ordered as constants, classes, then functions | 89% | 100% |
| Should | Put one space between a single-line comment marker and its text | 89% | 100% |
| Should | Write empty class and function bodies as {} on the same line when permitted by the brace rules | 89% | 93% |
| Should | Enable `sort-packages` in composer.json config so required packages stay alphabetized | 89% | 100% |
| Should | Flatten conditionals with guard clauses | 89% | 93% |
| Should | Document published configuration options with their purpose, accepted values, and relevant units | 89% | 93% |
| Should | Write union and intersection type operators without surrounding spaces | 89% | 100% |
| Should | Separate coherent statement groups with blank lines | 88% | 86% |
| Should | Keep queued payloads within the worker memory budget | 88% | 100% |
| Should | Use readable PHPUnit test names | 88% | 100% |
| Should | Use expectations for values in Pest | 88% | 100% |
| Should | Declare fixed section redirects directly in routes | 88% | 100% |
| Should | Index expiration columns in database cache tables | 88% | 100% |
| Should | Load page-specific JavaScript only on the pages using it | 88% | 100% |
| Should | Provide the stylesheet rule required by Alpine x-cloak | 88% | 100% |
| Should | Bound Laravel dependencies to compatible major versions | 88% | 100% |
| Should | Read signed URL values from route/query parameters and separately validate all unsigned body input | 88% |  |
| Should | Suffix controller class names with Controller | 88% | 100% |
| Should | Make package queue routing configurable | 88% | 100% |
| Should | Scope broadcast authorization explicitly | 88% |  |
| Should | Limit private-state reflection to unavoidable integration seams | 88% | 86% |
| Should | Resolve reusable library dependencies without a committed lockfile | 87% | 100% |
| Should | Namespace package implementation under src | 87% | 73% |
| Should | Exclude development-only files from package distributions | 87% | 82% |
| Should | Use method injection for dependencies needed by only one container-invoked action | 87% | 89% |
| Should | Filter nulls explicitly in collections | 87% |  |
| Should | Persist trusted security state without broadening public fillable fields | 87% | 100% |
| Should | Use canonical PHPUnit lifecycle signatures | 86% | 92% |
| Should | Declare dependencies compatible with Laravel 13 | 86% | 100% |
| Should | Put a blank line before return and continue statements where a preceding statement exists | 86% | 100% |
| Should | Keep model guard conventions consistent | 86% | 100% |
| Should | Reference class names with class constants | 86% | 73% |
| Should | Support application-owned generator stubs | 86% |  |
| Should | Declare the Illuminate support dependency in provider packages | 86% | 100% |
| Should | Consume a managed provider’s connection URL directly | 86% | 100% |
| Should | Provide a deleted_at column for every model using SoftDeletes | 86% |  |
| Should | Wire users into Laravel email verification | 86% | 100% |
| Should | Keep business logic out of Blade | 86% | 100% |
| Should | Keep secrets out of Livewire public state | 86% | 100% |
| Should | Keep large read-only datasets out of Livewire public state | 86% | 100% |
| Should | Use intention-revealing prefix and suffix checks | 86% | 89% |
| Should | Authenticate private-channel subscriptions | 86% |  |
| Should | Split oversized jobs into bounded work units | 86% |  |
| Should | Authorize administrators through managed permissions rather than hard-coded emails | 86% | 100% |
| Should | Disable arbitrary cache object unserialization when unnecessary | 86% | 100% |
| Should | Choose date or date_format validation | 86% |  |
| Should | Use the established action namespace | 86% | 100% |
| Should | Group distinct behaviors without unnecessary nesting | 86% |  |
| Should | Use strict closures for null or typed view data | 86% |  |
| Should | Keep cohesive multi-request scenarios together | 86% | 92% |
| Should | Prefer PHPUnit and Mockery for class-based unit suites | 86% | 60% |
| Should | Separate test suites by execution purpose | 86% | 100% |
| Should | Extract complex or reusable validation into Form Requests | 86% | 100% |
| Should | Show input requirements before validation fails | 86% |  |
| Should | Indent Blade with four spaces | 85% | 100% |
| Should | Use components for reusable interfaces with explicit inputs | 85% | 100% |
| Should | Commit a Pint configuration based on the Laravel preset with explicit project overrides | 85% | 80% |
| Should | Make broad record mutations an explicit opt-in | 85% | 33% |
| Should | Represent valid empty result sets consistently | 85% | 80% |
| Should | Choose heading levels by document structure | 85% | 67% |
| Should | Prefer PascalCase enum cases | 85% |  |
| Should | Enable reportUnmatchedIgnoredErrors in PHPStan | 84% | 100% |
| Should | Give meaningful icons action-oriented names | 84% | 100% |
| Should | Use singular descriptive enum class names | 84% | 100% |
| Should | Analyze traits through consuming classes | 84% | 100% |
| Should | Put one space on each side of binary operators | 84% | 100% |
| Should | Lint PHP syntax in CI | 84% | 93% |
| Should | Use PHPDoc for contracts native types cannot express | 84% | 67% |
| Should | Use PHPDoc for types PHP cannot express | 84% | 67% |
| Should | Normalize case-insensitive usernames before persistence | 83% | 100% |
| Should | Scope identifier validation to the caller's allowed records | 83% |  |
| Should | Append expensive or endpoint-specific computed attributes only where needed | 83% |  |
| Should | Encapsulate each email type in a mailable | 83% |  |
| Should | Guard optional package integrations | 83% |  |
| Should | Include Vue component filename extensions | 83% | 100% |
| Should | Use DeferrableProvider for deferred service providers | 83% |  |
| Should | Apply the intended authentication guard to protected logout routes | 83% |  |
| Should | List optional PHP extensions and packages under composer.json suggest, each with a short reason for when it is needed | 83% |  |
| Should | Enforce account eligibility during login | 83% |  |
| Should | Use kebab-case for human-readable public paths | 83% | 100% |
| Should | Handle partial batch effects explicitly | 83% |  |
| Should | Capture explicit actor context before queueing | 83% |  |
| Should | Handle synchronous dispatch failures in the caller | 83% |  |
| Should | Sign links whose parameters must resist tampering | 83% |  |
| Should | Offer TOTP or security keys rather than relying on SMS alone for high-assurance MFA | 83% | 100% |
| Should | Express argument-dependent return types accurately | 83% |  |
| Should | Reserve explicit storage filenames for application requirements | 83% |  |
| Should | Use format expectations for supported formats | 83% |  |
| Should | Use readable field and value labels in validation messages | 83% |  |
| Should | Use Laravel-13-compatible dependency constraints | 83% | 100% |
| Should | Use ?Type syntax for nullable single-type declarations | 83% | 100% |
| Should | Call never-returning helpers as statements | 82% | 100% |
| Should | Depend on the cache repository contract for store operations | 82% | 100% |
| Should | Extract repeated query predicates into named local scopes | 82% | 100% |
| Should | Use the established scope or custom-builder pattern for reusable query behavior | 82% | 100% |
| Should | Use protected #[Scope] methods for new local scopes unless the project retains legacy scope naming | 82% | 100% |
| Should | Name listeners after the action they perform | 82% |  |
| Should | Test the serialized field set at sensitive client boundaries | 82% |  |
| Should | Remove repeated blank lines and extra blank lines associated with throw statements and imports | 82% | 93% |
| Should | Offer multi-factor authentication for user accounts | 82% | 100% |
| Should | Reuse the route-bound model in request authorization | 82% |  |
| Should | Authenticate HTTP tests with the intended guard | 82% | 100% |
| Should | Enable class-based model factory discovery | 82% | 75% |
| Should | Namespace package Artisan command names | 81% | 100% |
| Should | Invalidate memoized results when their inputs change | 81% | 100% |
| Should | Configure usable fallback fonts when custom font metrics or processing may be unavailable | 81% | 100% |
| Should | Use Laravel-native service-provider APIs before adding another provider abstraction | 81% | 100% |
| Should | Set a shared schedule_timezone when scheduled tasks use the same timezone | 80% |  |
| Should | Handle failed filesystem writes explicitly or enable throwing writes | 80% |  |
| Should | Generate descriptively named timestamped migrations | 80% | 100% |
| Should | Reference only relationships that are defined and statically checked | 80% |  |
| Should | Use morph-map-aware type names in polymorphic filters | 80% |  |
| Should | Name global scope classes with a Scope suffix | 80% |  |
| Should | Prefer direct wire access over duplicated entangled state | 80% |  |
| Should | Use supported flex grow and shrink names | 80% | 67% |
| Should | Organize related Livewire components by domain | 80% |  |
| Should | Register console-only package resources during console execution | 80% | 100% |
| Should | Check formatting in CI | 80% | 92% |
| Should | Emit state-change events only after an actual transition | 80% |  |
| Should | Flush lifecycle-scoped monitoring and feature state | 80% | 67% |
| Should | Avoid registering package routes twice after caching | 80% | 86% |
| Should | Chunk large search-index imports | 80% |  |
| Should | Use Laravel 13 model attributes when adopting the attribute convention | 80% | 100% |
| Should | Broadcast user-specific data only on authorized resource-scoped private channels | 80% |  |
| Should | Store intended public local uploads on the public disk | 80% |  |
| Should | Enable local development diagnostics | 80% | 100% |
| Should | Reserve browser tests for browser-dependent behavior | 80% |  |
| Should | Defer request-dependent construction until request context is ready | 80% |  |
| Should | Associate model factories explicitly for static analysis | 80% | 100% |
| Should | Left-align PHPDoc tag contents and use two spaces after @param | 80% | 100% |
| Should | Use semantic page landmarks and controls | 79% | 88% |
| Should | Preserve the project enum case style | 79% | 75% |
| Should | Document the enum placement and representation convention | 79% | 75% |
| Should | Use Markdown mailables for conventional transactional messages | 79% | 100% |
| Should | Name action-specific Form Requests with the action, model and Request suffix | 79% | 100% |
| Should | Use camelCase route parameter names (e.g. {userId}, {newsItem}) | 79% | 100% |
| Should | Remove dependent migration indexes before dropping their columns | 78% |  |
| Should | Honor the configured guard and user provider in reusable user lookups | 78% | 75% |
| Should | Use Laravel email-verification services | 78% | 100% |
| Should | Persist a light dark or system appearance choice | 78% | 100% |
| Should | Route local mail to a development catcher | 78% | 100% |
| Should | Limit deep nesting of API routes; prefer /error-occurrences/1 over /projects/1/errors/1/error-occurrences/1 | 78% | 100% |
| Should | Keep generated files out of source-controlled server paths | 78% |  |
| Should | Use synchronous dispatch explicitly for synchronous commands | 78% |  |
| Should | Require authentication before displaying password-change settings | 78% | 100% |
| Should | Keep Workbench fixtures development-only | 78% | 78% |
| Should | Use fail-fast model lookup for required resources | 78% |  |
| Should | Check dependency and workflow-action versions on a regular documented schedule | 77% | 100% |
| Should | Provide accurate types for dynamic Eloquent attributes used by application code | 77% | 88% |
| Should | Split large application bootstrapping into focused service providers | 77% | 100% |
| Should | Space the logical NOT operator consistently | 77% | 100% |
| Should | Use PHP nullsafe access for simple optional chains | 76% | 67% |
| Should | Declare command inputs in the Artisan signature | 76% | 93% |
| Should | Inject container-managed collaborators explicitly where ordinary dependency injection is available | 76% | 71% |
| Should | Depend on contracts in cross-framework packages | 76% | 89% |
| Should | Generate public asset URLs through Laravel | 75% |  |
| Should | Use Artisan console components under the styled-output convention | 75% | 100% |
| Should | Preserve supported dark-mode behavior | 75% | 83% |
| Should | Import Laravel facade classes explicitly instead of relying on global aliases | 75% | 93% |
| Should | Use a zero flex basis for equal-width flexible items | 75% | 100% |
| Should | Offer usable MFA choices | 75% | 100% |
| Should | Describe Artisan commands for help output | 75% | 64% |
| Should | Reuse a test password hash across factory users | 75% | 100% |
| Should | Use collections for readable dataset transformations | 75% | 100% |
| Should | Expose formatter and analysis commands through Composer | 74% | 67% |
| Should | Expose standard checks through Composer scripts | 74% | 67% |
| Should | Reference controller actions with tuple notation [Controller::class, 'method'] instead of 'Controller@method' strings | 73% | 50% |
| Should | Create fakes in the tests that need them | 73% | 100% |
| Should | Type migration lifecycle methods as void | 73% | 100% |
| Should | Order interface lists consistently | 73% | 80% |
| Should | Use framework contracts for replaceable service dependencies | 73% | 100% |
| Should | Require explicit confirmation before risky interactive actions | 73% | 50% |
| Should | Protect destructive production commands with confirmation | 73% | 50% |
| Should | Start composable conditional model queries with query() and follow the established chain style | 73% |  |
| Should | Provide concrete application-user type information for authenticated-user values | 73% | 100% |
| Should | Reserve inline comments for complex logic | 73% | 60% |
| Should | Use concise higher-order collection operations | 73% | 100% |
| Should | Use intention-revealing collection queries | 73% | 100% |
| Should | Check formatting without mutating code in validation jobs | 73% | 50% |
| Should | Document custom facade methods with accurate @method annotations | 72% | 100% |
| Should | Omit redundant asc arguments in conventional query ordering | 72% | 100% |
| Should | Enforce foreign keys in SQLite tests | 72% | 83% |
| Should | Group related HTTP controllers in feature subnamespaces | 72% | 60% |
| Should | Capture the results of nonmutating collection methods | 72% | 85% |
| Should | Use explicit table names instead of evolving application models in migrations | 71% |  |
| Should | Decouple independent side effects with events | 71% | 100% |
| Should | Configure a class-driven dark variant for manual Tailwind themes | 71% | 75% |
| Should | Apply the resolved theme before the first page paint | 71% | 75% |
| Should | Complete the selected Tailwind build integration | 71% | 83% |
| Should | Expose an opt-out for automatically registered package routes | 71% | 88% |
| Should | Configure the API URI prefix centrally in route loading | 71% |  |
| Should | Use Request::boolean() for checkbox-style input conversion | 71% |  |
| Should | Load package routes from dedicated files | 71% | 100% |
| Should | Test endpoint authentication | 71% | 80% |
| Should | Keep framework compatibility matrices internally valid | 71% | 82% |
| Should | Keep intentionally HTTP-invoked commands registered | 71% | 100% |
| Should | Call Faker formatters as methods | 71% | 100% |
| Should | Use a consistent authorization call style | 71% | 67% |
| Should | Associate nonconventional factories with their model | 71% | 67% |
| Should | Keep contextual spacing outside reusable components | 70% | 100% |
| Should | Prefer short-lived workflow GITHUB_TOKEN over long-lived package CI secrets | 70% | 100% |
| Should | Automate repeatable upgrades with Rector | 70% |  |
| Should | Use territory locale directory identifiers consistently | 70% |  |
| Should | Use subject-specific assertions | 70% | 87% |
| Should | Include any required cache-key separator directly in the configured prefix | 69% | 100% |
| Should | Prefer transaction callbacks for automatic cleanup | 69% | 100% |
| Should | Name model-specific seeders with the singular model name and Seeder suffix | 69% | 100% |
| Should | Lower hashing cost in isolated tests | 69% | 60% |
| Should | Generate unique factory values for unique columns | 69% | 83% |
| Should | Allowlist keys in structured request arrays | 69% | 100% |
| Should | Normalize text line endings in Git | 68% | 100% |
| Should | Pipeline independent Redis commands in batches | 67% | 100% |
| Should | Centralize configurable application values in configuration files | 67% |  |
| Should | Use chunkById() or lazyById() when writes change the rows matching an iteration query | 67% |  |
| Should | Project scalar lists directly in SQL instead of hydrating models first | 67% |  |
| Should | Invalidate verification when the email changes | 67% | 100% |
| Should | Forward and merge component attributes | 67% | 67% |
| Should | Use block PHP directives when template PHP is necessary | 67% | 80% |
| Should | Build and deliver production frontend assets | 67% |  |
| Should | Split growing application routes into cohesive area files | 67% | 100% |
| Should | Keep domain logic out of HTTP and console entry points | 67% |  |
| Should | Run Composer noninteractively in CI | 67% | 100% |
| Should | Reference Laravel 13 request-forgery middleware in code and tests | 67% | 50% |
| Should | Isolate databases and shared resources for each parallel test process | 67% |  |
| Should | Adopt level six for complete type declarations | 67% | 75% |
| Should | Enable strict scalar argument checking | 67% |  |
| Should | Declare platform-dependent test requirements | 67% | 67% |
| Should | Write string concatenations without spaces around the dot operator | 66% | 100% |
| Should | Check method arguments at PHPStan level five or higher | 66% | 43% |
| Should | Generate application links from named routes | 66% | 64% |
| Should | Format fluent chains consistently | 65% | 87% |
| Should | Create prerequisite records before broad event fakes | 65% | 50% |
| Should | Make package Boost skills guide documentation searches before implementation | 64% | 100% |
| Should | Use in-memory SQLite for compatible isolated tests | 64% | 64% |
| Should | Index session expiration lookups | 64% | 100% |
| Should | Require password confirmation using the shared password rule | 64% | 100% |
| Should | Use a reproducible Testbench workbench application | 64% | 75% |
| Should | Use strict value comparisons | 63% | 57% |
| Should | Preserve column modifiers when changing a column | 62% |  |
| Should | Configure explicit OAuth callbacks | 62% |  |
| Should | Keep small single-use validation inline | 62% |  |
| Should | Index useful PostgreSQL foreign-key lookup columns explicitly | 62% | 100% |
| Should | Keep MySQL strict SQL mode enabled | 62% | 100% |
| Should | Name resource controllers after the singular resource with a Controller suffix | 62% | 60% |
| Should | Commit application Composer and frontend package-manager lockfiles | 62% | 0% |
| Should | Stop on required subprocess failures | 62% | 60% |
| Should | Mirror tested classes in test paths | 62% | 77% |
| Should | Use fluent strings for sequential transformations | 61% | 25% |
| Should | Keep substantial scripts and styles in asset files | 61% | 62% |
| Should | Guard optional package integrations with both an enable flag and a dependency-availability check | 60% |  |
| Should | Use a structured URI builder for nontrivial URI transformations | 60% | 67% |
| Should | Enforce one Laravel formatting preset | 60% | 83% |
| Should | Keep conditional query pipelines fluent | 60% | 57% |
| Should | Create actual foreign-key constraints | 60% | 75% |
| Should | Bound uniqueness lock lifetime | 60% |  |
| Should | Analyze configuration and migrations with Larastan | 59% | 86% |
| Should | Exempt only the required third-party webhook paths from browser request-forgery checks and verify provider signatures | 57% |  |
| Should | Keep core workflows independent of optional services | 57% |  |
| Should | Memoize repeated cache reads within a job | 57% |  |
| Should | Suppress model events for seeders that should not trigger side effects | 57% | 100% |
| Should | Allow package job classes to be replaced | 57% |  |
| Should | Express fixture ownership with factory relationships | 57% | 73% |
| Should | Configure a minimum release age or cooldown for automated dependency updates | 56% | 100% |
| Should | Describe console arguments and options | 56% | 77% |
| Should | Use API Resources as the established serialization boundary for model-backed APIs | 56% |  |
| Should | Allowlist model response fields at each endpoint boundary | 56% |  |
| Should | Choose rate-limit keys with shared-IP and lockout risks in mind | 56% | 100% |
| Should | Resolve the selected Laravel version in each CI lane | 56% | 0% |
| Should | Automate repeatable deployments in CI | 56% |  |
| Should | Reference model classes in model-backed unique rules | 56% | 100% |
| Should | Run static analysis at a useful level before release | 55% | 25% |
| Should | Use request->user() when a request object is already available | 55% | 100% |
| Should | Automatically fix and commit formatting in CI | 55% |  |
| Should | Grant formatting commit jobs write access | 55% |  |
| Should | Write useful PHPDoc descriptions as sentences | 53% | 87% |
| Should | Use Laravel's HTTP client for requests to external APIs | 53% |  |
| Should | Bind untrusted SQL values and allowlist dynamic SQL structure | 53% |  |
| Should | Define relationships that match the schema and declare their concrete return types | 53% | 33% |
| Should | Match queued dispatch arguments to constructors | 53% | 0% |
| Should | Prefer bound class or contract identifiers over equivalent legacy string container keys | 53% | 60% |
| Should | Declare void on methods that return no value | 52% | 47% |
| Should | Define first-party PHPUnit source paths | 52% | 40% |
| Should | Use Laravel current-date helpers | 50% | 67% |
| Should | Use the selected local filesystem abstraction consistently | 50% |  |
| Should | Organize application use cases as actions | 50% | 0% |
| Should | Use a bcrypt cost of at least 12 unless benchmarked requirements justify another value | 50% | 100% |
| Should | Use Arr::get() for nested optional array reads while preserving the original missing-value semantics | 50% | 50% |
| Should | Allow host applications to select or disable a package’s log channel | 50% | 67% |
| Should | Use the base query builder when model hydration and model value conversion are unnecessary | 50% |  |
| Should | Type relationship query-constraint callbacks with the appropriate builder | 50% |  |
| Should | Store stable aliases in polymorphic type columns through an enforced morph map | 50% |  |
| Should | Use supported, explicitly typed Livewire public state | 50% | 100% |
| Should | Invalidate dependent caches on every authoritative change path | 50% |  |
| Should | Use dedicated Blade authentication directives | 50% |  |
| Should | Use component-scoped Livewire scripts | 50% |  |
| Should | Pass exception classes to conditional throw helpers | 50% |  |
| Should | Prefer back() for straightforward redirects to the previous page | 50% | 100% |
| Should | Put shared endpoint access conditions in route middleware, using custom middleware for complex reusable checks | 50% | 50% |
| Should | Enforce PHP static analysis automatically | 50% | 40% |
| Should | Avoid redundant value wrappers | 50% |  |
| Should | Express batch dependencies as explicit chains | 50% |  |
| Should | Match Rector sets to the supported PHP floor | 50% |  |
| Should | Use provided application service fakes | 50% |  |
| Should | Restore global clock and random state | 50% | 25% |
| Should | Match Rector upgrade rules to installed framework versions | 50% |  |
| Should | Use a Form Request for each meaningful input action | 50% | 67% |
| Should | Use fluent rule builders for complex parameterized constraints | 50% | 100% |
| Should | Declare public method parameter and return types, including static for fluent mutators | 48% | 47% |
| Should | Test job logic separately from dispatch | 47% | 67% |
| Should | Use fluent route configuration methods instead of legacy array options | 47% | 80% |
| Should | Do not expose internal exception messages publicly | 47% |  |
| Should | Write multi-rule validation as arrays | 47% | 80% |
| Should | Disable checkout credential persistence when jobs do not need it | 46% | 100% |
| Should | Index the database queue name | 46% | 100% |
| Should | Annotate concrete model factory types | 46% | 70% |
| Should | Expose command metadata for lazy loading | 46% | 90% |
| Should | Prefer the shared fake helper in modern factories | 46% | 78% |
| Should | Centralize reusable response conversion with Responsable | 45% |  |
| Should | Automate import ordering and unused-import cleanup | 45% | 40% |
| Should | Import one trait per statement | 45% | 21% |
| Should | Use PATCH for partial resource updates | 44% |  |
| Should | Display per-field validation errors in Blade | 44% |  |
| Should | Report business validation failures with ValidationException | 44% |  |
| Should | Declare test environment configuration explicitly | 44% | 40% |
| Should | Use typed configuration getters where consumers require a specific configuration type | 44% | 43% |
| Should | Prove endpoints invoke authorization | 43% |  |
| Should | Register Carbon’s static-analysis extension | 43% | 30% |
| Should | Use all() when recursive conversion is unnecessary | 42% | 67% |
| Should | Declare least-privilege CI permissions | 42% | 73% |
| Should | Use gap for spacing sibling layout items | 42% | 33% |
| Should | Use strict comparisons when types matter | 41% | 50% |
| Should | Declare native parameter and return contracts | 41% | 47% |
| Should | Use first-class callables for simple method references | 41% | 25% |
| Should | Use cursor pagination for large changing lists with a suitable stable unique ordering | 40% |  |
| Should | Use maintained Laravel Rector rules | 40% |  |
| Should | Use the adopted Laravel PHPDoc alignment | 40% | 36% |
| Should | Separate browser tests from fast suites | 40% |  |
| Should | Prevent stray HTTP requests | 40% |  |
| Should | Use validated interactive console prompts | 39% | 80% |
| Should | Standardize PHP formatting with Pint | 39% | 40% |
| Should | Extract substantial or reusable business logic from controllers into focused application classes | 38% | 33% |
| Should | Choose a consistent business-logic boundary | 38% | 33% |
| Should | Prefer named status assertions | 38% | 58% |
| Should | Use implicit route model binding when its lookup and missing-model behavior match the endpoint | 38% |  |
| Should | Use the framework password broker for reset flows | 38% |  |
| Should | Compose package capabilities from contracts and concerns | 38% | 67% |
| Should | Scope event and queue fakes deliberately | 38% | 60% |
| Should | Ship version-aligned Boost guidance with supported packages | 37% | 30% |
| Should | Assert faked side effects and payloads | 37% | 62% |
| Should | Run Laravel-aware static analysis | 36% | 20% |
| Should | Test lowest and current supported dependencies | 35% | 18% |
| Should | Type-hint dispatched-object assertion predicates | 35% | 43% |
| Should | Promote ordinary constructor-only properties | 35% | 36% |
| Should | Keep isolated unit tests free of application boot | 35% | 45% |
| Should | Commit the selected formatter configuration and enforce it consistently in CI | 33% | 67% |
| Should | Fetch a scalar without hydrating a whole record | 33% |  |
| Should | Use enum backing values as PHP array keys | 33% |  |
| Should | Give modals an accessible dialog contract | 33% | 60% |
| Should | Select an explicit allowlist of request fields for application operations | 33% | 67% |
| Should | Stop retrying permanent failures | 33% |  |
| Should | Keep rate limiter declarations consistent | 33% | 60% |
| Should | Validate bcrypt password limits in bytes | 33% |  |
| Should | Validate the current password before sensitive credential changes | 33% |  |
| Should | Skip asset manifest lookup in server tests | 33% | 0% |
| Should | Share repeated profile validation rules | 33% | 100% |
| Should | Automate release changelog synchronization | 32% | 0% |
| Should | Restore test-owned external state | 32% |  |
| Should | Share the page layout through a Blade component | 31% | 50% |
| Should | Inject persistent collaborators through constructors | 31% |  |
| Should | Prevent browser MIME sniffing | 31% | 0% |
| Should | Enforce a maintained Content Security Policy | 31% | 0% |
| Should | Invalidate session data when logging out | 31% | 100% |
| Should | Use Blade directives for conditional boolean attributes | 30% |  |
| Consider | Replace unrelated boolean switches with cohesive operations | 100% |  |
| Consider | Accept Throwable in custom exception handlers | 100% |  |
| Consider | Keep exception-handler ignore public | 100% |  |
| Consider | Separate domain validity from authorization | 100% |  |
| Consider | Restrict upload types to the field purpose | 100% |  |
| Consider | Use input capabilities for pointer-specific behavior | 100% |  |
| Consider | Version the shared local development stack configuration | 100% |  |
| Consider | Skip only incompatible Rector rules or paths while retaining the useful remainder of the selected set | 100% |  |
| Consider | Cache an expensive value only after defining all events that invalidate it | 100% |  |
| Consider | Keep constant values compatible with declared and static-analysis-configured constant types | 100% |  |
| Consider | Initialize readonly properties only through write paths allowed by the supported PHP version | 100% |  |
| Consider | Configure a reusable file cache for repeated Rector analysis | 100% |  |
| Consider | Use laravel-package-tools lifecycle hooks for custom registration and boot behavior | 100% |  |
| Consider | Align package-tools configuration filenames with the declared package short name or explicit hasConfigFile() name | 100% |  |
| Consider | Prefer database locks for database-only consistency | 100% |  |
| Consider | Provide soft deletion when records need restoration | 100% |  |
| Consider | Store large generated files in object storage | 100% |  |
| Consider | Preserve domain validation and side effects during imports | 100% |  |
| Consider | Build Blade component variants on a shared base | 100% |  |
| Consider | Put repeatable reference data in dedicated seeders | 100% |  |
| Consider | Keep automated seed paths non-interactive and environment-aware | 100% |  |
| Consider | Give staff an authorized record-management interface | 100% |  |
| Consider | Isolate Alpine-owned regions from Livewire DOM updates | 100% |  |
| Consider | Keep administrative permissions in centrally reviewable records | 100% |  |
| Consider | Scope private resource lists to the authorized owner | 100% |  |
| Consider | Extract repeated domain checks into clearly named model methods | 100% |  |
| Consider | Store a state-change timestamp when the time of the transition is meaningful | 100% |  |
| Consider | Centralize repeated select-option construction in one consistent builder | 100% |  |
| Consider | Use model-aware introspection when building attribute or relationship tooling | 100% |  |
| Consider | Implement the intended dependent-record behavior explicitly for soft deletion | 100% |  |
| Consider | Use observers for lifecycle invariants that must run across every model entry point | 100% |  |
| Consider | Log selected model identifiers and safe fields instead of entire models | 100% |  |
| Consider | Pass SQL strings directly to DB::select() | 100% |  |
| Consider | Normalize broadcast-channel identifiers to the expected key type before ownership comparisons | 100% |  |
| Consider | Declare only available relationships in model eager-load defaults | 100% |  |
| Consider | Upload the processed file explicitly when client-side cropping or resizing precedes persistence | 100% |  |
| Consider | Change a child component key when its identity must be recreated for changed input | 100% |  |
| Consider | Preserve association rows during soft deletion unless the domain explicitly requires their removal | 100% |  |
| Consider | Route operational alerts through a dedicated configured recipient abstraction | 100% |  |
| Consider | Name relationships after their domain meaning when generic model names are ambiguous | 100% |  |
| Consider | Return explicit calculation/result objects instead of attaching transient fields to models | 100% |  |
| Consider | Migrate custom mail integrations from SwiftMailer to Symfony Mailer | 100% |  |
| Consider | Correlate audit entries with a stable actor | 100% |  |
| Consider | Avoid wildcard email domains for administrator grants | 100% |  |
| Consider | Preserve parent event-provider registration | 100% |  |
| Consider | Group related Blade field components in discoverable folders | 100% |  |
| Consider | Extend Blade through public extension points | 100% |  |
| Consider | Import applied PHP attribute classes | 100% |  |
| Consider | Access DOM refs after Alpine initialization | 100% |  |
| Consider | Let third-party libraries own their DOM | 100% |  |
| Consider | Break long workflows into coherent form steps | 100% |  |
| Consider | Write generated repository changes inside the checked-out GitHub workspace | 100% |  |
| Consider | Build APIs with Laravel's built-in features instead of Dingo API or similar packages | 100% |  |
| Consider | Do not mix HTTP and HTTPS between page load and subsequent API calls | 100% |  |
| Consider | Write declare directive assignments without spaces around the equals sign | 100% |  |
| Consider | Write declare(...) without spaces around its parentheses | 100% |  |
| Consider | End switch case labels with a colon instead of a semicolon | 100% |  |
| Consider | Use one space between case and its value and no extra space before its colon | 100% |  |
| Consider | Use protected Livewire properties only for request-local values; explicitly persist runtime state that must survive requests | 100% |  |
| Consider | Use signed routes for sensitive links that require tamper protection | 100% |  |
| Consider | Apply insert-only defaults with firstOrCreate | 100% |  |
| Consider | Update existing records during synchronization | 100% |  |
| Consider | Remove unreferenced uploads through lifecycle logic | 100% |  |
| Consider | Distinguish unauthenticated and forbidden responses | 100% |  |
| Consider | Separate lookup keys from values in updateOrCreate | 100% |  |
| Consider | Enforce numeric invariants inside atomic updates | 100% | 100% |
| Consider | Expire authentication JWTs | 100% |  |
| Consider | Preserve entered form data across transient failures | 100% |  |
| Consider | Preserve meaningful generic types instead of mixed | 100% |  |
| Consider | Specify generic arguments on implemented contracts | 100% |  |
| Consider | Use scheduled CI for small delay-tolerant tools | 100% |  |
| Consider | Clone shared query builders before specialization | 100% |  |
| Consider | Import facades by their namespaced class | 100% |  |
| Consider | Avoid retaining large request objects in singleton closures | 100% |  |
| Consider | Pass component markup through slots rather than escaped text props | 100% |  |
| Consider | Prefer supported framework primitives | 100% |  |
| Consider | Limit CSRF exemptions to appropriate non-cookie endpoints | 100% |  |
| Consider | Keep privilege flags out of public mass-assignment allowlists | 100% | 100% |
| Consider | Treat signed URLs as bearer capabilities rather than user identity | 100% |  |
| Consider | Keep Sanctum fallback guards from referring back to Sanctum | 100% |  |
| Consider | Reject expired JWTs during authentication | 100% |  |
| Consider | Reference actual model relationships in Eloquent relation queries | 100% |  |
| Consider | Write custom policies and generated application code outside vendor | 100% |  |
| Consider | Dispatch domain events independently of notification preferences | 100% |  |
| Consider | Review every administrative route for its required middleware and policy | 100% |  |
| Consider | Relax PHPDoc certainty deliberately at defensive library boundaries | 100% |  |
| Consider | Avoid removed PHPStan AST navigation attributes | 100% |  |
| Consider | Use PHP-Parser node types supported by the analyzer | 100% |  |
| Consider | Use PHPStan reflection for analyzed symbols | 100% |  |
| Consider | Follow constructor-parenthesis requirements of the selected alternate preset | 100% |  |
| Consider | Use class-first import groups under the selected alternate preset | 100% |  |
| Consider | Resolve displayed choices by stable record identity | 100% |  |
| Consider | Split behavior-switching boolean APIs | 100% |  |
| Consider | Mark intentional switch fall-through | 100% |  |
| Consider | Use compound offset updates only on compatible values | 100% |  |
| Consider | Remove duplicate or impossible switch cases | 100% |  |
| Consider | Validate date constructor strings | 100% |  |
| Consider | Request returned output before assigning it | 100% |  |
| Consider | Declare complete generator contracts | 100% |  |
| Consider | Honor native readonly property lifecycles | 100% |  |
| Consider | Respect readable and writable property directions | 100% |  |
| Consider | Respect reference parameter input and output contracts | 100% |  |
| Consider | Remove unused legacy application directories | 100% |  |
| Consider | Start Spatie-style packages with a compatible skeleton | 100% |  |
| Consider | Keep PHPStan assertion contracts truthful | 100% |  |
| Consider | Use macros for framework extension when suitable | 100% |  |
| Consider | Do not treat MIME validation as proof of a safe filename or executable | 100% |  |
| Consider | Invalidate only cache entries affected by a mutation | 100% |  |
| Consider | Call request validation on an injected request instance | 100% |  |
| Consider | Replace obsolete validator failure methods | 100% |  |
| Consider | Replace obsolete message replacement methods | 100% |  |
| Consider | Reference the current upload-size middleware | 100% |  |
| Consider | Give mixins valid complete object types | 95% | 75% |
| Consider | Put domain collection behavior on a custom collection | 94% |  |
| Consider | Keep reusable support code independent of HTTP | 94% |  |
| Consider | Remove unreferenced goto labels | 94% |  |
| Consider | Persist uploads through Laravel storage | 92% |  |
| Consider | Transform paginated items without discarding metadata | 92% |  |
| Consider | Keep transactions limited to consistency-critical work | 92% | 100% |
| Consider | Explain business-rule validation failures | 92% |  |
| Consider | Name security helpers according to their actual guarantees | 92% |  |
| Consider | Use direct class bindings when container autowiring is sufficient | 91% | 67% |
| Consider | Use one consistent JSON error shape (message plus field-level errors) across all API failures | 90% |  |
| Consider | Break foreach reference aliases after iteration | 90% |  |
| Consider | Bind untrusted values in database and process calls | 90% |  |
| Consider | Prefer Livewire's versioned script route | 89% |  |
| Consider | Centralize cache key construction and invalidation | 88% | 75% |
| Consider | Review sensitive production observability capture | 88% |  |
| Consider | Use method spoofing for HTML forms that target update or delete routes | 88% |  |
| Consider | Exclude inappropriate automated refactorings narrowly | 88% |  |
| Consider | Use Blade method-spoofing directives | 88% |  |
| Consider | Verify application ownership rules beyond scanners | 88% |  |
| Consider | Define every statically referenced file-based translation key in the configured translation files | 86% |  |
| Consider | Use middlewareAliases in a retained legacy HTTP kernel | 86% |  |
| Consider | Key authenticated API limits by user with an IP fallback | 86% |  |
| Consider | Group OR conditions so tenant and authorization constraints remain effective | 86% |  |
| Consider | Reject null and empty passwords before hashing | 86% |  |
| Consider | Analyse custom PHPStan extensions | 86% |  |
| Consider | Build class generators on GeneratorCommand | 86% |  |
| Consider | Test package provider integration contracts | 85% | 73% |
| Consider | Resolve fixture datasets after per-test setup | 85% |  |
| Consider | Resolve private static members through self | 84% | 33% |
| Consider | Extract repeated form field markup into components that forward input attributes | 83% |  |
| Consider | Scope personalized cache keys to the user or relevant owner | 83% |  |
| Consider | Give mailables purpose names with a Mail suffix | 83% |  |
| Consider | Define action-specific permissions | 83% |  |
| Consider | Choose explicit units and precision for displayed durations | 83% |  |
| Consider | Pass view data together | 83% |  |
| Consider | Use uppercase enum cases under that convention | 83% |  |
| Consider | Declare packages your application directly uses and avoid unnecessary constraints on unused transitive dependencies | 81% | 71% |
| Consider | Derive record ownership from the authenticated user | 80% |  |
| Consider | Keep classmap class identities unambiguous | 80% |  |
| Consider | Reuse the browser page within a cohesive scenario | 80% |  |
| Consider | Test the migration schema consumers actually install | 80% |  |
| Consider | Name listeners by their action with a Listener suffix | 79% |  |
| Consider | Exclude external code and intentionally invalid fixtures by path | 78% | 71% |
| Consider | Separate style checking from automatic fixing | 78% | 75% |
| Consider | Replace removed synchronous dispatch APIs | 78% |  |
| Consider | Cover actual API paths in CORS configuration | 78% |  |
| Consider | Extract repeated Blade validation error markup | 78% |  |
| Consider | Prefer libraries to shell commands | 77% | 33% |
| Consider | Load Larastan's extension in PHPStan configuration | 76% | 43% |
| Consider | Load relationships on the whole collection before iterating | 75% |  |
| Consider | Import Laravel classes explicitly instead of relying on global aliases | 75% |  |
| Consider | Associate form instructions with their input | 75% |  |
| Consider | Use context-independent callable syntax | 75% |  |
| Consider | Use snake_case for short translation keys | 75% |  |
| Consider | Group growing code by domain | 75% |  |
| Consider | Keep internal-only enums pure | 75% |  |
| Consider | Restore deletion-related state when undoing deletion | 73% |  |
| Consider | Use native DTO contracts with static analysis for internal type checking | 73% | 40% |
| Consider | Use the spaceship operator for three-way comparisons | 73% |  |
| Consider | Key Composer CI caches by runtime and dependency definition | 73% |  |
| Consider | Never put logic in route files | 72% | 50% |
| Consider | Remove PHPDoc types that repeat native declarations | 70% | 53% |
| Consider | Pass simple final projections directly to get | 70% |  |
| Consider | Make bulk commands restartable | 70% |  |
| Consider | Paginate API collections that can grow beyond a bounded small result set | 70% |  |
| Consider | Represent trusted generated HTML explicitly | 69% | 100% |
| Consider | Remove the obsolete root server router | 69% | 100% |
| Consider | Remove meaningless production debug messages | 69% |  |
| Consider | Remove empty public zero-argument constructors | 68% | 60% |
| Consider | Filter and batch backfill reads | 67% |  |
| Consider | Provide a plain-text alternative for HTML mail | 67% |  |
| Consider | Read request input with $request->name or request('name') instead of the verbose $request->input()/Request::get() forms | 67% | 88% |
| Consider | Rely on centralized request input normalization | 67% |  |
| Consider | Compare authentication MACs in constant time | 67% | 75% |
| Consider | Split complex workflows into focused pipeline stages | 67% |  |
| Consider | Defer to default exception rendering deliberately | 67% |  |
| Consider | Authorize sensitive field changes | 67% |  |
| Consider | Validate storage paths before normalization and use | 67% |  |
| Consider | Mask sensitive interactive command input | 67% |  |
| Consider | Preserve form values after validation failure | 67% |  |
| Consider | Use kebab-case for multi-word resource names in API routes (e.g. /error-occurrences) | 65% | 86% |
| Consider | Express cache lifetimes with readable duration values | 64% |  |
| Consider | Verify the persisted password after password changes | 64% | 75% |
| Consider | Index polymorphic association keys and uniquely constrain pure pivot pairs | 64% |  |
| Consider | Follow the surrounding validation syntax | 64% |  |
| Consider | Return named Artisan exit statuses | 63% | 71% |
| Consider | Provide one local formatting command | 62% | 67% |
| Consider | Centralize exception policy | 62% |  |
| Consider | Omit empty constructor parentheses under Laravel style | 62% | 43% |
| Consider | Send JSON with its correct Content-Type | 62% |  |
| Consider | Import class names referenced in PHPDoc | 60% | 53% |
| Consider | Centralize explicit runtime package versions in a release-maintained constant | 60% | 75% |
| Consider | Seed required session prerequisites in focused multi-step HTTP tests | 60% | 60% |
| Consider | Keep chart time buckets aligned across gaps | 60% | 67% |
| Consider | Delay model broadcasts and synchronous model-event effects until their transaction commits | 60% |  |
| Consider | Represent closed domain value sets with backed PHP enums | 60% |  |
| Consider | Use lifecycle-managed Livewire polling for periodic server refreshes | 60% |  |
| Consider | Render configured font CSS in the layout head | 60% | 75% |
| Consider | Use Laravel lotteries for infrequent inline housekeeping | 60% |  |
| Consider | Type-check frontend code in CI | 60% |  |
| Consider | Render boolean HTML attributes by presence | 60% |  |
| Consider | Name Livewire event payload arguments | 60% |  |
| Consider | Reuse existing gate assertions | 60% |  |
| Consider | Remove empty ordinary comments | 60% |  |
| Consider | Test email-verification state after profile updates | 60% | 75% |
| Consider | Fix browser viewport dimensions | 60% |  |
| Consider | Read testable application time through Laravel | 59% | 67% |
| Consider | Assign dynamically resolved services to explicitly typed locals before use when tooling cannot infer their type | 59% | 40% |
| Consider | Use consistent Pest declarations | 59% | 67% |
| Consider | Use existence queries for boolean checks | 58% | 50% |
| Consider | Type method parameters explicitly | 58% | 50% |
| Consider | Include the Larastan extension when using PHPStan to analyze Laravel framework behavior | 58% | 21% |
| Consider | Expose retry timing after rate limiting | 57% |  |
| Consider | Scope environment changes to the test lifecycle | 57% | 33% |
| Consider | Pin database service image versions | 56% | 40% |
| Consider | Enforce the agreed source and test coding standard | 56% | 40% |
| Consider | Move multi-step database workflows into focused orchestration | 56% |  |
| Consider | Run large data backfills outside schema migrations | 56% |  |
| Consider | Disable real broadcasting in ordinary tests | 56% | 33% |
| Consider | Use semantic links and buttons | 55% | 56% |
| Consider | Support native activation keys on custom controls | 55% | 56% |
| Consider | Automate dependency update visibility | 55% | 80% |
| Consider | Fix the test application locale | 55% | 50% |
| Consider | Promote simple constructor-injected properties | 54% | 36% |
| Consider | Test domain-critical relationship queries | 53% | 100% |
| Consider | Omit unused exception bindings from catch clauses | 50% | 50% |
| Consider | Report the Throwable object rather than logging only its message | 50% | 60% |
| Consider | Cache compact projections when callers do not need full Eloquent objects | 50% |  |
| Consider | Emit failed-authentication events from custom credential checks | 50% |  |
| Consider | Use the recipient locale for mail and notifications | 50% |  |
| Consider | Provide useful fallbacks for expected failures | 50% |  |
| Consider | Validate and authorize mass-assigned operations | 50% | 75% |
| Consider | Scope environment-specific password requirements explicitly | 50% | 33% |
| Consider | Use the verified email request contract before marking an address verified | 50% |  |
| Consider | Prune retained-row sets with set-based SQL | 50% | 67% |
| Consider | Explain conditional test skips | 50% | 70% |
| Consider | Pass delimited rule arguments as structured values | 50% |  |
| Consider | Pair numeric bounds with a numeric type rule | 50% |  |
| Consider | Depend on the validator contract in consumers | 50% |  |
| Consider | Assert status and content as distinct facts | 48% | 50% |
| Consider | Return conventional command exit statuses | 47% | 64% |
| Consider | Respect reduced-motion preferences | 47% | 40% |
| Consider | Run independent code checks in parallel CI jobs | 46% | 7% |
| Consider | Audit important model changes with the actor and operation | 46% | 0% |
| Consider | Return three-way values from comparators | 46% |  |
| Consider | Set a deterministic testing timezone | 46% | 42% |
| Consider | Namespace package-owned database tables to avoid host collisions | 45% | 40% |
| Consider | Publish actionable API deprecation metadata | 45% | 57% |
| Consider | Use modern password, MFA and recovery protections together | 45% | 100% |
| Consider | Log security-relevant application events without secrets | 44% |  |
| Consider | Name API error tests precisely | 44% | 75% |
| Consider | Use named routes for internal navigation assertions | 44% | 60% |
| Consider | Keep outbound HTTP calls interceptable by tests | 44% |  |
| Consider | Memoize stable attribute-reflection results | 43% |  |
| Consider | Constrain custom PHP attribute targets | 43% | 67% |
| Consider | Clean up Livewire JavaScript registrations | 43% | 67% |
| Consider | Set PHPStan’s target PHP version to the minimum runtime the project supports | 42% | 29% |
| Consider | Give loading indicators an accessible status announcement | 42% |  |
| Consider | Name Form Requests after the operation and resource | 42% | 100% |
| Consider | Honor prohibition policies before destructive command work | 40% | 44% |
| Consider | Clean up published application providers before package removal | 40% | 67% |
| Consider | Make repeatable seeders idempotent | 40% | 0% |
| Consider | Persist one explicit state for mutually exclusive workflow states and reject unknown values | 40% |  |
| Consider | Encode recurring review constraints as static-analysis rules | 40% |  |
| Consider | Use CarbonPeriod for date ranges | 40% |  |
| Consider | Do not treat remote disk paths as local files | 40% |  |
| Consider | Wait for UI readiness in browser automation | 40% |  |
| Consider | Throttle repeated operational alerts by incident identity | 40% |  |
| Consider | Serialize queued model references using the framework model serialization traits | 40% | 20% |
| Consider | Publish default translations only for customization | 40% |  |
| Consider | Test rejection of invalid OAuth and OIDC flow bindings | 40% |  |
| Consider | Test public TypeScript definitions with valid and invalid usage | 40% |  |
| Consider | Enable developer tooling only in intended environments | 40% |  |
| Consider | Quote raw SQL identifiers with the active connection grammar | 38% | 67% |
| Consider | Use a stable dependency lane for Larastan | 38% |  |
| Consider | Clear password fields after failed submissions as well as success | 38% | 33% |
| Consider | Preserve element types with PHPDoc generics | 38% | 44% |
| Consider | Separate long backfills from schema deployment | 38% |  |
| Consider | Clean generated static output before rebuilding | 38% |  |
| Consider | Smoke-test simple GET routes with valid bindings | 38% | 43% |
| Consider | Use Facade::expects or explicit call counts when a mocked interaction must occur | 36% |  |
| Consider | Normalize environment-derived configuration to the consumer type | 36% | 33% |
| Consider | Use notification channels for notification-like integrations | 36% | 67% |
| Consider | Represent package API failures with dedicated exception classes | 36% | 40% |
| Consider | Verify spies after exercising the code | 33% |  |
| Consider | Validate and consume one-time codes atomically | 33% |  |
| Consider | Use roving tabindex for tab controls | 33% |  |
| Consider | Handle nullable attributes explicitly rather than expecting their casts to convert null | 33% |  |
| Consider | Declare explicit targets on custom PHP attributes | 33% |  |
| Consider | Give ordered API collections deterministic default sorting | 33% |  |
| Consider | Provide a valid guest login destination | 33% |  |
| Consider | Slice data before manual pagination | 33% |  |
| Consider | Name batches and queued closures for monitoring | 33% |  |
| Consider | Preserve active authenticator secrets on repeated enable requests | 33% |  |
| Consider | Install the Laravel integration plugin for Pest suites | 33% | 33% |
| Consider | Offer interactive selection for routine human CLI workflows | 32% | 44% |
| Consider | Use query parameters for simple resource filtering | 31% | 33% |
| Consider | Expose useful package diagnostics through Artisan about | 30% | 22% |
| Consider | Exclude generated and tooling-only directories from Vite file watching | 30% | 40% |
| Consider | Describe package-specific IDE integrations in ide.json | 30% |  |
| Consider | Add a honeypot to public forms prone to spam | 30% | 0% |
