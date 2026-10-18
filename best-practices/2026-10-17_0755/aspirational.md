# Aspirational practices
Rules that scored at least Consider but that fewer than 30% of the applicable sample repos follow. They are kept out of the main list.

| Rule | Level by score | Score | Adoption (all) | First-party | L13 |
|---|---|---|---|---|---|
| Route testable delays through Sleep | Must | 17 | 27% | 50% | L13-current |
| Audit and remediate dependencies | Must | 15 | 5% | 0% | L13-current |
| Serve bounded stale results for expensive tolerant reads | Must | 13 | 0% |  | L13-current |
| Enable lazy-loading prevention outside production | Must | 12 | 8% | 0% | L13-current |
| Add structured exception context | Must | 12 | 0% | 0% | L13-current |
| Monitor critical scheduled job completion | Must | 11 | 0% |  | L13-current |
| Use concise Laravel conditional guards where equivalent | Must | 10 | 8% | 9% | L13-current |
| Retry only transient HTTP failures | Must | 10 | 20% |  | L13-current |
| Fail tests on relevant deprecations | Must | 10 | 14% | 33% | L13-current |
| Annotate relationship return types with the required model generics | Must | 9 | 24% | 33% | L13-current |
| Make promoted constructor properties readonly by default | Must | 9 | 9% | 0% | L13-current |
| Document structured arrays with PHPDoc shapes | Must | 8 | 11% | 22% | L13-current |
| Rotate production logs with bounded retention | Must | 8 | 14% | 0% | L13-current |
| Escape LIKE metacharacters when input should match literally and verify the driver escape semantics | Must | 8 | 20% |  | L13-current |
| Serialize expensive cache fills when duplicate work is unacceptable | Must | 7 | 0% |  | L13-current |
| Expose useful package diagnostics through Artisan about | Must | 7 | 29% | 22% | L13-current |
| Include critical dependencies in application health checks | Must | 7 | 8% | 0% | L13-current |
| Keep Slack log alerts at an actionable severity | Must | 7 | 0% | 0% | L13-current |
| Use the composed Foundation Queueable trait in new jobs | Must | 7 | 14% |  | L13-current |
| Fail loudly on discarded model attributes during development | Must | 7 | 0% | 0% | L13-current |
| Provide one composer validation command for required checks | Must | 7 | 19% | 33% | L13-current |
| Stream large filesystem uploads and close handles | Must | 7 | 27% |  | L13-current |
| Route PHP and dependency deprecation warnings to an intentional deprecation log channel | Should | 6 | 21% | 0% | L13-current |
| Provide a guided package installation command | Should | 6 | 29% | 50% | L13-current |
| Mark methods that override parent/framework methods with PHP's #[Override] attribute so signature drift is caught | Should | 6 | 0% | 0% | L13-updated |
| Keep application dependencies current | Should | 6 | 18% | 0% | L13-current |
| Layer per-IP and per-account limits for distributed login attacks | Should | 6 | 18% | 0% | L13-current |
| Resolve swappable drivers through a checked manager | Should | 6 | 17% | 0% | L13-current |
| Assert every supported browser and JSON response contract | Should | 6 | 21% | 25% | L13-current |
| Allowlist keys in structured request arrays | Should | 6 | 12% | 100% | L13-current |
| Cache CI dependency downloads with compatible cache keys | Should | 5 | 23% | 0% | L13-current |
| Enable Eloquent strictness outside production | Should | 5 | 0% | 0% | L13-current |
| Enable missing-attribute protection in development and tests | Should | 5 | 0% | 0% | L13-current |
| Type reusable scope builders with the appropriate model generic | Should | 5 | 10% |  | L13-current |
| Use type-specific Config accessors when the value contract is known | Should | 5 | 5% | 0% | L13-current |
| Return 201 Created for successful creates and 204 No Content (response()->noContent()) for deletes | Should | 5 | 29% |  | L13-current |
| Require MFA for privileged accounts | Should | 5 | 0% |  | L13-current |
| Reject TOTP replay | Should | 5 | 9% | 0% | L13-current |
| Consume recovery codes atomically and durably | Should | 5 | 10% | 0% | L13-current |
| Mark secret function parameters sensitive | Should | 5 | 27% | 25% | L13-current |
| Pin formatter versions when style updates require review | Should | 5 | 12% | 0% | L13-current |
| Represent stored callbacks with Closure or callable PHPDoc | Should | 5 | 27% | 20% | L13-current |
| Test listener behavior independently of event dispatch | Should | 5 | 29% | 40% | L13-current |
| Fail PHPUnit on warnings and risky tests | Should | 5 | 11% | 7% | L13-current |
| Build parameterized middleware references through typed factories | Should | 5 | 23% | 67% | L13-current |
| Restore consistent state after failed optimistic updates | Should | 4 | 29% |  | L13-current |
| Preserve application data when clearing framework caches | Should | 4 | 25% |  | L13-current |
| Use digit separators to make large numeric literals readable | Should | 4 | 26% | 17% | L13-current |
| Enable model-property checks and pass real attribute names to property-aware APIs | Should | 4 | 8% | 0% | L13-current |
| Annotate Attribute accessors with get and set generic types | Should | 4 | 20% |  | L13-current |
| Cover the full authentication lifecycle in the audit trail | Should | 4 | 20% |  | L13-current |
| Expose frontend quality checks through shared package scripts | Should | 4 | 25% | 29% | L13-current |
| Generate API documentation with Scribe | Should | 4 | 8% | 0% | L13-current |
| Add architecture checks for forbidden debugging calls and unsafe APIs | Should | 4 | 9% | 7% | L13-current |
| Use Pulse for application health diagnostics | Should | 4 | 15% | 0% | L13-current |
| Prohibit destructive custom commands in production | Should | 4 | 0% |  | L13-current |
| Check missing dot-key translations | Should | 4 | 0% |  | L13-current |
| Route external commands through Laravel Process | Should | 4 | 29% | 50% | L13-current |
| Analyze test code as well as package source | Should | 4 | 11% | 0% | L13-current |
| Reject conditionally forbidden nonempty fields | Should | 4 | 0% |  | L13-current |
| Expose package version and enabled status in Artisan about | Consider | 3 | 20% | 33% | L13-current |
| Start CSP from an explicit restrictive baseline | Consider | 3 | 0% | 0% | L13-current |
| Replace repeated type dispatch with polymorphic behavior | Consider | 3 | 0% |  | L13-current |
| Pad timing-sensitive work to a measured duration | Consider | 3 | 0% | 0% | L13-current |
| Keep file-cache locks outside the disposable cache-data tree | Consider | 3 | 7% | 20% | L13-current |
| Put PHPStan’s result cache in a project-local ignored directory | Consider | 3 | 13% | 0% | L13-current |
| Limit recorded SQL text to a configured maximum length | Consider | 3 | 10% | 33% | L13-current |
| Log only selected necessary query context when bindings are required for diagnosis | Consider | 3 | 10% |  | L13-current |
| Group related scheduled tasks that share scheduling and locking settings | Consider | 3 | 20% |  | L13-current |
| Exclude internal and non-production pages from indexing | Consider | 3 | 25% | 17% | L13-current |
| Declare JSON schemas for editable dependency manifests | Consider | 3 | 9% | 27% | L13-current |
| Make multi-write pivot operations transactional | Consider | 3 | 25% |  | L13-current |
| Identify configuration-driven defaults in Artisan option help | Consider | 3 | 12% | 33% | L13-current |
| Avoid authentication resolution in passive telemetry listeners | Consider | 3 | 29% | 67% | L13-updated |
| Prevent debug dumps from entering production code | Consider | 3 | 0% | 0% | L13-current |
| Preserve trusted rendered slots in published mail components | Consider | 3 | 17% |  | L13-current |
| Keep navigation keyboard-accessible and provide a skip link | Consider | 3 | 12% | 0% | L13-current |
| Enforce concurrent rate limits from atomic increments | Consider | 3 | 20% |  | L13-current |
| Use named Laravel arguments only with documented stability | Consider | 3 | 0% |  | L13-current |
| Redirect the SPA user to login after an expired session causes a 401 or 419 response | Consider | 3 | 11% | 0% | L13-current |
| Encrypt remote syslog delivery | Consider | 3 | 0% | 0% | L13-current |
| Audit Composer dependencies in delivery pipelines | Consider | 3 | 2% | 0% | L13-current |
| Flush plain streamed responses through the proxy | Consider | 3 | 14% |  | L13-current |
| Sample noisy diagnostic logs deliberately | Consider | 3 | 0% |  | L13-current |
| Lint changed files during local development | Consider | 3 | 9% | 0% | L13-current |
| Reject leftover debugging calls automatically | Consider | 3 | 2% | 0% | L13-current |
| Choose immutable dates as the application default | Consider | 3 | 23% | 75% | L13-current |
| Expose package HTTP budgets through configuration | Consider | 3 | 0% | 0% | L13-current |
| Run large data backfills outside schema migrations | Consider | 3 | 0% |  | L13-current |
| Stop validation chains after an earlier failure when appropriate | Consider | 3 | 8% | 0% | L13-current |
| Revoke outstanding reset tokens after an authenticated password change | Consider | 3 | 23% | 25% | L13-current |
| Offer useful password-strength feedback during entry | Consider | 3 | 15% | 0% | L13-current |
| Pin immutable CDN assets with Subresource Integrity | Consider | 3 | 25% |  | L13-current |
| Set an explicit referrer policy appropriate to data sensitivity | Consider | 3 | 17% | 0% | L13-current |
| Disable unused browser capabilities through Permissions-Policy | Consider | 3 | 17% | 0% | L13-current |
| Assert sensitive fields are absent from JSON responses | Consider | 3 | 25% | 20% | L13-current |
| Schedule cleanup of expired database reset tokens | Consider | 3 | 0% | 0% | L13-current |
| Review incoming dependency changes for known vulnerabilities in CI | Consider | 3 | 5% | 0% | L13-current |
| Group sequential prompts into a revisitable form | Consider | 3 | 0% |  | L13-current |
| Check the required filesystem entry type explicitly | Consider | 3 | 15% |  | L13-current |
| Exercise shipped environment-derived configuration defaults directly | Consider | 3 | 0% | 0% | L13-current |
| Provide a reproducible entrypoint for live-service integration tests | Consider | 3 | 6% | 0% | L13-current |
| Fake DNS validation lookups | Consider | 3 | 20% |  | L13-current |
| Fail browser smoke tests on unintended console logs | Consider | 3 | 0% |  | L13-current |
| Document the host type expected by a host-specific trait | Consider | 3 | 0% | 0% | L13-current |
| Validate external SDK response shape and batch cardinality before use | Consider | 3 | 14% | 25% | L13-current |
| Validate and encode identifiers used as outbound API path segments | Consider | 3 | 10% | 33% | L13-current |
| Order field rules to fail cheaply and deliberately | Consider | 3 | 8% | 0% | L13-current |
| Batch existence checks for arrays of database values | Consider | 3 | 29% |  | L13-current |
| Reject unknown fields for closed request contracts | Consider | 3 | 0% | 0% | L13-current |
| Reject booleans in coercive integer fields | Consider | 3 | 20% |  | L13-current |
| Make errors understandable without color or sight | Consider | 3 | 8% | 0% | L13-current |
| Show input requirements before validation fails | Consider | 3 | 23% | 0% | L13-current |
| Type macro parameters and returns | Consider | 2 | 21% |  | L13-current |
| Implement accessible tab roles and keyboard behavior | Consider | 2 | 25% |  | L13-current |
| Provide alternatives to drag-only interactions | Consider | 2 | 25% |  | L13-current |
| Retain PHPStan's result cache between CI runs | Consider | 2 | 3% | 0% | L13-current |
| Check configuration value types with Larastan | Consider | 2 | 0% | 0% | L13-current |
| Configure update automation for both Composer packages and GitHub Actions when both are used | Consider | 2 | 14% | 0% | L13-current |
| Use native lazy loading for eligible offscreen media | Consider | 2 | 18% |  | L13-current |
| Scope CI jobs to relevant source and tool-configuration changes | Consider | 2 | 16% | 0% | L13-current |
| Centralize repeated package configuration reads behind typed accessors | Consider | 2 | 13% | 11% | L13-current |
| Audit successful and failed authentication attempts | Consider | 2 | 17% | 0% | L13-current |
| Block internal destinations in user-controlled URL fetches | Consider | 2 | 0% |  | L13-current |
| Enable missing-attribute checks to expose incomplete selects | Consider | 2 | 0% | 0% | L13-current |
| Detect silently discarded mass-assignment input | Consider | 2 | 0% | 0% | L13-current |
| Use multipart POST with a PUT/PATCH method override when the server runtime cannot parse multipart updates | Consider | 2 | 20% |  | L13-current |
| Lint shell scripts on every pull request with (differential) ShellCheck at warning severity | Consider | 2 | 0% |  | L13-current |
| Include phpstan/phpstan-deprecation-rules in the PHPStan config to report usage of deprecated APIs | Consider | 2 | 11% | 0% | L13-current |
| Include the acting user, URL, route, and request time in audit records | Consider | 2 | 20% |  | L13-current |
| Schedule dependency security audits | Consider | 2 | 2% | 0% | L13-current |
| Build package distribution assets during release instead of tracking them | Consider | 2 | 14% | 0% | L13-current |
| Cancel superseded validation runs | Consider | 2 | 20% | 0% | L13-current |
| Audit workflow configuration changes | Consider | 2 | 2% | 0% | L13-current |
| Validate PSR-4 autoload mappings | Consider | 2 | 2% | 0% | L13-current |
| Alert when production debug mode is enabled | Consider | 2 | 8% | 0% | L13-current |
| Report exceptions caught by console commands | Consider | 2 | 12% | 0% | L13-current |
| Bound payloads in exception messages | Consider | 2 | 14% |  | L13-current |
| Associate form instructions with their input | Consider | 2 | 8% | 0% | L13-current |
| Provide controlled account containment and recovery | Consider | 2 | 0% | 0% | L13-current |
| Use static callbacks when no object binding is needed | Consider | 2 | 7% | 0% | L13-current |
| Support keyboard interaction in dropdown menus | Consider | 2 | 21% | 75% | L13-current |
| Test the next PHP version without blocking supported builds | Consider | 2 | 7% | 13% | L13-current |
| Bound CI job runtime | Consider | 2 | 14% | 7% | L13-current |
| Exercise development versions of critical upstream dependencies | Consider | 2 | 2% | 0% | L13-current |
| Queue persisted event identifiers | Consider | 2 | 20% |  | L13-current |
| Alert on suspicious repeated account login failures | Consider | 2 | 7% | 0% | L13-current |
| Consume single-use secrets atomically | Consider | 2 | 9% | 0% | L13-current |
| Defer package Blade and Gate extension registration until service resolution | Consider | 2 | 29% | 33% | L13-current |
| Analyze the supported PHP version range explicitly | Consider | 2 | 3% | 0% | L13-current |
| Keep late-static construction safe | Consider | 2 | 24% | 0% | L13-current |
| Keep repeatable benchmarks for critical package code | Consider | 2 | 8% | 0% | L13-current |
| Check browser accessibility in representative visual states | Consider | 2 | 17% |  | L13-current |
| Precompile shared Blade views before parallel tests | Consider | 2 | 20% |  | L13-current |
| Skip irrelevant test jobs for documentation-only changes | Consider | 2 | 12% | 0% | L13-current |
| Test multiple requests in a real long-lived worker | Consider | 2 | 7% | 17% | L13-current |
| Fail empty test-suite runs | Consider | 2 | 2% | 0% | L13-current |
| Collect all compatibility-matrix failures | Consider | 2 | 21% | 0% | L13-current |
| Fail unexpected test output | Consider | 2 | 11% | 0% | L13-current |
| Apply strict static analysis to maintained code | Consider | 2 | 3% | 0% | L13-current |
| Validate command argument formats before process execution | Consider | 2 | 17% | 0% | L13-current |
| Express required configuration as maintained executable checks | Consider | 2 | 0% | 0% | L13-current |
| Validate Composer metadata and lock consistency in CI | Consider | 2 | 2% | 0% | L13-current |
