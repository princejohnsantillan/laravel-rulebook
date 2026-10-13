# Laravel Codebase Rules

**Scope:** Application code for Laravel 13 and current first-party tools: architecture, routing, Eloquent, security, performance, testing, frontend stacks, and packages.

**Levels:** **Must** = non-negotiable · **Should** = default; deviate only with a clear reason · **Consider** = apply when the situation fits.

**Sub-bullets:** *Why* = the reason behind the rule, quotable when flagging a violation · *Flag when* = an observable signal in a diff that suggests a violation (omitted when not observable in code).

*Written for code-reviewing agents: use these rules to evaluate diffs, not as an implementation checklist.*

*Based on Laracon talks (2013–2026) and the research behind them; see the `.cited.md` variant for sources.*

## General

### Architecture & structure

- **Must** Keep Laravel, PHP, and dependencies on supported versions and apply security fixes promptly.
  - Why: reduces exposure to vulnerabilities in unsupported or unpatched software.
  - Flag when: Dependency constraints or runtime images pin unsupported versions or releases with known security fixes.
- **Should** Use Laravel and established packages where they fit; write custom code only when business needs justify maintaining it.
  - Why: frees development time for business features and reduces custom code maintenance.
  - Flag when: New custom infrastructure duplicates suitable Laravel facilities or an established package already in use.
- **Should** Document team conventions (naming, structure, branching, review) and enforce formatting with Pint.
  - Why: makes recurring implementations predictable and applies shared conventions consistently.
- **Should** Start simple, tolerate duplication until a stable shared responsibility emerges, and add abstractions only when they clearly pay off.
  - Why: avoids coupling merely similar business rules through premature abstractions.
  - Flag when: A shared abstraction gains feature-specific switches to accommodate unrelated business rules.
- **Should** Draw boundaries around business contexts, exposing small public interfaces named in domain language and hiding internals.
  - Why: keeps business terms unambiguous within contexts and isolates changes behind explicit interfaces.
  - Flag when: Code in one business context directly accesses another context’s internal implementation.
- **Should** Start with one deployable app; extract services only when concrete benefits outweigh coordination and ops costs.
  - Why: avoids extra operational and development overhead before measured constraints justify service extraction.
- **Should** Use the agreed business vocabulary consistently across operations, events, code, and docs.
  - Why: preserves shared understanding between developers and domain experts as terminology evolves.
  - Flag when: A renamed business concept retains conflicting names in related operations, events, or documentation.
- **Should** Inject typed dependencies and let the container autowire them; add bindings only where the implementation or construction must be chosen.
  - Why: makes collaborators explicit and substitutable in tests without requiring manual construction everywhere.
  - Flag when: A service constructs a replaceable collaborator internally instead of accepting a typed dependency.
- **Should** Use intention-revealing names and focused methods; write comments that explain why, not what.
  - Why: makes operations understandable without deciphering vague names or comments that merely narrate code.
  - Flag when: New vague method names or comments narrate operations that clearer names could express.
- **Should** Use guard clauses and early returns to keep the main path flat and readable.
  - Why: keeps the primary operation visible at the outer indentation level.
  - Flag when: Failed prerequisite checks wrap the main operation in successive nested conditionals.
- **Should** Split classes with too many responsibilities into objects for each distinct domain concept.
  - Why: makes behavior easier to describe and test with less extensive mocking.
  - Flag when: A class gains behavior for distinct domain concepts alongside its existing unrelated responsibilities.
- **Should** Refactor touched code in small, verified steps, proportionate to the change at hand.
  - Why: makes the next feature easier to implement while preserving existing behavior.
- **Should** Modernize legacy code in small reviewed batches using Pint, static analysis, and Rector.
  - Why: reduces manual cleanup and exposes defects while progressively improving frequently changed legacy code.
- **Should** Provide documented, repeatable commands for setup, dev, tests, formatting, and deploy, plus useful seed data.
  - Why: lets new developers get a fresh checkout running quickly with working tests and representative data.
  - Flag when: New setup requirements or required services lack corresponding setup commands or documentation.
- **Should** Review your own diff and get CI green before requesting review; get a second developer's eyes on every change.
  - Why: keeps repetitive mechanical fixes out of human reviews and brings another developer’s perspective.
- **Consider** Start from an official starter kit, then delete what the product doesn't need.
  - Why: provides established application conventions and scaffolding matched to required features.

### Routing & controllers

- **Must** Follow Adam Wathan's CRUDdy by Design: controllers expose only the 7 resourceful actions (`index`, `create`, `store`, `show`, `edit`, `update`, `destroy`).
  - Why: The seven-action constraint exposes missing resources and breaks large controllers into focused responsibilities.
  - Flag when: A route targets a controller action outside the seven resource methods.
- **Must** Never add custom controller actions; when you need one, model the concept as a new resource with its own controller (e.g. `PodcastSubscriptionController@store` instead of `PodcastController@subscribe`).
  - Why: Custom verbs can hide domain concepts with their own identity and lifecycle.
  - Flag when: A parent controller gains subscribe, unsubscribe, publish, or another custom resource operation.
- **Should** Use nested resources (and nested controllers) for actions on a child concept of a parent, e.g. `episodes.publish` → `EpisodePublicationController`.
  - Why: Dedicated child controllers separate operations with different loading, authorization, views, and response data.
  - Flag when: One action branches between global and parent-specific child operations using an optional parent parameter.
- **Should** Use `Route::resource()` / `apiResource()` with `only()` to declare routes; treat a controller that outgrows the 7 actions as a sign a resource is missing.
  - Why: Conventional resource routes reduce repetitive API plumbing while custom actions reveal missing resource boundaries.
  - Flag when: A resource route registers unimplemented actions, or its controller gains custom resource operations.
