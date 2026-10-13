# Laravel Infra & DevOps Rules

**Scope:** Deploying, running, scaling, and operating Laravel 13 applications.

**Levels:** **Must** = non-negotiable · **Should** = default; deviate only with a clear reason · **Consider** = apply when the situation fits.

**Sub-bullets:** *Why* = the reason behind the rule, quotable when flagging a violation · *Flag when* = an observable signal in a diff that suggests a violation (omitted when not observable in code).

*Written for code-reviewing agents: use these rules to evaluate diffs, not as an implementation checklist.*

*Each rule cites its Laracon talk(s) and up to two supporting references.*

## Deployment

- **Must** Build dependencies and assets from a versioned commit into an immutable release, separate from deploy, and reload processes to activate it; never edit production code in place. — [David McKay, Laracon EU 2018](https://www.youtube.com/watch?v=BvDXzyOP3NE&t=210s); [Justin Reasoner, Laracon India 2025](https://www.youtube.com/watch?v=DdMFP_K0gv4&t=150s) · see [laravel.com/docs/13.x/deployment](https://laravel.com/docs/13.x/deployment)
  - Why: Rebuilding alone leaves old code running until deployed processes are replaced or reloaded.
  - Flag when: Deployment rebuilds an image without updating or restarting running application instances.
- **Should** Automate ordered release steps (env setup, `storage:link`, migrations) using the release's own code and config. — [Alex Bilbie, Laracon EU 2018](https://www.youtube.com/watch?v=L5zAH3jIboQ); [Taylor Otwell, Laracon US 2019](https://www.youtube.com/watch?v=XsPeWjKAUt0) · see [laravel.com/docs/13.x/artisan](https://laravel.com/docs/13.x/artisan)
  - Why: Release tasks need the deployed code and configuration to operate against the intended application environment.
  - Flag when: Deployment runs migrations before environment setup or from a different release directory.
- **Should** Automate tested releases with health checks, atomic or blue-green/canary rollout, rollback paths, and post-release error and latency checks. — [Taylor Otwell, Laracon US 2019](https://www.youtube.com/watch?v=XsPeWjKAUt0); [Jack Ellis, Laracon Online 2020](https://www.youtube.com/watch?v=xWi64tVm_Qk) · see [docs.vapor.build/projects/deployments](https://docs.vapor.build/projects/deployments)
  - Why: Enables rapid recovery to a previous release when deployment fails.
  - Flag when: Deployment pipeline removes test gates or rollback steps.
- **Should** Keep config and secrets out of code, and reload consumers after changing them. — [Alex Bilbie, Laracon EU 2018](https://www.youtube.com/watch?v=L5zAH3jIboQ); [Taylor Otwell, Laracon US 2019](https://www.youtube.com/watch?v=XsPeWjKAUt0) · see [laravel.com/docs/13.x/database](https://laravel.com/docs/13.x/database)
  - Why: Allows service endpoints and credentials to change without source edits.
  - Flag when: Application code hard-codes deployment credentials or backing-service endpoints.
- **Must** Keep files, sessions, and cache in shared backing services so app nodes are disposable. — [Jeremy Lindblom, Laracon US 2014](https://speakerdeck.com/jeremeamia/aws-for-artisans); [Frank de Jonge, Laracon EU 2016](https://www.youtube.com/watch?v=0bq8619JijU&t=2070s) · see [laravel.com/docs/13.x/filesystem](https://laravel.com/docs/13.x/filesystem)
  - Why: Prevents node replacement or request routing from losing uploads and sessions.
  - Flag when: Multi-node production config stores sessions, uploads, or cache only on individual nodes.
- **Must** Define reproducible runtimes (PHP version, extensions, services) and keep data independent of rebuildable containers. — [Jonathan Barronville, Laracon US 2013](https://www.youtube.com/watch?v=S--SyXwwRfk&t=60s); [Francesco Ciulla, Laracon India 2026](https://www.youtube.com/watch?v=CMqhOhRnOY0&t=150s) · see [laravel.com/docs/13.x/sail](https://laravel.com/docs/13.x/sail)
  - Why: Prevents runtime mismatches and repeated manual environment setup.
  - Flag when: Container rebuild deletes the only copy of database data.
- **Should** Use multi-stage production images with only runtime artifacts; exclude compilers, dev deps, and Sail. — [David McKay, Laracon EU 2018](https://www.youtube.com/watch?v=BvDXzyOP3NE&t=1200s); [Francesco Ciulla, Laracon India 2026](https://www.youtube.com/watch?v=CMqhOhRnOY0&t=690s) · see [laravel.com/docs/13.x/deployment](https://laravel.com/docs/13.x/deployment)
  - Why: Reduces production image size by leaving build tools and test dependencies behind.
  - Flag when: Final production image includes compilers, development dependencies, or Sail tooling.
- **Consider** Use Sail or an equivalent container dev setup with wrappers for PHP, Composer, Artisan, and asset commands. — [David McKay, Laracon EU 2018](https://www.youtube.com/watch?v=BvDXzyOP3NE&t=930s); [Francesco Ciulla, Laracon India 2026](https://www.youtube.com/watch?v=CMqhOhRnOY0&t=810s) · see [laravel.com/docs/13.x/sail](https://laravel.com/docs/13.x/sail)
  - Why: Runs development commands with the application’s actual PHP runtime and dependencies.
- **Should** Gate startup on application-level readiness checks, not open ports or start order. — [Jeremy Lindblom, Laracon US 2014](https://speakerdeck.com/jeremeamia/aws-for-artisans); [David McKay, Laracon EU 2018](https://www.youtube.com/watch?v=BvDXzyOP3NE&t=1020s) · see [laravel.com/docs/13.x/artisan](https://laravel.com/docs/13.x/artisan)
  - Why: Prevents dependent operations from running before required services are ready.
  - Flag when: Startup runs migrations after only a port check or unconditional service start.
- **Should** Build assets with Vite (hashed filenames, consistent `ASSET_URL`) and give only versioned URLs long cache lifetimes. — [Taylor Otwell, Laracon US 2019](https://www.youtube.com/watch?v=XsPeWjKAUt0); [Ryuta Hamasaki, Laracon EU 2019](https://www.youtube.com/watch?v=nhpjNXHfa80) · see [docs.vapor.build/projects/deployments](https://docs.vapor.build/projects/deployments)
  - Why: Keeps changed assets fresh while allowing unchanged assets to remain cached.
  - Flag when: Long cache lifetimes applied to asset URLs without content versioning.
- **Consider** Choose a trusted minimal PHP base image after verifying versions, extensions, and libc compatibility. — [David McKay, Laracon EU 2018](https://www.youtube.com/watch?v=BvDXzyOP3NE&t=2310s); [Francesco Ciulla, Laracon India 2026](https://www.youtube.com/watch?v=CMqhOhRnOY0&t=630s) · see [laravel.com/docs/13.x/deployment](https://laravel.com/docs/13.x/deployment)
  - Why: Reduces image size while retaining required extensions and compatible runtime libraries.
- **Consider** Use a managed platform (Laravel Cloud, Forge, Vapor) when it cuts operating effort at acceptable cost. — [Jeremy Lindblom, Laracon US 2014](https://speakerdeck.com/jeremeamia/aws-for-artisans); [Geshan Manandhar, Laracon EU 2016](https://www.youtube.com/watch?v=SZ3UfwBACIo) · see [laravel.com/docs/13.x/deployment](https://laravel.com/docs/13.x/deployment)
  - Why: Reduces the setup and maintenance effort needed for deployment automation.
- **Consider** Run Kubernetes only when you can handle patching, ingress/TLS, replicas, and headroom; otherwise use a managed platform. — [Justin Reasoner, Laracon India 2025](https://www.youtube.com/watch?v=DdMFP_K0gv4&t=990s) · see [laravel.com/docs/13.x/deployment](https://laravel.com/docs/13.x/deployment)
  - Why: Avoids disproportionate maintenance of cluster security, routing, and availability.
- **Consider** Evaluate Octane (FrankenPHP, Swoole, RoadRunner) against your workload before adopting it. — [Kévin Dunglas, Laracon EU 2024](https://www.youtube.com/watch?v=PzOLyz1vr5I&t=180s); [Mateus Guimarães, Laracon US 2024](https://www.youtube.com/watch?v=dOXXL-98Zbs) · see [laravel.com/docs/13.x/deployment](https://laravel.com/docs/13.x/deployment)
  - Why: Reduces repeated framework boot overhead where persistent workers benefit the workload.
- **Should** Load-test under production-like conditions to verify capacity changes and each optimization. — [Leo Sjoberg, Laracon EU 2019](https://www.youtube.com/watch?v=UTXuslWrH1Q&t=105s); [Devon Garbalosa, Laracon US 2026](https://www.youtube.com/watch?v=toSMnEINXAo&t=826s) · see [laravel.com/docs/13.x/deployment](https://laravel.com/docs/13.x/deployment)
  - Why: Confirms capacity and scaling improvements under concurrent demand.
- **Should** Ship structured logs from every host and process (stdout/stderr) to a searchable destination. — [Kévin Dunglas, Laracon EU 2024](https://www.youtube.com/watch?v=PzOLyz1vr5I&t=660s); [Chris Fidao, Laracon US 2016](https://speakerdeck.com/fideloper/server-survival) · see [laravel.com/docs/13.x/octane](https://laravel.com/docs/13.x/octane)
  - Why: Makes access and operational events auditable across hosts.
  - Flag when: Production logging switches to uncollected local files or discards process output.
- **Must** Allow only key-based SSH for authorized users and firewall everything but required ports (IPv4 and IPv6). — [Chris Fidao, Laracon US 2016](https://speakerdeck.com/fideloper/server-survival); [Chris Fidao, Laracon Online 2022](https://www.youtube.com/watch?v=f4QShF42c6E&t=30221s) · see [laravel.com/docs/13.x/deployment](https://laravel.com/docs/13.x/deployment)
  - Why: Restricts server access to authorized users and required inbound traffic.
  - Flag when: SSH config enables password login or firewall config allows unnecessary inbound ports.

## CI/CD

- **Should** Integrate and deploy small increments frequently. — [Sam Sehnert, Laracon AU 2024](https://www.youtube.com/watch?v=IPbxscS0xiY&t=420s); [John Drexler, Laracon EU 2026](https://www.youtube.com/watch?v=YJmuKPk3d9M&t=11255s) · see [laravel.com/docs/13.x/deployment](https://laravel.com/docs/13.x/deployment)
  - Why: Reduces merge conflicts, deployment risk, and feedback delay.
- **Must** Run tests, style, static analysis, and builds in CI and require them to pass (plus an approved PR) before merge and release. — [Margaret Staples, Laracon EU 2018](https://www.youtube.com/watch?v=3IQrbUrb2E0); [David McKay, Laracon EU 2018](https://www.youtube.com/watch?v=BvDXzyOP3NE&t=1440s) · see [laravel.com/docs/13.x/testing](https://laravel.com/docs/13.x/testing)
  - Why: Prevents faulty changes from reaching production.
  - Flag when: CI deploys after failed checks or skips executing test targets.
- **Should** Enforce style and static analysis on new code automatically and baseline legacy violations to reduce over time. — [Ashley Hindle, Laracon Online 2022](https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=33720s); [Sam Sehnert, Laracon AU 2024](https://www.youtube.com/watch?v=IPbxscS0xiY&t=840s) · see [laravel.com/docs/13.x/pint](https://laravel.com/docs/13.x/pint)
  - Why: Introduces automated checks without requiring a complete legacy cleanup first.
  - Flag when: CI disables style or static analysis checks, or baselines new violations.
- **Should** Favor meaningful feature and integration tests over chasing a coverage number. — [Ashley Hindle, Laracon Online 2022](https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=33510s); [Sam Sehnert, Laracon AU 2024](https://www.youtube.com/watch?v=IPbxscS0xiY&t=990s) · see [laravel.com/docs/13.x/testing](https://laravel.com/docs/13.x/testing)
  - Why: Meaningful behavior tests provide useful feedback without pursuing an arbitrary coverage target.
- **Consider** Centralize deploy and ops actions in a visible shared tool or bot. — [Margaret Staples, Laracon EU 2018](https://www.youtube.com/watch?v=3IQrbUrb2E0); [James Mooring, Laracon AU 2025](https://www.youtube.com/watch?v=hpUrWWlBKCY&t=971s) · see [laravel.com/docs/13.x/deployment](https://laravel.com/docs/13.x/deployment)
  - Why: Makes deployments repeatable instead of depending on individual developers’ machines.

## Queues & workers

- **Must** Supervise long-running workers (Supervisor/systemd) with auto-restart, logs, and graceful shutdown long enough for in-flight jobs; restart them on deploy. — [Chris Fidao, Laracon US 2016](https://speakerdeck.com/fideloper/server-survival); [Mohamed Said, Laracon Online 2021](https://www.youtube.com/watch?v=of7zOVXtTec) · see [laravel.com/docs/13.x/queues](https://laravel.com/docs/13.x/queues)
  - Why: Keeps workers running after exits and loads updated application code on deployment.
  - Flag when: Worker config disables automatic restart, or deployment scripts omit worker restart.
- **Must** Keep job and worker timeouts below `retry_after` (or SQS visibility timeout) so stalled jobs end before retries overlap. — [Mohamed Said, Laracon Online 2021](https://www.youtube.com/watch?v=of7zOVXtTec); [Jack Ellis, Laracon Online 2020](https://www.youtube.com/watch?v=xWi64tVm_Qk) · see [laravel.com/docs/13.x/queues](https://laravel.com/docs/13.x/queues)
  - Why: Prevents a still-running job from being retried concurrently.
  - Flag when: Job or worker timeout equals or exceeds retry_after or SQS visibility timeout.
- **Must** Make jobs idempotent: deduplicate with stable IDs and DB uniqueness, and record remote run IDs before retrying. — [Seb Armand, Laracon US 2024](https://www.youtube.com/watch?v=9mDkf4popb8&t=1350s); [Joshua Alphonse, Laracon US 2026](https://www.youtube.com/watch?v=HhcieSa52SQ) · see [laravel.com/docs/13.x/queues](https://laravel.com/docs/13.x/queues)
  - Why: Prevents repeated events and retries from duplicating workflow effects or paid remote runs.
  - Flag when: Retried jobs submit remote work again without checking a persisted run ID.
- **Must** Persist workflow progress and failures and enqueue IDs, not big payloads, so workers can reload state and recover. — [Marten Westphal, Laracon EU 2018](https://www.youtube.com/watch?v=P0LfWwU7ToA&t=1230s); [Joshua Alphonse, Laracon US 2026](https://www.youtube.com/watch?v=HhcieSa52SQ) · see [laravel.com/docs/13.x/queues](https://laravel.com/docs/13.x/queues)
  - Why: Allows failed work to recover without rerunning entire workflows or leaving users indefinitely waiting.
  - Flag when: Workflow progress exists only in queue payloads, or exhausted retries never persist failure status.
- **Must** Batch independent jobs and chain dependent ones so steps run only after upstream success. — [Seb Armand, Laracon US 2024](https://www.youtube.com/watch?v=9mDkf4popb8&t=1410s); [Bert De Swaef, Laracon India 2026](https://www.youtube.com/watch?v=xxAv6s5045Y) · see [laravel.com/docs/13.x/queues](https://laravel.com/docs/13.x/queues)
  - Why: Prevents dependent steps from consuming incomplete upstream results.
  - Flag when: Dependent jobs are dispatched independently without checking prerequisite completion.
- **Should** Separate queues and worker pools by priority and resource needs; route heavy jobs to suitable machines. — [Chris Fidao, Laracon Online 2018](https://speakerdeck.com/fideloper/scaling-laravel-laracon-dot-net-2018); [Mohamed Said, Laracon Online 2021](https://www.youtube.com/watch?v=-l8o4KMaS4c&t=510s) · see [laravel.com/docs/13.x/queues](https://laravel.com/docs/13.x/queues)
  - Why: Matches worker resources to workload needs while giving urgent work priority.
  - Flag when: Heavy and urgent jobs share one queue and worker pool without priority routing.
- **Should** Queue slow side effects (mail, files, broadcasts, index updates) and expose progress or results durably. — [Chris Fidao, Laracon Online 2018](https://speakerdeck.com/fideloper/scaling-laravel-laracon-dot-net-2018); [Mohamed Said, Laracon Online 2021](https://www.youtube.com/watch?v=-l8o4KMaS4c&t=1470s) · see [laravel.com/docs/13.x/queues](https://laravel.com/docs/13.x/queues)
  - Why: Lets requests return without waiting for slow work that can finish in the background.
  - Flag when: Request handlers synchronously send mail or process files that the response does not require.
- **Should** Recycle workers with `--max-jobs`/`--max-time` and monitor their memory. — [Mohamed Said, Laracon Online 2021](https://www.youtube.com/watch?v=of7zOVXtTec); [Neel Shah, Laracon India 2026](https://www.youtube.com/watch?v=CdfW-bCKZ3E&t=960s) · see [laravel.com/docs/13.x/queues](https://laravel.com/docs/13.x/queues)
  - Why: Prevents growing worker memory from exhausting resources and interrupting processing.
  - Flag when: Worker configuration removes both max-jobs and max-time recycling limits.
- **Should** Process large exports and imports in batches or lazy streams sized by memory and runtime, not ID counts alone. — [Marten Westphal, Laracon EU 2018](https://www.youtube.com/watch?v=P0LfWwU7ToA&t=690s); [Jack Ellis, Laracon Online 2020](https://www.youtube.com/watch?v=xWi64tVm_Qk) · see [laravel.com/docs/13.x/queues](https://laravel.com/docs/13.x/queues)
  - Why: Bounds memory and runtime while avoiding uneven chunks caused by differing related-data volumes.
  - Flag when: Export or import loads the entire dataset with get() before processing.
- **Should** Limit concurrent DB-writing jobs to downstream capacity and back off retries during outages. — [Taylor Otwell, Laracon US 2019](https://www.youtube.com/watch?v=XsPeWjKAUt0); [Jack Ellis, Laracon Online 2020](https://www.youtube.com/watch?v=xWi64tVm_Qk) · see [docs.vapor.build/resources/queues](https://docs.vapor.build/resources/queues)
  - Why: Turns traffic spikes into manageable backlogs and preserves jobs through database outages.
  - Flag when: Database failures trigger immediate job retries without delay or backoff.
- **Consider** Use queued closures for trivial background work, remembering models are re-fetched at run time. — [Taylor Otwell, Laracon AU 2018](https://www.youtube.com/watch?v=0O6dLJX_k3w&t=600s) · see [laravel.com/docs/13.x/queues](https://laravel.com/docs/13.x/queues)
  - Why: Runs simple background tasks without a job class while reloading captured models at execution time.
- **Should** Profile representative requests, commands, and workers before changing concurrency or capacity. — [Marten Westphal, Laracon EU 2018](https://www.youtube.com/watch?v=P0LfWwU7ToA&t=1350s); [Mohamed Said, Laracon Online 2021](https://www.youtube.com/watch?v=-l8o4KMaS4c&t=870s) · see [laravel.com/docs/13.x/collections](https://laravel.com/docs/13.x/collections)
  - Why: Identifies costly operations at realistic scale before adding concurrency or infrastructure complexity.
- **Consider** Run independent I/O concurrently (e.g. `Concurrency`, Octane tasks); don't expect it to parallelize CPU work. — [Mohamed Said, Laracon Online 2021](https://www.youtube.com/watch?v=-l8o4KMaS4c&t=1620s); [Mateus Guimarães, Laracon US 2024](https://www.youtube.com/watch?v=dOXXL-98Zbs) · see [laravel.com/docs/13.x/octane](https://laravel.com/docs/13.x/octane)
  - Why: Overlapping independent waits reduces the time needed to collect results for a response.

## Scaling, Octane & performance

- **Should** Ship features behind disabled flags (Pennant), enable for cohorts or percentages, and expand while watching outcomes. — [Aaron Francis, Laracon Online 2022](https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=12420s); [Sam Sehnert, Laracon AU 2024](https://www.youtube.com/watch?v=IPbxscS0xiY&t=1260s) · see [laravel.com/docs/13.x/deployment](https://laravel.com/docs/13.x/deployment)
  - Why: Gradual exposure reveals regressions early and allows quickly disabling the affected feature.
  - Flag when: A newly introduced feature flag defaults to enabled for all production users.
- **Must** Never keep request or user state in statics, globals, or singletons under Octane; resolve per-request dependencies at runtime. — [Mateus Guimarães, Laracon US 2024](https://www.youtube.com/watch?v=dOXXL-98Zbs); [Khushboo Verma, Laracon India 2025](https://www.youtube.com/watch?v=Fj21ySHCz3I) · see [laravel.com/docs/13.x/octane](https://laravel.com/docs/13.x/octane)
  - Why: Persistent workers can reuse stale user data across requests.
  - Flag when: An Octane singleton or static property retains request objects or user-specific values.
- **Consider** Fix slow queries and request work and verify OPcache before benchmarking Octane's benefit. — [Mateus Guimarães, Laracon US 2024](https://www.youtube.com/watch?v=dOXXL-98Zbs); [Seb Armand, Laracon US 2024](https://www.youtube.com/watch?v=9mDkf4popb8&t=1170s) · see [laravel.com/docs/13.x/octane](https://laravel.com/docs/13.x/octane)
  - Why: Octane reduces repeated framework initialization but leaves expensive application work intact.
- **Should** Enable OPcache in production and reset it (and reload Octane/queues) on each deploy. — [Chris Fidao, Laracon Online 2018](https://speakerdeck.com/fideloper/scaling-laravel-laracon-dot-net-2018); [Marcel Pociot, Laracon EU 2025](https://www.youtube.com/watch?v=rk29Hyk59JY) · see [laravel.com/docs/13.x/deployment](https://laravel.com/docs/13.x/deployment)
  - Why: Reusing compiled bytecode avoids recompilation, while runtime refreshes prevent serving stale code.
  - Flag when: Production PHP configuration disables OPcache or deployment retains cached code with timestamp validation disabled.
- **Should** Add load balancing across nodes only when one node can't meet latency and throughput targets. — [Frank de Jonge, Laracon EU 2016](https://www.youtube.com/watch?v=0bq8619JijU&t=1710s); [Chris Fidao, Laracon Online 2018](https://speakerdeck.com/fideloper/scaling-laravel-laracon-dot-net-2018) · see [laravel.com/docs/13.x/deployment](https://laravel.com/docs/13.x/deployment)
  - Why: Distributing requests maintains throughput and response times when a single server reaches capacity.
- **Consider** Use on-demand or serverless capacity for suitable workloads, accounting for quotas, cold starts, and cost. — [Aaron Francis, Laracon Online 2022](https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=13170s); [Guus Leeuw, Laracon India 2024](https://www.youtube.com/watch?v=rpvhD-twpY4) · see [laravel.com/docs/13.x/concurrency](https://laravel.com/docs/13.x/concurrency)
  - Why: Capacity can grow with demand without permanently paying for peak resources.
- **Should** Stream large files instead of loading them into memory. — [Frank de Jonge, Laracon EU 2016](https://www.youtube.com/watch?v=0bq8619JijU&t=660s) · see [laravel.com/docs/13.x/filesystem](https://laravel.com/docs/13.x/filesystem)
  - Why: Full-file buffering multiplies memory demand under concurrent requests, increasing failures and latency.
  - Flag when: Large file transfers use `file_get_contents()` or `Storage::get()` to buffer the entire file.
- **Should** Use SQL counts and aggregates instead of hydrating models; use simple pagination when a total isn't needed. — [Blackfire, Laracon EU 2021](https://www.youtube.com/watch?v=la0ZtnqX6zQ&t=1217s); [Marcel Pociot, Laracon EU 2025](https://www.youtube.com/watch?v=rk29Hyk59JY) · see [laravel.com/docs/13.x/eloquent-relations](https://laravel.com/docs/13.x/eloquent-relationships)
  - Why: Database aggregates avoid model hydration, date conversion, and memory use for data needed only as counts.
  - Flag when: Related models are loaded and counted in PHP when only their count is used.
- **Should** Eager-load each page's relationships and detect lazy loading outside production. — [Chris Fidao, Laracon Online 2018](https://speakerdeck.com/fideloper/scaling-laravel-laracon-dot-net-2018); [Seb Armand, Laracon US 2024](https://www.youtube.com/watch?v=9mDkf4popb8&t=420s) · see [laravel.com/docs/13.x/eloquent-relations](https://laravel.com/docs/13.x/eloquent-relationships)
  - Why: Lazy relationship access adds a database query per model in a collection.
  - Flag when: A loop accesses unloaded relationships without collection-wide eager loading.
- **Should** Precompute expensive derived values and aggregates and refresh them on change, rather than recomputing on every read. — [Seb Armand, Laracon US 2024](https://www.youtube.com/watch?v=9mDkf4popb8&t=990s); [Devansh Bawari, Laracon India 2026](https://www.youtube.com/watch?v=eJJsGhu9gtI) · see [laravel.com/docs/13.x/scout](https://laravel.com/docs/13.x/scout)
  - Why: Precomputed read models avoid repeatedly executing expensive relational queries and storefront calculations.
  - Flag when: A read endpoint rebuilds expensive aggregates or derived catalog values on every request.
- **Should** Pick polling, WebSockets, or streaming based on freshness needs and cost; bound any polling. — [Joe Dixon, Laracon US 2024](https://www.youtube.com/watch?v=60g-Z-dAWNk&t=360s); [Joe Dixon, Laracon EU 2024](https://www.youtube.com/watch?v=yrL5eCMpqtc) · see [laravel.com/docs/13.x/broadcasting](https://laravel.com/docs/13.x/broadcasting)
  - Why: Frequent polling creates repeated HTTP work and delays visibility of server updates.
  - Flag when: Polling continues after the relevant operation completes or the component is removed.
- **Should** For realtime, run Reverb with queue workers for broadcasts and Echo on the frontend. — [Joe Dixon, Laracon US 2024](https://www.youtube.com/watch?v=60g-Z-dAWNk&t=270s); [Joe Dixon, Laracon India 2024](https://www.youtube.com/watch?v=IDYzAMDs2YI&t=750s) · see [laravel.com/docs/13.x/broadcasting](https://laravel.com/docs/13.x/broadcasting)
  - Why: The socket server delivers broadcast events to frontend listeners through persistent connections.
  - Flag when: Deployment removes queue workers while application events still implement `ShouldBroadcast`.
- **Must** Authorize private and presence channels against resource access and persist messages before broadcasting. — [Joe Dixon, Laracon EU 2024](https://www.youtube.com/watch?v=yrL5eCMpqtc); [Joe Dixon, Laracon India 2024](https://www.youtube.com/watch?v=IDYzAMDs2YI&t=1680s) · see [laravel.com/docs/13.x/broadcasting](https://laravel.com/docs/13.x/broadcasting)
  - Why: Channel membership checks restrict sensitive events, while persistence retains messages beyond live delivery.
  - Flag when: Channel authorization returns true unconditionally, or a durable message is broadcast before being saved.
- **Should** Size Reverb by concurrent connections and use Redis pub/sub to scale it horizontally. — [Joe Dixon, Laracon US 2024](https://www.youtube.com/watch?v=60g-Z-dAWNk&t=960s); [Joe Dixon, Laracon India 2024](https://www.youtube.com/watch?v=IDYzAMDs2YI&t=600s) · see [laravel.com/docs/13.x/reverb](https://laravel.com/docs/13.x/reverb)
  - Why: Shared Redis propagates broadcasts to subscribers connected to every Reverb instance.
  - Flag when: Multiple Reverb instances are configured without shared Redis pub/sub scaling.
- **Should** Run Reverb under a supervisor behind a proxy with connection limits, and monitor it with Pulse. — [Joe Dixon, Laracon US 2024](https://www.youtube.com/watch?v=60g-Z-dAWNk&t=600s); [Joe Dixon, Laracon EU 2024](https://www.youtube.com/watch?v=yrL5eCMpqtc) · see [laravel.com/docs/13.x/reverb](https://laravel.com/docs/13.x/reverb)
  - Why: Process management keeps Reverb running, and connection limits determine how many clients can connect.
  - Flag when: Reverb Supervisor configuration disables automatic startup or restart.
- **Must** Keep Reverb event-loop callbacks small and non-blocking. — [Joe Dixon, Laracon US 2024](https://www.youtube.com/watch?v=60g-Z-dAWNk&t=540s) · see [laravel.com/docs/13.x/reverb](https://laravel.com/docs/13.x/reverb)
  - Why: Blocking callbacks stall the shared event loop and can deadlock work awaiting later iterations.
  - Flag when: Reverb callbacks perform synchronous database queries, HTTP calls, or waits for future ticks.

## Caching

- **Should** Optimize slow operations before caching; caching must not hide backend cost. — [Frank de Jonge, Laracon EU 2016](https://www.youtube.com/watch?v=0bq8619JijU&t=1350s); [Jess Archer, Laracon EU 2025](https://www.youtube.com/watch?v=0j6z_oq4cZU&t=360s) · see [laravel.com/docs/13.x/cache](https://laravel.com/docs/13.x/cache)
  - Why: Fast cache hits conceal slow misses and expiry-driven backend load spikes.
- **Must** Include every variation (locale, auth, query params) in cache keys. — [Mattias Geniar, Laracon EU 2016](https://www.youtube.com/watch?v=nAFpkV5-vuI); [Thijs Feryn, Laracon EU 2022](https://www.youtube.com/watch?v=VFTeGtPLlS0) · see [laravel.com/docs/13.x/requests](https://laravel.com/docs/13.x/requests)
  - Why: Different representations sharing a cache key can receive the wrong cached response.
  - Flag when: Cache keys omit locale, user identity, or query parameters that change the cached result.
- **Consider** Cache public unauthenticated responses at a CDN/proxy with explicit freshness and purge integration. — [Thijs Feryn, Laracon EU 2022](https://www.youtube.com/watch?v=VFTeGtPLlS0); [Mattias Geniar, Laracon EU 2016](https://www.youtube.com/watch?v=nAFpkV5-vuI) · see [laravel.com/docs/13.x/responses](https://laravel.com/docs/13.x/responses)
  - Why: Proxy cache hits serve reusable responses without executing Laravel.
  - Flag when: Public response caching lacks explicit freshness headers or invalidation for content that cannot wait for expiry.
- **Should** Use `Cache::flexible()`-style stale-while-revalidate only within an acceptable staleness window. — [Mattias Geniar, Laracon EU 2016](https://www.youtube.com/watch?v=nAFpkV5-vuI); [Thijs Feryn, Laracon EU 2022](https://www.youtube.com/watch?v=VFTeGtPLlS0) · see [laravel.com/docs/13.x/cache](https://laravel.com/docs/13.x/cache)
  - Why: Bounded stale delivery avoids refresh latency and simultaneous origin work for content that tolerates staleness.
  - Flag when: Stale cache windows exceed the documented freshness requirement for the cached data.
- **Should** Prevent cache stampedes with locks and warm hot entries before traffic bursts. — [Frank de Jonge, Laracon EU 2016](https://www.youtube.com/watch?v=0bq8619JijU&t=1170s) · see [laravel.com/docs/13.x/cache](https://laravel.com/docs/13.x/cache)
  - Why: Simultaneous misses repeat expensive backend work and overload it during reader bursts.
  - Flag when: Expensive cache-miss regeneration runs without a lock or request coalescing.
- **Consider** Invalidate only affected entries using tags or dependency keys. — [Mattias Geniar, Laracon EU 2016](https://www.youtube.com/watch?v=nAFpkV5-vuI); [Seb Armand, Laracon US 2024](https://www.youtube.com/watch?v=9mDkf4popb8&t=480s) · see [laravel.com/docs/13.x/cache](https://laravel.com/docs/13.x/cache)
  - Why: Dependency invalidation removes changed results while preserving reusable unaffected components.
  - Flag when: A single model update flushes the entire cache instead of affected entries.
- **Should** Diagnose caching by inspecting hits, misses, latency, and Redis commands in request context. — [Mattias Geniar, Laracon EU 2016](https://www.youtube.com/watch?v=nAFpkV5-vuI); [Taylor Otwell, Laracon AU 2018](https://www.youtube.com/watch?v=0O6dLJX_k3w&t=1770s) · see [laravel.com/docs/13.x/http-tests](https://laravel.com/docs/13.x/http-tests)
  - Why: Request context reveals unexpected misses and the Redis activity responsible for cache behavior.

## Database operations

- **Must** With read replicas, write to the primary and use `sticky` or explicit primary reads for read-after-write. — [Tobias Petry, Laracon EU 2024](https://www.youtube.com/watch?v=QbsmLXmpYXo&t=630s); [Guus Leeuw, Laracon India 2024](https://www.youtube.com/watch?v=rpvhD-twpY4) · see [laravel.com/docs/13.x/database](https://laravel.com/docs/13.x/database)
  - Why: Replication lag can make immediate dependent reads miss newly written data.
  - Flag when: An immediate dependent read explicitly uses a replica after writing to the primary.
- **Should** Measure query count, latency, and N+1s and optimize queries, indexes, and schema before adding database servers. — [Blackfire, Laracon EU 2021](https://www.youtube.com/watch?v=S0F2TaZh2a8&t=60s); [Jess Archer, Laracon EU 2025](https://www.youtube.com/watch?v=0j6z_oq4cZU&t=750s) · see [laravel.com/docs/13.x/database](https://laravel.com/docs/13.x/database)
  - Why: Repeated fast queries can accumulate substantial database time and expose N+1 behavior.
- **Consider** Stay on a single database as long as cost and capacity allow; don't assume managed or serverless databases fix scaling. — [Tobias Petry, Laracon EU 2024](https://www.youtube.com/watch?v=QbsmLXmpYXo&t=300s); [Jilesh Patadiya, Laracon India 2024](https://www.youtube.com/watch?v=MBr5DxfEGLA&t=1020s) · see [laravel.com/docs/13.x/database](https://laravel.com/docs/13.x/database)
  - Why: A single server postpones replica consistency and cross-shard query complexity.
- **Consider** Shard only for measured capacity needs, with durable entity-to-shard mappings and explicit connections. — [Tobias Petry, Laracon EU 2024](https://www.youtube.com/watch?v=QbsmLXmpYXo&t=1020s) · see [laravel.com/docs/13.x/database](https://laravel.com/docs/13.x/database)
  - Why: Durable shard mappings preserve data location, while sharding introduces cross-boundary query assembly.
  - Flag when: Entity-to-shard mappings exist only in an expiring or disposable cache.
- **Consider** Move heavy analytics to a column store or TimescaleDB when transactional queries can't keep up. — [Jess Archer, Laracon US 2024](https://www.youtube.com/watch?v=_jjvaFWWKqg); [Jess Archer, Laracon India 2026](https://www.youtube.com/watch?v=G9Toicww8BU&t=630s) · see [laravel.com/docs/13.x/database](https://laravel.com/docs/13.x/database)
  - Why: Column-oriented storage handles high-volume analytical aggregations more efficiently than transactional storage.

## Observability & monitoring

- **Should** Instrument production early (Nightwatch, Pulse, or similar) with actionable alerts on route, job, and query thresholds. — [Geshan Manandhar, Laracon AU 2019](https://www.youtube.com/watch?v=6I1MK3lNZPI&t=900s); [Neel Shah, Laracon India 2026](https://www.youtube.com/watch?v=CdfW-bCKZ3E&t=570s) · see [laravel.com/docs/13.x/logging](https://laravel.com/docs/13.x/logging)
  - Why: Actionable alerts expose meaningful failures and saturation so operators can respond promptly.
- **Should** Correlate requests, users, queries, exceptions, logs, and job attempts, accounting for retries. — [Taylor Otwell, Laracon AU 2018](https://www.youtube.com/watch?v=0O6dLJX_k3w&t=210s); [Taylor Otwell, Jess Archer, Laracon AU 2024](https://www.youtube.com/watch?v=tesqB-DPP1I) · see [laravel.com/framework/docs/13.x/telescop](https://laravel.com/framework/docs/13.x/telescope#log-watcher)
  - Why: Connected events preserve the triggering context needed to diagnose failures in requests and asynchronous work.
- **Should** Propagate correlation IDs across services, queues, and child processes. — [David McKay, Laracon EU 2018](https://www.youtube.com/watch?v=BvDXzyOP3NE&t=2400s); [Neel Shah, Laracon India 2026](https://www.youtube.com/watch?v=CdfW-bCKZ3E&t=600s) · see [laravel.com/docs/13.x/deployment](https://laravel.com/docs/13.x/deployment)
  - Why: Shared trace context makes distributed application behavior observable across service boundaries.
  - Flag when: Outgoing service calls or queued jobs replace the incoming correlation ID with an unrelated identifier.
- **Should** Use Monolog channels and stacks with a consistent, parseable log schema and safe context. — [Geshan Manandhar, Laracon AU 2019](https://www.youtube.com/watch?v=6I1MK3lNZPI&t=600s) · see [laravel.com/docs/13.x/logging](https://laravel.com/docs/13.x/logging)
  - Why: Consistent structured context makes logs parseable and identifies the affected orders or other entities.
  - Flag when: New log messages concatenate variable fields instead of using the established structured context schema.
- **Must** Scrub secrets and personal data from telemetry and restrict production profiling to authorized, bounded investigations. — [Geshan Manandhar, Laracon AU 2019](https://www.youtube.com/watch?v=6I1MK3lNZPI&t=1200s); [Blackfire, Laracon EU 2021](https://www.youtube.com/watch?v=1awn4OfKF1A&t=300s) · see [laravel.com/docs/13.x/logging](https://laravel.com/docs/13.x/logging)
  - Why: Unsanitized diagnostics can reveal passwords, payment details, and personal data while production instrumentation adds overhead.
  - Flag when: Logging passes entire request payloads or unredacted sensitive fields to telemetry.
- **Should** Tune thresholds, sampling, and filters to the question you're answering; keep rare failures. — [Taylor Otwell, Laracon AU 2018](https://www.youtube.com/watch?v=0O6dLJX_k3w&t=1350s); [Jess Archer, Laracon AU 2023](https://www.youtube.com/watch?v=-9jV5ra5KB4&t=570s) · see [laravel.com/framework/docs/13.x/telescop](https://laravel.com/framework/docs/13.x/telescope#query-watcher)
  - Why: Irrelevant events obscure useful diagnostics, and sparse metrics lose useful representation with low sampling rates.
  - Flag when: Production telemetry filters discard reportable exceptions or failed jobs.
- **Should** Monitor queue backlog, consumers, and scheduled command exit codes, not just process liveness. — [Frank de Jonge, Laracon EU 2016](https://www.youtube.com/watch?v=0bq8619JijU&t=1860s); [Taylor Otwell, Laracon AU 2018](https://www.youtube.com/watch?v=0O6dLJX_k3w&t=330s) · see [laravel.com/docs/13.x/deployment](https://laravel.com/docs/13.x/deployment)
  - Why: Running processes alone do not establish that queues are being consumed or scheduled commands succeeded.
- **Should** Supervise Pulse and Nightwatch agents and verify data actually reaches the dashboard. — [Taylor Otwell, Laracon AU 2018](https://www.youtube.com/watch?v=0O6dLJX_k3w&t=540s); [Taylor Otwell, Jess Archer, Laracon AU 2024](https://www.youtube.com/watch?v=tesqB-DPP1I) · see [laravel.com/docs/13.x/telescope](https://laravel.com/docs/13.x/telescope)
  - Why: Captured telemetry needs a running delivery agent, and observability tools do not run or balance queue workers.
  - Flag when: Telemetry agent process configuration disables automatic startup or restart.
- **Consider** Keep telemetry off the request-critical path with cheap batched writes, isolating its storage if it competes. — [Geshan Manandhar, Laracon AU 2019](https://www.youtube.com/watch?v=6I1MK3lNZPI&t=1050s); [Jess Archer, Laracon India 2026](https://www.youtube.com/watch?v=G9Toicww8BU&t=90s) · see [laravel.com/docs/13.x/logging](https://laravel.com/docs/13.x/logging)
  - Why: Blocking remote delivery for every log event adds latency to application requests.
  - Flag when: Request handling synchronously sends each telemetry event to a remote service.
- **Consider** Version performance budgets for measured hotspots and check them before deploying. — [Blackfire, Laracon EU 2021](https://www.youtube.com/watch?v=1awn4OfKF1A&t=870s); [Blackfire, Laracon EU 2021](https://www.youtube.com/watch?v=l54v0sQopXk) · see [docs.blackfire.io/introduction](https://docs.blackfire.io/introduction)
  - Why: Automated performance assertions detect regressions before they reach production.
- **Consider** Debug locally with mail capture, dumps, and breakpoints, never real mail or public debug output. — [Taylor Otwell, Laracon AU 2018](https://www.youtube.com/watch?v=0O6dLJX_k3w&t=1590s); [Francesco Ciulla, Laracon India 2026](https://www.youtube.com/watch?v=CMqhOhRnOY0&t=900s) · see [laravel.com/docs/13.x/mail](https://laravel.com/docs/13.x/mail)
  - Why: Local mail capture prevents accidental delivery, and contextual dumps avoid inserting debug values into rendered output.
  - Flag when: Development mail configuration points to a live delivery provider instead of local capture.

## Security & ops

- **Must** Provision credentials outside the repo with least privilege per environment; prefer IAM roles or short-lived credentials and plan rotation. — [Jeremy Lindblom, Laracon US 2014](https://speakerdeck.com/jeremeamia/aws-for-artisans); [David McKay, Laracon EU 2018](https://www.youtube.com/watch?v=BvDXzyOP3NE&t=1170s) · see [laravel.com/docs/13.x/configuration](https://laravel.com/docs/13.x/configuration)
  - Why: Committed or publicly exposed credentials disclose access to application services.
  - Flag when: Application configuration or container build instructions embed real credentials.
- **Must** Automate access onboarding, offboarding, and credential rotation; share only SSH public keys. — [Chris Fidao, Laracon US 2016](https://speakerdeck.com/fideloper/server-survival); [Srdjan Vranac, Laracon EU 2019](https://www.youtube.com/watch?v=TF-E5FPHQaY&t=90s) · see [laravel.com/docs/13.x/deployment](https://laravel.com/docs/13.x/deployment)
  - Why: Former staff credentials can remain usable after departure unless access and outstanding tokens are revoked.
- **Should** Run services with minimal privileges, scan images, and keep OS and runtime patches current. — [Srdjan Vranac, Laracon EU 2019](https://www.youtube.com/watch?v=TF-E5FPHQaY&t=420s); [Francesco Ciulla, Laracon India 2026](https://www.youtube.com/watch?v=CMqhOhRnOY0&t=450s) · see [laravel.com/docs/13.x/deployment](https://laravel.com/docs/13.x/deployment)
  - Why: The runtime holds secrets, and even official container images can contain security vulnerabilities.
  - Flag when: Production service configuration runs application processes as root.
- **Consider** When higher-level tools fall short, inspect CPU, memory, OPcache, and PHP-FPM timings, or use strace. — [Mattias Geniar, Laracon EU 2018](https://www.youtube.com/watch?v=94GK6Ik-x98&t=1050s) · see [laravel.com/docs/13.x/deployment](https://laravel.com/docs/13.x/deployment)
  - Why: Process metrics and timed syscall traces reveal resource consumption and waits hidden from higher-level diagnostics.

## Pending decisions (excluded for now)

Talks disagree on these; no rule is included until decided.

- Staging before release: staging QA or direct production delivery.

## Outdated (reference only)

Not rules — these no longer apply to Laravel 13 / current tooling.

- Use HTTP/2 server push to send anticipated page resources before the browser requests them, with preload Link headers informing server push. *Why outdated:* Chrome disabled HTTP/2 server push by default in version 106 and Nginx made its push directives obsolete; preload/Early Hints are distinct mechanisms. — [Ben Ramsey, Laracon US 2016](https://speakerdeck.com/ramsey/http-is-dead-long-live-http2-laracon-2016), [laravel.com/docs/13.x/vite](https://laravel.com/docs/13.x/vite), [developer.chrome.com/blog/removing-push](https://developer.chrome.com/blog/removing-push)
