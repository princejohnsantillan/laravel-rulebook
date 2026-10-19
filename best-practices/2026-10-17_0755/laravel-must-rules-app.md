# Laravel Must rules for application code
Version 2026-10-17_0755. Target: Laravel 13. A subset of `laravel-best-practices.md`: **Must rules only, and only those that apply to Laravel application code** (including running and deploying an app: queues, caching, configuration, infrastructure). Rules that only matter when authoring a Composer package are left out and listed in the appendix.

**407 rules** (of the 413 Must rules in this version; 6 package-only rules excluded).

**What Must means here:** a rule scored 7 or more points from its sources and adoption (official docs, first-party code and Laravel Boost 3; packages and linters 2; Laracon 2, or 3 for a Laravel-employee speaker; teaching, blogs and community 1; plus up to +2 for adoption in the 44-repo sample), and Claude (Opus, xhigh) then confirmed in its Must check that it is worth blocking a pull request over. Rules followed by under 30% of the sample (aspirational) and rules not tagged L13-current or L13-updated are not included.

**How the app/package split was made:** a rule-based first pass, then Claude (Opus, high effort) reviewed every one of the 413 rules. Rules that apply to both apps and packages are included.

## Must (407)

### Architecture

#### Make automated Artisan calls noninteractive
- **Do:** Pass --no-interaction and all required explicit options in unattended Artisan invocations.
- **Flag when:** Flag when CI scripts or agent instructions invoke prompting Artisan commands without noninteractive mode or required options.
- **Why:** Unattended runs must complete or fail predictably instead of waiting for input.
- **Enforced by:** manual

Before:
```php
// CI: php artisan make:model Invoice
```

After:
```php
# deploy script
php artisan migrate --force --no-interaction
```

#### Provide recovery-code management for two-factor authentication
- **Do:** Provide authenticated, authorized UI flows to view and regenerate recovery codes and enable/disable two-factor authentication.
- **Flag when:** Flag when two-factor authentication is offered without a recovery-code management path or without access protection on those operations.
- **Why:** Users need a secure recovery path when their second factor is unavailable.
- **Enforced by:** manual

Before:
```php
// Settings expose only an enable-2FA button.
```

After:
```php
// Settings include protected enable/disable and show/regenerate recovery-code actions.
```

#### Use Unicode-aware string operations for text
- **Do:** Use suitable Laravel Str helpers for UTF-8 text, falling back to mb_* operations when no suitable helper exists.
- **Flag when:** Flag when user-facing UTF-8 text is measured, truncated or case-converted with byte-oriented strlen, substr or strtolower.
- **Why:** Byte operations can split encoded characters or produce incorrect lengths and casing.
- **Enforced by:** manual

Before:
```php
$preview = substr($title, 0, 40);
```

After:
```php
$preview = mb_substr($title, 0, 40, "UTF-8");
```

#### Detect inherited and nested trait use
- **Do:** Use class_uses_recursive() when behavior detection must include traits used by parent classes or other traits.
- **Flag when:** Flag when class_uses() is used as an inheritance-aware capability check and misses inherited or nested traits.
- **Why:** The shallow PHP function does not cover the complete trait composition tree.
- **Enforced by:** manual

Before:
```php
$audited = in_array(Auditable::class, class_uses($model), true);
```

After:
```php
$audited = in_array(Auditable::class, class_uses_recursive($model), true);
```

#### Replace legacy Laravel global helpers with Arr and Str
- **Do:** Replace removed Laravel array/string helper globals with their Illuminate\Support\Arr or Str equivalents; do not replace unrelated native PHP functions.
- **Flag when:** Flag when code calls legacy helpers such as array_get, array_dot, str_slug or camel_case, or installs compatibility globals solely to retain them.
- **Why:** Supported Arr/Str APIs avoid removed helpers and reduce compatibility dependencies.
- **Enforced by:** rector:FuncCallToStaticCallRector

Before:
```php
$slug = str_slug($title);
$value = array_get($data, "name");
```

After:
```php
$slug = Str::slug($title);
$value = Arr::get($data, "name");
```

#### Protect destructive production commands with confirmation
- **Do:** Use ConfirmableTrait::confirmToProceed() before destructive side effects and return failure if confirmation is declined. For new Laravel 13 commands, declare the explicit --force option in the class-level Signature attribute; retained legacy $signature declarations remain supported.
- **Flag when:** Flag destructive production commands without confirmation or an explicit force option, or commands that continue to side effects after confirmToProceed() returns false.
- **Why:** Laravel's production confirmation convention gives operators a predictable safeguard.
- **Enforced by:** manual

Before:
```php
public function handle(): int { $this->wipe(); return self::SUCCESS; }
```

After:
```php
use App\Actions\WipeDemoData;
use Illuminate\Console\Attributes\Signature;
use Illuminate\Console\Command;
use Illuminate\Console\ConfirmableTrait;

#[Signature('demo:wipe {--force}')]
class WipeDemo extends Command
{
    use ConfirmableTrait;

    public function handle(WipeDemoData $wipe): int
    {
        if (! $this->confirmToProceed()) {
            return self::FAILURE;
        }

        $wipe->handle();
        return self::SUCCESS;
    }
}
```

### Caching

#### Use cache-aside reads that preserve falsy values
- **Do:** Use Cache::remember() with a suitable TTL to populate a cache miss, and treat only null as a miss in equivalent manual code.
- **Flag when:** Flag when a cache-aside read treats a falsy cached value as a miss (truthiness test on Cache::get()) instead of using Cache::remember() or an explicit null/has() check.
- **Why:** False, zero, empty strings, and empty arrays can be valid cached results. Cache::remember() uses a null check and centralizes retrieval and population; it does not prevent concurrent recomputation.
- **Enforced by:** manual

Before:
```php
if (! $count = Cache::get('active-users')) {
    $count = User::where('active', true)->count();
    Cache::put('active-users', $count, 300);
}
```

After:
```php
$count = Cache::remember('active-users', 300,
    fn () => User::where('active', true)->count());
```

#### Use owned atomic locks for exclusive critical sections
- **Do:** Protect exclusive operations with Cache::lock() and release the owned lock through a callback or finally block.
- **Flag when:** Flag when code uses an ordinary cache key (has/add/put + forget) as a mutex, or acquires a Cache::lock() in one scope and releases it without a get()/block() callback or finally block. Do not flag locks deliberately left to expire or handed to another process via owner()/restoreLock().
- **Why:** An ordinary key lacks ownership-aware acquisition and release. Owned locks prevent one process from deleting another process's lock and callbacks release locks when work throws.
- **Enforced by:** manual

Before:
```php
if (Cache::add('billing-lock', true, 60)) {
    billCustomer();
    Cache::forget('billing-lock');
}
```

After:
```php
Cache::lock('billing-lock', 60)->block(5, function () {
    billCustomer();
});
```

#### Define a bounded freshness policy for cached data
- **Do:** Give cached data a TTL within its acceptable staleness or a complete mutation-driven invalidation path; give derived entries explicit bounded lifetimes and track related keys for grouped invalidation.
- **Flag when:** Flag when mutable or derived data is cached forever without complete invalidation, a cache() write omits its intended duration, or related derived entries survive a source-data change.
- **Why:** An explicit lifetime or invalidation policy bounds stale results. Tracking related keys prevents partial invalidation from leaving inconsistent derived values.
- **Enforced by:** manual

Before:
```php
Cache::forever('product:'.$id.':summary', $summary);
```

After:
```php
Cache::put('product:'.$id.':summary', $summary, now()->addMinutes(5));
// In the committed product-change handler:
foreach ($derivedKeys as $key) { Cache::forget($key); }
```

#### Restrict classes restored from persisted cache payloads
- **Do:** Prefer scalar or array cache payloads; when objects are intentional, explicitly allow-list every class in the object graph through cache.serializable_classes instead of enabling arbitrary class restoration.
- **Flag when:** Flag when persisted cache or Livewire computed-property values contain objects outside an explicit allow-list, or cache.serializable_classes is true.
- **Why:** Laravel 13 defaults new applications to disallowing object restoration from cache. An explicit payload or class policy avoids incomplete objects and reduces unsafe deserialization exposure.
- **Enforced by:** manual

Before:
```php
// config/cache.php
return ['serializable_classes' => true];
// In the consumer:
Cache::put($key, $dto, 300);
```

After:
```php
// config/cache.php
return ['serializable_classes' => false];
// In the consumer:
Cache::put($key, $dto->toArray(), 300);
// For intentional objects, configure an exhaustive class allow-list instead.
```

#### Use an atomic write when a key must be absent
- **Do:** Use Cache::add() for a write that must succeed only when the key is absent.
- **Flag when:** Flag when a write-if-absent operation checks has() or get() before calling put().
- **Why:** Separate existence and write operations race under concurrency. add() expresses the atomic operation and returns whether it stored the value.
- **Enforced by:** manual

Before:
```php
if (! Cache::has($key)) {
    Cache::put($key, $value, 60);
}
```

After:
```php
$inserted = Cache::add($key, $value, 60);
```

#### Share a supported lock store across participating servers
- **Do:** Configure a lock-capable cache store shared by every participating server for distributed locks, Cache::funnel(), isolated commands, scheduler single-server tasks, and distributed backup locks.
- **Flag when:** Flag when these operations resolve to a store without the required lock support, or participating nodes use separate array, local-file, local-database, or node-local cache backends.
- **Why:** Coordination works only when every contender observes the same lock namespace and the store supports the required lock operations. Choose the actual store used by each subsystem, including a scheduler store selected with Schedule::useCache().
- **Enforced by:** manual

#### Choose an Octane server that supports the required features
- **Do:** Choose Swoole or a supported Swoole-compatible server when using Octane concurrent tasks, ticks, the Octane cache, or Swoole tables.
- **Flag when:** Flag when application code uses these server-specific features while OCTANE_SERVER or the start command selects an incompatible server.
- **Why:** Octane server implementations expose different capabilities. Selecting a server from the features actually used prevents runtime incompatibility.
- **Enforced by:** manual

#### Keep authoritative lookup data outside disposable caches
- **Do:** Persist non-reconstructible lookup results and tenant-to-shard mappings in durable storage, and make ordinary cache misses recover from an authoritative source.
- **Flag when:** Flag when flushing or losing a cache permanently loses required lookup or routing data, or a normal miss leaves the application unable to function despite an available primary source.
- **Why:** Cache entries can disappear through expiry, eviction, outages, or failover. Data that must never be lost needs a durable source of truth separate from the cache.
- **Enforced by:** manual

Before:
```php
Cache::forever('tenant:'.$tenantId.':shard', $shard); // Only copy
```

After:
```php
TenantShard::updateOrCreate(['tenant_id' => $tenantId], ['shard' => $shard]);
Cache::put('tenant:'.$tenantId.':shard', $shard, 3600);
// Misses resolve the mapping from TenantShard.
```

#### Supervise Pulse Redis-ingest workers
- **Do:** Run pulse:work under a process monitor whenever Pulse uses Redis ingestion.
- **Flag when:** Flag when Pulse Redis ingestion is enabled but pulse:work is absent from supervised production processes or lacks automatic restart.
- **Why:** The worker transfers queued telemetry into persistent storage. An unsupervised stopped worker leaves ingestion unprocessed.
- **Enforced by:** manual

### Code style & static analysis

#### Limit localized number formatting to presentation
- **Do:** Use Number helpers for localized display values and keep stored values and calculations in their numeric or money representation.
- **Flag when:** Flag when a localized Number::format() or currency() result is persisted or used in arithmetic.
- **Why:** Localized separators and currency symbols are presentation text and can corrupt calculations or storage.
- **Enforced by:** manual

Before:
```php
$total = Number::format($amount) + $tax;
```

After:
```php
$total = $amount + $tax;
$displayTotal = Number::format($total);
```

#### Generate storage names for untrusted uploads
- **Do:** Let Laravel generate storage filenames for untrusted uploads; derive extensions from validated content and keep original names only as metadata.
- **Flag when:** Flag when stored paths use getClientOriginalName() or getClientOriginalExtension() from untrusted uploads.
- **Why:** Client-supplied names and extensions can cause collisions or unsafe serving.
- **Enforced by:** manual

Before:
```php
$file->storeAs('uploads', $file->getClientOriginalName());
```

After:
```php
$file->store('uploads');
```

#### Use blank and filled with intentional value semantics
- **Do:** Prefer blank()/filled() for application emptiness checks when their semantics match the requirement; explicitly preserve behavior for values such as 0 and false.
- **Flag when:** Flag when empty() unintentionally treats valid zero/false values as missing, or a mechanical helper replacement changes required semantics.
- **Why:** Laravel’s helpers differ from empty() on valid falsy values and whitespace; replacements need intent rather than syntax-only rewriting.
- **Enforced by:** partial: rector:EmptyToBlankAndFilledFuncRector

Before:
```php
if (empty($quantity)) { throw new InvalidArgumentException; } // zero is valid
```

After:
```php
if (blank($quantity)) { throw new InvalidArgumentException; }
```

#### Remove debug calls before committing
- **Do:** Remove debugging calls such as dd(), dump(), var_dump(), and their equivalents from maintained runtime code before committing.
- **Flag when:** Flag when runtime application code contains leftover debugging calls that print data or terminate execution.
- **Why:** Debug calls can interrupt requests or expose internal data.
- **Enforced by:** partial: rector:RemoveDumpDataDeadCodeRector

Before:
```php
dd($order);
```

After:
```php
// Continue using the validated order without a debug dump.
```

#### Restore temporary global state even when callbacks fail
- **Do:** Restore previous temporary state in finally, or use scoped APIs such as Number::withLocale() and withCurrency().
- **Flag when:** Flag when telemetry, locale, currency, or other shared overrides are reset only after successful callbacks.
- **Why:** An exception must not leak temporary settings into later work.
- **Enforced by:** manual

Before:
```php
$old = Number::defaultLocale();
Number::useLocale('de');
$work();
Number::useLocale($old);
```

After:
```php
Number::withLocale('de', fn () => $work());
```

#### Keep classes compatible with PSR-4 autoloading
- **Do:** Match class namespaces, filenames, and directory case to the declared Composer PSR-4 mappings.
- **Flag when:** Flag when a class namespace or filename cannot be resolved correctly from its PSR-4 prefix on a case-sensitive filesystem.
- **Why:** Correct mappings keep autoloading reliable across development and deployment platforms.
- **Enforced by:** partial: pint:psr_autoloading

Before:
```php
// app/Models/user.php
namespace App\Models; class User {}
```

After:
```php
// app/Models/User.php
namespace App\Models; class User {}
```

#### Respect internal API boundaries
- **Do:** Avoid calling @internal framework or package symbols from application code outside their supported namespace boundary.
- **Flag when:** Flag when application code references a symbol marked @internal outside the permitted package namespace.
- **Why:** Internal APIs may change without a public compatibility promise.
- **Enforced by:** partial: larastan:PHPStan\Rules\InternalTag\RestrictedInternalClassConstantUsageExtension, partial: larastan:PHPStan\Rules\InternalTag\RestrictedInternalClassNameUsageExtension, partial: larastan:PHPStan\Rules\InternalTag\RestrictedInternalFunctionUsageExtension, partial: larastan:PHPStan\Rules\InternalTag\RestrictedInternalMethodUsageExtension, partial: larastan:PHPStan\Rules\InternalTag\RestrictedInternalPropertyUsageExtension

Before:
```php
$name = \Illuminate\Support\enum_value($value);
```

After:
```php
$name = $value instanceof BackedEnum ? $value->value : $value;
```

#### Call accessible symbols with compatible arguments
- **Do:** Call existing concrete constructors, functions, methods, and callbacks with valid argument names, counts, types, and by-reference semantics.
- **Flag when:** Flag when a call targets an unresolved or inaccessible callable, instantiates an abstract type, or supplies arguments incompatible with its signature.
- **Why:** Checking the full callable contract prevents runtime errors at application boundaries.
- **Enforced by:** larastan:PHPStan\Rules\Classes\InstantiationRule, larastan:PHPStan\Rules\Functions\CallToFunctionParametersRule, larastan:PHPStan\Rules\Functions\DefineParametersRule, larastan:PHPStan\Rules\Methods\CallMethodsRule, larastan:PHPStan\Rules\Methods\CallStaticMethodsRule, larastan:PHPStan\Rules\Functions\CallCallablesRule

Before:
```php
function send(User $user): void {}
send(123);
```

After:
```php
function send(User $user): void {}
send(User::findOrFail(123));
```

#### Throw exception objects
- **Do:** Throw Throwable instances and convert error strings or other payloads into meaningful exception objects.
- **Flag when:** Flag when a throw expression can receive a non-Throwable value.
- **Why:** Callers can catch meaningful exception types and receive consistent error details.
- **Enforced by:** partial: larastan:PHPStan\Rules\Exceptions\ThrowExprTypeRule

Before:
```php
throw 'Order failed';
```

After:
```php
throw new RuntimeException('Order failed');
```

#### Preserve inherited method contracts
- **Do:** Keep overriding method visibility, staticness, parameter/return variance, final restrictions, and PHPDoc/purity contracts compatible with the inherited API.
- **Flag when:** Flag when an override narrows accepted inputs, widens outputs, weakens visibility, changes staticness, violates final restrictions, or breaks an inherited purity promise.
- **Why:** Code using a base interface must remain safe when given any implementing subtype.
- **Enforced by:** larastan:PHPStan\Rules\Methods\OverridingMethodRule, larastan:PHPStan\Rules\Methods\MethodSignatureRule

Before:
```php
// Base accepts Animal.
public function handle(Dog $animal): Animal {}
```

After:
```php
public function handle(Animal $animal): Dog {}
```

#### Use valid delimited regular expressions
- **Do:** Write preg_match-compatible patterns with balanced valid delimiters and supported modifiers, including regex and not_regex validation rules.
- **Flag when:** Flag when a literal pattern cannot compile or a validation pattern lacks delimiters.
- **Why:** Invalid patterns fail validation or runtime matching instead of enforcing the intended rule.
- **Enforced by:** partial: larastan:PHPStan\Rules\Regexp\RegularExpressionPatternRule

Before:
```php
'email' => ['regex:^.+@.+$'],
```

After:
```php
'email' => ['regex:/^.+@.+$/i'],
```

#### Describe MCP tool effects with accurate annotations
- **Do:** Annotate MCP tools with supported read-only, destructive, idempotent, and open-world metadata matching their behavior.
- **Flag when:** Flag when tool effect metadata contradicts the implementation or omits effect information required by the project’s clients.
- **Why:** Effect metadata helps clients plan execution; it does not enforce authorization.
- **Enforced by:** manual

Before:
```php
#[IsReadOnly]
class DeleteInvoice extends Tool {}
```

After:
```php
#[IsDestructive]
class DeleteInvoice extends Tool {}
```

#### Expose the public disk through its storage link
- **Do:** When serving the local public disk directly, create the configured public/storage link to storage/app/public during deployment.
- **Flag when:** Flag when local public-disk URLs are generated but the public/storage link is missing or points to the wrong root.
- **Why:** The link exposes only the intended public filesystem root to the web server.
- **Enforced by:** manual

#### Keep commands in discoverable directories
- **Do:** Place application commands in app/Console/Commands by default; register any intentional alternate command directories explicitly.
- **Flag when:** Flag when a command is outside the scanned directory and no withCommands() or equivalent registration exposes it.
- **Why:** Conventional placement makes commands discoverable without extra registration.
- **Enforced by:** manual

Before:
```php
// app/Operations/SyncUsers.php; no alternate directory registration
namespace App\Operations; class SyncUsers extends Command {}
```

After:
```php
// app/Console/Commands/SyncUsers.php
namespace App\Console\Commands; class SyncUsers extends Command {}
```

### Configuration & container

#### Read environment variables in configuration files and read those settings through config() in application code
- **Do:** Read environment variables in configuration files and read those settings through config() in application code.
- **Flag when:** Flag when application code outside configuration files calls env() for application settings.
- **Why:** Cached configuration bypasses .env loading, so direct application env() calls can return different values after deployment.
- **Enforced by:** larastan:NoEnvCallsOutsideOfConfigRule, shift:workbench/convert-env-to-config

Before:
```php
$key = env('PAYMENT_KEY');
```

After:
```php
// config/services.php
'payment' => ['key' => env('PAYMENT_KEY')];
// Application code
$key = config('services.payment.key');
```