- **Consider** Use a single-action invokable controller for a named business operation that has no natural resource.
  - Why: A dedicated controller keeps a named business operation separate from general CRUD responsibilities.
- **Must** Scope nested route-model binding to the parent (`scopeBindings()`) so a child ID can't resolve under an unrelated parent.
  - Why: Parent-scoped lookup prevents unrelated child records from resolving under another parent URL.
  - Flag when: A nested route resolves its child globally without scoped binding or a parent relationship lookup.
- **Should** Use implicit route-model binding with matching segment names so missing models 404 before the action runs.
  - Why: Implicit binding retrieves records and handles missing models before controller logic runs.
  - Flag when: A model parameter name differs from its route segment, or an action manually repeats a standard lookup.
- **Must** Authenticate APIs with distinct per-client credentials (Sanctum tokens) and use cookie sessions for first-party SPAs; authorize each resource separately.
  - Why: Distinct credentials identify clients, while ownership checks restrict access to protected resources.
  - Flag when: An API shares one credential across clients or accesses protected records without resource authorization.
- **Should** Keep stateless API routes free of sessions and cookies; keep session and CSRF for browser and Sanctum SPA routes.
  - Why: Stateless requests otherwise create unnecessary persistent session records and session cookies.
  - Flag when: A stateless token API adds session middleware, or a browser authentication flow removes session or CSRF middleware.
- **Should** Shape public API output with API Resources for consistent fields, types, dates, and pagination.
  - Why: Reusable resource mappings preserve predictable public response shapes when internal model attributes change.
  - Flag when: A public API returns raw models or duplicates field mappings across controllers.
- **Should** Paginate the query and pass the paginator to a resource collection for consistent pagination metadata.
  - Why: Passing the paginator preserves reusable item formatting and supplies consistent pagination metadata.
  - Flag when: A resource collection receives paginator items instead of the paginator, discarding pagination metadata.
- **Should** Map domain and validation exceptions to consistent responses centrally, returning JSON for APIs and redirects for browsers.
  - Why: Central error rendering gives failures consistent responses without repeated controller-level catch branches.
  - Flag when: Controllers duplicate validation or domain exception catches to construct error responses.
- **Should** Put cross-cutting request/response concerns in scoped middleware registered in `bootstrap/app.php`.
  - Why: Scoped middleware reuses shared request checks and response processing across the endpoints that need them.
  - Flag when: Multiple controllers repeat the same request check or response processing.
- **Should** Allowlist API filters, sorts, includes, and fields (e.g. Spatie Query Builder); never pass raw query params to Eloquent.
  - Why: Declared query capabilities reject unsupported includes and avoid repetitive request-to-Eloquent parsing.
  - Flag when: Request values directly choose query columns or relationships without an allowlist.
- **Should** Agree API contracts with consumers and keep a versioned OpenAPI spec in the repo, validated in CI.
  - Why: A shared, versioned specification lets clients develop in parallel against a consistent, testable contract.
  - Flag when: CI removes API specification validation, or an endpoint changes without updating its checked-in contract.
- **Consider** Evolve APIs backward-compatibly with announced deprecations; introduce a new major version only for breaking changes.
  - Why: Compatible changes avoid integrator migration costs while versions let incompatible contracts coexist.
  - Flag when: An existing API response removes or renames a public field without preserving the previous contract.

### Eloquent & database

- **Must** Follow Jonathan Reinink's Eloquent Performance Patterns: fetch only the columns, counts, aggregates, or single related row you need; never hydrate whole histories to read one value.
  - Why: Hydrating entire histories wastes memory and request time when only one value is needed.
  - Flag when: A related collection is loaded and sorted solely to read its latest value.
- **Must** Apply visibility and filter constraints in SQL before pagination, never by filtering fetched collections.
  - Why: SQL visibility constraints avoid hydrating unauthorized records and allow database pagination.
  - Flag when: Visibility or search filters run on a fetched collection instead of the query before pagination.
- **Should** Fetch a single related value (e.g. last login date) with a subquery select (`addSelect`) under a clear alias, plus query-time casts.
  - Why: A scalar subquery avoids loading entire histories while casting enables normal date formatting.
  - Flag when: A full relationship history is loaded solely to display one date.
- **Consider** Use one-of-many relationships (`latestOfMany`) or subquery-selected IDs with eager loading when you need several attributes from one related row.
  - Why: Fetching one related model avoids duplicating scalar subqueries for each of its attributes.
  - Flag when: Multiple subqueries independently select attributes from the same latest related row.
- **Should** Order by related or pivot data in SQL using joins or correlated subqueries, with a stable secondary sort, before paginating.
  - Why: SQL ordering sorts the full result set before pagination without loading everything into PHP.
  - Flag when: Related values are sorted in a fetched collection after pagination.
- **Should** Put reusable filters, visibility rules, selects, and required eager loads in named query scopes, passing user context explicitly.
  - Why: Named scopes let callers reuse and compose domain constraints without repeating query details.
  - Flag when: Recurring visibility predicates or related-value selections are duplicated across controllers.
- **Should** Give a recurring subset its own relationship (e.g. `recentComments()`) so it can be eager loaded directly.
  - Why: Naming the subset keeps relationship meaning clear and removes repeated filtering closures.
  - Flag when: Multiple callers redefine the same recurring subset with constrained eager-loading closures.
- **Should** Enable `Model::preventLazyLoading()` outside production and eager load relationships needed across model sets.
  - Why: Lazy-loading detection exposes missing eager loads that cause a separate relationship query for every model.
  - Flag when: A loop accesses an unloaded relationship on each model.
- **Should** Decide relationship loading at the query, and use `whenLoaded()`/conditional resource fields so rendering never triggers hidden N+1s.
  - Why: Queries during rendering create hidden repeated retrievals and counts that lazy-loading detection can miss.
  - Flag when: A Blade loop or resource serializer executes relationship queries or counts.
