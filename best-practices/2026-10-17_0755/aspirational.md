# Aspirational practices

> **INTERIM snapshot (2026-10-17 22:50, Asia/Manila).** Claude collection chunks and the stricter adoption re-run are still pending. Levels and counts will change when they are folded in. See README.md.

Rules that scored at least Consider but that fewer than 30% of the applicable sample repos follow. They are kept out of the main list.

| Rule | Level by score | Score | Adoption (all) | First-party | L13 |
|---|---|---|---|---|---|
| Build test records with expressive factories | Must | 16 | 21% | 36% | L13-current |
| Use temporary authorized links for private downloads | Must | 15 | 17% |  | L13-current |
| Audit and remediate dependencies | Must | 15 | 7% | 0% | L13-current |
| Route testable delays through Sleep | Must | 15 | 12% | 25% | L13-current |
| Freeze time for time-dependent tests | Must | 14 | 14% | 29% | L13-current |
| Serve bounded stale results for expensive tolerant reads | Must | 12 | 0% |  | L13-current |
| Organize resource controllers around one resource and its conventional actions | Must | 12 | 29% | 80% | L13-current |
| Pass application data to actions and services rather than HTTP requests | Must | 12 | 0% |  | L13-current |
| Use PHPDoc for information native types cannot express | Must | 12 | 7% | 0% | L13-current |
| Use factory sequences for varying fixture attributes | Must | 12 | 25% |  | L13-current |
| Assert console command outcomes | Must | 12 | 18% | 14% | L13-current |
| Enable lazy-loading prevention outside production | Must | 11 | 0% | 0% | L13-current |
| Use to_route for named redirects | Must | 11 | 15% | 67% | L13-current |
| Add structured exception context | Must | 11 | 0% | 0% | L13-current |
| Back off transient job failures | Must | 11 | 0% |  | L13-current |
| Preserve key and value types in generics | Must | 11 | 5% | 0% | L13-current |
| Assert complete write-operation outcomes | Must | 11 | 17% | 67% | L13-current |
| Fake delays in tests | Must | 11 | 29% | 50% | L13-current |
| Eager-load relationships that a collection consumer will access | Must | 10 | 0% |  | L13-current |
| Process unbounded data sets with bounded chunks or lazy iteration | Must | 10 | 12% |  | L13-current |
| Encode output for its destination context | Must | 10 | 27% | 20% | L13-current |
| Assert rejected operations have no effects | Must | 10 | 17% | 67% | L13-current |
| Test validation through behavior | Must | 10 | 0% | 0% | L13-current |
| Annotate relationship return types with the required model generics | Must | 9 | 24% | 33% | L13-current |
| Use stable unique keys for looped Livewire markup | Must | 9 | 29% | 67% | L13-current |
| Capture output and failures from background schedules | Must | 9 | 0% |  | L13-current |
| Bound outbound connection and response times | Must | 9 | 0% |  | L13-current |
| Queue mail that does not require immediate feedback | Must | 9 | 17% |  | L13-current |
| Fake external HTTP integrations | Must | 9 | 20% |  | L13-current |
| Assert outbound request contracts | Must | 9 | 20% |  | L13-current |
| Separate mail content and delivery tests | Must | 9 | 0% |  | L13-current |
| Create only required test records | Must | 9 | 17% |  | L13-current |
| Defer providers that only register bindings when eager loading is unnecessary | Must | 8 | 29% |  | L13-current |
| Document structured arrays with PHPDoc shapes | Must | 8 | 0% | 0% | L13-current |
| Represent monetary amounts in integer minor units | Must | 8 | 20% |  | L13-current |
| Preserve active filters in pagination URLs | Must | 8 | 0% |  | L13-current |
| Return explicit command success and failure codes | Must | 8 | 11% | 21% | L13-current |
| Aggregate and check existence in SQL instead of loading full collections | Must | 8 | 0% |  | L13-current |
| Separate reusable component behavior from page inclusion | Must | 8 | 0% | 0% | L13-current |
| Fail tests on relevant deprecations | Must | 8 | 7% | 7% | L13-current |
| Validate fixed-choice input against an allowlist | Must | 8 | 22% |  | L13-current |
| Create dedicated failure exceptions with descriptive named constructors | Must | 7 | 29% | 40% | L13-current |
| Monitor cumulative database time per request | Must | 7 | 8% | 0% | L13-current |
| Use whereBelongsTo() when filtering by an existing belongsTo model | Must | 7 | 17% |  | L13-current |
| Scope static-analysis suppressions by error identifier and path, with a reason | Must | 7 | 29% | 0% | L13-current |
| Use single-action (invokable) controllers when an endpoint is clearer as one focused action | Must | 7 | 24% | 29% | L13-current |
| In a custom login flow, call $request->session()->regenerate() after a successful login to prevent session fixation | Must | 7 | 20% |  | L13-current |
| Timebox authentication-sensitive failure paths | Must | 7 | 0% | 0% | L13-current |
| Use trailing commas in multiline lists | Must | 7 | 23% | 27% | L13-current |
| Make queue side effects idempotent | Must | 7 | 11% |  | L13-current |
| Monitor critical scheduled job completion | Must | 7 | 0% |  | L13-current |
| Use the composed Foundation Queueable trait in new jobs | Must | 7 | 0% |  | L13-current |
| Rehash outdated passwords after successful verification | Must | 7 | 10% |  | L13-current |
| Provide one composer validation command for required checks | Must | 7 | 2% | 7% | L13-current |
| Exercise real backend semantics in dedicated integration tests | Must | 7 | 23% | 40% | L13-current |
| Test Livewire components through their testing API | Must | 7 | 0% | 0% | L13-current |
| Test generic type inference contracts | Must | 7 | 25% | 33% | L13-current |
| Use explicit JSON request helpers | Must | 7 | 25% | 57% | L13-current |
| Constrain every uploaded file before storage | Must | 7 | 0% |  | L13-current |
| Route PHP and dependency deprecation warnings to an intentional deprecation log channel | Should | 6 | 21% | 0% | L13-current |
| Set explicit HTTP timeouts in third-party SDKs that bypass Laravel’s HTTP client | Should | 6 | 25% |  | L13-current |
| Provide a guided package installation command | Should | 6 | 29% | 50% | L13-current |
| Limit eager-loaded columns to the data the consumer needs | Should | 6 | 0% |  | L13-current |
| Document collection and array element types | Should | 6 | 23% | 33% | L13-current |
| Use stable named parameters to clarify optional flags and skipped arguments | Should | 6 | 20% | 50% | L13-current |
| Handle expected alternative external responses explicitly when graceful degradation is required | Should | 6 | 29% |  | L13-current |
| Decode third-party JSON payloads with json_decode(..., flags: JSON_THROW_ON_ERROR) so malformed payloads fail loudly | Should | 6 | 0% |  | L13-current |
| Mark methods that override parent/framework methods with PHP's #[Override] attribute so signature drift is caught | Should | 6 | 0% | 0% | L13-updated |
| Bound CI dependency-install retries | Should | 6 | 20% | 0% | L13-current |
| Prevent unsafe scheduled task overlap | Should | 6 | 20% |  | L13-current |
| Set explicit external-operation time budgets | Should | 6 | 20% |  | L13-current |
| Label form fields explicitly and persistently | Should | 6 | 12% | 29% | L13-current |
| Strip unneeded queued model relationships | Should | 6 | 0% |  | L13-current |
| Test application security boundaries | Should | 6 | 14% | 40% | L13-current |
| Disable raw HTML in untrusted Markdown | Should | 6 | 14% |  | L13-current |
| Keep the nonsecret environment template complete | Should | 6 | 0% | 0% | L13-current |
| Fail loudly on discarded model attributes during development | Should | 6 | 0% | 0% | L13-current |
| Regenerate the session ID and CSRF token after authentication | Should | 6 | 20% |  | L13-current |
| Provide contract-compatible fakes for reusable service clients | Should | 6 | 17% | 33% | L13-current |
| Test reported exceptions through the exception fake | Should | 6 | 0% | 0% | L13-current |
| Fix random strings when asserting generated identifiers | Should | 6 | 17% |  | L13-current |
| Assert contractual serialization shape | Should | 6 | 11% | 25% | L13-current |
| Assert query budgets on critical paths | Should | 6 | 0% | 0% | L13-current |
| Annotate generic factory traits | Should | 6 | 8% | 33% | L13-current |
| Validate every nested payload value | Should | 6 | 22% |  | L13-current |
| Validate command-line inputs before side effects | Should | 6 | 0% | 0% | L13-current |
| Isolate multiple forms with named validation error bags | Should | 6 | 0% |  | L13-current |
| Cache CI dependency downloads with compatible cache keys | Should | 5 | 23% | 0% | L13-current |
| Type reusable scope builders with the appropriate model generic | Should | 5 | 9% | 33% | L13-current |
| Suppress fixture model events when their side effects are irrelevant to the test setup | Should | 5 | 0% |  | L13-current |
| Keep substantial provider boot methods organized by concern | Should | 5 | 27% | 100% | L13-current |
| Handle expected Livewire request failures such as expired sessions with a deliberate user recovery path | Should | 5 | 0% |  | L13-updated |
| Automate committed asset and style regeneration | Should | 5 | 14% | 0% | L13-current |
| Keep CI actions updated automatically | Should | 5 | 9% | 0% | L13-current |
| Keep application dependencies current | Should | 5 | 18% | 0% | L13-current |
| Suppress dependency lifecycle scripts in CI installs | Should | 5 | 9% | 0% | L13-current |
| Check configured Rector refactorings in CI | Should | 5 | 10% |  | L13-current |
| Write actionable structured production logs | Should | 5 | 9% |  | L13-current |
| Explain non-obvious implementation constraints | Should | 5 | 19% | 43% | L13-current |
| Require MFA for privileged accounts | Should | 5 | 12% |  | L13-current |
| Document public exception contracts | Should | 5 | 5% | 18% | L13-current |
| Make pagination order deterministic | Should | 5 | 0% |  | L13-current |
| Run local background workers with the development stack | Should | 5 | 0% |  | L13-current |
| Provide concise repeatable project onboarding commands | Should | 5 | 25% | 0% | L13-current |
| Mark secret function parameters sensitive | Should | 5 | 6% | 0% | L13-current |
| Separate and validate shell arguments before execution | Should | 5 | 17% | 0% | L13-current |
| Escape interpolated values before marking generated markup safe | Should | 5 | 0% | 0% | L13-current |
| Use ForwardsCalls for framework-style method delegation | Should | 5 | 29% | 29% | L13-current |
| Restrict dependency auto-merge automation to verified bot updates and required checks | Should | 5 | 0% |  | L13-current |
| Authenticate and constrain serialized data before decoding it | Should | 5 | 0% | 0% | L13-current |
| Escape LIKE metacharacters when input should match literally and verify the driver escape semantics | Should | 5 | 6% | 25% | L13-current |
| Adopt level seven when checking union safety | Should | 5 | 14% |  | L13-current |
| Pin formatter versions when style updates require review | Should | 5 | 12% | 0% | L13-current |
| Resolve swappable drivers through a checked manager | Should | 5 | 17% | 0% | L13-current |
| Test container binding types and promised lifetimes | Should | 5 | 4% | 10% | L13-current |
| Ignore generated browser screenshots | Should | 5 | 20% |  | L13-current |
| Control nondeterministic dependencies | Should | 5 | 21% | 36% | L13-current |
| Cover handled HTTP failure paths | Should | 5 | 11% | 25% | L13-current |
| Extract large reusable literal fixtures | Should | 5 | 0% | 0% | L13-current |
| Test uploads with fake files and isolated storage | Should | 5 | 17% |  | L13-current |
| Fail PHPUnit on warnings and risky tests | Should | 5 | 9% | 7% | L13-current |
| Simulate Octane request reset in lifecycle tests | Should | 5 | 0% |  | L13-updated |
| Refresh Testbench package discovery after autoload changes | Should | 5 | 28% | 60% | L13-current |
| Build parameterized middleware references through typed factories | Should | 5 | 15% | 33% | L13-current |
| Stream large filesystem uploads and close handles | Should | 5 | 29% |  | L13-current |
| Use concise Laravel conditional guards where equivalent | Should | 4 | 8% | 9% | L13-current |
| Serialize expensive cache fills when duplicate work is unacceptable | Should | 4 | 0% |  | L13-current |
| Preserve application data when clearing framework caches | Should | 4 | 25% |  | L13-current |
| Enable Eloquent strictness outside production | Should | 4 | 0% | 0% | L13-current |
| Use whereRelation() for a single simple related-column condition | Should | 4 | 20% |  | L13-current |
| Enable model-property checks and pass real attribute names to property-aware APIs | Should | 4 | 15% | 0% | L13-current |
| Use type-specific Config accessors when the value contract is known | Should | 4 | 0% |  | L13-current |
| Use withWhereHas() when parent filtering and eager loading must use the same relation predicate | Should | 4 | 0% |  | L13-current |
| Annotate Attribute accessors with get and set generic types | Should | 4 | 20% |  | L13-current |
| Cover the full authentication lifecycle in the audit trail | Should | 4 | 8% | 0% | L13-current |
| Use Blade class directives for conditional class lists | Should | 4 | 21% | 40% | L13-current |
| Type query and tap closure parameters with their actual argument types | Should | 4 | 19% | 0% | L13-current |
| Prefer session() for straightforward session value retrieval | Should | 4 | 23% | 0% | L13-current |
| Group routes by access requirements | Should | 4 | 28% | 50% | L13-current |
| Use $request->query() or $request->post() when it matters whether a value comes from the query string or the request body | Should | 4 | 0% |  | L13-current |
| Call ->withQueryString() on paginators so pagination links keep the current request's query parameters (filters, sorting) | Should | 4 | 0% |  | L13-current |
| Rotate production logs with bounded retention | Should | 4 | 12% |  | L13-current |
| Use oldest for ascending creation order | Should | 4 | 17% |  | L13-current |
| Build production releases from locked production dependencies | Should | 4 | 0% |  | L13-current |
| Show progress and totals for batch commands | Should | 4 | 10% | 0% | L13-current |
| Type declared application properties | Should | 4 | 23% | 13% | L13-current |
| Persist shareable result state in the URL | Should | 4 | 0% |  | L13-current |
| Reindex filtered or unique collections for JSON lists | Should | 4 | 23% | 0% | L13-current |
| Propagate execution context into job logs | Should | 4 | 14% |  | L13-current |
| Skip optional live-service tests clearly when credentials are absent | Should | 4 | 17% |  | L13-current |
| Reject known compromised passwords | Should | 4 | 29% | 75% | L13-current |
| Test identity and abuse checks against database collation equivalents | Should | 4 | 5% | 0% | L13-current |
| Use timeboxing for custom timing-sensitive credential checks | Should | 4 | 0% | 0% | L13-current |
| Support password managers with correct autocomplete values | Should | 4 | 0% |  | L13-current |
| Keep private files off public disks and authorize delivery | Should | 4 | 17% |  | L13-current |
| Use direct filter/reject predicates instead of double negation | Should | 4 | 0% | 0% | L13-current |
| Authorize every intentional bypass of tenant scoping | Should | 4 | 0% |  | L13-current |
| Use the direct complementary helper instead of negating its counterpart | Should | 4 | 0% | 0% | L13-current |
| Check missing dot-key translations | Should | 4 | 0% |  | L13-current |
| Name actions with a verb and domain object | Should | 4 | 23% | 40% | L13-current |
| Test insufficient permissions across roles | Should | 4 | 10% |  | L13-current |
| Use expressive timestamp-ordering helpers | Should | 4 | 27% | 75% | L13-current |
| Compose upload constraints with the File rule builder | Should | 4 | 0% |  | L13-current |
| Restrict URL input to supported schemes | Should | 4 | 0% |  | L13-current |
| Expose package version and enabled status in Artisan about | Consider | 3 | 20% | 33% | L13-current |
| Start CSP from an explicit restrictive baseline | Consider | 3 | 0% | 0% | L13-current |
| Pad timing-sensitive work to a measured duration | Consider | 3 | 0% | 0% | L13-current |
| Keep file-cache locks outside the disposable cache-data tree | Consider | 3 | 7% | 20% | L13-current |
| Use digit separators to make large numeric literals readable | Consider | 3 | 20% | 17% | L13-current |
| Put PHPStan’s result cache in a project-local ignored directory | Consider | 3 | 13% | 0% | L13-current |
| Limit recorded SQL text to a configured maximum length | Consider | 3 | 10% | 33% | L13-current |
| Log only selected necessary query context when bindings are required for diagnosis | Consider | 3 | 10% |  | L13-current |
| Group related scheduled tasks that share scheduling and locking settings | Consider | 3 | 20% |  | L13-current |
| Exclude internal and non-production pages from indexing | Consider | 3 | 25% | 17% | L13-current |
| Declare JSON schemas for editable dependency manifests | Consider | 3 | 9% | 27% | L13-current |
| Make multi-write pivot operations transactional | Consider | 3 | 14% |  | L13-current |
| Include critical dependencies in application health checks | Consider | 3 | 8% | 0% | L13-current |
| Move substantial page-data preparation into a focused view model | Consider | 3 | 25% |  | L13-current |
| Identify configuration-driven defaults in Artisan option help | Consider | 3 | 8% | 33% | L13-current |
| Avoid redundant casts for conventional created_at and updated_at timestamps | Consider | 3 | 29% |  | L13-current |
| Review loaded relationships as part of the model serialization exposure surface | Consider | 3 | 12% |  | L13-current |
| Inject action-specific Form Requests for resource writes | Consider | 3 | 29% | 40% | L13-current |
| Prefer query-local eager loading for relations that are not universally needed | Consider | 3 | 20% |  | L13-current |
| Keep Slack log alerts at an actionable severity | Consider | 3 | 0% | 0% | L13-current |
| Avoid authentication resolution in passive telemetry listeners | Consider | 3 | 12% | 33% | L13-updated |
| Preserve trusted rendered slots in published mail components | Consider | 3 | 0% |  | L13-current |
| Declare language-aware Git diff attributes | Consider | 3 | 9% | 20% | L13-current |
| Omit redundant new-expression parentheses on PHP 8.4+ | Consider | 3 | 17% |  | L13-current |
| Keep navigation keyboard-accessible and provide a skip link | Consider | 3 | 8% | 0% | L13-current |
| Precompile Blade views during production deployment | Consider | 3 | 17% |  | L13-current |
| Expose frontend quality checks through shared package scripts | Consider | 3 | 11% | 29% | L13-current |
| Cancel obsolete frontend requests and ignore stale responses | Consider | 3 | 14% |  | L13-current |
| Use resource routes for conventional resource-controller actions | Consider | 3 | 0% | 0% | L13-current |
| Treat permission checks and row-level access constraints as separate requirements | Consider | 3 | 29% |  | L13-current |
| Delay loading indicators to avoid flicker on fast Livewire requests | Consider | 3 | 17% |  | L13-current |
| Use finally blocks to clean up temporary state and acquired resources | Consider | 3 | 24% |  | L13-current |
| Mark optional normalized input nullable | Consider | 3 | 18% |  | L13-current |
| Stream very large responses rather than materializing the entire payload | Consider | 3 | 0% |  | L13-current |
| Include Blade-only static assets in the Vite build and reference their versioned URLs | Consider | 3 | 17% |  | L13-current |
| Use full-string JSON keys under the selected translation convention | Consider | 3 | 18% |  | L13-current |
| Use named Laravel arguments only with documented stability | Consider | 3 | 0% | 0% | L13-current |
| Add architecture checks for forbidden debugging calls and unsafe APIs | Consider | 3 | 23% | 20% | L13-current |
| Encrypt remote syslog delivery | Consider | 3 | 0% | 0% | L13-current |
| Audit Composer dependencies in delivery pipelines | Consider | 3 | 2% | 0% | L13-current |
| Group schedules with shared constraints | Consider | 3 | 0% |  | L13-current |
| Import the Laravel defer helper explicitly | Consider | 3 | 0% |  | L13-current |
| Flush plain streamed responses through the proxy | Consider | 3 | 17% |  | L13-current |
| Set meaningful exception log levels | Consider | 3 | 15% | 0% | L13-current |
| Use schedule:work for local scheduling | Consider | 3 | 12% |  | L13-current |
| Keep diagnostic tables compact with an explicit full-output option | Consider | 3 | 14% |  | L13-current |
| Reset consumed one-shot options on reusable operation objects | Consider | 3 | 29% | 33% | L13-current |
| Prohibit destructive custom commands in production | Consider | 3 | 0% |  | L13-current |
| Build query strings through URL helpers | Consider | 3 | 8% |  | L13-current |
| Use transform for deliberately shared in-place mutation | Consider | 3 | 0% |  | L13-current |
| Group related primitive parameters into typed values | Consider | 3 | 11% | 33% | L13-current |
| Choose immutable dates as the application default | Consider | 3 | 23% | 75% | L13-current |
| Expose package HTTP budgets through configuration | Consider | 3 | 0% | 0% | L13-current |
| Prevent duplicate link-table assignments | Consider | 3 | 0% |  | L13-current |
| Handle terminal chain failure | Consider | 3 | 20% |  | L13-current |
| Checkpoint interruptible long-running jobs | Consider | 3 | 0% |  | L13-updated |
| Schedule failed-job retention | Consider | 3 | 12% |  | L13-current |
| Stop validation chains after an earlier failure when appropriate | Consider | 3 | 0% | 0% | L13-current |
| Notify the UI after background work completes | Consider | 3 | 11% | 33% | L13-current |
| Monitor individual slow queries separately | Consider | 3 | 0% | 0% | L13-current |
| Revoke outstanding reset tokens after an authenticated password change | Consider | 3 | 15% | 25% | L13-current |
| Test authentication rejection after the attempt budget is exhausted | Consider | 3 | 0% | 0% | L13-current |
| Offer JSON output for inspection commands used by automation | Consider | 3 | 9% | 25% | L13-current |
| Pin immutable CDN assets with Subresource Integrity | Consider | 3 | 9% | 0% | L13-current |
| Set an explicit referrer policy appropriate to data sensitivity | Consider | 3 | 15% | 0% | L13-current |
| Disable unused browser capabilities through Permissions-Policy | Consider | 3 | 15% | 0% | L13-current |
| Use production password defaults with at least 12 characters and compromised-password checks | Consider | 3 | 17% | 33% | L13-current |
| Test reset links through faked reset notifications | Consider | 3 | 29% | 60% | L13-current |
| Clear temporary MFA setup state when setup closes or restarts | Consider | 3 | 0% | 0% | L13-current |
| Exclude passwords and secrets from flashed input | Consider | 3 | 8% |  | L13-current |
| Test CSRF protection outside the default test bypass when required | Consider | 3 | 25% |  | L13-current |
| Exercise guarded write paths outside factories and seeders | Consider | 3 | 18% | 0% | L13-current |
| Assert sensitive fields are absent from JSON responses | Consider | 3 | 0% | 0% | L13-current |
| Use strict equality or timing-safe comparison for digest strings | Consider | 3 | 0% |  | L13-current |
| Support Escape and click-away dismissal for dismissible overlays | Consider | 3 | 15% | 67% | L13-current |
| Preserve callback result types through helper APIs | Consider | 3 | 21% | 40% | L13-current |
| Name dataset cases | Consider | 3 | 16% | 12% | L13-current |
| Detect deprecated APIs before upgrades | Consider | 3 | 11% | 0% | L13-current |
| Avoid depending on Laravel parameter names | Consider | 3 | 7% | 0% | L13-current |
| Assert every supported browser and JSON response contract | Consider | 3 | 0% | 0% | L13-current |
| Exercise two-factor enrollment and recovery flows | Consider | 3 | 27% | 25% | L13-current |
| Test dashboard access defaults outside the local environment | Consider | 3 | 12% | 33% | L13-current |
| Arrange the referring page in redirect-back tests | Consider | 3 | 25% | 67% | L13-current |
| Exercise shipped environment-derived configuration defaults directly | Consider | 3 | 0% | 0% | L13-current |
| Provide a reproducible entrypoint for live-service integration tests | Consider | 3 | 6% | 0% | L13-current |
| Match intended HTTP endpoints in fakes | Consider | 3 | 8% | 0% | L13-current |
| Disable global sleep and exception-reporting side effects | Consider | 3 | 0% | 0% | L13-current |
| Test escaping for rendered untrusted text | Consider | 3 | 0% | 0% | L13-current |
| Test protected attributes against unexpected input | Consider | 3 | 0% | 0% | L13-current |
| Protect schema-coupled queries with integration tests | Consider | 3 | 26% | 50% | L13-current |
| Keep conditional validation declarative | Consider | 3 | 0% |  | L13-current |
| Test interactive UI states | Consider | 3 | 0% | 0% | L13-current |
| Inspect real spreadsheets for content tests | Consider | 3 | 0% |  | L13-current |
| Exercise middleware through real HTTP requests | Consider | 3 | 4% | 0% | L13-current |
| Route external commands through Laravel Process | Consider | 3 | 27% | 25% | L13-current |
| Test login outcomes and authentication state | Consider | 3 | 0% |  | L13-current |
| Model process failure results in tests | Consider | 3 | 20% |  | L13-current |
| Fake DNS validation lookups | Consider | 3 | 14% |  | L13-current |
| Order by related values inside the database | Consider | 3 | 20% |  | L13-current |
| Exercise upload validation limits with deliberate file properties | Consider | 3 | 0% |  | L13-current |
| Document the host type expected by a host-specific trait | Consider | 3 | 0% | 0% | L13-current |
| Validate external SDK response shape and batch cardinality before use | Consider | 3 | 14% | 25% | L13-current |
| Validate and encode identifiers used as outbound API path segments | Consider | 3 | 10% | 33% | L13-current |
| Reject oversized uploads in the browser before sending | Consider | 3 | 20% |  | L13-current |
| Batch existence checks for arrays of database values | Consider | 3 | 0% |  | L13-current |
| Localize validation messages including custom rules | Consider | 3 | 0% |  | L13-current |
| Handle JSON validation failures as field error responses | Consider | 3 | 18% |  | L13-current |
| Reject unknown fields for closed request contracts | Consider | 3 | 0% |  | L13-current |
| Make errors understandable without color or sight | Consider | 3 | 8% | 33% | L13-current |
| Type macro parameters and returns | Consider | 2 | 21% |  | L13-current |
| Implement accessible tab roles and keyboard behavior | Consider | 2 | 0% |  | L13-current |
| Provide alternatives to drag-only interactions | Consider | 2 | 0% |  | L13-current |
| Retain PHPStan's result cache between CI runs | Consider | 2 | 3% | 0% | L13-current |
| Check configuration value types with Larastan | Consider | 2 | 0% | 0% | L13-current |
| Parse complex package configuration into typed objects at a defined boundary | Consider | 2 | 17% | 0% | L13-current |
| Configure update automation for both Composer packages and GitHub Actions when both are used | Consider | 2 | 11% | 0% | L13-current |
| Release Alpine-owned listeners, observers and loops during component cleanup | Consider | 2 | 25% |  | L13-current |
| Scope CI jobs to relevant source and tool-configuration changes | Consider | 2 | 5% | 0% | L13-current |
| Centralize repeated package configuration reads behind typed accessors | Consider | 2 | 0% | 0% | L13-current |
| Block internal destinations in user-controlled URL fetches | Consider | 2 | 0% |  | L13-current |
| Enable missing-attribute checks to expose incomplete selects | Consider | 2 | 0% |  | L13-current |
| Detect silently discarded mass-assignment input | Consider | 2 | 0% | 0% | L13-current |
| Resolve HTTP/session services by bound classes or contracts instead of equivalent string container keys | Consider | 2 | 0% |  | L13-current |
| Guard asynchronous callbacks after component destruction | Consider | 2 | 12% |  | L13-current |
| Lint shell scripts on every pull request with (differential) ShellCheck at warning severity | Consider | 2 | 0% |  | L13-current |
| Do not auto-merge Dependabot pull requests by default; review each dependency update before merging | Consider | 2 | 9% | 0% | L13-current |
| Keep high-frequency local Livewire state changes client-side until a server round trip is needed | Consider | 2 | 29% |  | L13-current |
| Run the committed project code-style check before merge | Consider | 2 | 30% | 20% | L13-current |
| Include phpstan/phpstan-deprecation-rules in the PHPStan config to report usage of deprecated APIs | Consider | 2 | 11% | 0% | L13-current |
| Schedule dependency security audits | Consider | 2 | 2% | 0% | L13-current |
| Build package distribution assets during release instead of tracking them | Consider | 2 | 14% | 0% | L13-current |
| Cancel superseded validation runs | Consider | 2 | 7% | 0% | L13-current |
| Audit workflow configuration changes | Consider | 2 | 2% | 0% | L13-current |
| Publish PHPStan diagnostics as CI annotations | Consider | 2 | 29% | 0% | L13-current |
| Validate PSR-4 autoload mappings | Consider | 2 | 2% | 0% | L13-current |
| Report exceptions caught by console commands | Consider | 2 | 0% | 0% | L13-current |
| Bound payloads in exception messages | Consider | 2 | 0% |  | L13-current |
| Use native dialog behavior for modals | Consider | 2 | 0% |  | L13-current |
| Hide decorative icons from assistive technology | Consider | 2 | 10% | 0% | L13-current |
| Preserve action-button space beside flexible inputs | Consider | 2 | 25% | 0% | L13-current |
| Verify webhook authenticity | Consider | 2 | 25% |  | L13-current |
| Use static callbacks when no object binding is needed | Consider | 2 | 7% | 0% | L13-current |
| Use browser-compatible URL parsing for host allowlists | Consider | 2 | 0% |  | L13-current |
| Support keyboard interaction in dropdown menus | Consider | 2 | 0% | 0% | L13-current |
| Keep personalized responses out of shared caches | Consider | 2 | 0% |  | L13-current |
| Test the next PHP version without blocking supported builds | Consider | 2 | 8% | 17% | L13-current |
| Invalidate related caches from domain events | Consider | 2 | 0% |  | L13-current |
| Bound CI job runtime | Consider | 2 | 5% | 7% | L13-current |
| Exercise development versions of critical upstream dependencies | Consider | 2 | 3% | 0% | L13-current |
| Retain a redacted record of important integration exchanges | Consider | 2 | 0% |  | L13-current |
| Allowlist schemes before rendering untrusted navigation URLs | Consider | 2 | 0% | 0% | L13-updated |
| Validate URL hosts and network destinations beyond scheme sanitization | Consider | 2 | 0% |  | L13-current |
| Sanitize untrusted CSS colors and cast numeric style values | Consider | 2 | 0% | 0% | L13-updated |
| Register package gate defaults only when the application has not defined them | Consider | 2 | 0% |  | L13-current |
| Review incoming dependency changes for known vulnerabilities in CI | Consider | 2 | 2% | 0% | L13-current |
| Defer package Blade and Gate extension registration until service resolution | Consider | 2 | 11% |  | L13-current |
| Translate user-facing exception messages | Consider | 2 | 0% | 0% | L13-current |
| Normalize absent optional secret values before enabling security features | Consider | 2 | 6% | 0% | L13-current |
| Scope relationship aggregates to authorized related records | Consider | 2 | 0% |  | L13-current |
| Replace backtick shell execution | Consider | 2 | 0% | 0% | L13-current |
| Keep late-static construction safe | Consider | 2 | 17% | 0% | L13-current |
| Keep repeatable benchmarks for critical package code | Consider | 2 | 4% | 0% | L13-current |
| Run project checks through the intended container environment | Consider | 2 | 0% |  | L13-current |
| Distinguish omitted request fields from null values in tests | Consider | 2 | 25% |  | L13-current |
| Mark code references clearly in Pest descriptions | Consider | 2 | 0% | 0% | L13-current |
| Check browser accessibility in representative visual states | Consider | 2 | 20% |  | L13-current |
| Scope one-file Pest helpers locally | Consider | 2 | 7% |  | L13-current |
| Keep Pest behavior declarations before local fixtures | Consider | 2 | 25% |  | L13-current |
| Precompile shared Blade views before parallel tests | Consider | 2 | 15% | 20% | L13-current |
| Analyze test code as well as package source | Consider | 2 | 13% | 0% | L13-current |
| Skip irrelevant test jobs for documentation-only changes | Consider | 2 | 5% | 0% | L13-current |
| Generate matching tests from project generators | Consider | 2 | 0% |  | L13-current |
| Test multiple requests in a real long-lived worker | Consider | 2 | 0% | 0% | L13-current |
| Test publishable resources through vendor:publish | Consider | 2 | 0% | 0% | L13-current |
| Fail empty test-suite runs | Consider | 2 | 2% | 0% | L13-current |
| Collect all compatibility-matrix failures | Consider | 2 | 19% | 0% | L13-current |
| Fail unexpected test output | Consider | 2 | 7% | 0% | L13-current |
| Scope PHPUnit notices and warnings to first-party source deliberately | Consider | 2 | 0% | 0% | L13-current |
| Set translation fallback locale in fallback tests | Consider | 2 | 0% | 0% | L13-current |
| Apply strict static analysis to maintained code | Consider | 2 | 5% | 0% | L13-current |
| Preserve class-name factory return types with generics | Consider | 2 | 0% | 0% | L13-current |
| Test pipeline steps independently | Consider | 2 | 17% |  | L13-current |
| Test destructive operations against records owned by another user | Consider | 2 | 0% |  | L13-current |
| Use self-validating value objects as trusted input contracts | Consider | 2 | 20% |  | L13-current |
| Validate command argument formats before process execution | Consider | 2 | 17% | 0% | L13-current |
| Ship PHPStan extensions for package magic APIs | Consider | 2 | 9% | 0% | L13-current |
| Validate release versions and tag uniqueness before building | Consider | 2 | 20% |  | L13-current |
| Validate dynamically supplied class names against their contract | Consider | 2 | 29% |  | L13-current |
| Validate Composer metadata and lock consistency in CI | Consider | 2 | 0% | 0% | L13-current |
| Validate configuration objects at construction | Consider | 2 | 0% |  | L13-current |