#### Bind variable boundary contracts to concrete implementations in a service provider’s register() method
- **Do:** Bind each injected boundary interface to its implementation centrally — in a service provider's register() method or with the container's #[Bind] attribute on the interface — instead of choosing implementations inside consumers.
- **Flag when:** Flag when an interface is type-hinted for injection without any container binding (provider binding or #[Bind]), or when consumers instantiate concrete boundary implementations directly instead of depending on the bound contract.
- **Why:** Central bindings separate infrastructure selection from application behavior.
- **Enforced by:** manual

Before:
```php
$gateway = new StripeGateway();
```

After:
```php
// AppServiceProvider::register()
$this->app->bind(PaymentGateway::class, StripeGateway::class);
// Consumer
public function __construct(private PaymentGateway $gateway) {}
```

#### Register application service providers in bootstrap/providers.php when using the streamlined skeleton
- **Do:** Register application service providers in bootstrap/providers.php when using the streamlined skeleton.
- **Flag when:** Flag when a provider is added only to config/app.php in an application using bootstrap/providers.php.
- **Why:** The modern skeleton loads application providers from bootstrap/providers.php.
- **Enforced by:** manual

Before:
```php
// config/app.php
'providers' => [App\Providers\BillingProvider::class],
```

After:
```php
// bootstrap/providers.php
return [App\Providers\AppServiceProvider::class, App\Providers\BillingProvider::class];
```

#### Bind dynamic values in raw SQL instead of interpolating them
- **Do:** Bind dynamic values in raw SQL instead of interpolating them.
- **Flag when:** Flag when DB raw statements or raw query clauses concatenate or interpolate external values.
- **Why:** Bindings preserve the boundary between SQL syntax and values and prevent SQL injection.
- **Enforced by:** manual

Before:
```php
$q->whereRaw("LOWER(name) = '$name'");
```

After:
```php
$q->whereRaw('LOWER(name) = ?', [$name]);
```

#### Use application environment predicates for environment checks
- **Do:** Check the environment with App::environment(...) / app()->isProduction() / app()->isLocal() instead of reading env('APP_ENV').
- **Flag when:** Flag when application code outside config files determines the environment via env('APP_ENV'), getenv() or $_ENV; treat string comparisons against app()->environment() as a style suggestion only.
- **Why:** Environment predicates use the bootstrapped application environment and support multiple environments.
- **Enforced by:** rector:AppEnvironmentComparisonToParameterRector

Before:
```php
if (env('APP_ENV') === 'production') { audit(); }
```

After:
```php
if (app()->isProduction()) { audit(); }
```

#### Register intentionally shared package services as container singletons
- **Do:** Register package-wide services and registries as container singletons when their state is intentionally shared.
- **Flag when:** Flag when repeated construction splits package-wide registry state or recreates services intended to have one shared instance.
- **Why:** One container-managed instance preserves shared registrations and avoids repeated construction. Request or tenant state requires an appropriately scoped lifetime.
- **Enforced by:** manual

Before:
```php
$registry = new PermissionRegistrar($cache);
```

After:
```php
$registry = app(PermissionRegistrar::class); // Provider registers it as a singleton
```

#### Configure trusted proxies and forwarded headers when HTTPS or host routing terminates at a reverse proxy
- **Do:** Configure trusted proxies and forwarded headers when HTTPS or host routing terminates at a reverse proxy.
- **Flag when:** Flag when proxied requests use incorrect scheme, host, port, or client IP because forwarded headers or trusted proxies are missing.
- **Why:** Laravel must trust the correct proxy metadata to generate accurate URLs and interpret client requests.
- **Enforced by:** manual

Before:
```php
URL::forceScheme('https'); // Masks proxy metadata
```

After:
```php
// bootstrap/app.php middleware configuration
$middleware->trustProxies(at: ['10.0.0.10']);
```

#### Store application timestamps in UTC and convert them at presentation boundaries
- **Do:** Store application timestamps in UTC and convert them at presentation boundaries.
- **Flag when:** Flag when application-wide timezone causes persisted timestamps to use local wall-clock time inconsistently.
- **Why:** UTC avoids ambiguous timestamps across daylight-saving transitions and deployments.
- **Enforced by:** manual

Before:
```php
'timezone' => 'America/New_York',
```

After:
```php
'timezone' => 'UTC',
```

#### Store persistent application files in durable shared storage when deployments use ephemeral or replicated filesystems
- **Do:** Store persistent application files in durable shared storage when deployments use ephemeral or replicated filesystems.
- **Flag when:** Flag when an app deployed to ephemeral or multi-replica infrastructure (Laravel Cloud, Vapor, containers, autoscaled or load-balanced servers) persists uploads or generated files on the local/public disk instead of shared object storage (e.g. an S3-compatible disk).
- **Why:** Instance-local files can disappear during deployment and are not shared across replicas.
- **Enforced by:** manual

#### Register custom manager drivers and mail transports in a service provider before consumers use them
- **Do:** Register custom manager drivers and mail transports in a service provider before consumers use them.
- **Flag when:** Flag when custom driver registration occurs inside a controller or is missing when configured by name.
- **Why:** Provider registration makes the extension available consistently across HTTP, queue, and console entrypoints.
- **Enforced by:** manual

Before:
```php
// Controller
Mail::extend('custom', fn () => new CustomTransport());
```

After:
```php
// Service provider boot()
Mail::extend('custom', fn () => new CustomTransport());
// config/mail.php uses transport => custom
```

#### Restrict register() to bindings and defer service-dependent bootstrapping to boot()
- **Do:** Restrict register() to bindings and defer service-dependent bootstrapping to boot().
- **Flag when:** Flag when register() executes service behavior, registers listeners or routes, or eagerly resolves services from providers that may not yet be registered.
- **Why:** All providers are registered before boot() runs, avoiding provider-order failures.
- **Enforced by:** manual

Before:
```php
public function register(): void { Event::listen(OrderPlaced::class, SendReceipt::class); }
```

After:
```php
public function boot(): void { Event::listen(OrderPlaced::class, SendReceipt::class); }
```

#### Use a StatefulGuard implementation for Fortify’s configured authentication guard
- **Do:** Use a StatefulGuard implementation for Fortify’s configured authentication guard.
- **Flag when:** Flag when the guard selected for Fortify lacks StatefulGuard behavior.
- **Why:** Fortify requires session-oriented authentication methods.
- **Enforced by:** manual

Before:
```php
// config/fortify.php
'guard' => 'api', // Configured guard implements only Guard, not StatefulGuard
```

After:
```php
// config/fortify.php
'guard' => 'web', // Session guard implements StatefulGuard
```

#### Require a valid two-factor code before presenting setup as complete, and explain the confirmation step
- **Do:** Require a valid two-factor code before presenting setup as complete, and explain the confirmation step.
- **Flag when:** Flag when two-factor enrollment is shown as complete immediately after secret generation.
- **Why:** A successful code proves the user configured an authenticator before relying on it.
- **Enforced by:** manual

Before:
```php
return 'Two-factor enabled'; // Immediately after generating secret
```

After:
```php
return 'Scan the QR code, then enter a valid code to confirm setup.';
```

#### Keep private files on nonpublic disks and reserve public disks for intentionally public content
- **Do:** Keep private files on nonpublic disks and reserve public disks for intentionally public content.
- **Flag when:** Flag when private documents are stored on a publicly linked disk or private roots are exposed by web configuration.
- **Why:** Disk visibility and web routing must match the intended access policy.
- **Enforced by:** manual

Before:
```php
Storage::disk('public')->put('invoices/'.$name, $pdf);
```

After:
```php
Storage::disk()->put('invoices/'.$name, $pdf); // Default disk is private
```

#### Put application bootstrapping and container bindings in service providers
- **Do:** Put application bootstrapping and container bindings in service providers.
- **Flag when:** Flag when cross-entrypoint bootstrapping is duplicated in controllers, routes, or commands.
- **Why:** Providers establish application services consistently for HTTP, queue, and console entrypoints.
- **Enforced by:** manual

Before:
```php
// Controller action repeats binding setup
app()->bind(PaymentGateway::class, StripeGateway::class);
```

After:
```php
// AppServiceProvider::register()
$this->app->bind(PaymentGateway::class, StripeGateway::class);
```

#### Configure application exception policy through withExceptions() in the streamlined bootstrap
- **Do:** Configure application exception policy through withExceptions() in the streamlined bootstrap.
- **Flag when:** Flag when an application using the streamlined bootstrap places exception configuration in an unregistered legacy Handler.
- **Why:** Exception policy must be registered through the active application bootstrap path.
- **Enforced by:** partial: shift:slimmer/bootstrap

Before:
```php
// app/Exceptions/Handler.php in an app that never registers this handler
protected $dontReport = [ExpectedDomainException::class];
```

After:
```php
->withExceptions(function (Exceptions $exceptions): void { $exceptions->dontReport([ExpectedDomainException::class]); })
```

#### Select authentication guards by names defined in config/auth.php
- **Do:** Select authentication guards by names defined in config/auth.php.
- **Flag when:** Flag when Auth::guard() references an undeclared guard.
- **Why:** Undeclared guards cannot be constructed by the authentication manager.
- **Enforced by:** manual

Before:
```php
Auth::guard('admins')->user(); // Only admin configured
```

After:
```php
Auth::guard('admin')->user();
```

#### Match Fortify login form field names to the configured username field
- **Do:** Match Fortify login form field names to the configured username field.
- **Flag when:** Flag when the login form submits email while Fortify expects another configured username key.
- **Why:** Field mismatch prevents authentication even with valid credentials.
- **Enforced by:** manual

Before:
```php
// Login form submits an email field
'username' => 'username',
```

After:
```php
// Matches the email input submitted by the login form
'username' => 'email',
```

#### Set explicit HTTP timeouts in third-party SDKs that bypass Laravel’s HTTP client
- **Do:** Set explicit HTTP timeouts in third-party SDKs that bypass Laravel’s HTTP client.
- **Flag when:** Flag when an external SDK request has no bounded connect or response timeout.
- **Why:** Laravel Http settings do not constrain an SDK’s separate transport.
- **Enforced by:** manual

Before:
```php
$client = new VendorClient(['key' => $key]);
```

After:
```php
$client = new VendorClient(['key' => $key, 'timeout' => 10, 'connect_timeout' => 3]); // Use SDK-supported options
```

#### Preserve required Laravel runtime directories without versioning their contents
- **Do:** Ensure bootstrap/cache and required storage subdirectories exist in every environment; retain self-ignoring .gitignore placeholders or create the directories during provisioning without committing runtime contents.
- **Flag when:** Flag a fresh checkout or deployment missing required runtime directories, or tracked session/cache/log contents used merely to preserve directory structure.
- **Why:** Git does not retain empty directories, while Laravel needs its runtime directory structure even before the first write.
- **Enforced by:** manual

#### Route unexpected application exceptions to configured logs or an error tracker
- **Do:** Route unexpected application exceptions to configured logs or an error tracker.
- **Flag when:** Flag when unexpected exceptions are swallowed or excluded from all reporting.
- **Why:** Recorded exceptions support diagnosis and incident response.
- **Enforced by:** manual

Before:
```php
try { performOperation(); } catch (Throwable $e) { return false; }
```

After:
```php
try { performOperation(); } catch (Throwable $e) { report($e); return false; }
```

#### Restrict environment-specific scheduled tasks with environments()
- **Do:** Restrict environment-specific scheduled tasks with environments().
- **Flag when:** Flag when a task intended only for named environments is scheduled without an environment restriction.
- **Why:** Explicit restrictions prevent staging or local schedulers from running production-only work.
- **Enforced by:** manual

Before:
```php
Schedule::command('billing:charge')->daily();
```

After:
```php
Schedule::command('billing:charge')->daily()->environments(['production']);
```

#### Reconcile published configuration with installed versions during upgrades
- **Do:** During framework or package upgrades, compare each published config file with the installed version's stub/upgrade notes and merge newly required or renamed keys and env variables while preserving your customizations.
- **Flag when:** Flag when, after an upgrade, a published config file lacks a key the installed version requires (incl. nested keys not covered by Laravel's default config merging) or still uses a renamed key/env variable the new version no longer reads.
- **Why:** Published copies do not automatically acquire every upstream change. Reviewing their diff prevents stale defaults and missing required options.
- **Enforced by:** partial: shift:public/laravel13/config-env

#### Set APP_URL to the real externally reachable application URL
- **Do:** Set APP_URL to the real externally reachable application URL.
- **Flag when:** Flag when generated absolute application or file URLs use localhost or an obsolete deployment URL.
- **Why:** Correct base URLs are required for links generated outside an HTTP request.
- **Enforced by:** manual

#### Return every provided service binding from provides() in a deferrable service provider
- **Do:** Return every provided service binding from provides() in a deferrable service provider.
- **Flag when:** Flag when a DeferrableProvider omits provides() or omits one of its registered services.
- **Why:** Laravel uses provides() to decide when to load the provider.
- **Enforced by:** partial: larastan:DeferrableServiceProviderMissingProvidesRule

Before:
```php
public function provides(): array { return []; }
```

After:
```php
public function provides(): array { return [ReportClient::class]; }
```

#### Configure Stripe webhooks to synchronize Cashier billing and asynchronous Checkout state
- **Do:** Configure Stripe webhooks to synchronize Cashier billing and asynchronous Checkout state.
- **Flag when:** Flag when Cashier subscriptions rely on local redirect completion without receiving required Stripe events.
- **Why:** Some payment methods complete asynchronously; webhooks keep application billing state aligned with Stripe.
- **Enforced by:** manual

### Database & migrations

#### Return anonymous migration classes
- **Do:** Use anonymous migration classes in migration files.
- **Flag when:** Flag when a migration defines a globally named class that can collide with another migration.
- **Why:** Anonymous classes avoid class-name collisions across application and package migrations.
- **Enforced by:** rector:AnonymousMigrationsRector, shift:workbench/adopt-anonymous-migrations

Before:
```php
class CreateUsersTable extends Migration {}
```

After:
```php
return new class extends Migration {};
```

#### Run production migrations in the deployment stage
- **Do:** Run required migrations with --force in the controlled production deployment stage.
- **Flag when:** Flag when a deployment leaves pending migrations or waits for interactive confirmation in a non-interactive pipeline.
- **Why:** The deployed code and schema must advance together under a predictable deployment process.
- **Enforced by:** manual

#### Implement truthful reversible down methods
- **Do:** Implement down() to reverse safely reversible schema changes and state any irreversible data loss explicitly.
- **Flag when:** Flag when a migration claims rollback support but down() omits the reverse operation or implies deleted data can be restored.
- **Why:** A reverse schema operation cannot recover destroyed data; rollback behavior must be accurate.
- **Enforced by:** manual

Before:
```php
public function down(): void {} // up() only added a nullable column.
```

After:
```php
public function down(): void {
    Schema::table('users', fn (Blueprint $table) => $table->dropColumn('nickname'));
}
```

#### Version all application schema objects in migrations
- **Do:** Create and alter required tables, indexes, generated columns, views, and other schema objects through versioned migrations.
- **Flag when:** Flag when required database structure is created only by manual edits or an untracked SQL script.
- **Why:** Versioned schema definitions keep all environments reproducible.
- **Enforced by:** manual

Before:
```php
// CREATE VIEW active_users AS ... run manually.
```

After:
```php
public function up(): void {
    DB::statement('CREATE VIEW active_users AS SELECT id, name FROM users WHERE active = 1');
}
public function down(): void { DB::statement('DROP VIEW IF EXISTS active_users'); }
```

#### Commit dependent writes atomically
- **Do:** Wrap database writes that must succeed or fail together in a transaction on the connection that owns them.
- **Flag when:** Flag when a workflow can persist only part of a required multi-write change after an exception.
- **Why:** Transactions keep related rows consistent when any operation fails.
- **Enforced by:** manual

Before:
```php
$order = Order::create($data);
$order->items()->createMany($items);
```

After:
```php
DB::transaction(function () use ($data, $items) {
    $order = Order::create($data);
    $order->items()->createMany($items);
});
```

#### Allowlist dynamic SQL identifiers
- **Do:** Map user-selected columns, tables, and sort directions to an explicit allowlist before building SQL.
- **Flag when:** Flag when request-controlled identifiers or directions reach select, orderBy, groupBy, table, or raw SQL without an allowlist.
- **Why:** Parameter bindings protect values, not SQL identifiers or query structure.
- **Enforced by:** manual

Before:
```php
User::orderBy($request->input('sort'))->get();
```

After:
```php
$sort = $request->validate(['sort' => ['required', Rule::in(['name', 'created_at'])]])['sort'];
User::orderBy($sort)->get();
```

#### Give conventional model tables identity and timestamps
- **Do:** Add id() and timestamps() to new ordinary Eloquent model tables unless their identity or timestamp contract deliberately differs.
- **Flag when:** Flag when a conventional timestamped model table lacks its primary key or created_at/updated_at columns.
- **Why:** The schema must match Eloquent’s expected identity and timestamp behavior.
- **Enforced by:** manual

Before:
```php
Schema::create('posts', fn ($t) => $t->string('title'));
```

After:
```php
Schema::create('posts', function (Blueprint $table) {
    $table->id();
    $table->string('title');
    $table->timestamps();
});
```

#### Share an atomic-lock cache for single-server scheduled tasks
- **Do:** Use onOneServer() for tasks that must run once across scheduler nodes and share a supported default cache store across those nodes.
- **Flag when:** Flag when a singleton scheduled job runs on multiple nodes without onOneServer(), or nodes use separate/unsupported lock stores.
- **Why:** The scheduler coordinates ownership through a shared atomic cache lock.
- **Enforced by:** manual

#### Serve Laravel from the public directory
- **Do:** Configure the web server document root as public/ and route application requests through public/index.php.
- **Flag when:** Flag when the web root exposes the repository root or bypasses Laravel’s front controller for application routes.
- **Why:** The public directory limits exposed files and provides the intended application entry point.
- **Enforced by:** manual

#### Prohibit destructive database reset commands in production
- **Do:** Block destructive reset commands in production and never run migrate:fresh against a shared or production database.
- **Flag when:** Flag when production permits or deploy scripts invoke migrate:fresh, migrate:refresh, or db:wipe.
- **Why:** These commands can erase application tables and their data.
- **Enforced by:** manual

Before:
```php
// No production guard; deploy runs migrate:fresh --force.
```

After:
```php
// AppServiceProvider::boot()
DB::prohibitDestructiveCommands(app()->isProduction());
```

#### Cascade deletion only for genuinely owned children
- **Do:** Declare database cascade deletion when a child cannot exist without its parent and parent deletion must remove it.
- **Flag when:** Flag when owned child rows can remain orphaned or deletion relies only on an Eloquent hook that bulk/direct deletes bypass.
- **Why:** A database cascade enforces ownership consistently across all writers.
- **Enforced by:** manual

Before:
```php
$table->foreignId('order_id'); // Items must disappear with the order.
```

After:
```php
$table->foreignId('order_id')->constrained()->cascadeOnDelete();
```

#### Keep SQLite foreign-key enforcement enabled
- **Do:** Enable SQLite foreign-key constraints unless a deliberate bounded maintenance operation requires disabling them.
- **Flag when:** Flag when SQLite foreign_keys is disabled for normal application access despite declared relational constraints.
- **Why:** Declared constraints provide no protection when enforcement is disabled.
- **Enforced by:** manual

Before:
```php
// config/database.php sqlite
'foreign_key_constraints' => false,
```

After:
```php
// config/database.php sqlite
'foreign_key_constraints' => env('DB_FOREIGN_KEYS', true),
```

#### Check engine-specific SQL against the deployed database
- **Do:** Use raw SQL syntax and features supported by every database engine and version on which that path runs.
- **Flag when:** Flag when raw SQL uses an engine-specific function, operator, or clause on an incompatible connection.
- **Why:** Query-builder bindings do not make SQL dialects portable. Engine-specific expressions need a matching connection and integration coverage.
- **Enforced by:** manual

Before:
```php
DB::connection('pgsql')->select('SELECT IFNULL(name, ?) FROM users', ['Unknown']);
```

After:
```php
DB::connection('pgsql')->select('SELECT COALESCE(name, ?) FROM users', ['Unknown']);
```

#### Declare foreign keys with deliberate targets and actions
- **Do:** Declare foreign-key constraints and choose the referenced table and delete/update action to match the relationship.
- **Flag when:** Flag when a relationship key lacks its intended constraint, constrained() infers the wrong table, or its default action conflicts with ownership semantics.
- **Why:** Constraints enforce referential integrity independently of Eloquent and prevent accidental cascade or orphan behavior.
- **Enforced by:** manual

Before:
```php
$table->foreignId('author_id');
```

After:
```php
$table->foreignId('author_id')->constrained('users')->restrictOnDelete();
```

#### Use attached Cloud resource configuration
- **Do:** Use Cloud-injected connection variables for attached resources and remove stale custom overrides.
- **Flag when:** Flag when custom environment variables override an attached resource with obsolete database/cache/storage connection details.
- **Why:** Explicit overrides take precedence and can silently keep an application connected to its old resource.
- **Enforced by:** manual

#### Declare nondefault migration connections explicitly
- **Do:** Set a migration’s connection property when its operations belong to a nondefault database.
- **Flag when:** Flag when a migration intended for a secondary database silently runs against the application default.
- **Why:** The explicit connection keeps schema changes on the intended database.
- **Enforced by:** manual

Before:
```php
return new class extends Migration { /* Intended reporting DB, no connection. */ };
```

After:
```php
return new class extends Migration {
    protected $connection = 'reporting';
};
```

#### Create one sessions table before using database sessions
- **Do:** Ensure a single sessions table migration exists and runs before database sessions are used; reuse the default skeleton migration when present.
- **Flag when:** Flag when the active database session driver has no sessions table or a duplicate migration recreates the skeleton table.
- **Why:** The session driver requires its table, and duplicate create migrations fail on fresh installs.
- **Enforced by:** manual

Before:
```php
// Add a second create_sessions_table migration although the users migration already creates it.
```

After:
```php
// Existing default create_users_table migration, used once:
Schema::create('sessions', function (Blueprint $table) {
    $table->string('id')->primary();
    $table->foreignId('user_id')->nullable()->index();
    $table->string('ip_address', 45)->nullable();
    $table->text('user_agent')->nullable();
    $table->longText('payload');
    $table->integer('last_activity')->index();
});
```

#### Create database cache and lock tables before using the store
- **Do:** When using the database cache store (the Laravel 13 default), make sure both the cache and cache_locks tables exist (the default 0001_01_01_000001_create_cache_table migration or php artisan make:cache-table) and are migrated before any cache operation runs.
- **Flag when:** Flag when database cache operations run before their required tables exist or the lock table is omitted.
- **Why:** The database store and its atomic locks use separate schema tables.
- **Enforced by:** manual

Before:
```php
// CACHE_STORE=database with only an application-data migration.
```

After:
```php
php artisan make:cache-table
php artisan migrate
// creates `cache` (key, mediumText value, bigInteger expiration) and `cache_locks` (key, owner, bigInteger expiration)
```

#### Lock rows inside the transaction that changes them
- **Do:** Read contested rows with lockForUpdate() inside the same transaction that validates and updates their state.
- **Flag when:** Flag when a read-modify-write workflow races with concurrent requests or acquires a row lock outside its transaction.
- **Why:** The lock must protect both the read and the write until commit or rollback.
- **Enforced by:** manual

Before:
```php
$account = Account::findOrFail($id);
$account->update(['balance' => $account->balance - $amount]);
```

After:
```php
DB::transaction(function () use ($id, $amount) {
    $account = Account::whereKey($id)->lockForUpdate()->firstOrFail();
    $account->update(['balance' => $account->balance - $amount]);
});
```

#### Give each application a least-privilege database account
- **Do:** Create a dedicated database user per application and grant only the privileges its runtime and deployment roles need.
- **Flag when:** Flag when applications share an overprivileged database account or use an administrator account for normal requests.
- **Why:** Per-application access limits the scope of credential compromise and accidental queries.
- **Enforced by:** manual

#### Connect database capacity monitoring to an alert handler
- **Do:** When you schedule db:monitor, pass --max (and --databases) so DatabaseBusy can fire, and listen for Illuminate\Database\Events\DatabaseBusy (e.g. in AppServiceProvider::boot) to send an actionable notification to the team.
- **Flag when:** Flag a scheduled db:monitor that has no --max threshold, or no DatabaseBusy listener that notifies someone.
- **Why:** Scheduling the command alone does not notify operators about exhausted connection capacity.
- **Enforced by:** manual

#### Select constrained eager-load columns inside the callback
- **Do:** Put the select() in the eager-load callback when combining column restrictions with relationship constraints.
- **Flag when:** Flag when with() uses a colon column-selector key and a callback as though both will be composed.
- **Why:** The callback must own both projection and constraints, including keys needed to match relationships.
- **Enforced by:** partial: larastan:RelationExistenceRule

Before:
```php
User::with(['accounts:id,user_id' => fn ($q) => $q->where('active', true)])->get();
```

After:
```php
User::with(['accounts' => fn ($q) => $q->select('id', 'user_id')->where('active', true)])->get();
```

#### Use full Unicode storage with a deliberate MySQL collation
- **Do:** Use utf8mb4 for MySQL/MariaDB connection and text storage, keeping Laravel's utf8mb4_unicode_ci default unless the domain requires another supported collation.
- **Flag when:** Flag when text expected to accept full Unicode is stored/connected through utf8mb3 or another insufficient charset, or collation comparison semantics contradict the field contract.
- **Why:** utf8mb4 supports four-byte characters; collation also determines case/accent comparison and must match the application's identity/search rules.
- **Enforced by:** manual

#### Make database nullability reflect actual absence semantics
- **Do:** Use nullable columns only for genuinely absent/unknown values and store SQL NULL for absence instead of sentinel strings such as "null" or an invented magic value.
- **Flag when:** Flag when a required domain value is arbitrarily nullable or absence is encoded as a string despite a nullable schema contract.
- **Why:** Truthful nullability makes constraints, comparisons and aggregate behavior match the domain.
- **Enforced by:** manual

#### Declare schemas for machine-parsed MCP output
- **Do:** Define outputSchema() for structured MCP tool results that clients must parse.
- **Flag when:** Flag when a tool returns a machine-consumed structured payload without a matching output schema.
- **Why:** An explicit schema makes the payload contract discoverable and verifiable.
- **Enforced by:** manual

Before:
```php
return Response::structured(['count' => 3]); // No output schema.
```

After:
```php
public function outputSchema(JsonSchema $schema): array {
    return ['count' => $schema->integer()->required()];
}
```

#### Describe and constrain MCP tool inputs
- **Do:** Define MCP tool parameters in schema() with descriptions and required/type/value constraints appropriate to each argument.
- **Flag when:** Flag when a tool consumes arguments absent from its schema or leaves important argument constraints undocumented.
- **Why:** Clients use the input schema to construct valid calls.
- **Enforced by:** manual

Before:
```php
public function schema(JsonSchema $schema): array { return []; } // handle() requires id.
```

After:
```php
public function schema(JsonSchema $schema): array {
    return ['id' => $schema->integer()->description('Record ID')->required()];
}
```

#### Make required query ordering explicit
- **Do:** Specify an explicit database order whenever feature behavior depends on result order.
- **Flag when:** Flag when a query relies on implicit row order for first/latest results, pagination, or an ordered display.
- **Why:** Databases do not guarantee order without ORDER BY.
- **Enforced by:** manual

Before:
```php
$latest = Post::first();
```

After:
```php
$latest = Post::orderByDesc('created_at')->orderByDesc('id')->first();
```

#### Normalize login rate-limit identifiers like database matching
- **Do:** Normalize login identifiers consistently with authentication/database comparison semantics before building rate-limit keys.
- **Flag when:** Flag when case or normalization variants authenticate as the same identity but receive different rate-limit buckets.
- **Why:** Equivalent identities must share a limit or attackers can evade throttling by changing their spelling.
- **Enforced by:** manual

Before:
```php
return Limit::perMinute(5)->by($request->input('email'));
```

After:
```php
RateLimiter::for('login', function (Request $request) {
    return Limit::perMinute(5)->by(
        Str::transliterate(Str::lower($request->input('email')).'|'.$request->ip())
    );
});
```

#### Keep implicit-commit statements out of managed transactions
- **Do:** Avoid MySQL statements that implicitly commit inside Laravel-managed transactions.
- **Flag when:** Flag when DDL or another implicit-commit statement executes inside a transaction expected to roll back as one unit.
- **Why:** The database can commit while Laravel still believes a transaction is open, breaking rollback and savepoint behavior.
- **Enforced by:** manual

Before:
```php
DB::transaction(function () { DB::statement('CREATE TABLE temp_data (id INT)'); $this->write(); });
```

After:
```php
// Run schema DDL separately from transactional DML.
DB::statement('CREATE TABLE temp_data (id INT)');
DB::transaction(fn () => $this->write());
```

#### Replace removed spatial column helpers with supported geometry definitions
- **Do:** Use Laravel 13 geometry()/geography() with an explicit subtype instead of removed spatial column helpers.
- **Flag when:** Flag when migrations still call removed spatial helpers such as point() or polygon().
- **Why:** Removed methods fail on current Laravel; the supported definitions preserve spatial intent.
- **Enforced by:** rector:RefactorBlueprintGeometryColumnsRector

Before:
```php
$table->point('location');
```

After:
```php
$table->geometry('location', subtype: 'point', srid: 4326);
```

#### Declare structured AI output contracts
- **Do:** Declare a JSON schema (HasStructuredOutput::schema() on the agent, or the schema: argument for anonymous agents) whenever callers need machine-parsed AI output, and read fields from the StructuredAgentResponse.
- **Flag when:** Flag when code json_decodes or regex-parses free-form model text as a stable structured payload instead of declaring an output schema.
- **Why:** A declared schema constrains output shape and makes the consumer contract explicit.
- **Enforced by:** manual

Before:
```php
$data = json_decode($agent->prompt($question)->text, true); // Free-form agent.
```

After:
```php
class Extractor implements Agent, HasStructuredOutput {
    use Promptable;
    public function instructions(): string { return 'Extract the person.'; }
    public function schema(JsonSchema $schema): array {
        return ['name' => $schema->string()->required()];
    }
}
$name = (new Extractor)->prompt($question)['name'];
```

### Eloquent

#### Pass explicit validated fields to mass-assignment operations
- **Do:** Pass explicit validated fields to mass-assignment operations.
- **Flag when:** Flag when create, update or fill receives request all or other unrestricted request arrays.
- **Why:** An explicit input boundary prevents unintended fields from being persisted.
- **Enforced by:** manual

Before:
```php
User::create($request->all());
```

After:
```php
User::create($request->safe()->only(['name', 'email']));
```

#### Hide sensitive model attributes from serialization
- **Do:** Hide sensitive model attributes from array and JSON serialization; use #[Hidden([...])] for declarative Laravel 13 model configuration.
- **Flag when:** Flag when passwords, tokens or other confidential model fields can appear in array/JSON output.
- **Why:** A model-level serialization safeguard protects consumers that serialize models directly.
- **Enforced by:** manual

Before:
```php
protected $hidden = [];
```

After:
```php
use Illuminate\Database\Eloquent\Attributes\Hidden;

#[Hidden(['password', 'remember_token'])]
class User extends Model {}
```

#### Process unbounded data sets with bounded chunks or lazy iteration
- **Do:** Process unbounded data sets with bounded chunks or lazy iteration.
- **Flag when:** Flag when a batch/background task iterates a table or query that can grow without bound via all() or get(), holding every model in memory, instead of chunk()/chunkById()/lazy()/lazyById()/cursor().
- **Why:** Bounded iteration keeps memory usage proportional to the batch size.
- **Enforced by:** manual

Before:
```php
foreach (User::all() as $user) { sendDigest($user); }
```

After:
```php
User::chunkById(500, fn ($users) => $users->each(fn ($u) => sendDigest($u)));
```

#### Use on-demand notifications for recipients without a persisted notifiable model
- **Do:** Use on-demand notifications for recipients without a persisted notifiable model.
- **Flag when:** Flag when code creates dummy database models solely to send a notification to an address.
- **Why:** Routing notifications directly avoids meaningless records and supports arbitrary recipients.
- **Enforced by:** manual

Before:
```php
$user = User::create(['email' => $address]); $user->notify(new Alert);
```

After:
```php
Notification::route('mail', $address)->notify(new Alert);
```

#### Load relationship counts with withCount() instead of hydrating children
- **Do:** Load relationship counts with withCount() instead of hydrating children.
- **Flag when:** Flag when code loads or queries each parent relationship solely to count it.
- **Why:** Database aggregates avoid unnecessary model hydration and per-parent count queries.
- **Enforced by:** partial: larastan:NoUnnecessaryCollectionCallRule

Before:
```php
$posts = Post::with('comments')->get(); // only comments->count() used
```

After:
```php
$posts = Post::withCount('comments')->get();
```

#### Scope nested route bindings to the parent resource
- **Do:** Scope nested route bindings to the parent resource.
- **Flag when:** Flag when a nested endpoint can bind a child belonging to a different parent.
- **Why:** Scoped bindings enforce the URL relationship before controller execution.
- **Enforced by:** manual

Before:
```php
Route::get('/users/{user}/posts/{post}', $handler);
```

After:
```php
Route::get('/users/{user}/posts/{post}', $handler)->scopeBindings();
```

#### Keep enum storage and PHP enum casts compatible
- **Do:** When a column is cast to a backed enum, keep every stored value (migration defaults, seeders, raw/bulk writes) a valid backing value of that enum, and migrate existing rows when enum cases change.
- **Flag when:** Flag when an enum-cast column can receive or already holds values outside the enum's backing values (e.g. a string default, raw update or seeder value that is not a case), or when a case is removed/renamed without migrating stored rows.
- **Why:** Compatible storage and casting make domain values explicit and prevent hydration errors.
- **Enforced by:** manual

Before:
```php
// provider stores arbitrary strings with no domain constraint.
```

After:
```php
// migration
$table->string('status')->default(OrderStatus::Pending->value);
// model
protected function casts(): array { return ['status' => OrderStatus::class]; }
```

#### Eager-load relationships that a collection consumer will access
- **Do:** Eager-load relationships that a collection consumer will access.
- **Flag when:** Flag when a loop or serializer lazily queries the same relationship once per parent.
- **Why:** Batch eager loading prevents query counts from growing with the number of rows.
- **Enforced by:** manual

Before:
```php
$users = User::all(); foreach ($users as $u) { echo $u->team->name; }
```

After:
```php
$users = User::with('team')->get(); foreach ($users as $u) { echo $u->team->name; }
```

#### Reuse an already-loaded parent for each child inverse relationship
- **Do:** When children of an already-loaded parent access their inverse parent, hydrate it instead of lazy-loading: use chaperone() on the hasMany relation (or when eager loading), or setRelation() on each child.
- **Flag when:** Flag when a loop over loaded children lazy-loads their known parent again.
- **Why:** Hydrating the inverse relation avoids additional queries and duplicate parent instances.
- **Enforced by:** manual

Before:
```php
foreach ($post->comments as $comment) { echo $comment->post->title; }
```

After:
```php
public function comments(): HasMany { return $this->hasMany(Comment::class)->chaperone(); }
// or at runtime:
$post->comments->each->setRelation('post', $post);
```

#### Choose compatible primary-key strategies in models and their schema
- **Do:** Choose compatible primary-key strategies in models and their schema.
- **Flag when:** Flag when model UUID, ULID or incrementing-key configuration disagrees with its migration or related foreign keys.
- **Why:** Matching key generation and storage prevents insert and relationship failures.
- **Enforced by:** manual

Before:
```php
$table->id(); // model uses HasUuids
```

After:
```php
$table->uuid('id')->primary(); // model uses HasUuids
```

#### Centralize translation of API filters, sorts and includes into a constrained query API
- **Do:** Translate API filter, sort and include parameters through an explicit allow-list (e.g. spatie/laravel-query-builder allowedFilters/allowedSorts/allowedIncludes or one shared query object); never pass request-supplied column or relation names directly to orderBy(), where() or with().
- **Flag when:** Flag when request-supplied sort, filter or include names reach orderBy(), where(), with() or similar query methods without an allow-list.
- **Why:** One explicit query boundary keeps accepted parameters consistent and reviewable.
- **Enforced by:** manual

Before:
```php
$query->orderBy($request->input('sort'));
```

After:
```php
QueryBuilder::for(User::class)->allowedSorts('name')->allowedFilters('name')->get();
```

#### Match model primary-key and timestamp metadata to its table
- **Do:** Match model primary-key and timestamp metadata to its table.
- **Flag when:** Flag when a model assumes id/created_at/updated_at despite a table using different names or lacking timestamps.
- **Why:** Correct metadata prevents invalid writes and lookups.
- **Enforced by:** manual

Before:
```php
// Table has flight_id but model assumes id.
```

After:
```php
#[Table(key: 'flight_id')]
class Flight extends Model { const CREATED_AT = 'created_on'; const UPDATED_AT = 'updated_on'; }
```

#### Include soft-deleted rows only in queries that intentionally require them
- **Do:** Include soft-deleted rows only in queries that intentionally require them.
- **Flag when:** Flag when normal model queries use withTrashed indiscriminately or a recovery workflow omits required trashed rows.
- **Why:** Explicit visibility preserves the normal live-record boundary.
- **Enforced by:** manual

Before:
```php
Post::withTrashed()->get(); // ordinary public list
```

After:
```php
Post::query()->get(); // recovery endpoint uses withTrashed explicitly
```

#### Retain every relationship matching key when narrowing selected columns
- **Do:** Retain every relationship matching key when narrowing selected columns.
- **Flag when:** Flag when a parent or related select omits a local, foreign or owner key needed for eager-load matching.
- **Why:** Missing matching keys can silently produce empty or incorrect relations.
- **Enforced by:** manual

Before:
```php
Post::select('title')->with('comments:id,body')->get();
```

After:
```php
Post::select('id', 'title')->with('comments:id,post_id,body')->get();
```

#### Scope entity-specific Pennant flags to the intended entity
- **Do:** Pass the intended entity with Feature::for($scope) when checking or changing a user/team-specific Pennant feature whenever that entity is not the default scope (the authenticated user), e.g. in admin actions, Artisan commands, queued jobs or team-scoped features.
- **Flag when:** Flag when a scope-specific feature is checked or changed without Feature::for() in code where the default scope is a different entity or null (admin acting on another user, commands, jobs), or when an entity-specific feature is changed with activateForEveryone()/globally().
- **Why:** Explicit scope prevents one entity feature state from affecting another.
- **Enforced by:** manual

Before:
```php
Feature::activate('new-dashboard');
```

After:
```php
Feature::for($user)->activate('new-dashboard');
```

#### Name model trait boot hooks boot{TraitName}()
- **Do:** Name model trait boot hooks boot{TraitName}().
- **Flag when:** Flag when a model trait declares boot and can override another boot implementation.
- **Why:** Named trait hooks let Eloquent initialize each trait without method collisions.
- **Enforced by:** manual

Before:
```php
trait Auditable { protected static function boot() {} }
```

After:
```php
trait Auditable { protected static function bootAuditable(): void {} }
```

#### Group OR predicates so they cannot escape relationship or global-scope constraints
- **Do:** Group OR predicates so they cannot escape relationship or global-scope constraints.
- **Flag when:** Flag when a top-level orWhere bypasses tenant, parent or other required predicates.
- **Why:** Parenthesized OR groups keep mandatory constraints applied to every alternative.
- **Enforced by:** manual

Before:
```php
$user->posts()->where('active', true)->orWhere('votes', '>', 100)->get();
```

After:
```php
$user->posts()->where(fn ($q) => $q->where('active', true)->orWhere('votes', '>', 100))->get();
```

#### Avoid model members that collide with inherited or trait-provided APIs
- **Do:** Before naming a model relationship or helper, check inherited methods, properties and installed trait APIs. Use a distinct domain name when the proposed member would conflict with casts(), Scout search() or another real existing member.
- **Flag when:** Flag a relationship named casts or a custom search method that shadows the Searchable trait entry point unintentionally.
- **Why:** Member collisions can prevent relationship loading or replace framework/package behavior.
- **Enforced by:** partial: larastan:PHPStan\Rules\Methods\OverridingMethodRule, partial: larastan:PHPStan\Rules\Properties\OverridingPropertyRule

Before:
```php
public function casts(): HasMany { return $this->hasMany(CastMember::class); }
```

After:
```php
public function castMembers(): HasMany { return $this->hasMany(CastMember::class); }
```

#### Use cursor() only for streaming reads that do not require eager-loaded relationships
- **Do:** Use cursor() only for streaming reads that do not require eager-loaded relationships.
- **Flag when:** Flag when cursor iteration accesses unloaded relationships or assumes driver buffering guarantees constant memory.
- **Why:** Cursors hydrate one model at a time but cannot eager-load relations, and PDO buffering can still consume memory.
- **Enforced by:** manual

Before:
```php
foreach (User::with('roles')->cursor() as $user) { /* ... */ }
```

After:
```php
foreach (User::cursor() as $user) { echo $user->email; }
```

#### List only defined computed accessors in model appends
- **Do:** List only existing computed accessors in #[Appends([...])], using their serialized snake_case names.
- **Flag when:** Flag when an appended attribute name does not resolve to a defined accessor.
- **Why:** Invalid appended fields break or corrupt serialization.
- **Enforced by:** larastan:ModelAppendsRule

Before:
```php
protected $appends = ['missing_label'];
```

After:
```php
use Illuminate\Database\Eloquent\Attributes\Appends;
use Illuminate\Database\Eloquent\Casts\Attribute;

#[Appends(['display_name'])]
class User extends Model
{
    protected function displayName(): Attribute
    {
        return Attribute::get(fn () => $this->first_name.' '.$this->last_name);
    }
}
```

#### Allowlist broadcast event payload fields
- **Do:** When a broadcast event carries models or data with fields subscribers must not see, define broadcastWith() (or narrow the public properties) so only the needed fields are sent.
- **Flag when:** Flag a ShouldBroadcast event whose public properties (with no broadcastWith()) expose sensitive or non-hidden internal attributes, such as email, tokens or private profile data, to channel subscribers, especially on public channels.
- **Why:** An explicit payload controls data exposure and network size.
- **Enforced by:** manual

Before:
```php
public User $user; // entire public model payload
```

After:
```php
public function broadcastWith(): array { return ['id' => $this->user->id]; }
```

#### Maintain and consult a path-scoped index of repository conventions
- **Do:** Commit .ai/rules, give every rule file a paths: frontmatter glob, record rules via Boost's record-rule tool (or run boost:index-rules) so .ai/rules/index.md stays current, and consult matching rules before editing.
- **Flag when:** Flag when .ai/rules is not committed, a rule file lacks accurate paths globs, or a hand-added rule file is missing from the regenerated .ai/rules/index.md.
- **Why:** Path scoping makes relevant conventions discoverable without applying unrelated rules.
- **Enforced by:** manual

Before:
```php
// .ai/rules/index.md: no mapping for model files
```

After:
```php
---
paths:
  - app/Models/**
---
# Models
## Store money as integer cents
...
// then: php artisan boost:index-rules  -> index.md row: | app/Models/** | .ai/rules/models.md |
```

#### Use lazy() with eager loading for large reads that need relationships
- **Do:** Use lazy() with eager loading for large reads that need relationships.
- **Flag when:** Flag when an unbounded get or cursor loop reads relationships across many models.
- **Why:** Lazy chunking bounds memory and supports eager loading once per chunk.
- **Enforced by:** manual

Before:
```php
foreach (User::cursor() as $user) { echo $user->roles->count(); }
```

After:
```php
foreach (User::with('roles')->lazy(500) as $user) { echo $user->roles->count(); }
```

#### Restrict Telescope access outside local development
- **Do:** Keep Telescope's Authorize middleware and a restrictive viewTelescope gate (explicit user/role allowlist) for non-local environments, and set APP_ENV=production in production.
- **Flag when:** Flag when viewTelescope allows any/guest user, Authorize is removed from telescope.middleware, or Telescope is enabled on a public staging/production host running with APP_ENV=local.
- **Why:** Telescope exposes request and query details that require access control.
- **Enforced by:** manual

#### Align Typesense ID and search-field settings with the indexed payload
- **Do:** For Typesense, return a toSearchableArray() payload that matches the collection schema: id cast to string, created_at as a UNIX timestamp, and field types matching the schema.
- **Flag when:** Flag when a Typesense-indexed model sends a non-string id, a non-integer created_at, or fields whose types contradict the configured collection schema.
- **Why:** Matching schema and payload types avoids rejected documents and unexpected search ranking.
- **Enforced by:** manual

Before:
```php
return ['id' => $this->id];
```

After:
```php
public function toSearchableArray(): array
{
    return array_merge($this->toArray(), [
        'id' => (string) $this->id,
        'created_at' => $this->created_at->timestamp,
    ]);
}
// Match these fields to config/scout.php's Typesense collection schema.
```

#### Declare Attribute accessors and attributed local scopes protected
- **Do:** Declare Attribute accessors and attributed local scopes protected.
- **Flag when:** Flag when Attribute-returning accessors or #[Scope] methods are public despite their framework-dispatched contract.
- **Why:** Protected visibility prevents accidental direct invocation and matches the intended model API.
- **Enforced by:** larastan:NoPublicModelScopeAndAccessorRule, rector:MakeModelAttributesAndScopesProtectedRector

Before:
```php
public function name(): Attribute { /* ... */ }
```

After:
```php
protected function name(): Attribute { /* ... */ }
```

#### Read query-expression SQL with getValue() and the connection grammar
- **Do:** Read query-expression SQL with getValue() and the connection grammar.
- **Flag when:** Flag when code casts an SQL expression object to string.
- **Why:** Current expressions require grammar-aware SQL rendering.
- **Enforced by:** rector:DatabaseExpressionCastsToMethodCallRector, rector:DatabaseExpressionToStringToMethodCallRector

Before:
```php
$sql = (string) DB::raw('now()');
```

After:
```php
$sql = DB::raw('now()')->getValue(DB::connection()->getQueryGrammar());
```

#### Replace the removed model dates property with date or datetime casts
- **Do:** Replace the removed model dates property with date or datetime casts.
- **Flag when:** Flag when a model relies on the obsolete $dates property for conversion.
- **Why:** Laravel 13 uses cast declarations for additional date attributes.
- **Enforced by:** rector:UnifyModelDatesWithCastsRector, shift:workbench/convert-dates-property

Before:
```php
protected $dates = ['published_at'];
```

After:
```php
protected function casts(): array { return ['published_at' => 'datetime']; }
```

#### Match the installed framework method signature in custom implementations and overrides
- **Do:** Match the installed framework method signature in custom implementations and overrides.
- **Flag when:** Flag when Castable::castUsing omits arguments or a Model::touch override omits its attribute parameter.
- **Why:** Compatible signatures preserve framework calls and avoid inheritance errors.
- **Enforced by:** partial: rector:ArgumentAdderRector, partial: larastan:PHPStan\Rules\Methods\OverridingMethodRule

Before:
```php
public static function castUsing() { return CustomCast::class; }
```

After:
```php
public static function castUsing(array $arguments) { return CustomCast::class; }
```

#### Specify explicit relationship keys when conventions cannot infer the intended association
- **Do:** Specify explicit relationship keys when conventions cannot infer the intended association.
- **Flag when:** Flag when custom foreign/type/table names are used without matching relationship arguments.
- **Why:** Explicit metadata prevents queries from using an inferred but nonexistent key.
- **Enforced by:** manual

Before:
```php
return $this->belongsTo(User::class); // column is owner_uuid
```

After:
```php
return $this->belongsTo(User::class, 'owner_uuid', 'uuid');
```

#### Refresh or reload a previously loaded relation after inserting through it when the current collection is needed
- **Do:** Refresh or reload a previously loaded relation after inserting through it when the current collection is needed.
- **Flag when:** Flag when code assumes save/saveMany appended newly stored records to an already-loaded relation collection.
- **Why:** Database writes do not automatically reconcile an existing relation snapshot.
- **Enforced by:** manual

Before:
```php
$post->comments()->save($comment); return $post->comments;
```

After:
```php
$post->comments()->save($comment); $post->load('comments'); return $post->comments;
```

#### Use authenticated private or presence channels for nonpublic broadcast data
- **Do:** Use authenticated private or presence channels for nonpublic broadcast data.
- **Flag when:** Flag when sensitive account or user events are broadcast on a public Channel.
- **Why:** Public channels permit unauthenticated subscription.
- **Enforced by:** manual

Before:
```php
return [new Channel('orders')];
```

After:
```php
return [new PrivateChannel('orders.'.$this->order->user_id)];
```

### Events / mail / notifications

#### Discover conventional event listeners automatically
- **Do:** Register each listener once: rely on event discovery for listeners in discovered directories (typed handle()/__invoke() event parameter), and register manually only listeners outside discovery paths or when discovery is disabled.
- **Flag when:** Flag when an ordinary listener in an enabled discovery path is also registered manually, or its method name/event parameter prevents intended discovery.
- **Why:** Discovery removes redundant event maps and prevents drift between listener signatures and registration.
- **Enforced by:** manual

Before:
```php
// app/Listeners/SendReceipt.php is already discovered.
Event::listen(OrderPaid::class, SendReceipt::class);
```

After:
```php
// app/Listeners/SendReceipt.php
class SendReceipt { public function handle(OrderPaid $event): void { /* send receipt */ } }
```

#### Prevent development mail from reaching real recipients
- **Do:** Use the log transport, an isolated SMTP catcher, or an enforced development recipient for non-production mail; keep log-only delivery out of production workflows that require real delivery.
- **Flag when:** Flag when development or staging mail can reach real recipients, or a production workflow that requires delivery is configured only to log mail.
- **Why:** A safe development transport makes messages inspectable without sending test content to customers. Log transport records messages but does not deliver them.
- **Enforced by:** manual

Before:
```php
# .env.local
MAIL_MAILER=smtp
MAIL_HOST=smtp.production.example
```

After:
```php
# .env.local
MAIL_MAILER=log
# Alternative: SMTP to an isolated mail catcher such as Mailpit.
```

#### Use immutable dates for calculations
- **Do:** Use immutable date objects when calculations should derive a new date without changing the original value.
- **Flag when:** Flag when a date calculation unintentionally mutates an object reused as the original date elsewhere in the workflow.
- **Why:** Immutability prevents chained calculations or shared references from changing the source timestamp.
- **Enforced by:** manual

Before:
```php
$start = Carbon::parse($input);
$end = $start->addDay(); // also changes $start
```

After:
```php
$start = CarbonImmutable::parse($input);
$end = $start->addDay();
```

#### Send notifications synchronously when the caller needs the outcome
- **Do:** Send a notification synchronously when the operation must immediately know whether delivery through the channel succeeded or failed.
- **Flag when:** Flag when a workflow depends on immediate notification failure feedback but dispatches a queued notification and reports completion before it runs.
- **Why:** Queued execution cannot return delivery failures to the initiating operation. A successful transport call still does not prove the recipient read the message.
- **Enforced by:** manual

Before:
```php
$user->notify(new CriticalNotice($data)); // CriticalNotice implements ShouldQueue
return response()->noContent();
```

After:
```php
$user->notifyNow(new CriticalNotice($data)); // handle transport exceptions here
return response()->noContent();
```

#### Restrict who can frame the application
- **Do:** Restrict framing of pages that must not be embedded cross-origin with CSP frame-ancestors 'self' (or stricter) and/or X-Frame-Options: SAMEORIGIN/DENY, set in middleware or at the edge.
- **Flag when:** Flag when an application that should not be embedded cross-origin sends neither a frame-ancestors directive nor an X-Frame-Options header (or permits arbitrary framing).
- **Why:** Restricting framing reduces clickjacking exposure.
- **Enforced by:** manual

Before:
```php
$response->headers->set("Content-Security-Policy", "default-src 'self'");
```

After:
```php
$response->headers->set("Content-Security-Policy", "default-src 'self'; frame-ancestors 'self'");
```

### Frontend & views

#### Use the Laravel Vite asset pipeline
- **Do:** Build application assets with Vite and the Laravel plugin, and load them with @vite; complete dependency, script, module-import, and template migration when replacing Mix.
- **Flag when:** Flag when a new asset pipeline uses abandoned Mix tooling, or a Vite migration leaves Mix scripts, require calls incompatible with ESM, or mix() asset references.
- **Why:** One integrated build pipeline produces versioned assets and keeps development and production resolution aligned.
- **Enforced by:** partial: shift:vite-converter

Before:
```php
{{ mix('js/app.js') }}
```

After:
```php
@vite('resources/js/app.js')
```

#### Define new Tailwind 4 theme tokens in CSS
- **Do:** Define Tailwind 4 theme tokens with @theme for new CSS-first configuration; preserve legacy JavaScript config only through an explicit supported migration path.
- **Flag when:** Flag when a Tailwind v4 project relies on theme values from tailwind.config.js without loading it through an explicit @config directive, or adds new theme tokens to a JS config instead of @theme.
- **Why:** CSS-first theme tokens generate consistent utilities; v4 does not automatically load legacy JavaScript configuration.
- **Enforced by:** manual

Before:
```php
// tailwind.config.js is present but never loaded
```

After:
```php
@theme { --color-brand: #123456; }
```

#### Namespace package and module views
- **Do:** Namespace package and module views.
- **Flag when:** Flag when a package/module registers global unqualified views that collide with application or other package names.
- **Why:** A namespace preserves package ownership while allowing documented application overrides.
- **Enforced by:** manual

Before:
```php
View::addLocation($packageViews);
```

After:
```php
$this->loadViewsFrom($packageViews, 'courier');
```

#### Pass parent props explicitly before using aware
- **Do:** Use @aware for explicitly passed ancestor props and pass required parent values through attributes rather than relying on parent defaults or slots being inherited.
- **Flag when:** Flag when a child expects @aware to receive an ancestor default or slot that was never explicitly passed as a prop.
- **Why:** Aware resolves explicitly supplied ancestor data; relying on implicit defaults yields missing or inconsistent values.
- **Enforced by:** manual

Before:
```php
<x-menu><x-menu.item /></x-menu>
```

After:
```php
<x-menu color="gray"><x-menu.item /></x-menu>
```

#### Translate user-facing text in localized applications
- **Do:** Store user-facing strings in language resources and render translations with the translation helper in localized applications.
- **Flag when:** Flag when a localized interface hard-codes user-facing strings instead of using its language resources.
- **Why:** Language resources let translations change without editing templates.
- **Enforced by:** manual

Before:
```php
<label>Email address</label>
```

After:
```php
<label>{{ __('Email address') }}</label>
```

#### Declare Blade component data inputs
- **Do:** Declare class component data as constructor properties and anonymous component data with @props, including intentional defaults.
- **Flag when:** Flag when component data is accidentally treated as an HTML attribute or an anonymous component reads undeclared input variables.
- **Why:** Declared props distinguish data from forwarded HTML attributes and avoid accidental encoding or missing inputs.
- **Enforced by:** manual

Before:
```php
<div>{{ $message }}</div>
```

After:
```php
@props(['message' => ''])
<div>{{ $message }}</div>
```

#### Bind PHP component expressions explicitly
- **Do:** Prefix component attributes with : when passing PHP expressions or variables.
- **Flag when:** Flag when a component receives a literal "$variable" string where the caller intended a PHP value.
- **Why:** Explicit binding preserves types and avoids passing expression text.
- **Enforced by:** manual

Before:
```php
<x-alert message="$message" />
```

After:
```php
<x-alert :message="$message" />
```

#### Merge paginated data when implementing infinite scroll
- **Do:** Use merging props or Inertia scroll pagination when later pages must append rather than replace earlier data.
- **Flag when:** Flag when an infinite-scroll partial reload replaces the existing item list or concatenates it through fragile custom state.
- **Why:** Server-declared merging preserves accumulated pages across reloads.
- **Enforced by:** manual

Before:
```php
'items' => Item::paginate()
```

After:
```php
'items' => Inertia::scroll(fn () => Item::paginate())
```

#### Lock server-controlled Livewire public properties
- **Do:** Protect server-controlled public identifiers and similar state from client tampering with #[Locked] (or by storing the Eloquent model itself as the property), and still validate and authorize every action that uses them.
- **Flag when:** Flag when a security-sensitive public property (e.g. a record ID used in an action) is a plain, unlocked value that the client can change and the action uses it without authorizing it.
- **Why:** Locking rejects client-side property changes but does not replace authorization.
- **Enforced by:** manual

Before:
```php
public int $postId;
```

After:
```php
#[Locked]
public int $postId;
```

#### Use stable unique keys for looped Livewire markup
- **Do:** Give each looped Livewire element a stable wire:key based on entity identity and namespace keys when different loops could collide.
- **Flag when:** Flag when a Livewire loop omits wire:key, uses a changing index, or duplicates keys across entities.
- **Why:** Stable unique keys let DOM morphing track the correct item after reordering or filtering.
- **Enforced by:** manual

Before:
```php
@foreach($posts as $post)<div>{{ $post->title }}</div>@endforeach
```

After:
```php
@foreach($posts as $post)<div wire:key="post-{{ $post->id }}">{{ $post->title }}</div>@endforeach
```

#### Render one root element per Livewire component
- **Do:** Wrap each Livewire component template in exactly one root HTML element.
- **Flag when:** Flag when a Livewire component template renders multiple sibling roots.
- **Why:** Livewire needs a stable root for its component DOM boundary.
- **Enforced by:** manual

Before:
```php
<h1>Title</h1><p>Body</p>
```

After:
```php
<div><h1>Title</h1><p>Body</p></div>
```

#### Initialize Livewire state in lifecycle hooks
- **Do:** Initialize component state in mount and use updated-property hooks for reactive side effects rather than component constructors.
- **Flag when:** Flag when a Livewire component defines __construct to accept parameters or initialize state instead of using mount() (or boot()/updated hooks for per-request and reactive logic).
- **Why:** Lifecycle hooks run at the points Livewire expects during hydration and updates.
- **Enforced by:** manual

Before:
```php
public function __construct() { $this->title = 'New'; }
```

After:
```php
public function mount(): void { $this->title = 'New'; }
```

#### Use the Tailwind 4 CSS import
- **Do:** Import Tailwind 4 with @import "tailwindcss" rather than the v3 @tailwind layer directives.
- **Flag when:** Flag when a Tailwind 4 CSS entry still relies on @tailwind base/components/utilities.
- **Why:** The v4 entrypoint loads the supported engine layers.
- **Enforced by:** manual

Before:
```php
@tailwind base; @tailwind components; @tailwind utilities;
```

After:
```php
@import "tailwindcss";
```

#### Run and supervise the production Inertia SSR service
- **Do:** Run the production Inertia SSR service under a process supervisor and keep its lifecycle tied to deployment.
- **Flag when:** Flag when SSR is enabled in configuration but no supervised rendering service is started or restarted for the deployed bundle.
- **Why:** Configuration alone cannot render pages; a running service must load the current server bundle.
- **Enforced by:** manual

#### Load one Alpine instance with Livewire
- **Do:** Use Livewire bundled Alpine and its bundled plugins; remove duplicate Alpine imports, CDN scripts, and overlapping plugins.
- **Flag when:** Flag when a Livewire page loads a second Alpine instance or re-registers a plugin already bundled by its installed Livewire version.
- **Why:** Duplicate instances cause initialization errors and inconsistent state.
- **Enforced by:** manual

Before:
```php
import Alpine from 'alpinejs'; Alpine.start();
```

After:
```php
@livewireScripts {{-- supplies the installed Livewire/Alpine runtime --}}
```

#### Provide fallbacks for optional layout slots
- **Do:** Guard optional slots and provide fallback content; use hasActualContent when comment-only content should count as empty.
- **Flag when:** Flag when a reusable layout/component echoes an optional named slot or variable without a fallback (??, @props default, isset/$slots->has()), so omitting it throws an undefined-variable error or renders an empty wrapper.
- **Why:** Pages can omit optional regions without undefined-variable errors or empty wrappers.
- **Enforced by:** manual

Before:
```php
<title>{{ $title }}</title>
```

After:
```php
<title>{{ $title ?? config('app.name') }}</title>
```

#### Build and deliver production frontend assets
- **Do:** Run the production asset build (e.g. npm ci && npm run build) during deployment and ship the generated public/build manifest and assets before serving pages that use @vite.
- **Flag when:** Flag when a deployment serves @vite pages without a successful, current production build (missing or stale public/build/manifest.json) or ships a leftover public/hot file.
- **Why:** Missing manifests or assets break page rendering and asset URLs.
- **Enforced by:** manual

#### Keep Vite entry declarations and template references aligned
- **Do:** Declare every @vite entry in the Laravel Vite plugin input configuration and keep the paths synchronized.
- **Flag when:** Flag when a template references a source entry absent from the configured Vite inputs and production manifest.
- **Why:** Only built entries can be resolved from the production manifest.
- **Enforced by:** manual

Before:
```php
@vite('resources/js/admin.js') // not configured
```

After:
```php
laravel({ input: ['resources/js/admin.js'] })
```

#### Set the asset origin for CDN builds
- **Do:** Set ASSET_URL to the origin serving the built assets when building and deploying assets to a CDN.
- **Flag when:** Flag when a CDN deployment builds or emits asset URLs against the application origin or another incorrect asset origin.
- **Why:** The build and Laravel must agree on where the generated files are served.
- **Enforced by:** manual

#### Use lifecycle-managed Inertia polling
- **Do:** Use usePoll for periodic page refreshes and stop or delay its automatic start when polling is not yet needed.
- **Flag when:** Flag when a page starts unmanaged intervals that survive unmounting or polls before its required user action.
- **Why:** Lifecycle-managed polling avoids orphan timers and unnecessary requests.
- **Enforced by:** manual

Before:
```php
setInterval(() => router.reload(), 5000);
```

After:
```php
usePoll(5000, { only: ['stats'] });
```

#### Prefer direct wire access over duplicated entangled state
- **Do:** Read and write Livewire properties through $wire for routine Alpine interaction and avoid new @entangle or unnecessary entangle state.
- **Flag when:** Flag when Alpine duplicates routine Livewire state with deprecated @entangle or unnecessary synchronization.
- **Why:** Direct access avoids maintaining two copies of the same state.
- **Enforced by:** manual

Before:
```php
<div x-data="{ open: @entangle('open') }">
```

After:
```php
<div x-data><button @click="$wire.open = !$wire.open">Toggle</button></div>
```

#### Preserve client template expressions when composing Blade and JavaScript
- **Do:** Preserve client template expressions when composing Blade and JavaScript.
- **Flag when:** Flag when Blade compiles double-curly expressions intended for a client framework.
- **Why:** Escaped echoes or a verbatim block preserve the client template syntax.
- **Enforced by:** manual

Before:
```php
<span>{{ clientName }}</span>
```

After:
```php
<span>@{{ clientName }}</span>
```

#### Describe when and why an MCP tool should be used
- **Do:** Describe when and why an MCP tool should be used.
- **Flag when:** Flag when an MCP tool has no meaningful Description attribute explaining its purpose and invocation conditions.
- **Why:** The client needs enough context to select the right tool; descriptions are not inferred automatically.
- **Enforced by:** manual

Before:
```php
#[Description('Tool')]
```

After:
```php
#[Description('Search customer records when a user needs to find an existing customer by name')]
```

### HTTP / routing / APIs

#### Configure middleware, exception handling and route loading in bootstrap/app.php in streamlined applications
- **Do:** Configure middleware, exception handling and route loading in bootstrap/app.php in streamlined applications.
- **Flag when:** Flag when a streamlined application places its active HTTP bootstrap configuration in an unused legacy kernel.
- **Why:** The streamlined application builder owns these registrations.
- **Enforced by:** partial: shift:slimmer/bootstrap

Before:
```php
// app/Http/Kernel.php in an app that never loads this kernel
```

After:
```php
// bootstrap/app.php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->alias(['paid' => EnsurePaid::class]);
})
```

#### Customize packages through supported overrides rather than vendor edits
- **Do:** Customize packages through supported published resources or extension points rather than editing vendor files.
- **Flag when:** Flag when files under vendor/ (code, views, styles or config) are modified instead of using the package's configuration, published resources, resources/views/vendor overrides, or documented extension points.
- **Why:** Dependency installation overwrites vendor edits.
- **Enforced by:** manual

Before:
```php
// vendor/tallstackui/...: custom styles edited here
```

After:
```php
// resources/views/vendor/package/card.blade.php
<div class="application-card">{{ $slot }}</div>
```

#### Declare development tools in Composer require-dev
- **Do:** Declare development tools in Composer require-dev.
- **Flag when:** Flag when development-only tools are declared as runtime requirements.
- **Why:** Development tools need not be deployed to production.
- **Enforced by:** manual

Before:
```php
{"require":{"phpstan/phpstan":"^2.0"}}
```

After:
```php
{"require-dev":{"phpstan/phpstan":"^2.0"}}
```

#### Do not store request-specific state in Octane singletons or static properties
- **Do:** Do not store request-specific state in Octane singletons or static properties.
- **Flag when:** Flag when request-specific mutable data survives in a singleton or static variable across Octane requests.
- **Why:** Long-lived workers reuse memory between users and requests.
- **Enforced by:** partial: larastan:OctaneCompatibilityRule

Before:
```php
static $user;
$user ??= request()->user();
```

After:
```php
$user = request()->user();
```

#### Protect authenticated application pages with auth and verified middleware
- **Do:** Protect pages that require a signed-in user with auth middleware, and add verified middleware where the application requires a verified email address.
- **Flag when:** Flag when pages requiring signed-in verified users omit auth or verified middleware.
- **Why:** Both identity and verification requirements must be enforced.
- **Enforced by:** manual

Before:
```php
Route::get('/dashboard', $handler);
```

After:
```php
Route::get('/dashboard', $handler)->middleware(['auth', 'verified']);
```

#### Explicitly render API exceptions as JSON when the API contract requires it regardless of Accept headers
- **Do:** Explicitly render API exceptions as JSON when the API contract requires it regardless of Accept headers.
- **Flag when:** Flag when API failures return HTML when a client omits Accept: application/json.
- **Why:** An explicit route-aware exception policy keeps the error contract consistent.
- **Enforced by:** manual

Before:
```php
// API errors rely only on the Accept header.
```

After:
```php
->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->shouldRenderJsonWhen(fn (Request $request, Throwable $e) => $request->is('api/*') || $request->expectsJson());
})
```

#### Check expected HTTP status codes or throw before consuming an outbound success payload
- **Do:** Check expected HTTP status codes or throw before consuming an outbound success payload.
- **Flag when:** Flag when a 4xx or 5xx response is decoded and used as if it were successful.
- **Why:** Laravel HTTP responses do not throw for ordinary error statuses unless instructed.
- **Enforced by:** manual

Before:
```php
$id = Http::post($url, $data)->json('id');
```

After:
```php
$id = Http::post($url, $data)->throw()->json('id');
```

#### In a custom login flow, call $request->session()->regenerate() after a successful login to prevent session fixation
- **Do:** In a custom login flow, call $request->session()->regenerate() after a successful login to prevent session fixation.
- **Flag when:** Flag when a custom successful login does not regenerate the session identifier.
- **Why:** Session regeneration prevents session fixation.
- **Enforced by:** manual

Before:
```php
Auth::login($user);
```

After:
```php
Auth::login($user);
$request->session()->regenerate();
```

#### Return the continuing middleware or pipeline result
- **Do:** Return the next invocation result, or the framework-supported response alternative, from each continuing middleware or pipeline stage.
- **Flag when:** Flag when middleware or a pipeline stage calls the next callback but discards its result or returns an unsupported value.
- **Why:** Dropping the result breaks the pipeline’s response contract.
- **Enforced by:** manual

Before:
```php
public function handle($value, $next) { $next($value); }
```

After:
```php
public function handle($value, $next) { return $next($value); }
```

#### Modify responses in after middleware; reserve terminable middleware for post-response processing
- **Do:** Modify responses in after middleware; reserve terminable middleware for post-response processing.
- **Flag when:** Flag when terminate() is used to modify a response already sent to the client.
- **Why:** Response changes must occur before the response is sent.
- **Enforced by:** manual

Before:
```php
public function terminate($request, $response) { $response->headers->set('X-App', '1'); }
```

After:
```php
public function handle($request, Closure $next) { $response = $next($request); $response->headers->set('X-App', '1'); return $response; }
```

#### Resolve current request/container/config dependencies at use time rather than retaining stale instances in Octane singletons
- **Do:** Resolve current request/container/config dependencies at use time rather than retaining stale instances in Octane singletons.
- **Flag when:** Flag when an Octane singleton retains a request, container or config repository from an earlier lifecycle.
- **Why:** Long-lived services must resolve the active request/config context at use time.
- **Enforced by:** partial: larastan:OctaneCompatibilityRule

Before:
```php
public function __construct(private Request $request) {}
```

After:
```php
public function __construct(private Closure $requestResolver) {}
// ($this->requestResolver)() resolves the current request.
```

#### Use HasMiddleware::middleware() for controller-owned middleware in new streamlined applications
- **Do:** In Laravel 11+ applications, declare controller-owned middleware with the HasMiddleware interface's static middleware() method or the #[Middleware] attribute instead of calling $this->middleware() in the constructor.
- **Flag when:** Flag when a controller calls $this->middleware() in its constructor and its base controller does not extend Illuminate\Routing\Controller (the Laravel 11+ skeleton default).
- **Why:** Static middleware declarations avoid constructor-time request assumptions.
- **Enforced by:** shift:upgrade-12/controller-middleware

Before:
```php
public function __construct() { $this->middleware('auth'); }
```

After:
```php
class OrderController implements HasMiddleware { public static function middleware(): array { return ['auth']; } }
```

#### Rate-limit abuse-prone and resource-intensive endpoints with workload-appropriate keys and quotas
- **Do:** Rate-limit abuse-prone and resource-intensive endpoints with workload-appropriate keys and quotas.
- **Flag when:** Flag when authentication/API/expensive operations have no workload-appropriate rate limits.
- **Why:** Throttles bound abuse and protect shared application capacity.
- **Enforced by:** manual

Before:
```php
Route::post('/export', ExportController::class);
```

After:
```php
Route::post('/export', ExportController::class)->middleware('throttle:exports');
```

#### Apply required authentication and verification middleware to protected Folio pages
- **Do:** Apply required authentication and verification middleware to protected Folio pages.
- **Flag when:** Flag when a private Folio page can be reached without its required auth or verified middleware.
- **Why:** File-based routing still needs explicit access controls.
- **Enforced by:** manual

Before:
```php
<?php // private Folio page without middleware
```

After:
```php
<?php
use function Laravel\Folio\middleware;
middleware(['auth', 'verified']);
?>
```

#### Prepare PHP data before rendering and use context-safe encoding when passing it to JavaScript
- **Do:** Prepare PHP data before rendering and use context-safe encoding when passing it to JavaScript.
- **Flag when:** Flag when PHP values are interpolated into executable JavaScript with unsafe or complex JSON expressions.
- **Why:** Context-aware encoding prevents syntax errors and script injection.
- **Enforced by:** manual

Before:
```php
<script>const value = {!! json_encode($value) !!};</script>
```

After:
```php
<script>const value = {{ Js::from($value) }};</script>
```

#### Never trust user-supplied fields like user_id, role or is_admin from the request
- **Do:** Never trust user-supplied fields like user_id, role or is_admin from the request.
- **Flag when:** Flag when client input controls ownership or privileged fields without authorization.
- **Why:** Ownership and privilege must be derived from trusted server context.
- **Enforced by:** manual

Before:
```php
Order::create($request->all());
```

After:
```php
$request->user()->orders()->create($request->safe()->only(['description']));
```

#### Use $request->input() instead of $request->get(); input() supports dot notation for nested values
- **Do:** Read request data with $request->input() (or query(), post(), string(), integer(), boolean(), etc.) instead of the deprecated Symfony-inherited $request->get(); input() also supports dot notation for nested values.
- **Flag when:** Flag any $request->get(...) or Request::get(...) call on Illuminate\Http\Request (deprecated in Laravel 13).
- **Why:** input() has Laravel’s explicit input behavior and supports dotted keys.
- **Enforced by:** rector:RenameMethodRector, shift:public/laravel13/request-input

Before:
```php
$city = $request->get('address.city');
```

After:
```php
$city = $request->input('address.city');
```

#### Review browser-facing SPA and component payloads for sensitive fields as well as public API responses
- **Do:** Review browser-facing SPA and component payloads for sensitive fields as well as public API responses.
- **Flag when:** Flag when SPA/Inertia/Livewire payloads expose fields considered private merely because they are not public API responses.
- **Why:** All browser-delivered data is visible to the user.
- **Enforced by:** manual

Before:
```php
return Inertia::render('Users', ['user' => $user->makeVisible('api_token')]);
```

After:
```php
return Inertia::render('Users', ['user' => $user->only(['id', 'name'])]);
```

#### Have users confirm their two-factor setup by submitting a valid TOTP code to POST /user/confirmed-two-factor-authentication
- **Do:** Have users confirm their two-factor setup by submitting a valid TOTP code to POST /user/confirmed-two-factor-authentication.
- **Flag when:** Flag when two-factor authentication is enabled before proving the user can generate a valid TOTP.
- **Why:** Confirmation prevents locking users into an unusable second factor.
- **Enforced by:** manual

Before:
```php
// Mark two-factor enabled immediately after generating a secret.
```

After:
```php
// Require a valid TOTP confirmation before marking setup complete.
```

#### Name the email verification link handler route verification.verify and assign it both the auth and signed middleware
- **Do:** Name the email verification link handler route verification.verify and assign it both the auth and signed middleware.
- **Flag when:** Flag when email-verification links lack signature or authenticated-user checks.
- **Why:** Both checks bind the link to an untampered authenticated verification flow.
- **Enforced by:** manual

Before:
```php
Route::get('/email/verify/{id}/{hash}', $handler);
```

After:
```php
Route::get('/email/verify/{id}/{hash}', $handler)->middleware(['auth', 'signed'])->name('verification.verify');
```

#### Submit Livewire forms through wire:submit
- **Do:** When using livewire/livewire, submit forms with wire:submit on the <form> element (rather than wire:click on a button) so Livewire disables inputs during the request and prevents double submission.
- **Flag when:** Flag when form submission uses only a button click handler instead of the form submit event.
- **Why:** wire:submit supports keyboard submission and in-flight submission protection.
- **Enforced by:** manual

Before:
```php
<form><button wire:click="save">Save</button></form>
```

After:
```php
<form wire:submit="save"><button type="submit">Save</button></form>
```

#### Configure the API URI prefix centrally in route loading
- **Do:** Let the API route loader apply the API prefix (withRouting(api: ...) in bootstrap/app.php, default apiPrefix 'api'); don't repeat the prefix on individual routes in routes/api.php or expect apiResource() to add it.
- **Flag when:** Flag when API routes repeat prefixes individually or rely on apiResource() to supply one.
- **Why:** A central prefix prevents inconsistent or duplicated API paths.
- **Enforced by:** manual

Before:
```php
// routes/api.php
Route::apiResource('api/posts', PostController::class);
```

After:
```php
// bootstrap/app.php: withRouting(api: __DIR__.'/../routes/api.php') applies the /api prefix (change it with apiPrefix:)
// routes/api.php
Route::apiResource('posts', PostController::class);
```

#### Isolate production, staging and preview environments with separate resources and deployment settings
- **Do:** Isolate production, staging and preview environments with separate resources and deployment settings.
- **Flag when:** Flag when nonproduction deployments share writable production resources or production deployment settings.
- **Why:** Environment isolation prevents testing and preview traffic from affecting production data.
- **Enforced by:** manual

#### Leave SESSION_DOMAIN null unless you need cross-subdomain cookies, and never set it to an IP or mismatched domain
- **Do:** Leave SESSION_DOMAIN null unless you need cross-subdomain cookies, and never set it to an IP or mismatched domain.
- **Flag when:** Flag when the session cookie sets a Domain without a cross-subdomain need or uses a mismatched/IP domain.
- **Why:** Host-only cookies minimize scope and avoid browser rejection.
- **Enforced by:** manual

Before:
```php
'domain' => '.example.com', // only one host needs this session
```

After:
```php
'domain' => null,
```

#### Register specific routes before resource parameters
- **Do:** Register explicit routes such as /receipts/destroy-multiple before Route::apiResource()/Route::resource() so the {parameter} route does not capture them.
- **Flag when:** Flag when a resource parameter route shadows an explicit static endpoint.
- **Why:** Specific routes must be registered before matching parameter routes.
- **Enforced by:** manual

Before:
```php
Route::apiResource('receipts', ReceiptController::class);
Route::delete('receipts/destroy-multiple', BulkDestroyController::class);
```

After:
```php
Route::delete('receipts/destroy-multiple', BulkDestroyController::class);
Route::apiResource('receipts', ReceiptController::class);
```

#### Decode third-party JSON payloads with json_decode(..., flags: JSON_THROW_ON_ERROR) so malformed payloads fail loudly
- **Do:** Decode third-party JSON payloads with json_decode(..., flags: JSON_THROW_ON_ERROR) so malformed payloads fail loudly.
- **Flag when:** Flag when third-party JSON decoding silently returns null on malformed data.
- **Why:** Explicit decoding errors keep invalid payloads from masquerading as valid empty results.
- **Enforced by:** manual

Before:
```php
$data = json_decode($json, true);
```

After:
```php
$data = json_decode($json, true, flags: JSON_THROW_ON_ERROR);
```

#### Allowlist external redirect destinations or map configured destination identifiers to URLs
- **Do:** Allowlist external redirect destinations or map configured destination identifiers to URLs.
- **Flag when:** Flag when user input controls an unrestricted external redirect destination.
- **Why:** An allowlist or destination identifier prevents open redirects.
- **Enforced by:** manual

Before:
```php
return redirect()->away($request->input('next'));
```

After:
```php
$destination = config('redirects.allowed')[$request->input('destination')] ?? abort(422);
return redirect()->away($destination);
```

#### Store intended post-login redirects in the session instead of trusting arbitrary redirect parameters
- **Do:** Store intended post-login redirects in the session instead of trusting arbitrary redirect parameters.
- **Flag when:** Flag when post-login redirects trust arbitrary request parameters.
- **Why:** Session-managed intended destinations reduce attacker control.
- **Enforced by:** manual

Before:
```php
return redirect($request->input('redirect'));
```

After:
```php
return redirect()->intended(route('dashboard'));
```

#### Stream uploads exceeding Lambda's request limit directly from the browser to S3
- **Do:** Stream uploads exceeding Lambda's request limit directly from the browser to S3.
- **Flag when:** Flag when uploads exceeding the platform request-size limit pass through the Lambda application.
- **Why:** Direct object-storage uploads bypass the application body limit.
- **Enforced by:** manual

Before:
```php
// Browser posts the large file to the application controller.
```

After:
```php
return Storage::disk('s3')->temporaryUploadUrl('uploads/'.$uploadId, now()->addMinutes(5)); // client sends returned headers
```

#### Do not declare optional parameters before required parameters
- **Do:** Declare required parameters before optional ones; a default value on a parameter that is followed by a required parameter is unreachable and deprecated since PHP 8.0.
- **Flag when:** Flag when a function, method or closure declares a parameter with a default value before a required parameter.
- **Why:** Consistent formatting keeps code review focused on behavior.
- **Enforced by:** pint:no_unreachable_default_argument_value

Before:
```php
function f($optional = null, $required) {}
```

After:
```php
function f($required, $optional = null) {}
```

#### Use the appropriate browser or API middleware group
- **Do:** Define browser routes in `routes/web.php` and API routes in `routes/api.php`, relying on Laravel to apply the `web` and `api` middleware groups to those files automatically.
- **Flag when:** Flag when browser session routes and stateless API routes are placed in groups with incompatible middleware.
- **Why:** Route groups supply the appropriate session/forgery behavior.
- **Enforced by:** manual

Before:
```php
// routes/api.php: session-dependent HTML checkout page.
```

After:
```php
// routes/web.php
Route::get('/checkout', CheckoutController::class)->middleware('auth');
```

#### Set Vary headers for request headers that change the response representation
- **Do:** Set Vary headers for request headers that change the response representation.
- **Flag when:** Flag when a response changes with a request header without a matching cache variant.
- **Why:** Vary prevents a cache from serving the wrong representation.
- **Enforced by:** manual

Before:
```php
// Response differs by Accept-Language but omits Vary.
```

After:
```php
return response($body)->header('Vary', 'Accept-Language');
```

#### Keep slash-containing route captures in the final segment
- **Do:** When a route must capture nested slash-separated paths, use a final wildcard parameter with an appropriate where constraint; keep subsequent path components out of that route design.
- **Flag when:** Flag nested-path captures without a slash-matching constraint or slash-containing parameters before another route segment.
- **Why:** Laravel supports encoded forward slashes only within the final route segment.
- **Enforced by:** manual

#### Use apiResource() for API-only resources and configure the API prefix through route loading
- **Do:** Use apiResource() for API-only resources and configure the API prefix through route loading.
- **Flag when:** Flag when API-only resources expose HTML create/edit routes or assume apiResource adds an API prefix.
- **Why:** API resources omit form pages; prefixing belongs to the route group or application route configuration.
- **Enforced by:** manual

Before:
```php
Route::resource('posts', PostController::class);
```

After:
```php
// routes/api.php, loaded with the API prefix
Route::apiResource('posts', PostController::class);
```

#### Handle expected alternative external responses explicitly when graceful degradation is required
- **Do:** Handle expected alternative external responses explicitly when graceful degradation is required.
- **Flag when:** Flag when a known optional dependency failure produces an unhandled exception or a success-shaped invalid payload.
- **Why:** Explicit fallback behavior keeps expected dependency failures within the application contract.
- **Enforced by:** manual

Before:
```php
return Http::get($recommendationsUrl)->throw()->json();
```

After:
```php
try {
    $response = Http::connectTimeout(3)->timeout(5)->get($recommendationsUrl);
} catch (\Illuminate\Http\Client\ConnectionException) {
    return [];
}

return $response->successful() ? $response->json() : [];
```

#### Authenticate web-exposed MCP servers unless they are intentionally public
- **Do:** Authenticate web-exposed MCP servers unless they are intentionally public.
- **Flag when:** Flag when a nonpublic MCP endpoint is registered without authentication middleware or OAuth protection.
- **Why:** MCP tools and resources can expose application capabilities and data.
- **Enforced by:** manual

Before:
```php
Mcp::web('/mcp', PrivateServer::class);
```

After:
```php
Mcp::web('/mcp', PrivateServer::class)->middleware('auth:sanctum');
```

#### Terminate TLS at the web server or proxy, redirect HTTP to HTTPS and set a Strict-Transport-Security header
- **Do:** Terminate TLS at the web server or proxy, redirect HTTP to HTTPS and set a Strict-Transport-Security header.
- **Flag when:** Flag when public traffic permits plaintext HTTP or lacks the intended HTTPS/HSTS policy.
- **Why:** TLS protects traffic in transit and HSTS reduces downgrade exposure.
- **Enforced by:** manual

#### Set production session cookies to Secure and HttpOnly with a suitable SameSite policy, using Lax by default
- **Do:** Set production session cookies to Secure and HttpOnly with a suitable SameSite policy, using Lax by default.
- **Flag when:** Flag when production session cookies lack Secure, HttpOnly or the intended SameSite protection.
- **Why:** Cookie attributes limit transport and script exposure.
- **Enforced by:** manual

Before:
```php
'secure' => false, 'http_only' => false, 'same_site' => null,
```

After:
```php
'secure' => true, 'http_only' => true, 'same_site' => 'lax',
```

#### Type-hint the Request in controller methods rather than injecting it into the controller constructor
- **Do:** Type-hint the Request in controller methods rather than injecting it into the controller constructor.
- **Flag when:** Flag when a controller constructor depends on current request state before route middleware runs.
- **Why:** Action injection resolves the request at the point the endpoint executes.
- **Enforced by:** manual

Before:
```php
public function __construct(Request $request) { $this->user = $request->user(); }
```

After:
```php
public function store(Request $request) { $user = $request->user(); }
```

#### Do not pass complex or multi-line expressions to Blade directives such as @json or {{ }}; they are regex-based and can break
- **Do:** Prepare complex values first (controller, view model or @php block) and pass only existing variables to Js::from()/@json and other Blade directive arguments; avoid multi-line or complex expressions inside regex-parsed directives.
- **Flag when:** Flag when Js::from(), @json or another Blade directive receives a multi-line or complex inline expression (closures, nested calls, array literals) instead of a prepared variable.
- **Why:** Preparing values first avoids parser ambiguity and keeps templates readable.
- **Enforced by:** manual

Before:
```php
@json(array_map(fn ($x) => complicated($x), $items))
```

After:
```php
@json($preparedItems)
```

#### Group routes by access requirements
- **Do:** Register routes inside middleware groups by access level (guest, auth, auth:guard, verified, can:...) so every new route gets an explicit access decision; if access is declared on the controller instead (#[Middleware], HasMiddleware, #[Authorize]), keep it equally explicit.
- **Flag when:** Flag when a new non-public route is registered outside any auth/authorization middleware group and neither its controller nor the action declares authentication or authorization.
- **Why:** Grouping by access level makes access decisions visible during route review.
- **Enforced by:** manual

Before:
```php
Route::get('/admin/orders', [OrderController::class, 'index']);
```

After:
```php
Route::middleware(['auth', 'can:view-orders'])->prefix('admin')->group(fn () => Route::get('/orders', [OrderController::class, 'index']));
```

#### Authenticate signed API and webhook payloads before processing them
- **Do:** Authenticate signed API and webhook payloads before processing them.
- **Flag when:** Flag when trusted API/webhook data is consumed before its HMAC signature is validated.
- **Why:** A signature authenticates the payload and detects tampering.
- **Enforced by:** manual

Before:
```php
$handler->handle($request->json()->all());
```

After:
```php
abort_unless(hash_equals($expectedSignature, $providedSignature), 403);
$handler->handle($payload);
```

#### Avoid request-dependent URL helpers in configuration
- **Do:** Do not call url() or asset() helpers inside config files (e.g. config/app.php); the request/URL generator is not available when config is loaded.
- **Flag when:** Flag when configuration files call request-dependent url() or asset() helpers during bootstrap.
- **Why:** Configuration must load before a current request URL generator is reliably available.
- **Enforced by:** manual

Before:
```php
'logo' => asset('logo.svg'),
```

After:
```php
'logo_path' => 'logo.svg', // resolve with asset() at rendering time
```

#### Remove obsolete translation roots after migration
- **Do:** After upgrading to Laravel 9+, remove the old resources/lang directory once translations live in the root lang/ directory, because resources/lang takes precedence when it exists.
- **Flag when:** Flag when translations are moved to lang/ but an obsolete resources/lang tree still takes precedence.
- **Why:** Duplicate language roots can hide the migrated translations.
- **Enforced by:** partial: shift:upgrade-9/lang-folder

#### Use Laravel request, server and session APIs instead of PHP request/session superglobals
- **Do:** Use Laravel request, server and session APIs instead of PHP request/session superglobals.
- **Flag when:** Flag when application request/session handling reads or writes raw PHP superglobals.
- **Why:** Framework APIs preserve Laravel’s request normalization and session lifecycle.
- **Enforced by:** rector:RequestVariablesToRequestFacadeRector, rector:ServerVariableToRequestFacadeRector, rector:SessionVariableToSessionFacadeRector

Before:
```php
$name = $_POST['name'];
$_SESSION['name'] = $name;
```

After:
```php
$name = $request->input('name');
$request->session()->put('name', $name);
```

#### Return generic login failures without flashing secrets
- **Do:** On a failed login attempt, redirect back with a generic "credentials do not match our records" error and re-flash only the email input using onlyInput('email').
- **Flag when:** Flag when failed login responses disclose which credential was wrong or reflash passwords.
- **Why:** Generic errors avoid account enumeration and safe flashing avoids secret persistence.
- **Enforced by:** manual

Before:
```php
return back()->withErrors(['email' => 'User does not exist'])->withInput();
```

After:
```php
return back()->withErrors(['email' => 'Credentials do not match our records'])->onlyInput('email');
```

#### Include Blade-only static assets in the Vite build and reference their versioned URLs
- **Do:** With Laravel Vite plugin 3 / Vite 8, list Blade-only static assets in the Laravel plugin assets option, run the production build, and reference their versioned URLs with Vite::asset().
- **Flag when:** Flag Blade-only assets missing from the Laravel Vite plugin assets configuration, legacy import.meta.glob-only inclusion on plugin 3 / Vite 8, or unversioned resource URLs.
- **Why:** Build inclusion and Vite URLs keep assets available and cache-safe.
- **Enforced by:** manual

Before:
```php
// vite.config.js has no assets option; image is referenced only in Blade.
<img src="/resources/images/logo.png">
```

After:
```php
// vite.config.js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
export default defineConfig({ plugins: [laravel({
    input: ['resources/css/app.css', 'resources/js/app.js'],
    assets: ['resources/images/**'],
})] });
// Run npm run build during deployment.
{{-- Blade --}}
<img src="{{ Vite::asset('resources/images/logo.png') }}">
```

#### Keep provider register methods focused on bindings
- **Do:** Do not register event listeners, routes or other functionality in a service provider's `register` method, because you may use a service whose provider has not loaded yet.
- **Flag when:** Flag when provider register() executes route/listener behavior requiring other providers to have booted.
- **Why:** Registration should declare bindings; boot runs after providers register.
- **Enforced by:** manual

Before:
```php
public function register(): void { Route::get('/x', $handler); }
```

After:
```php
public function boot(): void { Route::get('/x', $handler); }
```

#### Do not evaluate code supplied in HTTP requests
- **Do:** Do not evaluate code supplied in HTTP requests.
- **Flag when:** Flag when request-controlled content is executed as PHP or another executable expression.
- **Why:** Executing untrusted code gives attackers application-process capabilities.
- **Enforced by:** manual

Before:
```php
eval($request->input('code'));
```

After:
```php
$data = json_decode($request->input('data'), true, flags: JSON_THROW_ON_ERROR);
```

### Infrastructure & operations

#### Encode server data safely for JavaScript
- **Do:** Encode PHP data embedded in JavaScript with Js::from() or an equivalent context-safe serializer.
- **Flag when:** Flag when Blade templates insert raw server strings or unescaped JSON into script contexts.
- **Why:** HTML and JavaScript have different escaping rules. Context-safe encoding prevents data from becoming executable script.
- **Enforced by:** manual

Before:
```php
<script>const data = {!! json_encode($data) !!};</script>
```

After:
```php
<script>const data = {{ Js::from($data) }};</script>
```

#### Pin GitHub Actions to immutable revisions
- **Do:** Pin external GitHub Actions and reusable workflows to full commit SHAs, with version comments where useful.
- **Flag when:** Flag when an external uses entry refers to a mutable tag or branch instead of a full commit SHA.
- **Why:** Immutable revisions prevent an upstream tag change from silently changing code executed by CI.
- **Enforced by:** manual

#### Store durable user files on object storage
- **Do:** Store user uploads and other durable files on persistent object storage shared by application replicas.
- **Flag when:** Flag when durable files are written only to a release directory or an ephemeral application container filesystem.
- **Why:** Deployments and replica replacement must not destroy user data or make it unavailable to other nodes.
- **Enforced by:** manual

#### Align web and command-line PHP runtimes
- **Do:** Run deployment, Composer, and Artisan commands with the PHP version configured for the site. In Forge use `$FORGE_PHP` or plain `php` (Forge resolves both to the site's version); don't hard-code a different `phpX.Y` binary.
- **Flag when:** Flag when deploy/CI scripts hard-code a PHP CLI version (e.g. php8.2) that differs from the site's PHP-FPM runtime, or build dependencies with a different PHP version than production runs.
- **Why:** Different PHP versions or extensions can make a successful build fail at runtime.
- **Enforced by:** manual

#### Declare nullable parameters explicitly
- **Do:** Use ?T or T|null for parameters that accept null instead of implicit nullability from a null default.
- **Flag when:** Flag when a non-nullable parameter type has a null default, such as string $name = null.
- **Why:** Implicit nullable parameters are deprecated in PHP 8.4 and obscure the actual contract.
- **Enforced by:** pint:nullable_type_declaration_for_default_null_value

Before:
```php
function greet(string $name = null): void {}
```

After:
```php
function greet(?string $name = null): void {}
```

#### Provision Laravel 13 runtime requirements
- **Do:** Deploy Laravel 13 on PHP 8.3 or newer with the framework and application-required extensions installed.
- **Flag when:** Flag when the production web or CLI runtime is below PHP 8.3 or lacks a required extension.
- **Why:** Laravel 13 declares PHP ^8.3 and depends on its required runtime extensions.
- **Enforced by:** manual

#### Rate-limit authentication endpoints
- **Do:** Apply appropriate rate limits to login and other authentication endpoints.
- **Flag when:** Flag when an authentication endpoint accepts unlimited repeated attempts.
- **Why:** Rate limits constrain brute-force and repeated credential attacks.
- **Enforced by:** manual

Before:
```php
Route::post('/login', LoginController::class);
```

After:
```php
RateLimiter::for('login', fn (Request $request) => Limit::perMinute(5)->by($request->ip()));
Route::post('/login', LoginController::class)->middleware('throttle:login');
```

#### Follow server pagination metadata
- **Do:** Drive client pagination from the server's pagination metadata: follow the provided next link or cursor (and stop when it is null), and never compute current page + 1 against a cursor-paginated endpoint.
- **Flag when:** Flag client code that computes the next page as current page + 1 for a cursor- or link-paginated endpoint, or that keeps paging without checking the next link/cursor, has-more or last_page indicator.
- **Why:** The server determines pagination boundaries, and cursor-based results may have no numeric next page.
- **Enforced by:** manual

Before:
```php
$nextPage = $response['current_page'] + 1;
```

After:
```php
$nextUrl = $response['next_page_url'];
if ($nextUrl !== null) { /* request the provided URL */ }
```

#### Scope activity-log suppression to one operation
- **Do:** Use withoutLogging() around explicitly unaudited operations so logging is restored even if the operation throws.
- **Flag when:** Flag when code disables activity logging globally and may fail before re-enabling it.
- **Why:** Scoped suppression prevents an exception from silently disabling later audit records.
- **Enforced by:** manual

Before:
```php
activity()->disableLogging();
importInternalRecords();
activity()->enableLogging();
```

After:
```php
activity()->withoutLogging(fn () => importInternalRecords());
```

#### Prevent script execution in upload storage
- **Do:** Store uploads outside the application web root or on separate storage that never executes server-side scripts.
- **Flag when:** Flag when a user upload can land in an executable PHP location or be handled as server-side code by the web server.
- **Why:** An upload must remain data even when its filename or content resembles a script.
- **Enforced by:** manual

#### Prevent unsafe scheduled task overlap
- **Do:** Apply withoutOverlapping() to scheduled tasks that must not run concurrently, and choose a lock expiry suitable for their runtime.
- **Flag when:** Flag when a non-reentrant scheduled task can still be running when its next invocation starts and has no overlap lock.
- **Why:** An overlap lock prevents concurrent work from corrupting state or duplicating side effects.
- **Enforced by:** manual

Before:
```php
Schedule::command('imports:run')->everyMinute();
```

After:
```php
Schedule::command('imports:run')->everyMinute()->withoutOverlapping();
```

#### Deny web access to sensitive dotfiles
- **Do:** Deny HTTP access to hidden application files while permitting required well-known challenge paths.
- **Flag when:** Flag when the web server can serve .env, .git metadata, or other sensitive dotfiles.
- **Why:** Hidden files can expose secrets or source history if served as static files.
- **Enforced by:** manual

#### Include the S3 filesystem adapter in production dependencies
- **Do:** Declare league/flysystem-aws-s3-v3 as a production dependency before using S3-compatible storage.
- **Flag when:** Flag when production config selects an S3-compatible disk but the S3 adapter is absent or only in require-dev.
- **Why:** The configured disk cannot be constructed without its runtime adapter.
- **Enforced by:** manual

#### Configure S3-compatible endpoints explicitly
- **Do:** Reuse the s3 disk driver for S3-compatible providers and configure their endpoint, credentials, and region rather than assuming AWS defaults.
- **Flag when:** Flag when an S3-compatible disk points to the AWS default endpoint or lacks its provider-specific endpoint settings.
- **Why:** The S3 protocol can be shared while endpoints and connection settings differ.
- **Enforced by:** manual

#### Default-deny inbound server traffic
- **Do:** Keep a default-deny firewall and allow only required service ports; restrict SSH sources where practical.
- **Flag when:** Flag when production disables the default-deny firewall or exposes ports/services it does not need publicly (e.g. database, Redis, Horizon/admin or debug endpoints open to the internet); treat unrestricted SSH as a hardening note when key-only auth is enforced.
- **Why:** A narrow network boundary reduces externally reachable services.
- **Enforced by:** manual

#### Maintain a patch process for the full stack
- **Do:** Keep Composer dependencies, PHP and server/OS software on supported releases and apply security patches promptly (automated OS security updates, regular dependency updates), upgrading before versions reach end of life.
- **Flag when:** Flag when production software remains on known-vulnerable or unsupported releases with no update plan.
- **Why:** Application and operating-system weaknesses can both compromise the deployed service.
- **Enforced by:** manual

#### Redact sensitive data before logging
- **Do:** Remove or redact secrets and sensitive data before writing log messages and context, including logs sent to Nightwatch.
- **Flag when:** Flag when raw passwords, tokens, or sensitive payload values are passed into log calls or telemetry logs.
- **Why:** Nightwatch collects log content as supplied; a remote collector cannot be assumed to sanitize it.
- **Enforced by:** manual

Before:
```php
Log::info('request', $request->all());
```

After:
```php
Log::info('request', $request->except(['password', 'token']));
```

#### Use fallback reads during object-storage migration
- **Do:** When moving object storage to a new disk/provider, either copy all existing objects before switching the disk, or use the `read-through` filesystem driver (Laravel 13.26+; `primary` = new disk, `fallback` = old disk) so missing files are served from, and copied over from, the old provider during migration.
- **Flag when:** Flag when storage is switched to a new bucket before existing objects are copied and no fallback read path exists.
- **Why:** A fallback read path keeps existing files available while migration proceeds.
- **Enforced by:** manual

#### Run the production scheduler every minute
- **Do:** Configure one active schedule:run invocation per minute on each intended scheduler node, or use an equivalent managed scheduler.
- **Flag when:** Flag when the deployed schedule has no active runner or cron runs less frequently than the schedule requires.
- **Why:** Defining tasks does not execute them without a scheduler entry point.
- **Enforced by:** manual

### Miscellaneous

#### Use temporary authorized links for private downloads
- **Do:** Keep private files on a non-public disk and deliver them only after an authorization check, through a short-lived temporaryUrl() or an authorized download response, never a permanent public URL.
- **Flag when:** Flag when private files use permanent public URLs or a temporary URL is issued without checking access.
- **Why:** Expiring URLs limit link lifetime while private storage prevents direct public access.
- **Enforced by:** manual

Before:
```php
return Storage::disk('s3')->url($path);
```

After:
```php
Gate::authorize('download', $document);
return Storage::disk('s3')->temporaryUrl($document->path, now()->addMinutes(5));
```

#### Use cryptographic randomness for secrets
- **Do:** Generate security-sensitive random strings and integers with random_bytes, random_int or Str::random.
- **Flag when:** Flag when tokens, challenges or unpredictable values are generated using rand, mt_rand, str_shuffle or reversible encoding alone.
- **Why:** Predictable randomness lets attackers guess values intended to be secret.
- **Enforced by:** manual

Before:
```php
$token = md5((string) rand());
```

After:
```php
$token = Str::random(40);
```

#### Serialize mutually exclusive work with an atomic lock
- **Do:** Protect operations that must not overlap with a shared atomic lock.
- **Flag when:** Flag when concurrent requests or workers can enter a critical section without mutual exclusion.
- **Why:** An atomic lock prevents concurrent processes from performing incompatible operations simultaneously.
- **Enforced by:** manual

Before:
```php
$this->rebuildReport();
```

After:
```php
Cache::lock('report:rebuild', 120)->block(5, fn () => $this->rebuildReport());
```

#### Define feature flags before checking them
- **Do:** Define each Pennant feature before checking it.
- **Flag when:** Flag when a feature name is checked without an application definition, discoverable feature class or external definition.
- **Why:** Misspelled or undefined flags can silently produce disabled behavior.
- **Enforced by:** manual

Before:
```php
Feature::active('new-checkout');
```

After:
```php
Feature::define('new-checkout', fn (User $user) => $user->is_beta);
Feature::active('new-checkout');
```

#### Supply Laravel-specific context to coding agents
- **Do:** When AI coding agents work on a Laravel app, install Laravel Boost as a dev dependency (composer require laravel/boost --dev; php artisan boost:install) so agents get version-specific docs, application tools and Laravel guidelines, as the skeleton's AGENTS.md instructs.
- **Flag when:** Flag when an AI-agent-assisted Laravel project has no laravel/boost in require-dev or still ships the skeleton's unprocessed bootstrap AGENTS.md/CLAUDE.md (boost:install never run).
- **Why:** Framework-aware context helps agents choose APIs compatible with installed versions.
- **Enforced by:** manual

#### Provide usable MFA recovery codes
- **Do:** Provide one-time MFA recovery codes and let users save them separately from their authenticator.
- **Flag when:** Flag when MFA enrollment completes without a recoverable alternative to the primary device.
- **Why:** Recovery codes reduce lockout without requiring a weak recovery mechanism.
- **Enforced by:** manual

Before:
```php
return view('mfa.enabled');
```

After:
```php
return view('mfa.recovery', ['codes' => $user->recoveryCodes()]);
```

#### Timebox authentication-sensitive failure paths
- **Do:** Use a reviewed minimum-duration timebox for authentication checks whose early failures would reveal account state.
- **Flag when:** Flag when early authentication failures return observably faster than comparable failures that perform credential work.
- **Why:** Timing differences can reveal whether an account or other secret state exists.
- **Enforced by:** manual

Before:
```php
return $this->verifyCredentials($credentials);
```

After:
```php
return $timebox->call(fn () => $this->verifyCredentials($credentials), 200000);
```

#### Represent monetary amounts in integer minor units
- **Do:** Pass Cashier and other payment-API amounts as integer minor units of the currency (e.g. cents), and store/compute money as integer minor units or an exact decimal/money type, never as floats.
- **Flag when:** Flag when floating-point major-unit amounts are stored or passed to a minor-unit payment API.
- **Why:** Integer units avoid binary floating-point rounding and payment scale mistakes.
- **Enforced by:** manual

Before:
```php
$amount = 19.99;
$user->charge($amount, $paymentMethod);
```

After:
```php
$amount = 1999;
$user->charge($amount, $paymentMethod);
```

#### Preserve active filters in pagination URLs
- **Do:** Append the active allowlisted filters and sort values to pagination links.
- **Flag when:** Flag when moving to another page silently loses the current search/filter/sort state.
- **Why:** Stable query state keeps pagination within the same result set.
- **Enforced by:** manual

Before:
```php
$posts->links();
```

After:
```php
$posts->appends(request()->only(['filter', 'sort']))->links();
```

#### Do not expose internal exception messages publicly
- **Do:** Render a safe user-facing message and report internal exception details privately.
- **Flag when:** Flag when a public error response directly includes an internal framework exception message.
- **Why:** Internal messages can reveal queries, paths and configuration details.
- **Enforced by:** manual

Before:
```php
return response()->json(['error' => $e->getMessage()], 500);
```

After:
```php
report($e);
return response()->json(['error' => 'Unable to complete the request.'], 500);
```

#### Keep installed dependency trees out of Git
- **Do:** Ignore vendor and node_modules while versioning the dependency manifests and lockfiles.
- **Flag when:** Flag when installed dependency directories are tracked without a deliberate distribution requirement.
- **Why:** Lockfiles reproduce dependencies without vendoring large generated trees.
- **Enforced by:** manual

Before:
```php
# .gitignore
/storage/*.key
```

After:
```php
# .gitignore
/vendor/
/node_modules/
```

#### Generate OAuth state and PKCE verifiers securely
- **Do:** Generate fresh cryptographic OAuth state and PKCE verifiers, using the required 43–128 unreserved characters for PKCE.
- **Flag when:** Flag when OAuth state is predictable or reused, or a PKCE verifier violates its entropy/format requirement.
- **Why:** These values bind an authorization flow and protect its code exchange.
- **Enforced by:** manual

Before:
```php
$state = (string) time();
$verifier = "fixed";
```

After:
```php
$state = Str::random(40);
$verifier = Str::random(64);
```

#### Return explicit command success and failure codes
- **Do:** When a command can fail, signal it with a non-zero exit code: return Command::FAILURE (self::FAILURE) or call $this->fail(); don't just print an error and fall through to exit code 0.
- **Flag when:** Flag when a command detects or reports a failure (e.g. $this->error(...) or a caught exception) but still finishes with exit code 0.
- **Why:** Shell scripts and schedulers depend on exit status to detect failure.
- **Enforced by:** manual

Before:
```php
public function handle() { if (! $this->import()) { $this->error('Failed'); } }
```

After:
```php
public function handle(): int {
    if (! $this->import()) { return self::FAILURE; }
    return self::SUCCESS;
}
```

#### Keep private uploads outside public storage
- **Do:** Store uploads privately by default and use public visibility only for intentionally public files.
- **Flag when:** Flag when private or unclassified uploads are placed under public/ or on a publicly readable disk.
- **Why:** Private storage prevents direct URL access from bypassing authorization.
- **Enforced by:** manual

Before:
```php
$request->file('document')->store('documents', 'public');
```

After:
```php
$request->file('document')->store('documents', 'local');
```

#### Prefer non-mutating collection transforms when shared state is unnecessary
- **Do:** Use map when a transformed collection should be independent of the original.
- **Flag when:** Flag when transform mutates a collection whose original values are still needed elsewhere.
- **Why:** Returning a new collection avoids unexpected changes through shared references.
- **Enforced by:** manual

Before:
```php
$labels = $items->transform(fn ($item) => $item->name);
```

After:
```php
$labels = $items->map(fn ($item) => $item->name);
```

#### Trust forwarded headers only from controlled proxies
- **Do:** Trust explicit proxy addresses where possible; trust all proxies only when network controls make the origin reachable solely through trusted proxies.
- **Flag when:** Flag when wildcard proxy trust is enabled while clients can reach the origin directly.
- **Why:** Forwarded headers affect IP, scheme and host decisions. Network isolation is necessary for wildcard trust to be safe.
- **Enforced by:** manual

Before:
```php
$middleware->trustProxies(at: '*');
```

After:
```php
$middleware->trustProxies(at: ['10.0.0.10']);
```

#### Use Laravel authentication components
- **Do:** Use Laravel's built-in authentication (guards, Auth::attempt, Hash, password reset, starter kits/Fortify) instead of hand-rolling credential checks, login sessions or MFA; if a custom flow is unavoidable, model it on Fortify's implementation.
- **Flag when:** Flag custom authentication code that compares passwords manually, sets session user IDs by hand, or reimplements login throttling, password reset or MFA that Laravel's auth components already provide, without a documented requirement.
- **Why:** Supported authentication components provide established security behavior and reduce custom attack surface.
- **Enforced by:** manual

Before:
```php
if ($user->password === $request->password) { session(['user_id' => $user->id]); }
```

After:
```php
if (Auth::attempt($request->only('email', 'password'))) {
    $request->session()->regenerate();

    return redirect()->intended('dashboard');
}
```

#### Deny access by default and grant least privilege
- **Do:** Deny capabilities by default and grant only those required by the user or role.
- **Flag when:** Flag when a policy grants broad access by default or a new role receives unrelated privileges.
- **Why:** Explicit minimal grants limit the impact of mistaken configuration and compromised accounts.
- **Enforced by:** manual

Before:
```php
public function delete(User $user, Invoice $invoice): bool { return true; }
```

After:
```php
public function delete(User $user, Invoice $invoice): bool { return $user->can('delete invoices'); }
```

#### Use explicit nullable parameter types
- **Do:** Declare nullable typed parameters explicitly (?T or T|null) instead of relying on a null default to make the type implicitly nullable.
- **Flag when:** Flag when a typed parameter has a null default but its declared type is not nullable, e.g. string $name = null.
- **Why:** Explicit nullability avoids deprecated implicit-nullability behavior and clarifies the contract.
- **Enforced by:** pint:nullable_type_declaration, pint:nullable_type_declaration_for_default_null_value

Before:
```php
function run(string $name = null): void {}
```

After:
```php
function run(?string $name = null): void {}
```

#### Do not treat remote disk paths as local files
- **Do:** Use storage streams or a deliberate local temporary copy when a consumer needs local filesystem access to remote content.
- **Flag when:** Flag when an S3 disk path is passed to a function expecting an absolute local filename.
- **Why:** Storage::path on S3 returns an object key, not a local file.
- **Enforced by:** manual

Before:
```php
$image = imagecreatefromjpeg(Storage::disk('s3')->path($key));
```

After:
```php
$bytes = Storage::disk('s3')->get($key);
$image = imagecreatefromstring($bytes);
```

#### Label form fields explicitly and persistently
- **Do:** Give inputs explicit accessible labels and keep required instructions visible after text entry.
- **Flag when:** Flag when placeholder text is the only field label or explanation.
- **Why:** Placeholders disappear and do not provide a reliable persistent label.
- **Enforced by:** manual

Before:
```php
<input name="email" placeholder="Email">
```

After:
```php
<label for="email">Email</label><input id="email" name="email" type="email">
```

#### Keep authentication factors independently enforced
- **Do:** Prevent possession of one authentication factor from disabling or bypassing the account’s other required factors.
- **Flag when:** Flag when a password-only or alternate-factor flow removes another required factor without the appropriate verification or controlled recovery.
- **Why:** An alternate path must preserve the protection promised by the enrolled factor set.
- **Enforced by:** manual

Before:
```php
$this->verifyPassword($user, $password);
$this->skipRequiredMfa($user);
```

After:
```php
$this->verifyPassword($user, $password);
$this->requireEnrolledSecondFactor($user);
```

### Queues & jobs

#### Queue slow or retryable work durably
- **Do:** Implement ShouldQueue on deferred job/listener classes and dispatch slow or reliability-critical work on a durable asynchronous connection.
- **Flag when:** Flag when a request handler performs slow external I/O (third-party API calls, mail sending, media processing) inline, or when defer(), dispatchAfterResponse() or terminating callbacks are used for work that must survive failures or be retried.
- **Why:** Durable queues decouple response latency from processing and allow recovery after failure. Jobs must still tolerate retries.
- **Enforced by:** manual

Before:
```php
Http::post($url, $payload);
```

After:
```php
// SyncCustomer implements ShouldQueue
SyncCustomer::dispatch($customerId);
```

#### Scope dispatch uniqueness to the logical operation
- **Do:** Implement ShouldBeUnique with a stable uniqueId when duplicate pending work for the same logical operation should be suppressed.
- **Flag when:** Flag when a ShouldBeUnique job whose constructor takes arguments declares no uniqueId (so distinct jobs share one class-wide lock), or when work that must not run twice concurrently for the same resource is dispatched without a uniqueness key.
- **Why:** A logical key suppresses redundant work without collapsing jobs for different resources.
- **Enforced by:** partial: larastan:UniqueJobDeclaresUniqueIdRule

Before:
```php
class Reindex implements ShouldQueue { public function __construct(public int $id) {} }
```

After:
```php
class Reindex implements ShouldQueue, ShouldBeUnique { public function uniqueId(): string { return (string) $this->id; } }
```

#### Enforce mutable invariants atomically
- **Do:** Use database constraints, atomic updates, or transactions to enforce uniqueness and inventory invariants.
- **Flag when:** Flag when correctness relies solely on a prior validation or existence check under concurrent requests.
- **Why:** Concurrent requests can both pass validation before either writes.
- **Enforced by:** manual

Before:
```php
if (! User::where("email", $email)->exists()) User::create($data);
```

After:
```php
$table->unique("email"); // also handle conflict in the write path
```

#### Serialize model identifiers rather than live dependencies
- **Do:** Use SerializesModels, directly or through Foundation Queueable, for queued objects carrying Eloquent models, and keep other payload data serializable.
- **Flag when:** Flag when a queued object embeds full models without model serialization or includes live service/connection objects.
- **Why:** Identifier serialization keeps payloads small and reloads model state at execution.
- **Enforced by:** partial: larastan:JobWithModelPropertyDeclaresSerializesModelsRule

Before:
```php
class ProcessOrder implements ShouldQueue { public function __construct(public Order $order) {} }
```

After:
```php
class ProcessOrder implements ShouldQueue { use \Illuminate\Foundation\Queue\Queueable; public function __construct(public Order $order) {} }
```

#### Inject job services at execution time
- **Do:** Keep queued payloads limited to serializable data and model identifiers. Inject runtime services into container-invoked job/listener handle() methods; for mailables resolve them at delivery time (build() supports method injection), and for notifications use a container-resolved custom channel or resolve services inside the delivery method.
- **Flag when:** Flag when a queued job, listener, mailable or notification serializes a live container service, or when service injection assumes a notification/mailable has a framework-invoked handle() method.
- **Why:** Execution-time injection avoids serializing services and makes dependencies explicit.
- **Enforced by:** manual

Before:
```php
public function __construct(public Mailer $mailer) {}
```

After:
```php
class SendReceipt implements ShouldQueue
{
    use \Illuminate\Foundation\Queue\Queueable;

    public function __construct(public int $orderId) {}

    public function handle(ReceiptSender $sender): void
    {
        $sender->send($this->orderId);
    }
}
```

#### Bound outbound connection and response times
- **Do:** Set explicit connection and response timeouts appropriate to the service and shorter than the caller execution budget.
- **Flag when:** Flag when outbound HTTP or socket operations lack timeouts or can exceed the job runtime budget.
- **Why:** Blocking IO may not honor worker timeouts and can monopolize workers.
- **Enforced by:** manual

Before:
```php
Http::get($url);
```

After:
```php
Http::connectTimeout(3)->timeout(15)->get($url);
```

#### Back off transient job failures
- **Do:** Configure delayed, progressively increasing retries for transient dependency failures.
- **Flag when:** Flag when jobs retry dependency outages immediately or at a sustained aggressive rate.
- **Why:** Backoff reduces retry load and gives the dependency time to recover.
- **Enforced by:** manual

Before:
```php
#[Backoff(0)] class SyncVendor implements ShouldQueue {}
```

After:
```php
#[Backoff([1, 5, 30])] class SyncVendor implements ShouldQueue {}
```

#### Queue external notification delivery
- **Do:** Implement ShouldQueue on notifications whose external delivery need not finish before the response.
- **Flag when:** Flag when notification delivery calls external channels synchronously without a requirement for immediate feedback.
- **Why:** Queued notifications shorten requests and use the existing notification retry mechanism.
- **Enforced by:** manual

Before:
```php
class OrderReady extends Notification {}
```

After:
```php
class OrderReady extends Notification implements ShouldQueue { use Queueable; }
```

#### Make queue side effects idempotent
- **Do:** Guard important job side effects with stable operation identifiers, atomic state transitions, or database constraints, even for unique jobs.
- **Flag when:** Flag when reprocessing a job can repeat an irreversible effect without a duplicate guard.
- **Why:** Acknowledgement can fail after a successful side effect. Cache uniqueness locks do not provide exactly-once execution.
- **Enforced by:** manual

Before:
```php
Payment::create($data);
```

After:
```php
Payment::firstOrCreate(["operation_id" => $operationId], $data); // unique operation_id index
```

#### Handle exhausted jobs in domain state
- **Do:** Implement failed (or jobFailed for queued actions) when exhausted work must update domain state, compensate, or alert with useful context.
- **Flag when:** Flag when a failed background workflow can leave domain status permanently pending without a failure handler.
- **Why:** Failure hooks let the application expose terminal failure and support recovery.
- **Enforced by:** manual

Before:
```php
public function handle(): void { $this->runImport(); }
```

After:
```php
public function failed(?Throwable $e): void { Import::whereKey($this->importId)->update(["status" => "failed"]); }
```

#### Bound job and queued-listener execution time
- **Do:** Bound job and queued-listener execution with a suitable worker --timeout or #[Timeout] override, with PCNTL enabled for worker enforcement. Keep the effective timeout several seconds below retry_after for drivers using that setting, or below the SQS visibility timeout.
- **Flag when:** Flag an unbounded or excessive worker/job timeout, a job that needs more time than its effective timeout allows, or a timeout at or above retry_after (or SQS visibility timeout).
- **Why:** Runtime bounds free stuck workers and make failure timing predictable.
- **Enforced by:** manual

Before:
```php
class Sync implements ShouldQueue {}
```

After:
```php
use Illuminate\Queue\Attributes\Timeout;

#[Timeout(60)]
class Sync implements ShouldQueue {}
// Redis/database/etc.: configure retry_after = 90.
// SQS: configure VisibilityTimeout = 90 in AWS instead.
```

#### Run workers on every routed queue
- **Do:** Configure an asynchronous queue and run matching workers for every routed connection and named queue, including mail, notifications, broadcasting, Scout, and queued commands.
- **Flag when:** Flag when jobs route to a nondefault queue that no worker subscribes to.
- **Why:** Workers process only their configured queues, so routing alone cannot provide execution.
- **Enforced by:** manual

#### Reserve in-process deferral for small disposable work
- **Do:** Use defer or after-response execution only for short work that can be lost without retries.
- **Flag when:** Flag when slow or reliability-critical work runs in defer, dispatchAfterResponse, or a deferred connection.
- **Why:** These mechanisms continue within the application process and do not provide durable queue execution.
- **Enforced by:** manual

Before:
```php
ChargeCard::dispatchAfterResponse($order);
```

After:
```php
ChargeCard::dispatch($order); // durable queue: retries + failed_jobs
defer(fn () => Metrics::reportOrder($order)); // small, disposable
```

#### Do not rely on batch or bulk dispatch uniqueness
- **Do:** Dispatch unique jobs individually through a uniqueness-enforcing dispatch path; provide independent deduplication for batches or bulk operations.
- **Flag when:** Flag when batch or bulk dispatch relies on ShouldBeUnique to suppress duplicate jobs.
- **Why:** Batch constraints do not enforce unique-job locks, and bulk dispatch bypasses the usual pending-dispatch uniqueness path.
- **Enforced by:** partial: larastan:NoBatchedUniqueJobRule

Before:
```php
Bus::batch([new Reindex($id), new Reindex($id)])->dispatch();
```

After:
```php
Reindex::dispatch($id); // Reindex implements ShouldBeUnique
```

#### Use Horizon with Redis-backed queues
- **Do:** Use Horizon only with standalone Redis queue connections; run queue:work under a process monitor for database, SQS or other drivers.
- **Flag when:** Flag when Horizon supervisors reference a non-Redis queue connection or a Redis Cluster connection.
- **Why:** Horizon depends on Redis queue internals and cannot supervise database or SQS workers.
- **Enforced by:** manual

#### Ignore only the trusted current record in unique validation
- **Do:** Exclude the trusted current model from update uniqueness checks; never accept the ignore identifier from request input.
- **Flag when:** Flag when update validation rejects an unchanged value or passes user-controlled data to ignore.
- **Why:** Trusted record exclusion allows unchanged values without exposing query manipulation.
- **Enforced by:** manual

Before:
```php
Rule::unique("users")->ignore($request->input("id"));
```

After:
```php
Rule::unique("users")->ignore($request->user());
```

#### Scope mutable services to the request or job
- **Do:** Bind services holding request/job-specific state as scoped rather than singleton.
- **Flag when:** Flag when a singleton carries actor, tenant, mutable logging, or configuration-derived lifecycle state across executions.
- **Why:** Scoped instances reset between worker lifecycles and prevent cross-request contamination.
- **Enforced by:** manual

Before:
```php
$this->app->singleton(ActorResolver::class);
```

After:
```php
$this->app->scoped(ActorResolver::class);
```

#### Queue mail that does not require immediate feedback
- **Do:** Implement ShouldQueue on mailables intended for background delivery.
- **Flag when:** Flag when ordinary asynchronous mailables are sent inline or wrapped in redundant dispatch jobs.
- **Why:** The mail subsystem already supports queueing and retries.
- **Enforced by:** manual

Before:
```php
class Receipt extends Mailable {}
```

After:
```php
class Receipt extends Mailable implements ShouldQueue { use Queueable; }
```

#### Use retryUntil for a bounded retry window
- **Do:** Define retryUntil when elapsed time is the correct retry limit; omit redundant unlimited tries settings.
- **Flag when:** Flag when a time-bounded workflow uses unlimited tries without a deadline or assumes tries overrides retryUntil.
- **Why:** A deadline bounds retries by business relevance and takes precedence over attempt counts.
- **Enforced by:** manual

Before:
```php
public $tries = 0;
```

After:
```php
public function retryUntil(): DateTime { return now()->addMinutes(10); }
```

#### Reload persistent workers after deployments
- **Do:** Gracefully restart queue workers and Horizon after code or configuration changes, with a monitor to start replacement processes outside managed Cloud restarts.
- **Flag when:** Flag when deployments change loaded code or configuration without reloading persistent workers.
- **Why:** Persistent workers retain their booted code and configuration.
- **Enforced by:** manual

#### Use shared Redis for horizontal Reverb scaling
- **Do:** Enable Reverb scaling and connect all Reverb replicas to a shared Redis Pub/Sub backend.
- **Flag when:** Flag when multiple Reverb replicas cannot exchange broadcasts through a shared backend.
- **Why:** Shared Pub/Sub distributes events to clients attached to other replicas.
- **Enforced by:** manual

#### Bound ordinary job and listener attempts
- **Do:** Set finite job/listener/notification attempts or an explicit retry deadline according to recoverable failure behavior.
- **Flag when:** Flag when jobs retry indefinitely without a deliberate alternative bound.
- **Why:** A retry budget prevents permanently failing work from circulating forever.
- **Enforced by:** manual

Before:
```php
#[Tries(0)] class Sync implements ShouldQueue {}
```

After:
```php
#[Tries(5)] class Sync implements ShouldQueue {}
```

#### Set SQS visibility above job runtime
- **Do:** Configure SQS visibility timeout above the maximum effective job execution time with a safety margin.
- **Flag when:** Flag when an SQS visibility timeout can expire while the first worker still executes the job.
- **Why:** SQS uses visibility timeout rather than Laravel retry_after to control redelivery.
- **Enforced by:** manual

#### Enable a Cloud scheduler and coordinate singleton tasks
- **Do:** Enable scheduler execution on an App or Worker cluster and use onOneServer for tasks that must run once across replicas.
- **Flag when:** Flag when scheduled tasks have no active scheduler or replica schedulers duplicate singleton work.
- **Why:** Scheduler activation and shared locking are both needed for reliable recurring work.
- **Enforced by:** manual

Before:
```php
Schedule::job(new DailyReport)->daily();
```

After:
```php
Schedule::job(new DailyReport)->daily()->onOneServer();
```

#### Use Livewire 3 event dispatch APIs
- **Do:** Use $this->dispatch() (and #[On] or $listeners for listening) instead of the removed emit(), emitTo(), emitUp() and dispatchBrowserEvent() APIs in Livewire 3+ components.
- **Flag when:** Flag when Livewire 3+ components call emit(), emitTo(), emitUp(), emitSelf() or dispatchBrowserEvent().
- **Why:** The current event API integrates server and browser events.
- **Enforced by:** manual

Before:
```php
$this->emit("saved");
```

After:
```php
$this->dispatch("saved");
```

#### Give Eloquent models one identifying primary key
- **Do:** Keep a single-column model primary key and express composite business uniqueness with additional unique indexes.
- **Flag when:** Flag when an Eloquent model depends on a composite primary key.
- **Why:** Eloquent model identity does not support composite primary keys.
- **Enforced by:** manual

Before:
```php
$table->primary(["tenant_id", "code"]);
```

After:
```php
$table->id(); $table->unique(["tenant_id", "code"]);
```

#### Respect the SQS fifteen-minute dispatch delay limit
- **Do:** Keep SQS message delays at or below fifteen minutes and use staged releases toward an absolute deadline for longer waits.
- **Flag when:** Flag when SQS dispatch requests a delay over 900 seconds.
- **Why:** Native SQS delay is bounded; longer business deadlines need another scheduling step.
- **Enforced by:** manual

Before:
```php
CheckStatus::dispatch($id)->delay(now()->addHour());
```

After:
```php
CheckStatus::dispatch($id, $deadline)->delay(min(900, now()->diffInSeconds($deadline)));
```

#### Reset application-owned state between worker lifecycles
- **Do:** Avoid accumulating request or job data in static properties and reset application-owned mutable state at lifecycle boundaries.
- **Flag when:** Flag when persistent workers retain or grow request-specific static data across requests or jobs.
- **Why:** Persistent workers reuse the application and can leak data or memory across users.
- **Enforced by:** manual

Before:
```php
static::$requests[] = $request;
```

After:
```php
// Keep request data in scoped objects; clear owned static caches on lifecycle events.
```

#### Create required queue tables without duplicating skeleton migrations
- **Do:** Ensure configured jobs, failed-job, and batch tables exist before use; reuse skeleton migrations and set custom names before generation.
- **Flag when:** Flag when a database queue/failed store/batch references a missing table or generates a duplicate existing table.
- **Why:** The database drivers need their tables, and duplicate generators can break migrations.
- **Enforced by:** manual

Before:
```php
$table->create("jobs"); // duplicate of skeleton migration
```

After:
```php
// Reuse create_jobs_table; generate only missing tables with configured names.
```

#### Supervise persistent queue managers and workers
- **Do:** Run queue workers or one Horizon manager under a process monitor with automatic startup/restart and appropriate group shutdown behavior.
- **Flag when:** Flag when workers exit after timeout/reload and no process manager replaces them.
- **Why:** A persistent worker command alone does not maintain service after exits.
- **Enforced by:** manual

#### Allow active jobs to finish during process shutdown
- **Do:** Set process-manager stopwaitsecs above the longest supported job runtime.
- **Flag when:** Flag when the process manager kills workers before the longest legitimate job can finish.
- **Why:** A graceful queue restart still needs enough outer shutdown time.
- **Enforced by:** manual

#### Correlate jobs by payload UUID across retries
- **Do:** Use the job payload UUID for lifecycle correlation instead of a backend database row ID.
- **Flag when:** Flag when job tracing keys change when a job is released or retried.
- **Why:** Backend row IDs can change while the payload UUID represents the logical queued execution.
- **Enforced by:** manual

Before:
```php
$traceId = $job->getJobId();
```

After:
```php
$traceId = $job->uuid();
```

#### Keep route names unique
- **Do:** Give every named route a unique name.
- **Flag when:** Flag when two routes register the same name.
- **Why:** Name collisions make URL generation and caching ambiguous.
- **Enforced by:** manual

Before:
```php
Route::get("/a", A::class)->name("show"); Route::get("/b", B::class)->name("show");
```

After:
```php
Route::get("/a", A::class)->name("a.show"); Route::get("/b", B::class)->name("b.show");
```

#### Make pagination order deterministic
- **Do:** Add a unique tie-breaker to nonunique pagination ordering.
- **Flag when:** Flag when a paginated query orders only by columns whose values can tie.
- **Why:** A total ordering prevents tied rows from changing position across pages.
- **Enforced by:** manual

Before:
```php
Post::latest()->paginate();
```

After:
```php
Post::latest()->orderByDesc("id")->paginate();
```

#### Treat unique conflicts as already-completed creation
- **Do:** Back derived-record creation with a unique index and handle the expected unique-constraint race by loading the existing result.
- **Flag when:** Flag when concurrent derived-record creation leaks duplicate-key errors for an otherwise satisfied operation.
- **Why:** The constraint arbitrates concurrent creators and the loser can reuse the winner.
- **Enforced by:** manual

Before:
```php
return DerivedRecord::create($attributes);
```

After:
```php
return DerivedRecord::createOrFirst($key, $attributes); // unique index on the $key columns; on a duplicate it returns the existing row
```

#### Honor batch cancellation before doing work
- **Do:** Check batch cancellation at the start of batchable work or use SkipIfBatchCancelled middleware.
- **Flag when:** Flag when a batch job continues costly or harmful work after its batch is cancelled.
- **Why:** Cancellation metadata does not automatically terminate each pending job.
- **Enforced by:** partial: larastan:BatchableJobChecksCancellationRule

Before:
```php
public function handle(): void { $this->import(); }
```

After:
```php
public function handle(): void { if ($this->batch()?->cancelled()) return; $this->import(); }
```

#### Enqueue transaction-dependent jobs after commit
- **Do:** Use afterCommit, ShouldQueueAfterCommit, or connection after_commit when queued work depends on transaction writes.
- **Flag when:** Flag when a job can run before its required transaction commits.
- **Why:** Workers can otherwise observe missing records or act on data that later rolls back.
- **Enforced by:** partial: larastan:JobDispatchedInTransactionUsesAfterCommitRule

Before:
```php
DB::transaction(fn () => ProcessOrder::dispatch(Order::create($data)));
```

After:
```php
DB::transaction(function () use ($data) { $order = Order::create($data); ProcessOrder::dispatch($order)->afterCommit(); $order->items()->createMany($data['items']); }); // BEFORE: same closure with ProcessOrder::dispatch($order); (no afterCommit)
```

#### Replace legacy job retry configuration names
- **Do:** Rename retryAfter/retryAfter() to backoff/backoff() and timeoutAt/timeoutAt() to retryUntil/retryUntil().
- **Flag when:** Flag when jobs still declare legacy retryAfter or timeoutAt configuration.
- **Why:** Current payload creation reads the current retry configuration names.
- **Enforced by:** rector:RenameMethodRector, rector:RenamePropertyRector

Before:
```php
public function timeoutAt() { return now()->addMinutes(5); }
```

After:
```php
public function retryUntil(): DateTime { return now()->addMinutes(5); }
```

#### Use execution locks for concurrent-resource exclusion
- **Do:** Use WithoutOverlapping keyed by the protected resource for execution exclusion, and call shared() when different job classes can mutate that resource. Use a lock-capable cache shared by workers and an expiry appropriate to the maximum runtime; keep dispatch deduplication separate.
- **Flag when:** Flag concurrent resource mutations without an execution lock, dispatch uniqueness used as the execution mutex, or different job classes using the same resource key without shared().
- **Why:** Dispatch deduplication and execution exclusion protect different windows.
- **Enforced by:** manual

Before:
```php
// Both DebitAccount and ReconcileAccount use this, but their locks are class-scoped.
public function middleware(): array {
    return [new WithoutOverlapping("account:".$this->accountId)];
}
```

After:
```php
// In each job class that modifies the account; import Illuminate\Queue\Middleware\WithoutOverlapping.
public function middleware(): array {
    return [(new WithoutOverlapping("account:".$this->accountId))->shared()];
}
```

#### Reload Octane after application deployments
- **Do:** Gracefully reload Octane workers after code, provider, environment, or configuration deployments.
- **Flag when:** Flag when a deployment leaves existing Octane workers serving old code or configuration.
- **Why:** Octane retains its booted application between requests.
- **Enforced by:** manual

#### Use JSON session serialization when objects are unnecessary
- **Do:** Set session serialization to json when the application session does not require PHP objects.
- **Flag when:** Flag when sessions use PHP object serialization despite containing only JSON-compatible data.
- **Why:** JSON reduces exposure to PHP deserialization gadget chains.
- **Enforced by:** manual

Before:
```php
"serialization" => "php",
```

After:
```php
"serialization" => "json",
```

#### Keep queued payloads within the worker memory budget
- **Do:** Pass compact identifiers and necessary scalar data to jobs; store bulky content externally and reload it during execution. Include payload size when budgeting concurrent workers.
- **Flag when:** Flag serialized full documents or large object graphs that materially inflate worker memory when an identifier or storage reference suffices.
- **Why:** Every concurrent job must deserialize its payload before performing useful work.
- **Enforced by:** manual

Before:
```php
ProcessUpload::dispatch(file_get_contents($path));
```

After:
```php
ProcessUpload::dispatch($storedObjectKey);
```

#### Release uniqueness before processing only intentionally
- **Do:** Use ShouldBeUniqueUntilProcessing when duplicate pending dispatches should be suppressed but new work may enqueue during processing.
- **Flag when:** Flag when ShouldBeUniqueUntilProcessing is used where uniqueness must cover the running job.
- **Why:** The lock is released before handle and does not protect execution overlap.
- **Enforced by:** manual

Before:
```php
class SettleBalance implements ShouldBeUniqueUntilProcessing {}
```

After:
```php
class SettleBalance implements ShouldBeUnique {}
```

#### Read persisted state in failure handlers
- **Do:** Use constructor data or persisted state in failed instead of relying on handle mutations.
- **Flag when:** Flag when failed reads a property that is only set during handle.
- **Why:** Laravel invokes failed on a fresh job instance.
- **Enforced by:** manual

Before:
```php
public function failed(?Throwable $e): void { $this->result->markFailed(); }
```

After:
```php
public function failed(?Throwable $e): void { Result::find($this->resultId)?->markFailed(); }
```

#### Use managed queues for Cloud background workloads
- **Do:** On Laravel Cloud, run jobs that must keep processing on managed queues (recommended) or an always-awake Worker cluster, not on a Scale-to-Zero App cluster or a retired queue cluster.
- **Flag when:** Flag when reliability-critical queued jobs are processed only by an App cluster that can scale to zero, or still depend on a deprecated/retired Cloud queue cluster.
- **Why:** Managed processing separates job execution from App cluster sleep and supplies worker scaling.
- **Enforced by:** manual

#### Reindex filtered or unique collections for JSON lists
- **Do:** Call values after key-preserving transformations when the result must serialize as a JSON array.
- **Flag when:** Flag when filter or unique results with gaps in numeric keys are returned as an API list.
- **Why:** Gapped numeric keys serialize as an object instead of a sequential list.
- **Enforced by:** manual

Before:
```php
return $items->filter($keep);
```

After:
```php
return $items->filter($keep)->values();
```

#### Configure Horizon for every deployed environment
- **Do:** Provide a matching APP_ENV entry or wildcard Horizon environment with the required supervisors.
- **Flag when:** Flag when the deployed APP_ENV has no matching Horizon supervisor configuration.
- **Why:** Horizon otherwise starts without the intended workers.
- **Enforced by:** manual

#### Add Batchable to jobs submitted to batches
- **Do:** Use Illuminate\Bus\Batchable on every job dispatched through Bus::batch.
- **Flag when:** Flag when a batched job lacks Batchable behavior.
- **Why:** Batch tracking attaches a batch identifier through that trait.
- **Enforced by:** larastan:BatchedJobIsBatchableRule

Before:
```php
class ImportPart implements ShouldQueue {}
```

After:
```php
class ImportPart implements ShouldQueue { use Batchable; }
```

#### Replace obsolete framework event-name strings with event classes
- **Do:** Use the corresponding framework event class constants instead of legacy event-name strings.
- **Flag when:** Flag when listeners subscribe to obsolete framework event strings such as illuminate.queue.before or auth.login.
- **Why:** Current framework events are dispatched as class events; obsolete names leave listeners inactive.
- **Enforced by:** rector:EventStringToClassConstantRector

Before:
```php
Event::listen("illuminate.queue.before", $listener);
```

After:
```php
Event::listen(\Illuminate\Queue\Events\JobProcessing::class, $listener);
```

#### Limit real exceptions separately from benign releases
- **Do:** Combine a release-friendly retry budget or deadline with MaxExceptions when repeated execution errors need a tighter bound.
- **Flag when:** Flag when a frequently released job either exhausts attempts before execution or tolerates unbounded actual exceptions.
- **Why:** Releases consume attempts without necessarily indicating a failed execution.
- **Enforced by:** manual

Before:
```php
#[Tries(2)] class PollVendor implements ShouldQueue {}
```

After:
```php
#[Tries(50)] #[MaxExceptions(3)] class PollVendor implements ShouldQueue {}
```

### Security

#### Encrypt recoverable sensitive attributes
- **Do:** Use encrypted casts or equivalent authenticated encryption for stored secrets that must be recovered.
- **Flag when:** Flag when recoverable sensitive values such as OAuth tokens are stored in plaintext.
- **Why:** Encryption limits disclosure from a database-only leak.
- **Enforced by:** manual

Before:
```php
protected function casts(): array { return ["token" => "string"]; }
```

After:
```php
protected function casts(): array { return ["token" => "encrypted"]; }
```

#### Guard request-derived mass assignment
- **Do:** Declare the intended mass-assignable attributes with #[Fillable([...])] (or a deliberate #[Guarded([...])]) and pass only operation-appropriate validated fields; never mass assign untrusted input into an unguarded model.
- **Flag when:** Flag when request-derived mass assignment can set attributes outside the intended writable set.
- **Why:** Model guards constrain which attributes mass assignment may change.
- **Enforced by:** manual

Before:
```php
protected $guarded = [];
```

After:
```php
use Illuminate\Database\Eloquent\Attributes\Fillable;

#[Fillable(['name', 'email'])]
class User extends Model {}
```

#### Throttle abuse-prone endpoints
- **Do:** Apply route throttle middleware or an equivalent effective limiter to login, recovery, verification, token issuance and other abuse-prone HTTP endpoints.
- **Flag when:** Flag when an abuse-prone endpoint has no effective limiter.
- **Why:** Limits reduce brute force and resource or messaging abuse.
- **Enforced by:** manual

Before:
```php
Route::post("/login", LoginController::class);
```

After:
```php
Route::post("/login", LoginController::class)->middleware("throttle:login");
```

#### Exclude plaintext secrets from version control
- **Do:** Keep .env files and plaintext production credentials out of version control.
- **Flag when:** Flag when tracked files contain real credentials or an unencrypted deployment .env.
- **Why:** Repository access and history can expose production credentials.
- **Enforced by:** manual

#### Enforce permission checks on protected actions
- **Do:** Authorize permission-dependent operations with policies, gates or Form Request authorization.
- **Flag when:** Flag when a protected action relies only on authentication, validation, hidden attributes or encryption.
- **Why:** These mechanisms do not decide whether this actor may perform this operation.
- **Enforced by:** manual

Before:
```php
return $account->secret;
```

After:
```php
Gate::authorize("viewSecret", $account); return $account->secret;
```

#### Send CSRF credentials from browser clients
- **Do:** Configure session-authenticated browser AJAX clients to send X-CSRF-TOKEN (from the csrf-token meta tag) or the XSRF-TOKEN cookie as X-XSRF-TOKEN, so requests pass PreventRequestForgery's token fallback when Sec-Fetch-Site origin verification is unavailable; never exclude routes from CSRF protection to fix token mismatches.
- **Flag when:** Flag when browser-issued state-changing requests to session-authenticated routes send no CSRF header while the app relies on the token fallback (not originOnly mode), or when routes are excluded from request-forgery protection to work around 419 token mismatches.
- **Why:** Correct client credentials allow the middleware token fallback to validate requests.
- **Enforced by:** manual

Before:
```php
fetch("/profile", {method: "POST"});
```

After:
```php
fetch("/profile", {method: "POST", headers: {"X-CSRF-TOKEN": document.querySelector("meta[name=csrf-token]").content}});
```

#### Escape untrusted HTML output
- **Do:** Render untrusted text with escaped Blade echoes.
- **Flag when:** Flag when user-controlled text reaches raw Blade echoes or PHP echo in HTML.
- **Why:** Escaped output prevents text from becoming executable markup.
- **Enforced by:** manual

Before:
```php
{!! $comment->body !!}
```

After:
```php
{{ $comment->body }}
```

#### Sanitize intentional rich HTML
- **Do:** Sanitize user-controlled rich HTML for its rendering context before using raw output.
- **Flag when:** Flag when raw HTML output contains user content without an appropriate sanitizer.
- **Why:** Raw echoes bypass Blade escaping and can execute injected markup.
- **Enforced by:** manual

Before:
```php
{!! $article->body !!}
```

After:
```php
{!! $htmlSanitizer->sanitize($article->body) !!}
```

#### Encode output for its destination context
- **Do:** Encode values for the exact HTML, URL, JavaScript or CSS context where they are inserted.
- **Flag when:** Flag when HTML escaping is used as the only protection for JavaScript, CSS or URL sinks.
- **Why:** Escaping rules differ by context.
- **Enforced by:** manual

Before:
```php
<script>const name = "{{ $name }}";</script>
```

After:
```php
<script>const name = {{ Illuminate\Support\Js::from($name) }};</script>
```

#### Include CSRF tokens in state-changing forms
- **Do:** Include @csrf in Blade forms that submit state-changing requests through web middleware.
- **Flag when:** Flag when a POST/PUT/PATCH/DELETE form using session authentication lacks a CSRF token.
- **Why:** Tokens preserve request-forgery protection when origin verification does not apply.
- **Enforced by:** partial: shift:workbench/blade/csrf

Before:
```php
<form method="POST" action="/profile"></form>
```

After:
```php
<form method="POST" action="/profile">@csrf</form>
```

#### Centralize password validation defaults
- **Do:** Centralize password validation defaults.
- **Flag when:** Flag when registration and password-change paths duplicate divergent password requirements.
- **Why:** Shared rules prevent a weaker alternate password enrollment path.
- **Enforced by:** manual

Before:
```php
"password" => Password::min(8),
```

After:
```php
Password::defaults(fn () => Password::min(12)->uncompromised());
```

#### Choose limiter identities deliberately
- **Do:** Choose normalized actor/account/IP limiter keys that match the abuse model and shared-network behavior.
- **Flag when:** Flag when an untrusted or overly broad key lets attackers evade limits or block unrelated users.
- **Why:** Key choice determines who shares the budget and can be denied service.
- **Enforced by:** manual

Before:
```php
Limit::perMinute(5)->by($request->input("email"));
```

After:
```php
Limit::perMinute(5)->by(strtolower($request->string("email"))."|".$request->ip());
```

#### Size storage for ciphertext
- **Do:** Use TEXT or larger columns for encrypted casts and avoid querying the randomized ciphertext as plaintext.
- **Flag when:** Flag when an encrypted attribute uses a short column or is queried with plaintext equality.
- **Why:** Ciphertext expands and encrypted casts cannot be searched like plaintext.
- **Enforced by:** manual

Before:
```php
$table->string("token", 64);
```

After:
```php
$table->text("token");
```

#### Preserve decryption access during planned key rotation
- **Do:** Preserve decryption access during planned key rotation.
- **Flag when:** Flag when a planned APP_KEY rotation removes keys still needed by persisted ciphertext.
- **Why:** APP_PREVIOUS_KEYS allows old data to remain readable while new encryption uses the current key; do not retain compromised keys indefinitely.
- **Enforced by:** manual

#### Authenticate and constrain serialized data before decoding it
- **Do:** Authenticate and constrain serialized data before decoding it.
- **Flag when:** Flag when request-controlled serialized PHP reaches unserialize without authenticated integrity or allowed-class restrictions.
- **Why:** Untrusted object deserialization can invoke dangerous gadget chains.
- **Enforced by:** manual

Before:
```php
unserialize($request->input("state"));
```

After:
```php
json_decode(Crypt::decryptString($request->input("state")), true, flags: JSON_THROW_ON_ERROR);
```

#### Keep browser request-forgery protection enabled
- **Do:** Fix session, origin and token configuration instead of disabling request-forgery protection to resolve mismatches.
- **Flag when:** Flag when session-authenticated web routes are excluded broadly to work around CSRF failures.
- **Why:** The bypass exposes state-changing browser actions to forged requests.
- **Enforced by:** manual

Before:
```php
$middleware->preventRequestForgery(except: ["*"]);
```

After:
```php
$middleware->preventRequestForgery();
```

#### Centralize reusable authorization decisions
- **Do:** Put reusable resource permission decisions in policies or gates and call them from each execution path.
- **Flag when:** Flag when ownership or permission conditions are duplicated across controllers, jobs and components.
- **Why:** Central decisions avoid divergent access rules.
- **Enforced by:** manual

Before:
```php
if ($user->id !== $post->user_id) abort(403);
```

After:
```php
Gate::authorize("update", $post);
```

#### Use framework password hashing
- **Do:** Hash passwords with Laravel Hash and verify them with Hash::check or the authentication provider.
- **Flag when:** Flag when passwords use plaintext, reversible encryption or homemade fast hashes.
- **Why:** Adaptive password hashing resists offline guessing.
- **Enforced by:** manual

Before:
```php
$hash = md5($password);
```

After:
```php
$hash = Hash::make($password); Hash::check($candidate, $hash);
```

#### Restrict production diagnostic dashboards to authorized users
- **Do:** Restrict production diagnostic dashboards to authorized users.
- **Flag when:** Flag when Horizon, Telescope, Pulse or similar diagnostics are publicly reachable without access checks.
- **Why:** Dashboards expose operational data and sometimes privileged actions.
- **Enforced by:** manual

Before:
```php
Gate::define("viewPulse", fn () => true);
```

After:
```php
Gate::define("viewPulse", fn (User $user) => $user->is_admin);
```

#### Disable production debug mode
- **Do:** Disable production debug mode.
- **Flag when:** Flag when production configuration enables APP_DEBUG or detailed exception pages.
- **Why:** Detailed pages expose configuration, paths and secrets.
- **Enforced by:** manual

#### Compare shared secrets with timing-safe equality
- **Do:** Compare shared secrets with timing-safe equality.
- **Flag when:** Flag when raw API keys, signatures or secret tokens are compared with == or ===.
- **Why:** hash_equals prevents type juggling and reduces comparison timing leakage; passwords still require a password hasher.
- **Enforced by:** partial: rector:StaticCallToFuncCallRector

Before:
```php
if ($request->token == config("services.hook.secret")) {}
```

After:
```php
if (hash_equals(config("services.hook.secret"), (string) $request->token)) {}
```

#### Rehash outdated passwords after successful verification
- **Do:** Let outdated password hashes be upgraded on login: authenticate via Auth::attempt()/starter kits with hashing.rehash_on_login left enabled, implement rehashPasswordIfRequired() in custom user providers, and in hand-rolled Hash::check() flows call Hash::needsRehash()/Hash::make() after successful verification.
- **Flag when:** Flag custom login flows that verify with Hash::check() and then Auth::login() without rehashing, custom UserProvider implementations whose rehashPasswordIfRequired() is a no-op, or rehash_on_login disabled without a stated reason.
- **Why:** Login provides plaintext needed to upgrade a stored hash.
- **Enforced by:** manual

Before:
```php
if (Hash::check($password, $user->password)) return true;
```

After:
```php
if (Hash::check($password, $user->password)) {
    if (Hash::needsRehash($user->password)) {
        $user->forceFill(['password' => Hash::make($password)])->save();
    }

    return true;
}
// Prefer Auth::attempt($credentials), which rehashes automatically.
```

#### Require recent password confirmation for sensitive account operations
- **Do:** Require recent password confirmation for sensitive account operations.
- **Flag when:** Flag when security settings or privileged account changes are accessible without the required recent-password check.
- **Why:** Step-up confirmation limits damage from an unattended authenticated session.
- **Enforced by:** manual

Before:
```php
Route::get("/settings/security", $handler)->middleware("auth");
```

After:
```php
Route::get("/settings/security", $handler)->middleware(["auth", "password.confirm"]);
```

#### Exclude authentication secrets from model serialization
- **Do:** Exclude password hashes, remember tokens, MFA secrets, recovery codes and other authentication secrets from model array/JSON output; use #[Hidden([...])] for declarative Laravel 13 configuration.
- **Flag when:** Flag when password hashes, MFA secrets, recovery codes or reset tokens can appear in array/JSON output.
- **Why:** Serialization may expose secrets even when database access is controlled.
- **Enforced by:** manual

Before:
```php
protected $hidden = [];
```

After:
```php
use Illuminate\Database\Eloquent\Attributes\Hidden;

#[Hidden(['password', 'remember_token', 'two_factor_secret', 'two_factor_recovery_codes'])]
class User extends Authenticatable {}
```

#### Validate the current password before sensitive credential changes
- **Do:** Validate the current password before sensitive credential changes.
- **Flag when:** Flag when a password/account mutation lacks required current_password verification.
- **Why:** The check provides proof beyond possession of an existing session.
- **Enforced by:** manual

Before:
```php
"current_password" => "string",
```

After:
```php
"current_password" => ["required", "current_password"],
```

#### Authenticate and authorize private-channel subscription signing
- **Do:** Authenticate and authorize private-channel subscription signing.
- **Flag when:** Flag when a private-channel auth endpoint signs subscriptions without checking the user and channel access.
- **Why:** A signed subscription grants access to private events.
- **Enforced by:** manual

Before:
```php
return $broadcaster->auth($request); // no authenticated user
```

After:
```php
// routes/channels.php (auth route registered via withRouting(channels: ...))
Broadcast::channel('orders.{orderId}', function (User $user, int $orderId) {
    return $user->id === Order::findOrNew($orderId)->user_id;
});
```

#### Authenticate protected API and MCP endpoints
- **Do:** Apply the appropriate authentication middleware to nonpublic API and MCP endpoints.
- **Flag when:** Flag when an API or MCP route exposes nonpublic operations without its configured authentication middleware.
- **Why:** The guard provides a verified identity before protected handlers run.
- **Enforced by:** manual

Before:
```php
Route::get("/api/me", $handler);
```

After:
```php
Route::get("/api/me", $handler)->middleware("auth:sanctum");
```

#### Use the current framework password-strength rule builder
- **Do:** Use Illuminate\Validation\Rules\Password for password-strength validation instead of the deprecated Laravel\Fortify\Rules\Password class, preserving the application's required defaults and confirmation checks.
- **Flag when:** Flag password-strength validation that still instantiates the deprecated Fortify rule or needlessly reimplements criteria already supported by the current builder.
- **Why:** The supported rule builder integrates current password-policy options and application-wide defaults.
- **Enforced by:** manual

After:
```php
use Illuminate\Validation\Rules\Password;
'password' => ['required', 'string', Password::defaults(), 'confirmed'],
```

#### Test application security boundaries
- **Do:** Test authorization, validation, output escaping, query construction and abuse controls where untrusted input crosses them.
- **Flag when:** Flag when an application security boundary has no adversarial input or denied-access test.
- **Why:** Framework features only protect paths where the application uses them correctly.
- **Enforced by:** manual

Before:
```php
// Only the valid owner request is tested.
```

After:
```php
$this->actingAs($outsider)->patchJson("/accounts/1", ["is_admin" => true])->assertForbidden();
```

#### Move deployment-specific credentials into config
- **Do:** Read credentials and deployment-specific contact destinations from configuration backed by environment values.
- **Flag when:** Flag when code hard-codes a real credential or environment-specific administrative destination.
- **Why:** Configuration prevents accidental disclosure and wrong-environment delivery.
- **Enforced by:** manual

Before:
```php
$token = "live-secret";
```

After:
```php
$token = config("services.vendor.token");
```

#### Register macros, gates and view extensions during provider boot
- **Do:** Register macros, gates and view extensions during provider boot.
- **Flag when:** Flag when runtime registrations occur before the provider boot phase without a requirement.
- **Why:** The boot phase runs after service registration.
- **Enforced by:** manual

Before:
```php
public function register(): void { Gate::define("manage", $callback); }
```

After:
```php
public function boot(): void { Gate::define("manage", $callback); }
```

#### Default authentication cookies to SameSite=Lax
- **Do:** Default authentication cookies to SameSite=Lax.
- **Flag when:** Flag when session cookies use SameSite=None without a cross-site requirement.
- **Why:** Lax limits ambient cross-site cookie sending while supporting common navigation.
- **Enforced by:** manual

#### Invalidate sessions and regenerate CSRF tokens on logout or account deletion
- **Do:** Invalidate sessions and regenerate CSRF tokens on logout or account deletion.
- **Flag when:** Flag when logout/deletion only clears authentication while retaining old session data or CSRF state.
- **Why:** Invalidation removes stale authenticated-session state.
- **Enforced by:** manual

Before:
```php
Auth::logout();
```

After:
```php
Auth::logout(); $request->session()->invalidate(); $request->session()->regenerateToken();
```

#### Limit cookie-encryption exceptions to named required cookies
- **Do:** Keep EncryptCookies in the web middleware group and configure only required named plaintext cookies through bootstrap/app.php middleware encryptCookies(except: [...]).
- **Flag when:** Flag when cookie encryption is removed from the web group or bypassed globally to accommodate one plaintext cookie.
- **Why:** Narrow exceptions retain protection for other application cookies.
- **Enforced by:** manual

Before:
```php
// bootstrap/app.php, inside withMiddleware():
$middleware->web(remove: [\Illuminate\Cookie\Middleware\EncryptCookies::class]);
```

After:
```php
// bootstrap/app.php, inside withMiddleware():
$middleware->encryptCookies(except: ['theme']);
```

#### Use session-cookie authentication for the first-party browser application
- **Do:** Use session-cookie authentication for the first-party browser application.
- **Flag when:** Flag when a first-party browser stores long-lived bearer API tokens instead of using supported session authentication.
- **Why:** HttpOnly session cookies and CSRF defenses reduce token exposure in browser code.
- **Enforced by:** manual

Before:
```php
localStorage.setItem("token", apiToken);
```

After:
```php
axios.defaults.withCredentials = true;
axios.defaults.withXSRFToken = true;
await axios.get('/sanctum/csrf-cookie');
await axios.post('/login', credentials); // session cookie, no token in localStorage
```

#### Forward Authorization headers through the deployed web server
- **Do:** Forward Authorization headers through the deployed web server.
- **Flag when:** Flag when FastCGI/Apache drops Authorization before PHP authenticates a protected API.
- **Why:** Authentication middleware cannot validate a missing header.
- **Enforced by:** manual

#### Confirm a second-factor code before enabling MFA and require step-up for management
- **Do:** Confirm a second-factor code before enabling MFA and require step-up for management.
- **Flag when:** Flag when MFA becomes active before verifying enrollment or management bypasses required password confirmation.
- **Why:** Confirmation prevents locking accounts to an incorrect authenticator and protects factor changes.
- **Enforced by:** manual

Before:
```php
Features::twoFactorAuthentication(),
```

After:
```php
Features::twoFactorAuthentication(["confirm" => true, "confirmPassword" => true]),
```

#### Keep session-based browser routes in the web middleware group
- **Do:** Keep session-based browser routes in the web middleware group.
- **Flag when:** Flag when browser routes relying on sessions/cookies bypass the web security middleware.
- **Why:** The web stack provides session, cookie and request-forgery controls.
- **Enforced by:** manual

Before:
```php
// Session login endpoint in a bare stateless group.
```

After:
```php
Route::middleware("web")->post("/login", $handler);
```

#### Regenerate the session ID and CSRF token after authentication
- **Do:** Regenerate the session ID and CSRF token after authentication.
- **Flag when:** Flag when custom login only migrates the session ID or retains its prior token.
- **Why:** Both session and CSRF state should cross the authentication boundary safely.
- **Enforced by:** manual

Before:
```php
$request->session()->migrate();
```

After:
```php
$request->session()->regenerate();
```

#### Record the telemetry aggregates that dashboards query
- **Do:** When a custom Pulse card reads aggregate()/aggregateTotal() for a type, record that type with Pulse::record() chaining every aggregation the card queries (sum, count, max, min, avg).
- **Flag when:** Flag when a custom Pulse card requests an aggregate (e.g. 'avg' or 'max') for a type whose Pulse::record() calls never chain that aggregation method.
- **Why:** Unrecorded aggregates cannot provide correct dashboard data.
- **Enforced by:** manual

Before:
```php
// Query an hourly aggregate absent from recorder configuration.
```

After:
```php
Pulse::record('user_sale', $user->id, $sale->amount)->sum()->count();
// in the card:
$this->aggregate('user_sale', ['sum', 'count']);
```

#### Encrypt versioned environment files
- **Do:** Encrypt environment files before storing their secret values in version control and keep the decryption key separate.
- **Flag when:** Flag when a versioned environment bundle contains plaintext secrets or its decryption key.
- **Why:** Encryption permits controlled distribution without storing plaintext credentials in git.
- **Enforced by:** manual

#### Allowlist fields for each write operation
- **Do:** Pass only validated and operation-appropriate fields to mass assignment and keep guards aligned with that operation.
- **Flag when:** Flag when create/fill/update receives request->all() or validated fields that include unauthorized attributes.
- **Why:** Validation alone can still admit attributes the actor must not write.
- **Enforced by:** manual

Before:
```php
$user->update($request->all());
```

After:
```php
$user->update($request->safe()->only(["name", "email"]));
```

#### Validate and authorize Livewire actions
- **Do:** Treat Livewire actions as public request handlers and validate inputs and authorize protected operations on the server.
- **Flag when:** Flag when an action trusts public properties or UI visibility for validation or permission.
- **Why:** Clients can invoke actions and modify request data.
- **Enforced by:** manual

Before:
```php
$post->update(["title" => $this->title]);
```

After:
```php
Gate::authorize("update", $post); $post->update($this->validate(["title" => "required|string"]));
```

#### Configure and benchmark an adequate password hashing cost
- **Do:** Keep production password hashing at or above Laravel's defaults (bcrypt rounds >= 12 via BCRYPT_ROUNDS / hashing.bcrypt.rounds) and raise the work factor over time; Laravel rehashes on login.
- **Flag when:** Flag when production config sets bcrypt rounds below 12, or lowers Argon2 memory below 65536 KiB or time below 2 (test-only low rounds in phpunit.xml are fine).
- **Why:** An adequate cost slows offline guessing; benchmark higher costs against authentication capacity.
- **Enforced by:** manual

Before:
```php
"bcrypt" => ["rounds" => 8],
```

After:
```php
"bcrypt" => ["rounds" => 12],
```

#### Issue minimally scoped API tokens with deliberate lifetimes
- **Do:** Issue minimally scoped API tokens with deliberate lifetimes.
- **Flag when:** Flag when tokens receive wildcard abilities or indefinite lifetime without a need.
- **Why:** Narrow scope and expiry reduce damage from a stolen token.
- **Enforced by:** manual

Before:
```php
$user->createToken("integration", ["*"]);
```

After:
```php
$user->createToken("integration", ["sites:read"], now()->addDays(30));
```

#### Escape interpolated values before marking generated markup safe
- **Do:** Escape interpolated values before marking generated markup safe.
- **Flag when:** Flag when HtmlString wraps generated HTML containing unescaped user values.
- **Why:** HtmlString bypasses normal escaping of the whole value.
- **Enforced by:** manual

Before:
```php
new HtmlString("<b>".$name."</b>");
```

After:
```php
new HtmlString("<b>".e($name)."</b>");
```

#### Validate OAuth state before exchanging callback codes
- **Do:** Validate OAuth state before exchanging callback codes.
- **Flag when:** Flag when a browser OAuth callback disables or omits state validation without a stateless protocol requirement.
- **Why:** State binds the callback to the initiating browser flow.
- **Enforced by:** manual

Before:
```php
$provider->stateless()->user();
```

After:
```php
$provider->user();
```

#### Make policy discovery or explicit registration resolve every intended policy
- **Do:** Make policy discovery or explicit registration resolve every intended policy.
- **Flag when:** Flag when a nonconventional policy is never registered or conventional class names/locations do not match.
- **Why:** An undiscovered policy cannot enforce the intended ability.
- **Enforced by:** manual

Before:
```php
class PostPermissions {} // unregistered
```

After:
```php
Gate::policy(Post::class, PostPermissions::class);
```

#### Provide API-token revocation
- **Do:** Provide API-token revocation.
- **Flag when:** Flag when a bearer token has no supported invalidation path before its expiry.
- **Why:** Revocation is necessary after loss or changed access.
- **Enforced by:** manual

Before:
```php
// No token deletion/revocation endpoint.
```

After:
```php
$request->user()->currentAccessToken()->delete();
```

#### Match anonymous authentication routes to Fortify's configured guard
- **Do:** Apply the intended guest middleware to login, registration, recovery and pending-challenge screens using the same authentication guard configured for Fortify.
- **Flag when:** Flag when a custom Fortify route checks the default web guard while Fortify authenticates another guard, or anonymous-only flows remain accessible to users already signed into that guard.
- **Why:** Guard mismatches make guest checks and authentication state disagree.
- **Enforced by:** manual

#### Authenticate CSRF-exempt webhooks
- **Do:** Verify provider signatures or equivalent authenticity on third-party webhook endpoints excluded from CSRF checks.
- **Flag when:** Flag when a CSRF-exempt webhook accepts a payload without authenticating its source.
- **Why:** An exemption removes browser token checks and needs an independent authenticity control.
- **Enforced by:** manual

Before:
```php
$payload = $request->all(); process($payload);
```

After:
```php
$provider->verifySignature($request->getContent(), $request->header("Provider-Signature")); process($request->all());
```

#### Authorize each sensitive MCP primitive
- **Do:** Authorize the actual user, action and requested resource for every sensitive MCP operation. Use shouldRegister() for conditional availability, recognizing that a false result also prevents invocation; ensure the operation-specific permission check runs before data access or side effects.
- **Flag when:** Flag sensitive MCP operations that rely on authentication alone or on an availability check that does not enforce permissions for the requested action and resource.
- **Why:** Registration controls visibility rather than per-invocation permission.
- **Enforced by:** manual

Before:
```php
// Authenticated and registered, but no authorization for the requested secret.
public function handle(Request $request): Response {
    $secret = Secret::findOrFail($request->get("secret_id"));
    return Response::text($secret->value);
}
```

After:
```php
// Imports: Laravel\Mcp\Request, Laravel\Mcp\Response, Illuminate\Support\Facades\Gate.
public function handle(Request $request): Response {
    $secret = Secret::findOrFail($request->get("secret_id"));
    Gate::forUser($request->user())->authorize("view", $secret);
    return Response::text($secret->value);
}
```

#### Disable raw HTML in untrusted Markdown
- **Do:** When rendering untrusted Markdown (Str::markdown/inlineMarkdown or league/commonmark), set html_input to 'escape' or 'strip' and allow_unsafe_links to false, or pass the rendered HTML through an HTML purifier when some raw HTML must be allowed.
- **Flag when:** Flag when user-supplied Markdown is rendered with raw HTML allowed (default options or html_input 'allow') and the output is not purified before being displayed unescaped.
- **Why:** Markdown rendering can otherwise preserve executable markup.
- **Enforced by:** manual

Before:
```php
new Environment(["html_input" => "allow"]);
```

After:
```php
new Environment(["html_input" => "strip", "allow_unsafe_links" => false]);
```

#### Match returning social users by provider identity
- **Do:** Use the provider namespace and stable provider user ID to match returning users.
- **Flag when:** Flag when callbacks identify social accounts only by mutable profile data.
- **Why:** Stable provider identity avoids creating duplicate or wrong users.
- **Enforced by:** manual

Before:
```php
User::firstOrCreate(["name" => $social->getName()]);
```

After:
```php
SocialAccount::updateOrCreate(["provider" => "github", "provider_id" => $social->getId()], ["name" => $social->getName()]);
```

#### Redact secrets from telemetry
- **Do:** Scrub credentials, tokens, cookies, authorization headers and sensitive personal data from error reports and logs.
- **Flag when:** Flag when telemetry captures raw sensitive request or integration data.
- **Why:** Error-reporting access should not expose reusable credentials.
- **Enforced by:** manual

#### Enforce UI permissions on the server
- **Do:** Enforce authorization in the server handler even when the UI hides unauthorized controls.
- **Flag when:** Flag when a protected handler relies on @can or hidden buttons alone.
- **Why:** Clients can invoke endpoints and actions directly.
- **Enforced by:** manual

Before:
```php
// Button hidden; destroy() has no permission check.
```

After:
```php
public function destroy(Post $post) { Gate::authorize("delete", $post); $post->delete(); }
```

#### Let non-super-admin decisions fall through
- **Do:** Return true for authorized super-admin bypasses and null for other users in Gate::before.
- **Flag when:** Flag when a super-admin Gate::before returns false for all ordinary users.
- **Why:** False short-circuits policies and denies ordinary legitimate access.
- **Enforced by:** manual

Before:
```php
Gate::before(fn ($user) => $user->isSuperAdmin());
```

After:
```php
Gate::before(fn ($user) => $user->isSuperAdmin() ? true : null);
```

#### Use precise validation and throttle statuses
- **Do:** Return HTTP 422 for invalid input and 429 for exhausted rate limits.
- **Flag when:** Flag when validation or throttle failures return a success status or unrelated error code.
- **Why:** Clients need distinguishable failure semantics.
- **Enforced by:** manual

Before:
```php
return response()->json(["error" => "Too many requests"], 200);
```

After:
```php
return response()->json(["error" => "Too many requests"], 429);
```

#### Initialize Sanctum CSRF state before SPA login
- **Do:** Initialize Sanctum CSRF state before SPA login.
- **Flag when:** Flag when a cookie-authenticated SPA posts login without first initializing its CSRF cookie.
- **Why:** The login request needs the session token/cookie handshake.
- **Enforced by:** manual

Before:
```php
await axios.post("/login", data);
```

After:
```php
await axios.get("/sanctum/csrf-cookie"); await axios.post("/login", data);
```

#### Generate security tokens from cryptographically secure randomness
- **Do:** Generate security tokens from cryptographically secure randomness.
- **Flag when:** Flag when tokens or private identifiers derive from timestamps, model data, uniqid or weak PRNGs.
- **Why:** Predictable inputs and fast hashes do not create secret entropy.
- **Enforced by:** manual

Before:
```php
$token = md5($user->id.time());
```

After:
```php
$token = Str::random(64);
```

#### Reject known compromised passwords
- **Do:** Reject known compromised passwords.
- **Flag when:** Flag when registration or password changes accept known leaked passwords and rely only on complexity.
- **Why:** Compromised-password checks address reuse that complexity rules miss.
- **Enforced by:** manual

Before:
```php
Password::min(8)->symbols();
```

After:
```php
Password::min(8)->uncompromised();
```

#### Choose cryptographic hashes suited to their purpose
- **Do:** Choose cryptographic hashes suited to their purpose.
- **Flag when:** Flag when MD5 or SHA-1 is used for passwords or security signatures/integrity.
- **Why:** Passwords require adaptive hashing; integrity/signatures require suitable modern primitives.
- **Enforced by:** manual

Before:
```php
$digest = sha1($payload);
```

After:
```php
$digest = hash("sha256", $payload); // Use HMAC for authentication; Hash for passwords.
```

#### Use established cryptographic implementations
- **Do:** Use established cryptographic implementations.
- **Flag when:** Flag when custom encryption, signing or token cryptography duplicates vetted primitives.
- **Why:** Cryptographic composition errors can defeat otherwise correct access controls.
- **Enforced by:** manual

Before:
```php
$ciphertext = $plaintext ^ $key;
```

After:
```php
$ciphertext = Crypt::encryptString($plaintext);
```

#### List trusted SPA hosts in Sanctum stateful domains
- **Do:** List trusted SPA hosts in Sanctum stateful domains.
- **Flag when:** Flag when a cookie-authenticated SPA host is missing from stateful configuration or untrusted hosts are included.
- **Why:** Sanctum must identify exactly which frontends receive session authentication.
- **Enforced by:** manual

Before:
```php
SANCTUM_STATEFUL_DOMAINS=*
```

After:
```php
SANCTUM_STATEFUL_DOMAINS=app.example.com
```

#### Revoke persistent remember-me credentials during password reset
- **Do:** Revoke persistent remember-me credentials during password reset.
- **Flag when:** Flag when a reset updates the password but leaves the prior remember token active.
- **Why:** Stolen remember credentials can survive a password change.
- **Enforced by:** manual

Before:
```php
$user->password = Hash::make($password);
```

After:
```php
$user->forceFill(["password" => Hash::make($password), "remember_token" => Str::random(60)])->save();
```

#### Protect and provision Passport signing keys outside source control
- **Do:** Protect and provision Passport signing keys outside source control.
- **Flag when:** Flag when Passport private keys are tracked or each replica uses an unrelated signing identity.
- **Why:** Protected consistent signing keys preserve token trust across servers.
- **Enforced by:** manual

#### Mask sensitive interactive command input
- **Do:** Mask sensitive interactive command input.
- **Flag when:** Flag when Artisan commands read passwords or secrets with a visible ask/text prompt.
- **Why:** Echoing secrets exposes them to nearby viewers and terminal recordings.
- **Enforced by:** manual

Before:
```php
$password = $this->ask("Password");
```

After:
```php
$password = $this->secret("Password");
```

### Testing

#### Use framework fakes at framework boundaries
- **Do:** Use Laravel fakes for framework-managed side effects rather than manually mocking facade internals.
- **Flag when:** Flag when tests manually mock queue, mail, notification, storage or event internals despite a suitable fake.
- **Why:** Framework fakes expose domain assertions and avoid coupling to implementation calls.
- **Enforced by:** partial: rector:ReplaceExpectsMethodsInTestsRector, partial: rector:ReplaceWithoutJobsEventsAndNotificationsWithFacadeFakeRector

Before:
```php
Mail::shouldReceive("to")->andReturnSelf();
```

After:
```php
Mail::fake();
// invoke application
Mail::assertSent(Receipt::class);
```

#### Freeze time for time-dependent tests
- **Do:** Use Laravel time helpers to fix the clock and assert known time-dependent results.
- **Flag when:** Flag when a test's expected result depends on the current date or time (now(), today(), timestamps, expiry/overdue logic) but the test neither freezes nor travels the clock, or computes the expected value from now() with the implementation's own logic.
- **Why:** A fixed clock prevents calendar and execution-time failures.
- **Enforced by:** partial: rector:CarbonSetTestNowToTravelToRector

Before:
```php
expect($invoice->isOverdue())->toBeTrue();
```

After:
```php
$this->travelTo('2026-01-10 09:00:00');
$invoice = Invoice::factory()->create(['due_at' => '2026-01-05']);
expect($invoice->isOverdue())->toBeTrue();
```

#### Build test records with expressive factories
- **Do:** Use model factories and existing named states for test data, and add reusable states for recurring domain scenarios.
- **Flag when:** Flag when tests repeatedly hand-insert records or duplicate attributes already expressed by a factory state.
- **Why:** Factories keep fixtures valid as models evolve; named states explain the scenario.
- **Enforced by:** manual

Before:
```php
$user = User::create(["name" => "A", "email" => "a@example.test", "email_verified_at" => null]);
```

After:
```php
$user = User::factory()->unverified()->create();
```

#### Test validation through behavior
- **Do:** Send invalid input through the application and cover each distinct validation contract, including required fields.
- **Flag when:** Flag when validation tests only inspect rule arrays or omit invalid requests.
- **Why:** Request-level assertions prove the rules are applied to real input.
- **Enforced by:** manual

Before:
```php
expect($request->rules()["email"])->toContain("required");
```

After:
```php
$this->postJson("/users", [])->assertInvalid(["email"]);
```

#### Mock external boundaries and use real in-process logic
- **Do:** Mock nondeterministic or out-of-process collaborators and exercise deterministic application logic and database queries for real.
- **Flag when:** Flag when feature tests mock their own core behavior, Eloquent queries or deterministic value objects.
- **Why:** Real internal collaboration catches wiring errors that matching mocks conceal.
- **Enforced by:** manual

Before:
```php
$this->mock(PlaceOrder::class, fn ($mock) => $mock->shouldReceive('handle')->once());
$this->postJson('/orders', $data)->assertCreated();
```

After:
```php
$this->postJson('/orders', $data)->assertCreated();
$this->assertDatabaseHas('orders', ['sku' => $data['sku']]);
```

#### Use the application base case for framework tests
- **Do:** Extend the project Tests\TestCase for tests that use Laravel services, factories or configuration.
- **Flag when:** Flag when a framework-dependent test extends only PHPUnit\Framework\TestCase.
- **Why:** Framework services require the application lifecycle supplied by the base case.
- **Enforced by:** manual

Before:
```php
class OrderTest extends \PHPUnit\Framework\TestCase {}
```

After:
```php
class OrderTest extends \Tests\TestCase {}
```

#### Assert complete write-operation outcomes
- **Do:** Assert the response, persisted changes and contractual dispatched side effects of successful writes.
- **Flag when:** Flag when a write test checks only success status while omitting the intended data change or contractual side effects.
- **Why:** A successful response can conceal a missing write or dispatch.
- **Enforced by:** manual

Before:
```php
$this->postJson("/orders", $data)->assertCreated();
```

After:
```php
$this->postJson("/orders", $data)->assertCreated();
$this->assertDatabaseHas("orders", ["reference" => $data["reference"]]);
Queue::assertPushed(ProcessOrder::class);
```

#### Fake external HTTP integrations
- **Do:** Fake remote HTTP integrations in the ordinary automated suite.
- **Flag when:** Flag when ordinary tests depend on live HTTP service availability.
- **Why:** Fakes isolate application behavior and prevent unintended remote side effects.
- **Enforced by:** manual

Before:
```php
$response = Http::get("https://api.example.test/orders");
```

After:
```php
Http::fake(["https://api.example.test/orders" => Http::response(["id" => 1])]);
```

#### Bind collaborator mocks through the container
- **Do:** Bind test doubles for container-resolved dependencies into the container (e.g. $this->mock(), $this->instance(), $this->partialMock()) so the application receives them.
- **Flag when:** Flag when a test creates a mock of a container-resolved dependency without binding it into the container, so the code under test never receives the double.
- **Why:** Container bindings ensure production resolution receives the intended double.
- **Enforced by:** manual

Before:
```php
$mock = Mockery::mock(Gateway::class);
```

After:
```php
$mock = $this->mock(Gateway::class);
```

#### Follow PHPUnit discovery and naming conventions
- **Do:** Make every PHPUnit test method discoverable with the test prefix or the #[Test] attribute (doc-comment @test is ignored by PHPUnit 12), following the convention of neighbouring test files.
- **Flag when:** Flag when a PHPUnit test method has neither the test prefix nor the #[Test] attribute, including methods that rely only on a @test doc-comment.
- **Why:** Discoverable consistent methods prevent silently skipped tests and unclear reports.
- **Enforced by:** pint:php_unit_method_casing, shift:workbench/format-test-cases

Before:
```php
public function savesOrder(): void {}
```

After:
```php
public function test_saves_order(): void {}
```

#### Make exception expectations explicit
- **Do:** Assert the expected exception class and contractual message before triggering an exceptional operation.
- **Flag when:** Flag when a test manually swallows an exception or can pass when the operation throws nothing.
- **Why:** Explicit expectations ensure the failure path is actually taken.
- **Enforced by:** manual

Before:
```php
try { $service->run(); } catch (DomainException $e) { $this->assertTrue(true); }
```

After:
```php
$this->expectException(DomainException::class);
$this->expectExceptionMessage("Account is closed");
$service->run();
```

#### Cover changed behavior with regression tests
- **Do:** Add meaningful regression tests for changed decisions and important failure paths.
- **Flag when:** Flag when behavior changes without tests that fail if that behavior breaks.
- **Why:** Tests protect the changed contract and make later refactoring safer.
- **Enforced by:** manual

Before:
```php
public function test_home(): void { $this->get("/")->assertOk(); }
```

After:
```php
public function test_rejects_closed_order(): void { $this->postJson("/orders/1/pay")->assertConflict(); }
```

#### Reset database state between database-backed tests
- **Do:** Use RefreshDatabase or another compatible reset strategy and arrange the records each test needs.
- **Flag when:** Flag when database-backed tests reuse mutable leftovers from prior cases.
- **Why:** A clean baseline prevents order-dependent results.
- **Enforced by:** manual

Before:
```php
class OrderTest extends TestCase {}
```

After:
```php
class OrderTest extends TestCase { use RefreshDatabase; }
```

#### Wait for browser conditions
- **Do:** Wait for the required element or state and set appropriate browser timeouts instead of fixed sleeps.
- **Flag when:** Flag when browser tests use fixed pauses to wait for asynchronous work.
- **Why:** Condition waits adapt to machine speed and fail on the missing condition.
- **Enforced by:** manual

Before:
```php
$browser->pause(3000)->assertSee("Saved");
```

After:
```php
$browser->waitForText("Saved")->assertSee("Saved");
```

#### Declare PHPUnit providers public and static
- **Do:** Declare PHPUnit data providers as public static methods.
- **Flag when:** Flag when a PHPUnit provider is an instance or non-public method.
- **Why:** Modern PHPUnit requires provider data without constructing the test instance.
- **Enforced by:** manual

Before:
```php
public function cases(): array { return [[1]]; }
```

After:
```php
public static function cases(): array { return [[1]]; }
```

#### Make nonconventional model factories discoverable
- **Do:** Follow ModelFactory naming or explicitly associate nonconventional factories with their models.
- **Flag when:** Flag when Model::factory() relies on convention for a factory stored under a nonconventional name or namespace.
- **Why:** Explicit discovery prevents resolution failures after naming or namespace changes.
- **Enforced by:** manual

Before:
```php
class BillingInvoiceFactory extends Factory {} // undiscoverable from Invoice
```

After:
```php
#[UseFactory(BillingInvoiceFactory::class)]
class Invoice extends Model { use HasFactory; }
```

#### Use class-based factories and named state methods
- **Do:** Replace legacy closure factories and string state lists with class-based factories and named state methods.
- **Flag when:** Flag when tests use legacy factory(Model::class) definitions or string states unavailable in the modern factory API.
- **Why:** Class-based factories provide discoverable, typed fixture construction on Laravel 13.
- **Enforced by:** rector:FactoryApplyingStatesRector, rector:FactoryDefinitionRector, rector:FactoryFuncCallToStaticCallRector

Before:
```php
factory(User::class)->states("active")->create();
```

After:
```php
User::factory()->active()->create();
```

#### Match intended HTTP endpoints in fakes
- **Do:** Fake the intended URL patterns instead of using a catch-all response for every request.
- **Flag when:** Flag when a generic Http::fake() lets incorrect hosts or paths succeed.
- **Why:** Specific matching detects endpoint mistakes when combined with stray-request prevention.
- **Enforced by:** manual

Before:
```php
Http::fake();
```

After:
```php
Http::preventStrayRequests();
Http::fake(["https://api.example.test/orders" => Http::response(["id" => 1])]);
```

#### Cover handled HTTP failure paths
- **Do:** Test handled timeouts, connection failures and error responses from external services.
- **Flag when:** Flag when an integration handles failures but tests only successful responses.
- **Why:** Failure paths often control retries, fallbacks and user-visible errors.
- **Enforced by:** manual

Before:
```php
Http::fake(["*" => Http::response(["id" => 1])]);
```

After:
```php
Http::fake(["api.example.test/*" => Http::failedConnection()]);
expect(fn () => $gateway->fetch())->toThrow(ServiceUnavailable::class);
```

#### Test engine-sensitive behavior against the production engine
- **Do:** Run critical database integration tests against the production database engine, even when SQLite is used for fast local tests.
- **Flag when:** Flag when MySQL/PostgreSQL-specific behavior is validated only with SQLite.
- **Why:** Engines differ in constraints, JSON, transactions and sorting semantics.
- **Enforced by:** manual

#### Stay current within supported releases
- **Do:** Keep Laravel and PHP on supported versions and apply compatible patch/security updates.
- **Flag when:** Flag when the project pins an unsupported runtime/framework or leaves available security fixes unapplied.
- **Why:** Supported releases receive fixes required for reliable operation.
- **Enforced by:** partial: shift:public/linter/current-version

Before:
```php
"laravel/framework": "13.0.0"
```

After:
```php
"laravel/framework": "^13.0"
```

#### Use strict comparisons when types matter
- **Do:** Compare values and types with assertSame() or equivalent strict expectations.
- **Flag when:** Flag when a contract requires a specific type but tests use loose equality.
- **Why:** Loose comparisons can let incorrect types pass.
- **Enforced by:** pint:strict_comparison

Before:
```php
$this->assertEquals(0, $result);
```

After:
```php
$this->assertSame(0, $result);
```

#### Test reported exceptions through the exception fake
- **Do:** Use Exceptions::fake() to check reporting while retaining normal exception handling in response-contract tests.
- **Flag when:** Flag when a response or reporting test disables exception handling and thereby bypasses the response being asserted.
- **Why:** The exception fake observes reporting without changing normal rendering.
- **Enforced by:** manual

Before:
```php
$this->withoutExceptionHandling();
```

After:
```php
$exceptions = Exceptions::fake();
$this->getJson("/broken")->assertServerError();
Exceptions::assertReported(DomainException::class);
```

#### Set mock expectations before acting
- **Do:** Declare mock expectations before the action and inspect spies after the action.
- **Flag when:** Flag when shouldReceive expectations are added only after the tested call or spies are asserted before execution.
- **Why:** Mocks must intercept the call when it occurs; spies verify recorded calls.
- **Enforced by:** manual

Before:
```php
$service->run(); $mock->shouldReceive("send")->once();
```

After:
```php
$mock->shouldReceive("send")->once(); $service->run();
```

#### Fix random strings when asserting generated identifiers
- **Do:** Replace random-string generation with a deterministic generator when the generated value is asserted.
- **Flag when:** Flag when tests guess random identifiers or depend on repeated random generation producing a particular value.
- **Why:** Controlling randomness makes exact assertions reliable.
- **Enforced by:** manual

Before:
```php
expect($slug)->toBe("random-value");
```

After:
```php
Str::createRandomStringsUsing(fn () => "known-id");
expect($service->newId())->toBe("known-id");
```

#### Namespace resources for parallel tests
- **Do:** Give each parallel process separate file paths, cache keys and queue resources.
- **Flag when:** Flag when parallel tests mutate the same filesystem path, cache entry or queue.
- **Why:** Database isolation alone does not prevent shared-resource races.
- **Enforced by:** manual

Before:
```php
$path = "tests/tmp/output.pdf";
```

After:
```php
$path = "tests/tmp/".ParallelTesting::token()."/output.pdf";
```

#### Keep HTTP tests to one principal request
- **Do:** Make one application request per ordinary HTTP test.
- **Flag when:** Flag when unrelated request scenarios share a test and reuse the same booted application state.
- **Why:** Laravel does not reset the application between requests inside one test.
- **Enforced by:** manual

Before:
```php
$this->post("/orders", $a); $this->post("/orders", $b);
```

After:
```php
// Separate tests for the two independent request scenarios.
```

#### Use a dedicated test database
- **Do:** Point automated tests at an isolated testing database rather than development or production data.
- **Flag when:** Flag when test configuration reuses a developer or production database.
- **Why:** Reset traits and fixtures are destructive to the database they target.
- **Enforced by:** manual

Before:
```php
<env name="DB_DATABASE" value="app"/>
```

After:
```php
<env name="DB_DATABASE" value="app_test"/>
```

#### Test Livewire components through their testing API
- **Do:** Exercise Livewire rendering, state changes, actions and validation with Livewire component tests.
- **Flag when:** Flag when Livewire-specific actions or state transitions have only ordinary route-render assertions.
- **Why:** Component tests exercise Livewire hydration and action behavior without browser overhead.
- **Enforced by:** manual

Before:
```php
get("/profile")->assertOk();
```

After:
```php
Livewire::test(EditProfile::class)->set("name", "Ada")->call("save")->assertHasNoErrors();
```

#### Set real test configuration instead of mocking Config
- **Do:** Set configuration values through config() or Config::set() in tests.
- **Flag when:** Flag when a test mocks Config facade lookups merely to change configuration values.
- **Why:** A real configuration repository preserves unrelated lookups and normal application behavior.
- **Enforced by:** manual

Before:
```php
Config::shouldReceive("get")->with("payments.enabled")->andReturn(false);
```

After:
```php
config(["payments.enabled" => false]);
```

#### Declare PHPUnit metadata with attributes
- **Do:** Use supported PHPUnit attributes such as Test, DataProvider and TestWith instead of docblock test metadata on current PHPUnit versions.
- **Flag when:** Flag when current-runner tests rely on @test, @dataProvider or similar removed docblock metadata.
- **Why:** PHPUnit 12 removed support for docblock test metadata; attributes preserve discovery and parameterization.
- **Enforced by:** shift:phpunit/attributes

Before:
```php
/** @dataProvider cases */
public function test_value($value): void { /* ... */ }
```

After:
```php
#[\PHPUnit\Framework\Attributes\DataProvider('cases')]
public function test_value($value): void { /* ... */ }
```

#### Control nondeterministic dependencies
- **Do:** Control the clock, random values, sleeping and network boundaries in tests that depend on them.
- **Flag when:** Flag when a test outcome varies with wall-clock time, randomness, real delays or external availability.
- **Why:** Controlled inputs isolate failures to the application behavior.
- **Enforced by:** manual

Before:
```php
expect($token)->toBe(Str::random(10));
```

After:
```php
Str::createRandomStringsUsing(fn () => "fixed-token");
expect($service->token())->toBe("fixed-token");
```

#### Control lottery outcomes in tests
- **Do:** Fix lottery win/loss outcomes explicitly when testing lottery-dependent branches.
- **Flag when:** Flag when a test relies on random Lottery outcomes to reach a branch.
- **Why:** Deterministic outcome control makes both branches reproducible.
- **Enforced by:** manual

Before:
```php
Lottery::odds(1, 100)->winner(fn () => $service->run())->choose();
```

After:
```php
Lottery::alwaysWin();
// Exercise and assert the winning branch; restore lottery state afterward.
```

#### Reset mutable static state after tests
- **Do:** Provide reset/flush operations for mutable framework or application static state and invoke them during teardown.
- **Flag when:** Flag when a test mutates static registries or caches that survive into subsequent tests.
- **Why:** Explicit resets prevent hidden dependencies between application lifecycles.
- **Enforced by:** manual

Before:
```php
Registry::$items[] = $item; // never reset
```

After:
```php
protected function tearDown(): void { Registry::flushState(); parent::tearDown(); }
```

#### Create Laravel-dependent fixtures only after application boot
- **Do:** Keep PHPUnit data providers and Pest beforeAll/afterAll hooks free of Laravel container, facade and database operations. Resolve those fixtures in per-test hooks or bound dataset closures after the application has booted.
- **Flag when:** Flag when a data provider or class-wide Pest hook accesses app(), config(), facades or factories without a booted test application.
- **Why:** These suite-discovery/class lifecycle phases run outside the ordinary Laravel per-test application lifecycle.
- **Enforced by:** manual

#### Test insufficient permissions across roles
- **Do:** Test roles and permissions that must be refused as well as allowed users.
- **Flag when:** Flag when permission-protected behavior is covered only for an administrator or another allowed user.
- **Why:** Authorization errors often affect particular roles rather than all users.
- **Enforced by:** manual

Before:
```php
// Test only admin access.
```

After:
```php
$this->actingAs($viewer)->deleteJson("/orders/{$order->id}")->assertForbidden();
```

#### Assert contractual validation messages
- **Do:** Assert user-visible validation message text when the wording is part of the application contract.
- **Flag when:** Flag when custom or contractual validation messages are tested only by the presence of an error key.
- **Why:** The correct rule may run while users still receive the wrong explanation.
- **Enforced by:** manual

Before:
```php
$response->assertInvalid(["email"]);
```

After:
```php
$response->assertInvalid(["email" => "Enter a valid work email."]);
```

#### Assert mail using its delivery mode
- **Do:** Use assertQueued() for queued mail and assertSent() for synchronous mail.
- **Flag when:** Flag when queued mailables are asserted as synchronously sent, or vice versa.
- **Why:** The assertion must match whether the application queued or sent the mailable.
- **Enforced by:** manual

Before:
```php
Mail::assertSent(QueuedReceipt::class);
```

After:
```php
Mail::assertQueued(QueuedReceipt::class);
```

#### Assert contractual serialization shape
- **Do:** Assert the exact set of serialized keys (order-insensitive) when a model's or API response's serialized shape is a public contract.
- **Flag when:** Flag when tests of a contractual serialized model/API response only check selected keys or fragments (toHaveKey, assertJson) so new or sensitive keys can appear without a failing test.
- **Why:** Exact shape assertions detect accidental field exposure.
- **Enforced by:** manual

Before:
```php
expect($user->toArray())->toHaveKey("name");
```

After:
```php
expect($user->toArray())->toHaveKeys(["id", "name", "email"])->toHaveCount(3);
// HTTP: $response->assertExactJsonStructure(["data" => ["id", "name", "email"]]);
```

#### Fake and assert external process execution
- **Do:** Fake Process calls in ordinary tests and assert command invocation contracts.
- **Flag when:** Flag when ordinary tests execute external binaries or assert results without checking the intended command was invoked.
- **Why:** Process fakes isolate environment-dependent tooling while checking orchestration.
- **Enforced by:** manual

Before:
```php
Process::run("convert input.png output.png");
```

After:
```php
Process::fake();
$service->convert();
Process::assertRan("convert input.png output.png");
```

#### Test uploads with fake files and isolated storage
- **Do:** Submit fake uploads to the endpoint, fake every storage disk touched by the test, and assert stored and missing files.
- **Flag when:** Flag when upload tests touch real storage or assert only the response without checking stored-file outcomes.
- **Why:** Isolated files exercise upload orchestration without leaving external artifacts.
- **Enforced by:** manual

Before:
```php
$this->post("/avatar", ["avatar" => new UploadedFile("real.jpg", "real.jpg")]);
```

After:
```php
Storage::fake("public");
$file = UploadedFile::fake()->image("avatar.jpg");
$this->post("/avatar", ["avatar" => $file]);
Storage::disk("public")->assertExists($file->hashName());
```

### Validation

#### Validate external input on the server before use
- **Do:** Validate data at every public input boundary before processing or persistence, even when the client enforces the same constraints.
- **Flag when:** Flag when external input reaches processing or persistence without server-side rules, or HTML input types and client checks are its only validation.
- **Why:** Clients can bypass browser controls and send arbitrary payloads. Server-side validation establishes the input contract for every caller.
- **Enforced by:** manual

Before:
```php
Post::create($request->all());
```

After:
```php
$data = $request->validate(['title' => ['required', 'string', 'max:255']]);
Post::create($data);
```

#### Pass validated input to persistence and downstream actions
- **Do:** Use validate(), validated(), or safe() results when passing request data to models or services, and retain model mass-assignment protection.
- **Flag when:** Flag when an action validates input but later passes all() or raw request fields to a model or service instead of the validated result.
- **Why:** Validation does not remove arbitrary fields from the original request. Reading the validated result prevents unchecked input from bypassing the input contract.
- **Enforced by:** manual

Before:
```php
$request->validate($rules);
Post::create($request->all());
```

After:
```php
$data = $request->validate($rules);
Post::create($data);
```

#### Map only fields intended for the current operation
- **Do:** Select an explicit subset of validated data when rules also cover control fields or values the operation must not persist.
- **Flag when:** Flag when an entire validated payload containing confirmation, control or unrelated fields is passed into persistence or another operation.
- **Why:** A value can be valid without belonging to a particular write. Explicit field mapping limits each operation to its intended inputs.
- **Enforced by:** manual

Before:
```php
User::create($request->validated()); // includes accept_terms
```

After:
```php
User::create($request->safe()->only(['name', 'email', 'password']));
```

#### Constrain every uploaded file before storage
- **Do:** Validate uploads against an explicit content-type allowlist and maximum size, adding image and dimension constraints when required by their use.
- **Flag when:** Flag when an upload is stored or processed without content-type validation and a size limit, including each element of a multi-upload array.
- **Why:** Unrestricted uploads admit unexpected content and excessive resource use. Validation is one layer of upload security and does not make arbitrary content safe to execute.
- **Enforced by:** manual

Before:
```php
$request->file('photo')->store('photos');
```

After:
```php
$request->validate(['photo' => ['required', File::image()->max('2mb')]]);
$request->file('photo')->store('photos');
```

#### Validate command-line inputs before side effects
- **Do:** Validate Artisan arguments, options and free-form prompt answers up front; report invalid input and return a nonzero exit code.
- **Flag when:** Flag when a mutating command uses raw arguments or options, treats autocomplete as an allowlist, or reports invalid input with a success exit status.
- **Why:** Command inputs can be arbitrary strings just like HTTP inputs. Early validation prevents partial mutations and lets scripts detect failure.
- **Enforced by:** manual

Before:
```php
$days = $this->option('days');
$this->purge($days);
```

After:
```php
$v = Validator::make(['days' => $this->option('days')], ['days' => ['required', 'integer', 'min:1']]);
if ($v->fails()) { $this->components->error($v->errors()->first()); return self::FAILURE; }
$this->purge((int) $v->validated()['days']);
```

#### Implement reusable custom rules with ValidationRule
- **Do:** Implement reusable custom checks as ValidationRule classes with validate(string $attribute, mixed $value, Closure $fail): void and call $fail for each failure.
- **Flag when:** Flag when new reusable rules use the legacy passes()/message() contract, duplicate custom checks, or return false without invoking the failure callback.
- **Why:** The current contract reports failures directly and supports composing reusable application-specific checks. Returning false from validate() does not record an error.
- **Enforced by:** manual

Before:
```php
public function passes($attribute, $value) { return $this->valid($value); }
```

After:
```php
public function validate(string $attribute, mixed $value, Closure $fail): void {
    if (! $this->valid($value)) { $fail('The :attribute is invalid.'); }
}
```

#### Validate fixed-choice input against an allowlist
- **Do:** Use Rule::in() or an equivalent server-side allowlist for roles, statuses and every element of fixed-choice arrays.
- **Flag when:** Flag when a select value, role, status or fixed-choice array is validated only as a string or trusted because the UI limits choices.
- **Why:** Attackers can submit values the interface never offered. An explicit allowlist enforces the actual domain of accepted values.
- **Enforced by:** manual

Before:
```php
'status' => ['required', 'string']
```

After:
```php
'status' => ['required', Rule::in(['draft', 'published'])]
```

#### Check upload contents rather than trusting names or client metadata
- **Do:** Validate file contents with mimes, mimetypes or File::types(); pair filename-extension checks with content validation and enforce this on the server.
- **Flag when:** Flag when an upload is accepted based only on extensions, client MIME metadata or an HTML accept attribute.
- **Why:** The client controls filenames and declared MIME types. Content-based rules establish the file type independently of those claims.
- **Enforced by:** manual

Before:
```php
'attachment' => ['extensions:pdf']
```

After:
```php
'attachment' => ['required', 'file', 'mimes:pdf', 'extensions:pdf']
```

#### Invalidate derived caches when source data changes
- **Do:** Invalidate every cache entry derived from changed data, including forever entries and alternate tenant lookup keys.
- **Flag when:** Flag when a source mutation leaves a derived cached response, forever entry, or old tenant routing key valid.
- **Why:** Cache expiry alone cannot refresh forever entries and may leave stale routing or output after updates.
- **Enforced by:** manual

Before:
```php
$tenant->update(['domain' => $newDomain]);
```

After:
```php
$oldDomain = $tenant->domain;
$tenant->update(['domain' => $newDomain]);
Cache::forget('tenant:'.$oldDomain);
Cache::forget('tenant:'.$newDomain);
```

#### Preserve automatic failure behavior for manual validators
- **Do:** Call validate() on a manually constructed validator when normal redirect or JSON 422 failure behavior is intended.
- **Flag when:** Flag when a Validator::make() instance is never executed or checked (no validate(), validated(), fails()/passes()) before the input is used.
- **Why:** Constructing the validator alone performs no validation; validate() executes it and uses the standard exception response.
- **Enforced by:** manual

Before:
```php
$validator = Validator::make($data, $rules);
save($data);
```

After:
```php
$data = Validator::make($data, $rules)->validate();
save($data);
```

#### Use a tag-capable store for tagged caching
- **Do:** Configure a cache store that supports tags before enabling tagged cache or response-cache invalidation.
- **Flag when:** Flag when Cache::tags() or tagged response caching uses a store without tag support, such as file or database.
- **Why:** Tag APIs depend on the store implementation; an unsupported store fails instead of invalidating data.
- **Enforced by:** manual

Before:
```php
// .env
RESPONSE_CACHE_DRIVER=file
```

After:
```php
// .env
RESPONSE_CACHE_DRIVER=redis
```

#### Request JSON explicitly from API clients
- **Do:** Send Accept: application/json from clients that expect JSON validation and exception responses.
- **Flag when:** Flag when an API client expects JSON errors but omits an appropriate Accept header and receives redirects or HTML.
- **Why:** The Accept header tells Laravel which response format the caller expects. Explicit negotiation makes error handling predictable.
- **Enforced by:** manual

Before:
```php
Http::post($url, $data);
```

After:
```php
Http::acceptJson()->post($url, $data);
```

#### Validate every nested payload value
- **Do:** Validate every consumed nested field with dot/wildcard rules (e.g. users.*.email) and restrict nested arrays to their allowed keys (array:key1,key2 / list) so validated() carries no unvalidated children.
- **Flag when:** Flag when a nested array or object is accepted with only a parent array rule (no allowed-keys list or child rules) while its child values are used, persisted or mass-assigned.
- **Why:** Validating the container alone does not establish the validity of its members. Wildcards apply the same contract to each submitted element.
- **Enforced by:** manual

Before:
```php
'users' => ['required', 'array']
```

After:
```php
'users' => ['required', 'list'],
'users.*' => ['required', 'array:email,name'],
'users.*.email' => ['required', 'email'],
'users.*.name' => ['required', 'string', 'max:255'],
```

## Appendix: package-only Must rules excluded (6)

These are in `laravel-best-practices.md` but apply only when writing a Composer package.

| Id | Rule | Why excluded |
|---|---|---|
| `architecture-132` | Release breaking public changes only in a major version | Semantic versioning of published Composer APIs; apps do not publish versioned APIs |
| `architecture-144` | Fail clearly when optional package features need a newer framework | Supporting multiple Laravel majors with compatibility guards is a package-author concern |
| `config-092` | Merge package configuration defaults in the provider’s register() method | mergeConfigFrom package defaults belongs in package service providers |
| `config-305` | Respect cached configuration and skip unnecessary package-config file reads or merges when configuration is cached | Package config loading/merging under config:cache; apps use config files directly |
| `database-410` | Make package migration publication repeat-safe | Repeat-safe migration publishing is package installer behaviour |
| `http-661` | Load package views under a namespace with loadViewsFrom() so applications can override them in resources/views/vendor/<package> | loadViewsFrom registers package views for app override; package-only API |