- **Should** Design composite indexes with equality columns before range columns, and verify with `EXPLAIN`.
  - Why: Leading equality constraints narrow the index scan before a range limits usable subsequent columns.
  - Flag when: A composite index puts a range-filter column before equality-filter columns used by the same query.
- **Should** Profile query count, latency, memory, and hydrated models against realistic data volumes; extra bounded queries are fine if total work drops.
  - Why: Fewer queries can still make requests slower by hydrating thousands of unnecessary models.
- **Consider** Denormalize or add read models only when measured cost justifies the freshness and maintenance tradeoff.
  - Why: Efficient related-record queries can avoid cached derived state and its real-time freshness problems.
- **Should** Create and associate related records through relationship methods, wrapping repeated linking in named domain methods.
  - Why: Relationship methods assign linking keys while domain methods hide repeated association details.
  - Flag when: Child creation manually assigns the parent foreign key instead of using the existing relationship.
- **Should** Use pivot attributes for simple association metadata and `syncWithoutDetaching()` to add links without dropping existing ones.
  - Why: Pivot attributes keep simple assignment metadata on the association it describes.
  - Flag when: `sync()` is used to add links while existing associations must be preserved.
- **Must** Enforce every business uniqueness invariant with a database unique constraint.
  - Why: Database constraints reject duplicate business keys that error-prone application checks can miss.
  - Flag when: Business uniqueness is checked only in application code without a matching database unique constraint.
- **Must** Encrypt sensitive attributes with the `encrypted` casts, size columns for ciphertext, and don't bulk-write or search them in plaintext.
  - Why: Model encryption protects sensitive database fields only when writes pass through the encryption layer.
  - Flag when: Bulk insert or update writes plaintext directly to an attribute configured with an encrypted cast.
- **Must** In shared-database multi-tenancy, apply a global tenant scope and set the tenant key automatically; bypass it only deliberately.
  - Why: Automatic scoping and ownership keep ordinary reads and writes confined to the active tenant.
  - Flag when: A tenant-owned model lacks the tenant scope or automatic tenant-key assignment.
- **Must** Commit migrations with the code that needs them; never rely on copying production databases to recreate schema.
  - Why: Versioned migrations let developers and tests recreate the schema without production database copies or coordinated manual edits.
  - Flag when: Code uses a new table or column without a corresponding migration.
- **Should** Maintain realistic factories and seeders with meaningful relations and states; never distribute production data.
  - Why: Synthetic data supports development and testing without exposing sensitive production records.
  - Flag when: A committed fixture or seeder contains identifiable production records.
- **Should** Separate required settings seeders from demo/fixture seeders, and never truncate live data when seeding production.
  - Why: Purpose-specific seeders install required settings without inserting demo records or destroying live data.
  - Flag when: A production seeder calls demo seeders or truncates live tables.
- **Should** Prefer readable collection pipelines with built-in methods; convert to arrays only at output boundaries.
  - Why: Focused collection operations reduce interleaved conditionals, temporary state, and repeated intermediate array conversions.
  - Flag when: A collection is converted to an array for intermediate transformations supported by collection methods.
- **Consider** Put recurring operations on sets of models in a custom Eloquent collection class.
  - Why: Domain-named collection methods make dense model-set pipelines understandable and reusable.
  - Flag when: The same model-set pipeline is duplicated across callers.
- **Should** Annotate collection and relationship generics in PHPDoc so model types survive iteration and chaining.
  - Why: Generic annotations preserve concrete model types through iteration and query chaining for static analysis.
  - Flag when: A collection or relationship declaration omits model generic types where inference loses them.

### Validation & security

- **Must** Always use parameter bindings; only allow application-controlled identifiers (columns, tables) in raw SQL.
  - Why: Interpolated input lets attackers alter queries and extract database records through SQL injection.
  - Flag when: Request data is interpolated into raw SQL or used as an unrestricted SQL identifier.
- **Must** Map user-selected sort keys to predefined orderings; never pass raw input as a column or SQL expression.
  - Why: User-controlled ordering identifiers can introduce SQL injection even when query values are bound.
  - Flag when: Request sort input reaches `orderBy` or `orderByRaw` without a fixed allowlist mapping.
- **Must** Authorize every protected read and write on the server, regardless of authentication, route binding, hidden URLs, or frontend flags.
  - Why: Clients can alter frontend permissions, and authentication alone does not grant access to protected resources.
  - Flag when: A protected endpoint relies on authentication or frontend flags without enforcing server authorization.
- **Must** Centralize authorization in gates and policies and check abilities, not role names.
  - Why: Shared abilities reuse authorization decisions without coupling callers to role names.
  - Flag when: Controllers or templates compare role names to decide access instead of checking abilities.
- **Must** Read `env()` only inside config files; use `config()` everywhere else.
  - Why: Cached configuration stops loading `.env`, leaving direct `env()` reads without those values.
  - Flag when: An `env()` call is added outside configuration files.
- **Must** Protect `.env` and `APP_KEY`; never reuse the app key for unrelated encryption.
  - Why: An encryption service sharing `APP_KEY` can produce attacker-chosen ciphertext trusted by other application components.
  - Flag when: An arbitrary-content encryption endpoint uses the application key.
- **Must** Serve only `public/` and disable debug mode in production and any public environment.
  - Why: Public debug output can disclose private records, sharing keys, and sensitive configuration.
  - Flag when: Public deployment config enables debug mode or serves the project root instead of `public/`.
- **Must** Hash passwords with Laravel's `Hash`; never store plaintext, encrypted, or MD5 passwords.
  - Why: Slow password hashing limits exposure of reusable credentials when the database is stolen.
  - Flag when: Password persistence uses plaintext, reversible encryption, or MD5 instead of Laravel password hashing.
- **Must** Return safe, useful error responses for expected failures and report full diagnostics internally without leaking sensitive details.
  - Why: Useful responses help users handle failures while internal diagnostics support debugging without exposing sensitive information.
  - Flag when: An error response includes exception traces, SQL, secrets, or other sensitive diagnostic details.
- **Must** Hide sensitive attributes from API responses and expose privileged fields only after a permission check.
  - Why: prevent API responses from exposing credentials or private fields to unauthorized callers.
  - Flag when: API resources include credentials or privileged fields without checking the caller’s permission.
- **Must** Use Secure, HttpOnly cookies with Sanctum's stateful CSRF flow for first-party SPAs; never store bearer tokens in JavaScript-accessible storage.
  - Why: reduce authentication-token exposure to JavaScript when browser code is compromised.
  - Flag when: Bearer tokens are written to localStorage or sessionStorage, or session cookies disable Secure or HttpOnly.
- **Must** Restrict user-supplied file paths to intended storage locations and never `unserialize()` untrusted data.
  - Why: prevent secret-file disclosure and attacker-controlled object execution.
  - Flag when: User input reaches file reads without containment checks, or reaches unserialize().
- **Should** Rate-limit logins, APIs, and expensive endpoints per caller with `RateLimiter`, watching for abuse.
  - Why: prevent one caller’s traffic bursts from consuming resources shared by other users.
  - Flag when: Login or expensive API routes omit throttling, or limiter keys fail to distinguish callers.
- **Should** Use Laravel's validator and put reusable constraints in `ValidationRule` objects.
  - Why: reuse Laravel’s validation engine and avoid repeating domain checks across controllers.
  - Flag when: The same custom validation checks are duplicated across controllers instead of shared rule objects.
- **Should** Throw `ValidationException::withMessages()` for rejected operations and make sure the UI shows those errors.
  - Why: give users visible, contextual feedback when a business operation is rejected.
  - Flag when: Backend emits a synthetic validation error key that the frontend never displays.
- **Should** Audit security-relevant events (auth, authorization, notifications) with filtered context.
  - Why: make allowed and denied access traceable for investigating policy or role mistakes.
  - Flag when: Audit listeners persist passwords or unfiltered request, mail, or notification payloads.
- **Should** Set up exception tracking and searchable logs early, fix recurring errors, and add regression tests for them.
  - Why: make failures traceable to their originating code so recurring causes can be fixed.

### Performance

- **Should** Queue slow work the response doesn't need, on a real async connection with supervised workers.
  - Why: keep expensive work such as sending email from delaying the web response.
  - Flag when: Request handlers synchronously perform deferrable work, or queued jobs explicitly use the sync connection.
- **Should** Process large datasets incrementally (`chunk`, `lazy`, cursors) instead of loading everything into memory.
  - Why: keep memory use from growing with the entire recipient list or import input.
  - Flag when: Large recipient sets or import inputs are fully materialized before processing.
- **Should** Register a cumulative query-time threshold (`DB::whenQueryingForLongerThan`) to catch requests with many small queries.
  - Why: detect accumulated database latency that individually fast queries hide.
- **Should** Cache routes in production and regenerate on deploy; don't cache routes in local dev.
  - Why: avoid repeating route registration and discovery work on every production request.
  - Flag when: Deployment clears route caches without rebuilding them, or local startup runs route:cache.
- **Should** Set explicit connect and total timeouts and retries on every outbound HTTP/SDK call.
  - Why: keep slow external dependencies from holding up user requests.
  - Flag when: Outbound clients have no explicit connection or total timeout in their call, shared builder, or configuration.
- **Should** Measure before optimizing: profile app logic, queries, rendering, and blocking calls before adding capacity or blaming the framework.
  - Why: target the actual bottleneck before spending money on capacity that may not solve it.
- **Should** Keep service providers lean; don't eagerly build unused services or capture request state in singletons.
  - Why: avoid paying bootstrap costs for dependency graphs the request never uses.
  - Flag when: Providers eagerly resolve unused services or singleton factories capture the current request.
- **Consider** Load-test critical paths at increasing concurrency against explicit latency and error targets.
  - Why: detect latency and failure regressions as request concurrency increases.

### Design patterns & code style

- **Must** Register bindings in `register()` and do work that depends on other services in `boot()`.
  - Why: Dependent setup needs services registered by all providers.
  - Flag when: Service-dependent setup in `register()`, or container bindings in `boot()`.
- **Should** Keep business logic out of service providers; use them only for construction and bindings.
  - Why: Separating service construction from behavior keeps business services independently reusable and testable.
  - Flag when: Service provider executes business calculations or domain operations.
- **Should** Use contextual or factory bindings when values or implementations can't be inferred.
  - Why: Autowiring cannot infer required configuration values or consumer-specific implementations.
  - Flag when: Container-resolved constructor adds a required scalar or unbound interface without a resolution strategy.
- **Should** Dispatch domain events for meaningful business moments with independent listeners; queue slow listeners and dispatch commit-dependent work after commit.
  - Why: Independent reactions keep notifications, mail, and logging separate from the core business operation.
  - Flag when: Business operation directly performs multiple independent notification, mail, and logging side effects.
- **Should** Replace growing type-based conditionals with polymorphic implementations behind a shared interface.
  - Why: New behaviors can be added without duplicating orchestration or modifying every consumer.
  - Flag when: Growing `switch` or `instanceof` branches select behavior by concrete type.
- **Should** Use immutable value objects for recurring related primitives (money, ranges), validated on construction.
  - Why: Related values share validation and calculations instead of duplicating them across callers.
  - Flag when: Recurring amount/currency or date pairs repeat validation and calculations across callers.
- **Should** Wrap third-party APIs behind an application-owned class so tests can fake it.
  - Why: Tests can exercise application behavior without destructive, costly, or rate-limited external calls.
  - Flag when: Business logic directly constructs or invokes a third-party API client.
- **Should** Use typed DTOs with explicit mapping for complex inputs, results, and API contracts instead of loose arrays.
  - Why: Explicit data types expose misspelled fields and incompatible assignments to static analysis.
  - Flag when: Complex input or response contracts use loose arrays with repeated string-key access.
- **Should** Keep interfaces small and specific to what each consumer needs.
  - Why: Consumers avoid depending on capabilities they never use.
  - Flag when: Interface adds unrelated capabilities that existing consumers do not need.
- **Should** Ensure subclasses and implementations honor the behavior their contract promises.
  - Why: Consumers rely on promised behavior when implementations are substituted.
  - Flag when: Implementation leaves required behavior empty or throws an unsupported-operation exception.
- **Should** Degrade gracefully when a dependency fails, using fallbacks or deferred work so unrelated features keep working.
  - Why: A dependency outage should leave unrelated features usable.
  - Flag when: Shared request setup aborts on an optional external service failure without recovery.
- **Consider** Use `scoped()` bindings for request/job state and `singleton()` only for truly shared state in long-lived workers.
  - Why: Singleton resolutions reuse the same object and its accumulated state.
  - Flag when: Singleton binding stores mutable request-specific or job-specific state.
- **Consider** Use facades or real-time facades as concise, testable proxies to container services.
  - Why: Container-backed proxies offer concise calls while allowing collaborators to be replaced in tests.
- **Consider** Use decorators to layer caching, logging, or eligibility checks around an operation without changing its contract.
  - Why: Independently varying behaviors can be composed without multiplying subclasses.
- **Should** If you adopt `declare(strict_types=1)`, apply it consistently across all calling files.
  - Why: Scalar argument checking follows the calling file, so inconsistent declarations allow unintended coercion.
  - Flag when: Calling application file omits `declare(strict_types=1)` within a namespace adopting strict typing.

### Tooling & AI

- **Should** Run PHPStan with Larastan locally and in CI, starting at an attainable level and raising it over time.
  - Why: Laravel-aware analysis catches undefined members and incompatible types before affected endpoints run.
  - Flag when: CI removes PHPStan analysis, or PHPStan configuration omits the Larastan extension.
- **Should** Use PHPDoc generics/templates where they preserve concrete types through callbacks and collections.
  - Why: Preserves concrete input, callback, and result types so analyzers can detect incompatible values.
  - Flag when: Reusable collection or callback APIs return `mixed` despite a concrete input-to-output type relationship.
- **Consider** In modular codebases, enforce allowed dependencies between modules with Deptrac in CI.
  - Why: Catches dependencies on other modules' implementation classes before merging.
  - Flag when: CI removes the Deptrac check or suppresses a dependency forbidden by the module ruleset.
- **Should** Declare frontend entry points explicitly in Vite config with dev, build, and watch scripts.
  - Why: Makes asset dependencies explicit and build commands repeatable without memorizing project-specific arguments.
  - Flag when: New frontend entry files lack Vite inputs, or package scripts omit the required build modes.
- **Must** Build optimized production assets and reference them via `@vite`/`Vite::asset()` for versioned URLs.
  - Why: Reduces production asset size and lets content changes invalidate cached URLs.
  - Flag when: Templates hard-code compiled asset filenames or deployment scripts use development assets without a production build.
- **Should** Rely on Laravel's discovery for commands, events, and listeners instead of manual registration lists.
  - Why: Avoids maintaining redundant command and listener registration lists.
  - Flag when: Adds manual registrations for commands or listeners already covered by configured discovery.
- **Consider** Generate TypeScript types from PHP DTOs and enums rather than hand-duplicating schemas.
  - Why: Reduces repeated schema definitions and keeps frontend field names and nested shapes aligned with PHP contracts.
  - Flag when: Handwritten TypeScript interfaces or enums duplicate existing PHP DTOs or enums.

## Testing

- **Must** Add a Pest arch test banning `dd`, `dump`, and `ray` in application code.
  - Why: Catches leftover debugging calls even in paths behavioral tests never execute.
- **Should** Write readable HTTP feature tests with explicit fixtures and auth, asserting responses, redirects, validation errors, and persisted data.
  - Why: Verifies application behavior through requests, including successful writes and failure responses.
  - Flag when: New write-endpoint tests assert only HTTP status, omitting promised redirects, errors, or persisted effects.
- **Should** Run real queries in database tests and assert resulting data, not SQL strings or builder call sequences.
  - Why: Verifies query results rather than merely confirming that expected builder methods were called.
  - Flag when: Query tests mock builder calls or compare SQL strings without executing queries and checking results.
- **Should** Test both allowed and forbidden access for every authorized action.
  - Why: Verifies permission checks on each action even when ownership rules look repetitive.
  - Flag when: A newly protected action has permitted-access tests but no forbidden-access case.
- **Should** Fake mail, notifications, queues, and events before acting, then assert recipients, payloads, and counts.
  - Why: Detects wrong recipients and payloads without coupling tests to internal callback order.
  - Flag when: Side-effect tests fake after execution or assert only message classes without relevant recipients or payloads.
- **Should** Give each test only the fixtures it needs and reset database and shared state so tests can't affect each other.
  - Why: Keeps unrelated setup and data left by other tests from changing outcomes.
  - Flag when: Tests rely on preexisting records, execution order, or shared mutable state without cleanup.
- **Should** Assert independently known outcomes; never recompute the expected value with the code under test.
  - Why: Prevents copied implementation logic from making incorrect behavior pass its own assertions.
  - Flag when: Expected values call the code under test or duplicate its calculation algorithm.
- **Should** Collapse duplicate tests of the same behavior into datasets/data providers.
  - Why: Runs multiple input and expected-output cases without duplicating the same test body.
  - Flag when: Multiple tests repeat identical setup, action, and assertions with only input and expected values changed.
- **Should** Give factories sensible defaults, named states, and relationship methods without hidden unconditional relations.
  - Why: Makes fixture conditions readable and prevents optional related records from appearing unexpectedly.
  - Flag when: An unconditional factory callback creates optional related records for every fixture.
- **Consider** Keep relevant fixture details visible in the test and extract big repeated setups into domain-named factory helpers.
  - Why: Keeps tests understandable locally while hiding cumbersome incidental fixture preparation behind meaningful scenario names.
  - Flag when: Helpers hide behavior-relevant fixture values, or tests repeat large manual relationship-creation graphs.
- **Should** Before refactoring, pin existing behavior with tests and keep them green through small steps; add new behavior separately.
  - Why: Existing tests catch unintended behavior changes while internal structure is rearranged.
  - Flag when: Refactoring weakens existing test expectations or mixes structural changes with new behavior.
- **Should** Cover success, alternate paths, invalid input, forbidden access, and edge cases in acceptance scenarios.
  - Why: Happy-path tests leave alternative inputs and failure branches unverified.
  - Flag when: New acceptance tests cover success but omit implemented validation failures or alternative branches.
- **Should** Organize suites so relevant tests run fast after changes, and always run the full suite in CI before delivery.
  - Why: Focused suites shorten feedback loops while broader tests verify interacting components.
  - Flag when: CI changes restrict test execution to one suite without a full-suite verification job.
- **Should** Use isolated test environments; match the production database engine for engine-specific behavior and use SQLite only where representative.
  - Why: SQLite can miss production-specific query behavior, and shared database state makes tests interfere.
  - Flag when: Engine-specific database tests use SQLite instead of the production engine.
- **Should** Use `--profile` to find and fix the slowest tests.
  - Why: Measured slow tests reveal bottlenecks that make the suite impractical to run locally.
- **Should** Use coverage reports to find untested code, not as a target in itself.
  - Why: Coverage reveals application code the test suite never executes without proving assertion quality.
- **Consider** Use mutation testing on critical code to expose weak assertions.
  - Why: Surviving mutations expose missing cases and weak assertions even when line coverage is complete.
- **Consider** Pin legacy user journeys with browser tests before and during a migration.
  - Why: Automated legacy journeys catch behavior changes when undocumented interfaces are replaced.
- **Should** Test UI components with representative and empty props, interactions, and rendered results, not just mounting.
  - Why: Mounting alone leaves prop rendering and user-visible interaction outcomes unverified.
  - Flag when: Component tests only mount or assert existence without checking rendered data or interaction outcomes.
- **Consider** Use small snapshot tests only where clearer than explicit assertions, and review every baseline change.
  - Why: Unchecked snapshots can preserve buggy output, while large artifacts hinder review and cause merge conflicts.
  - Flag when: New snapshots capture large output unrelated to the specific behavior under test.
- **Consider** Adopt Pest incrementally, keeping existing PHPUnit tests working.
  - Why: Pest runs existing PHPUnit tests, allowing adoption without rewriting the entire suite.
  - Flag when: Pest migration deletes existing PHPUnit tests without preserving their behavioral assertions.

## Frontend: Blade

- **Must** Escape all untrusted output with `{{ }}` and pass PHP data to JS via `Js::from()`/`@js`; never concatenate raw HTML or JS.
  - Why: Unescaped user content can execute scripts in administrator views, and manual JavaScript serialization breaks quoting.
  - Flag when: User-controlled values appear in raw Blade output or are concatenated into JavaScript expressions.
- **Should** Extract reusable markup and repeated utility classes into Blade components with explicit props.
  - Why: Reusable components remove repeated field markup and keep component-specific presentation work out of page controllers.
  - Flag when: Repeated form-field markup and styling are copied across templates instead of using a shared component.
- **Should** Use anonymous components with `@props` for simple templates and class components when PHP logic is needed.
  - Why: Anonymous components avoid unnecessary PHP classes for presentation-only primitives.
  - Flag when: New component class only returns a static view and supplies no data or behavior.
- **Should** Forward `$attributes` to the right element and merge default classes so callers can extend styling.
  - Why: Attribute forwarding preserves caller control, while class merging avoids duplicate class attributes.
  - Flag when: Component drops caller attributes or renders separate default and caller class attributes.
- **Should** Compose layouts and fields with slots and simple common-case props.
  - Why: Independent field pieces let callers rearrange markup and reuse controls without embedding every layout variation.
  - Flag when: Input component embeds many label, error, and layout variants instead of accepting composed slot content.
- **Should** Move substantial view preparation into view models, composers, or components; keep templates rendering prepared data.
  - Why: keeps shared presentation data available wherever its view is rendered.
  - Flag when: Blade templates repeat substantial data preparation or display formatting.
- **Should** Use semantic HTML and proper heading hierarchy; add ARIA only with the behavior it requires.
  - Why: gives assistive technology meaningful structure for navigating and understanding the interface.
  - Flag when: Clickable divs replace native controls, or heading levels follow styling instead of document hierarchy.
- **Should** Show errors beside their fields, move focus to failures, and announce status to assistive tech without relying on color.
  - Why: helps users identify and correct failures, including through assistive technology.
  - Flag when: Validation failures change only field colors without associated corrective messages.
- **Should** Use accessible modals that manage focus entry, trapping, restoration, and keyboard behavior.
  - Why: keeps keyboard focus inside the dialog and restores it when the dialog closes.
  - Flag when: Custom modal opens without focus management or keyboard handling.
- **Should** Give custom Alpine controls named hidden inputs and `x-modelable` values so native forms and bindings still work.
  - Why: lets custom control state submit with native forms and bind through the parent.
  - Flag when: Custom Alpine form control lacks a named hidden input or exposed modelable value.
- **Consider** Extend Blade only via supported directives/compiler hooks and inspect compiled views when debugging.
  - Why: avoids freezing per-render behavior into cached compiled PHP.
  - Flag when: Custom Blade compilation evaluates per-render values instead of emitting PHP that evaluates them at runtime.

## Frontend: Livewire

- **Must** Validate Livewire properties before persisting and authorize every referenced resource in actions.
  - Why: prevents invalid component input from being persisted.
  - Flag when: Livewire action persists editable properties without validation or accesses referenced resources without authorization.
- **Must** Mark server-owned IDs with `#[Locked]` and still authorize operations on them.
  - Why: prevents browser tampering with server-selected record identifiers.
  - Flag when: Server-selected public Livewire ID lacks client-write protection.
- **Must** When rendering components by name (e.g. modal hosts), allowlist the component names.
  - Why: restricts dynamic rendering to components intended as allowed modal content.
  - Flag when: Client-supplied component name reaches dynamic rendering without an allowlist check.
- **Should** Keep browser-only state in Alpine; use Livewire (`$wire`, PHP actions) only for server-dependent behavior.
  - Why: lets local UI interactions respond without a server round trip.
  - Flag when: PHP action only toggles modal visibility or another browser-only state.
- **Should** Use deferred `wire:model` for ordinary forms and `wire:model.live.debounce` for search-as-you-type.
  - Why: batches ordinary input changes until an action needs their values.
  - Flag when: Ordinary form inputs use live synchronization although intermediate values need no server processing.
- **Should** Put derived queries in `#[Computed]` methods, with explicit cache keys and invalidation when caching across requests.
  - Why: preserves derived query data that public model collections can lose during hydration.
  - Flag when: Public model collection stores custom joined query results needed across Livewire requests.
- **Should** Use `$this->dispatch()` and Alpine `x-on` listeners for UI reactions like closing a modal after save.
  - Why: lets successful server actions update local UI without a full-page refresh.
- **Should** Lazy-load expensive components (`#[Lazy]`) with a placeholder so they don't block initial render.
  - Why: lets the page or modal render promptly while expensive content loads.
  - Flag when: Initially mounted component performs expensive initialization synchronously without lazy loading or a placeholder.
- **Should** Build reusable wrappers with `#[Modelable]`, slots, and attribute forwarding.
  - Why: lets parents bind extracted inputs and supply content and attributes to reusable wrappers.
  - Flag when: Reusable wrapper discards caller attributes or extracted input lacks a modelable value.
- **Consider** Use single-file components for small components and multi-file directories when complexity grows.
  - Why: keeps related PHP, Blade, and JavaScript together as component complexity grows.
- **Consider** Use a shared, constrained component library (e.g. Flux) for consistent typography, variants, and dark mode.
  - Why: Shared defaults keep controls aligned and support dark mode without repeated styling decisions.

## Frontend: Inertia

- **Should** Return pages with `Inertia::render()` from ordinary routes and controllers, passing prepared props.
  - Why: Controller props supply initial page data without separate API fetches in component mount hooks.
  - Flag when: An Inertia page fetches its initial data from a separate API in its mount hook.
- **Must** Send only the attributes each page needs and never sensitive data; Inertia props are visible in the browser.
  - Why: Every returned prop is browser-visible, including unnecessary attributes and sensitive data.
  - Flag when: Inertia props explicitly include secrets or unrelated model attributes.
- **Should** Submit with Inertia's form helpers, validate in Laravel, redirect after success, and display returned errors.
  - Why: Server validation returns field errors while retaining inputs, and successful mutations redirect to a GET page.
  - Flag when: An Inertia form bypasses server validation or never displays returned field errors.
- **Should** Navigate with Inertia links/visits; preserve state and scroll for debounced query-string filters on the same page.
  - Why: Same-page filtering retains input focus and scroll position without full document reloads.
  - Flag when: Live query-string filtering sends every keystroke or omits state and scroll preservation.
- **Should** Use partial reloads and wrap expensive props in closures so omitted props aren't computed.
  - Why: Omitted props save response bytes and database work only when expensive queries are evaluated lazily.
  - Flag when: A partial reload omits a prop whose expensive query still executes eagerly.
- **Consider** Use deferred props with placeholders so cheap content renders first.
  - Why: Slow secondary data otherwise delays displaying inexpensive, useful page content.
  - Flag when: An initial dashboard response waits for slow secondary data before returning cheap content.
- **Should** Share only truly common, small data via `HandleInertiaRequests`.
  - Why: Shared data accompanies page responses, so unnecessary sharing adds payload and request costs.
  - Flag when: HandleInertiaRequests::share() loads large collections needed by only one page.
- **Should** Use `useRemember`/keyed `useForm` to restore unfinished drafts on Back, excluding passwords.
  - Why: Returning with Back restores unfinished inputs without carrying the draft into fresh visits.
  - Flag when: An editable draft uses only local state without useRemember or a keyed useForm.

## Frontend: Filament

- **Should** Generate a Filament resource per managed model, with fields and columns named after model attributes; rely on app validation and model policies.
  - Why: Model resources provide create, edit, and list interfaces with minimal application plumbing.
  - Flag when: A Filament resource field name mismatches the model attribute it edits.
- **Should** Use relationship selects and dot-notation columns instead of hand-assembling related data.
  - Why: Relationship bindings supply related choices and display values without manually assembling UI data.
  - Flag when: A Filament select manually queries relationship options instead of using relationship().
- **Should** Add select filters and `searchable()` columns instead of building custom search endpoints.
  - Why: Declarative filters and searchable columns provide table search without bespoke endpoints.
  - Flag when: A Filament table adds a custom endpoint for ordinary column search.
- **Should** Use record actions for single-record operations and bulk actions for the same operation on selections.
  - Why: Users can perform the same operation on one record or a selected group.
  - Flag when: A record action updates an unscoped model query instead of its current record.
- **Consider** Limit re-rendering in reactive forms to fields that actually depend on the change.
  - Why: Targeted rendering avoids re-rendering unaffected form fields after a state update.
  - Flag when: A reactive field re-renders the whole form when only one dependent field changes.
- **Consider** Use standalone Filament forms, tables, or actions inside Livewire pages when a full panel isn't needed.
  - Why: Existing Livewire pages can reuse Filament interfaces without adopting a full panel.

## Packages

- **Must** Keep runtime deps in `require` and tools/tests in `require-dev`/`autoload-dev`.
  - Why: Consumers receive runtime dependencies without inheriting package test tooling or test autoload mappings.
  - Flag when: composer.json puts test tools in require, test namespaces in autoload, or runtime dependencies in require-dev.
- **Must** Include a LICENSE file and matching `license` in `composer.json` for distributed packages.
  - Why: Without an explicit license, publishing source does not grant consumers reuse rights.
  - Flag when: A distributed package lacks LICENSE or has conflicting Composer license metadata.
- **Should** Start packages from a current skeleton (e.g. Spatie's package skeleton) with clear src/tests namespaces.
  - Why: A skeleton supplies package namespaces, Composer metadata, providers, and test setup before feature work.
- **Should** Implement package logic in plain services; register providers and optional facades via Composer auto-discovery.
  - Why: An optional facade proxies an ordinary service that consumers can also instantiate directly.
  - Flag when: Package business logic is implemented inside a facade instead of its underlying service.
- **Consider** Build reusable components inside the app first, then extract mature ones into packages (Composer path repositories).
  - Why: Mature, separable features can be reused and maintained independently while the application stays focused on its domain.
- **Should** Test framework-dependent packages with Orchestra Testbench, loading providers and migrations explicitly.
  - Why: Bare PHPUnit tests lack the Laravel services and database schema needed by framework-dependent package behavior.
  - Flag when: Framework-dependent package tests extend bare PHPUnit TestCase without bootstrapping Laravel.
- **Should** Follow semantic versioning and keep a changelog and upgrade guide.
  - Why: Predictable versions and migration instructions help consumers understand release changes and upgrade requirements.
- **Should** Document the install command, usage examples, and how to run tests and contribute.
  - Why: Consumers and future maintainers need usable instructions without knowing the package internals.
  - Flag when: Documented installation commands or usage examples contradict the changed package API.
- **Must** Verify Cashier webhook signatures, trust provider state for orders and subscriptions, and keep custom effects in listeners.
  - Why: Authenticated provider callbacks establish billing state while asynchronous checkout completion may still be pending.
  - Flag when: Checkout return handlers mark orders paid without provider confirmation, or webhook routes bypass signature verification.
- **Should** Configure Scout searchable fields explicitly, import existing records, and keep the index in sync.
  - Why: Irrelevant indexed attributes create noisy matches, while imports and synchronization keep searchable records current.
  - Flag when: Searchable attributes include irrelevant fields, or indexed model updates bypass Scout lifecycle events.
- **Consider** Use a maintained media package (e.g. Spatie Media Library) with purpose-specific collections and disks.
  - Why: A media package handles repeated upload, model association, derivative, and storage workflows.
- **Should** Define named media conversions and queue them unless needed immediately.
  - Why: Queued conversions move expensive image processing out of requests while named derivatives suit different display contexts.
  - Flag when: Conversions use nonQueued() without requiring immediate availability.
- **Consider** Use Spatie Laravel Data where it removes duplicated shapes, keeping validation explicit.
  - Why: One typed definition reduces repeated schemas across request validation, API responses, and frontend types.
- **Consider** Use scoped route discovery only for repetitive routes, keeping manual routes where control matters.
  - Why: Scoped discovery removes repetitive declarations while manual routes preserve deliberate control over important URLs.
  - Flag when: Route discovery scans all controllers or views instead of a dedicated directory.
- **Should** Keep API docs current: purpose, auth, parameters, validation, responses, and error examples.
  - Why: Code-derived documentation and explicit examples help consumers understand endpoint contracts beyond successful responses.
  - Flag when: Changed authentication, validation, or response behavior contradicts endpoint documentation or examples.

## Pending decisions (excluded for now)

Talks disagree on these; no rule is included until decided.

- Persistence coupling: independent domain entities or Eloquent-backed domains.
- Test isolation: collaborator-focused tests or request-boundary tests with real persistence.
- Event payload semantics: event-time values or participating model references.
- Validation location: model-centered rules or operation-specific validation.
- Universal TDD obligation: every kind of work or risk-based testing discipline.
- Form Requests: avoid them or use them for substantial validation.
- Client validation timing: early feedback or submission-only backend errors.
- Interactive frontend preference: JavaScript page components or Blade with Livewire and Alpine.
- Compromised-password policy: permit warnings or require rejection.
- Reusable-action validation: validate inside actions or require validated inputs.
- Workflow status: stored current status or event-history-derived state.

## Outdated (reference only)

Not rules — these no longer apply to Laravel 13 / current tooling.

- Use Gate::after to override earlier authorization decisions or enforce non-bypassable administrator constraints. *Why outdated:* Laravel 13 after-hook return values cannot override a non-null allow or deny, so mandatory constraints belong in the decision path. — [Chris Brown, Laracon EU 2019](https://www.youtube.com/watch?v=B2M3FCjRz6w&t=930s), [laravel.com/docs/13.x/authorization](https://laravel.com/docs/13.x/authorization)
