# Laravel Techniques

**Version:** 2026-10-13 18:19 (Asia/Manila)

**Purpose:** refactoring strategies and design patterns that improve Laravel 13 code when a specific smell shows up. A reviewing agent uses this to *suggest* improvements, not to flag violations.

**Levels:** **Recommended** = score 7+ · **Useful** = 4–6 · **Situational** = 2–3 (same source tiers and points as the rules docs; each person or org counted once).

## Controllers & business logic

### Action classes (Recommended)

- **Smell:** Controllers or Livewire components contain long business workflows that other entry points duplicate.
- **Technique:** Extract a single business operation into a class with execute or __invoke, accepting plain values or DTOs. Let controllers and Livewire components validate, authorize, and manage UI responses while the action coordinates persistence and domain behavior.
- **When to use:** An operation is reused from HTTP, jobs, commands, or multiple components.
- **When not to:** A trivial CRUD operation may not justify a class; action proliferation can fragment a simple workflow.
- **Packages:** `filament/actions`, `filament/tables`, `livewire/livewire`

Before:
```php
public function save() {
    $user = User::create($this->validate());
    // business steps mixed with UI updates
    $this->redirect("/users");
}
```
After:
```php
class CreateUser {
    public function execute(array $data): User {
        return User::create($data);
    }
}
$user = app(CreateUser::class)->execute($this->validate());
$this->redirect("/users");
```

### Authorization policies (Recommended)

- **Smell:** Ownership and capability conditionals are repeated inside controllers and business operations.
- **Technique:** Move resource authorization into a Laravel policy with methods such as update(User $user, Invoice $invoice). Invoke it through Gate::authorize, route can middleware, or Blade @can so the same decision is reused across entry points.
- **When to use:** Several entry points authorize the same resource or authorization obscures business logic.
- **When not to:** A single unrelated capability may only need a Gate; policies do not replace authentication or query scoping.
- **Packages:** `api-platform/laravel`, `filament/filament`, `inertiajs/inertia-laravel`, `laravel/nova`

Before:
```php
if ($invoice->user_id !== auth()->id()) {
    abort(403);
}
$invoice->update($data);
```
After:
```php
class InvoicePolicy {
    public function update(User $user, Invoice $invoice): bool {
        return $invoice->user_id === $user->id;
    }
}
Gate::authorize("update", $invoice);
$invoice->update($data);
```

### Convention-based route discovery (Recommended)

- **Smell:** Domain modules cannot render their pages through a resolver that only searches the global pages directory.
- **Technique:** Render namespaced identifiers such as booking::passenger/create and map them to module files using import.meta.glob and resolvePageComponent. Retain the ordinary-page fallback and align any root Blade page-asset paths with the same mapping.
- **When to use:** An Inertia modular monolith colocates pages within each business module.
- **When not to:** Default page resolution is sufficient without modules; inconsistent Vite and server mappings cause production asset failures.
- **Packages:** `inertiajs/inertia-laravel`, `laravel/folio`, `spatie/laravel-route-discovery`

Before:
```php
Inertia::render("booking/passenger/create");
// Resolver searches only resources/js/pages.
```
After:
```php
Inertia::render("booking::passenger/create");
// Inertia client resolve callback:
const modules = import.meta.glob("/app-modules/*/resources/js/pages/**/*.tsx");
const [module, page] = name.split("::");
return resolvePageComponent(
  `/app-modules/${module}/resources/js/pages/${page}.tsx`, modules
);
// Keep the standard resolver branch for non-module names.
```

### Resource-oriented controllers (Recommended)

- **Smell:** Controllers collect unrelated custom verbs and many routes repeat CRUD wiring.
- **Technique:** Represent each meaningful resource with a dedicated controller using conventional index, show, store, update, and destroy operations. For a domain action such as publishing, introduce a publication resource and its store operation rather than adding another verb to a broad controller.
- **When to use:** Controller methods grow beyond cohesive resource operations and routes become inconsistent.
- **When not to:** Do not force every RPC-like operation into an artificial resource; standard CRUD controllers are sufficient for simple entities.
- **Packages:** `laravel-shift/blueprint`

Before:
```php
Route::post("/posts/{post}/publish", [PostController::class, "publish"]);
```
After:
```php
Route::post("/posts/{post}/publication", [PublicationController::class, "store"]);
class PublicationController {
    public function store(Post $post, PublishPost $publish) {
        $publish->execute($post);
        return response()->noContent();
    }
}
```

### Guard clauses (Recommended)

- **Smell:** The main operation is buried under nested prerequisite checks.
- **Technique:** Return early for failed prerequisites so the successful path remains at the outer indentation level. Preserve each branch response and Boolean semantics in Laravel 13 controller or service methods.
- **When to use:** Nested conditionals mostly reject invalid cases before one main operation.
- **When not to:** Multiple exits can complicate shared cleanup or genuinely interdependent branches.

Before:
```php
if ($user->active) {
    if ($invoice->payable) {
        return $invoice->pay();
    }
}
return null;
```
After:
```php
if (! $user->active) return null;
if (! $invoice->payable) return null;
return $invoice->pay();
```

### Application services (Useful)

- **Smell:** Business arithmetic sits in a Livewire render method and requires framework/database setup to test.
- **Technique:** Move the calculation into a plain PHP service and inject it into entry points through Laravel's container. Test its business rules directly while retaining integration tests for the UI wiring.
- **When to use:** Calculations need reuse or focused tests independent of UI and storage.
- **When not to:** A one-line calculation used once may not justify a separate class; extraction alone does not verify framework integration.

Before:
```php
$ltv = $this->loan / $this->propertyValue * 100;
```
After:
```php
class CalculateLtv {
    public function __invoke(float $loan, float $value): float {
        return $loan / $value * 100;
    }
}
$ltv = app(CalculateLtv::class)($loan, $value);
```

### Domain exceptions (Useful)

- **Smell:** Generic exceptions carry opaque strings and expected domain failures clutter error reports.
- **Technique:** Create a named exception for each domain failure and provide descriptive static constructors for contextual messages. Implement Illuminate\Contracts\Debug\ShouldntReport for expected failures; translate them at the HTTP or job boundary.
- **When to use:** Callers need to distinguish meaningful business failures and throw sites need to explain the problem.
- **When not to:** Do not suppress reporting for unexpected defects or infrastructure failures; many exception types add maintenance.

Before:
```php
throw new Exception("Cannot pay");
```
After:
```php
class InvoiceAlreadyPaid extends DomainException implements ShouldntReport {
    public static function forInvoice(Invoice $invoice): self {
        return new self("Invoice {$invoice->id} is already paid");
    }
}
throw InvoiceAlreadyPaid::forInvoice($invoice);
```

### Exception-local HTTP rendering (Useful)

- **Smell:** Controllers repeat catch blocks to translate the same error into an HTTP response.
- **Technique:** Implement render(Request $request) on a custom exception to centralize its Laravel HTTP representation. Alternatively use a type-specific renderer in bootstrap/app.php when the exception should remain transport-independent.
- **When to use:** A recurring application exception needs a deliberate status and response shape.
- **When not to:** Exception-local rendering couples the class to HTTP and Laravel.

Before:
```php
try { $service->reserve(); } catch (SoldOut $e) {
  return response()->json(["error" => "Sold out"], 409);
}
```
After:
```php
class SoldOut extends Exception {
  public function render(Request $request) {
    return response()->json(["error" => "Sold out"], 409);
  }
}
```

### Extract method (Useful)

- **Smell:** A vaguely named method hides multiple distinct operations.
- **Technique:** Extract focused methods whose names reveal the steps, leaving the entry method to describe their sequence. This is ordinary PHP refactoring usable in Laravel 13 services and controllers.
- **When to use:** A workflow mixes invoice creation, PDF generation, delivery, or similar separate operations.
- **When not to:** Tiny forwarding methods can add navigation overhead without hiding meaningful detail.

Before:
```php
public function updateData() {
    // create invoice, generate PDF, send email
}
```
After:
```php
public function sendInvoice() {
    $invoice = $this->createInvoice();
    $pdf = $this->generatePdf($invoice);
    $this->emailInvoice($invoice, $pdf);
}
```

### HTTP-independent application operations (Useful)

- **Smell:** API business logic depends on Laravel Request and response serialization, preventing reuse outside HTTP.
- **Technique:** Keep Laravel controllers as adapters that validate input, create plain DTOs, invoke a framework-independent operation, and transform its output. Place Request and response handling at the edge rather than in the operation.
- **When to use:** The same application operation serves an API, a command, and a job.
- **When not to:** An independent domain layer adds mapping and interfaces; straightforward Laravel-only CRUD may not need it.

Before:
```php
class ChargeInvoice {
    public function execute(Request $request): JsonResponse {
        return response()->json(charge($request->input("invoice_id")));
    }
}
```
After:
```php
$input = ChargeInput::fromArray($request->validated());
$result = $chargeInvoice->execute($input);
return new ChargeResource($result);
```

### Middleware composition (Useful)

- **Smell:** Controllers repeat request filtering and before/after processing across endpoints.
- **Technique:** Extract reusable request behavior into Laravel middleware and compose it through the middleware stack. Each layer can reject early or call $next and process the returned response; register application middleware in bootstrap/app.php.
- **When to use:** Multiple endpoints share request/response concerns.
- **When not to:** Business operations tied to one use case may belong in that use case rather than a global request layer.
- **Packages:** `laravel/ai`

Before:
```php
public function index(Request $request) {
    if (! $request->user()) abort(401);
    // Repeated in other controllers.
}
```
After:
```php
Route::middleware("auth")->group(function () {
    Route::get("/orders", [OrderController::class, "index"]);
});
```

### Named exception factories (Useful)

- **Smell:** Related domain failures produce many tiny classes or repeated string messages.
- **Technique:** Group related failures in one domain exception and expose named static constructors carrying the relevant context. Catch the exception family at the application boundary while keeping each throw site expressive.
- **When to use:** Failures share handling behavior and separate catchable types provide little benefit.
- **When not to:** Use distinct types when callers must handle individual failures differently.

Before:
```php
throw new Exception("Missing balance");
throw new Exception("Payment rejected");
```
After:
```php
class PaymentFailed extends DomainException {
    public static function insufficientBalance(): self {
        return new self("Insufficient balance");
    }
}
throw PaymentFailed::insufficientBalance();
```

### Centralized exception reporting (Useful)

- **Smell:** Type-specific reporting logic is scattered across catch blocks or a growing global handler.
- **Technique:** Define report() on the custom exception so Laravel invokes its reporting behavior centrally. Return false from the exception method when default reporting should take over; this differs from false in a global reporting callback.
- **When to use:** A custom exception has specialized reporting requirements.
- **When not to:** Shared domain exceptions may be better kept independent of infrastructure via central callbacks.

Before:
```php
try { $gateway->charge(); } catch (PaymentFailed $e) {
  Log::warning("Payment failed", ["message" => $e->getMessage()]);
  throw $e;
}
```
After:
```php
class PaymentFailed extends Exception {
  public function report(): void {
    Log::warning("Payment failed", ["message" => $this->getMessage()]);
  }
}
```

### Route groups and route-file modularization (Useful)

- **Smell:** One large route file mixes unrelated areas and middleware.
- **Technique:** Move coherent areas into separate files and register file-backed middleware groups. In Laravel 13, additional registration can run through withRouting(then: ...) in bootstrap/app.php.
- **When to use:** Routes have grown into distinct application areas.
- **When not to:** Excessive splitting makes endpoint discovery harder.
- **Packages:** `spatie/laravel-route-discovery`

Before:
```php
// routes/web.php mixes public, account and admin routes.
```
After:
```php
->withRouting(then: function () {
    Route::middleware(["web", "auth", "can:admin"])
        ->group(base_path("routes/admin.php"));
})
```

### Signed URLs (temporary access links) (Useful)

- **Smell:** Emailed downloads require custom token tables or expose permanent unauthenticated file URLs.
- **Technique:** Create a temporarySignedRoute for a download endpoint and apply signed middleware before serving a private file. Bind the link to a recipient or add authorization where downloads require identity checks.
- **When to use:** An expiring bearer link is an appropriate access model for an emailed file.
- **When not to:** Anyone possessing the link can use it unless identity is checked; revocation needs additional state.

Before:
```php
Mail::to($email)->send(new ReportReady("/storage/report.pdf"));
```
After:
```php
$url = URL::temporarySignedRoute("reports.download", now()->addMinutes(30), ["report" => $report->id]);
Route::get("/reports/{report}/download", DownloadReport::class)
    ->name("reports.download")->middleware("signed");
```

### Content negotiation (Situational)

- **Smell:** Separate JSON and PDF endpoints repeat lookup and authorization for the same resource.
- **Technique:** Serve representations from one resource endpoint using Accept negotiation. Laravel 13 request prefers()/accepts() helpers select a supported media type, with appropriate handling of unsupported requests.
- **When to use:** The same resource has several representations.
- **When not to:** Separate endpoints may be clearer when operations or permissions differ; caching must vary by Accept.

Before:
```php
Route::get("/invoices/{invoice}", JsonInvoiceController::class);
Route::get("/invoices/{invoice}/pdf", PdfInvoiceController::class);
```
After:
```php
return match ($request->prefers(["application/json", "application/pdf"])) {
    "application/pdf" => $pdf->render($invoice),
    "application/json" => new InvoiceResource($invoice),
    default => abort(406),
};
```

### Pipelines (Situational)

- **Smell:** A long workflow mixes many ordered responsibilities in one method.
- **Technique:** Extract named, focused pipe classes and compose them with Laravel 13 Pipeline::send()->through()->thenReturn(). Continuing pipes accept the payload and continuation and return $next($payload); container resolution supports injected dependencies.
- **When to use:** Several meaningful stages are hard to maintain or reuse.
- **When not to:** One- or two-step flows are clearer directly; ordering dependencies remain even after extraction.

Before:
```php
$price = validatePrice($input);
$price = convertCurrency($price);
$price = applyFees($price);
return formatPrice($price);
```
After:
```php
return Pipeline::send($input)
  ->through([ValidatePrice::class, ConvertCurrency::class,
    ApplyFees::class, FormatPrice::class])
  ->thenReturn();
```

## Models & domain state

### Event sourcing (Recommended)

- **Smell:** Legacy behavior is poorly understood and mutable rows do not preserve enough history to reconstruct it.
- **Technique:** Capture meaningful operations as typed events in a durable event store before changing the visible behavior. Use application projections to reconstruct state; ordinary Laravel event dispatch alone does not persist replayable history.
- **When to use:** A risky legacy migration needs recoverable historical behavior.
- **When not to:** Event storage, schema evolution and replay add considerable complexity; ordinary CRUD may be sufficient.
- **Packages:** `hirethunk/verbs`, `spatie/laravel-event-sourcing`

Before:
```php
LegacyAnswer::create($request->all());
```
After:
```php
// Application event-store API:
$eventStore->append(new AnswerSubmitted($userId, $questionId, $answer));
// Projectors derive persisted state from stored events.
```

### Custom Eloquent relationship adapters (Recommended)

- **Smell:** Proximity queries are repeated manually because a foreign key cannot express nearby shops.
- **Technique:** Expose an application-defined spatial Relation that constrains by spherical distance and matches results to each parent. The demonstrated HasNear uses MySQL distance calculations; its eager-loading implementation must support multiple parent locations.
- **When to use:** Geographic proximity is a useful model association.
- **When not to:** Distance SQL and indexing are database-specific, and per-parent matching can be expensive.
- **Packages:** `filament/tables`, `prism-php/prism`

Before:
```php
$shops = Shop::selectRaw("*, ST_Distance_Sphere(location, ?) AS d", [$point])
    ->having("d", "<", 500)->get();
```
After:
```php
// App-defined HasNear owns distance constraints and matching.
public function nearbyShops(): HasNear {
    return new HasNear(Shop::query(), $this);
}
$venues = Venue::with("nearbyShops")->get();
```

### Custom Eloquent collections (Useful)

- **Smell:** Callers repeat operations on sets of domain models using anonymous collection callbacks.
- **Technique:** Extract the set-level behavior into an Eloquent Collection subclass and return it through the model newCollection method. Domain-named collection methods make callers express intent rather than repeat filtering or aggregation.
- **When to use:** Several callers manipulate the same kind of model collection in the same way.
- **When not to:** Keep database filtering in queries for large sets; not every one-off callback needs a named method.

Before:
```php
$paid = $invoices->filter(fn ($invoice) => $invoice->status === "paid");
```
After:
```php
class InvoiceCollection extends Collection {
    public function paid(): static {
        return $this->filter(fn ($invoice) => $invoice->status === "paid");
    }
}
// Invoice model:
public function newCollection(array $models = []): InvoiceCollection { return new InvoiceCollection($models); }
$paid = $invoices->paid();
```

### Reusable bootable model traits (Useful)

- **Smell:** Several models duplicate lifecycle hooks for the same behavior.
- **Technique:** Extract the behavior into a trait with a boot method named boot followed by the trait name. Eloquent calls it when booting each model that uses the trait, centralizing event registration.
- **When to use:** A lifecycle concern such as slug initialization genuinely applies to several models.
- **When not to:** Traits can conceal side effects and conflict; model-specific behavior may be clearer in an observer.

Before:
```php
protected static function booted(): void {
    static::creating(fn ($model) => $model->slug = Str::slug($model->title));
}
```
After:
```php
trait HasSlug {
    protected static function bootHasSlug(): void {
        static::creating(fn ($model) => $model->slug = Str::slug($model->title));
    }
}
class Post extends Model { use HasSlug; }
```

### State pattern (state classes) (Useful)

- **Smell:** Status checks and state-dependent side effects are repeated across invoice entry points.
- **Technique:** Put each state behind an event contract, with base methods rejecting unsupported events and concrete states implementing allowed transitions. Resolve the state from the model status and delegate through calls such as $invoice->state()->pay().
- **When to use:** The same workflow event has different behavior or legality depending on state.
- **When not to:** Extra classes are costly for simple workflows; direct status writes and concurrent transitions still require control.
- **Packages:** `hirethunk/verbs`

Before:
```php
if ($invoice->status === "pending") {
    $invoice->update(["status" => "paid"]);
    Mail::to($invoice->customer)->send(new Receipt($invoice));
}
```
After:
```php
abstract class InvoiceState { public function pay(): void { throw new InvalidTransition(); } }
class PendingState extends InvoiceState {
    public function __construct(private Invoice $invoice) {}
    public function pay(): void {
        $this->invoice->update(["status" => "paid"]);
        Mail::to($this->invoice->customer)->send(new Receipt($this->invoice));
    } }
$invoice->state()->pay();
```

### Relationship-based persistence (Useful)

- **Smell:** Ad hoc attach/remove endpoints proliferate for each relationship.
- **Technique:** Expose a JSON:API relationship endpoint with linkage payloads for attaching, detaching, or replacing related records. Implement validation, authorization, and Eloquent relationship mutations separately from JsonApiResource.
- **When to use:** Clients need a consistent relationship editing contract.
- **When not to:** Do not expose relationships whose writes need a different domain workflow.
- **Packages:** `filament/forms`

Before:
```php
POST /posts/7/add-tag
{"tag_id": 4}
```
After:
```php
POST /posts/7/relationships/tags
{"data": [{"type": "tags", "id": "4"}]}
```

### Projection replay (Useful)

- **Smell:** Fixing a calculation bug affects new writes but leaves the old leaderboard incorrect.
- **Technique:** Correct the replayable projection or state-application logic and rebuild affected derived data from preserved events. Reset only derived data or use idempotent handlers; validation-only fixes do not alter replay of previously accepted events.
- **When to use:** Recorded events retain enough information to calculate corrected results.
- **When not to:** Replay does not clear arbitrary models automatically and can repeat unguarded external effects.
- **Packages:** `hirethunk/verbs`, `spatie/laravel-event-sourcing`

Before:
```php
// Update the calculation; old totals remain wrong.
$score += correctedBonus($event);
```
After:
```php
// Correct replayable apply()/handle() logic.
// Reset affected derived leaderboard data safely.
php artisan verbs:replay
```

### Rich domain models (Useful)

- **Smell:** Callers construct posts with low-level field arrays and repeat domain defaults.
- **Technique:** Expose a publish method on the Post model that expresses the operation and supplies its domain defaults. Controllers call this method instead of reconstructing persistence details.
- **When to use:** A cohesive entity operation recurs across entry points.
- **When not to:** Models can become too large if they absorb cross-resource orchestration.

Before:
```php
Post::create(["profile_id" => $profile->id, "content" => $text]);
```
After:
```php
Post::publish($profile, $text);
```

### Split model responsibilities (Useful)

- **Smell:** A growing model mixes persistence, query construction, and set-level behavior.
- **Technique:** Move responsibilities to the Eloquent extension point that owns them, retaining the model as the normal query entry point. Laravel 13 supports custom builders, collections, and observers; these are illustrative implementation options for the abstract’s broader recommendation.
- **When to use:** Models have multiple independently growing responsibilities.
- **When not to:** Extra classes obscure a small model with only a few simple methods.

Before:
```php
class Invoice extends Model {
    // Persistence, query helpers, and set operations...
}
```
After:
```php
class Invoice extends Model {
    // Persistence and single-record behavior.
}
// Query behavior: InvoiceBuilder.
// Set behavior: InvoiceCollection.
// Lifecycle reactions: InvoiceObserver.
```

### Array-backed Eloquent models (Situational)

- **Smell:** Small reference datasets need Eloquent queries and relationships but database migrations add unnecessary storage work.
- **Technique:** Use Sushi to expose a static array as an Eloquent model’s rows. Existing Laravel code can query that model and relate to the reference data.
- **When to use:** A small, mostly static reference list is maintained in code.
- **When not to:** Not suitable for mutable user data or large datasets; array data and cached SQLite state need care.
- **Packages:** `calebporzio/sushi`

Before:
```php
$locations = collect([["id" => 1, "name" => "London"]]);
```
After:
```php
class Location extends Model {
    use \Sushi\Sushi;
    protected $rows = [["id" => 1, "name" => "London"]];
}
```

### Attribute-based model registration (Situational)

- **Smell:** Model observers and global scopes are registered in scattered boot methods or service providers.
- **Technique:** Use ObservedBy and ScopedBy attributes to attach observers and global scopes beside the model declaration. Laravel 13 discovers these declarations during model setup.
- **When to use:** A model has stable observer or scope registrations that should be visible locally.
- **When not to:** Environment-dependent registrations may be clearer in a provider; global scopes can hide query constraints.

Before:
```php
protected static function booted(): void {
    static::observe(UserObserver::class);
    static::addGlobalScope(ActiveScope::class);
}
```
After:
```php
#[ObservedBy([UserObserver::class])]
#[ScopedBy([ActiveScope::class])]
class User extends Model {}
```

### Property hooks (Situational)

- **Smell:** Local property invariants are repeated in lifecycle hooks or assignment sites.
- **Technique:** On PHP 8.4 or newer, encapsulate simple assignment invariants in a property setter hook. Verify Livewire hydration behavior for the installed version before replacing framework lifecycle hooks; Laravel 13 itself can run on PHP 8.3 where these hooks are unavailable.
- **When to use:** A local value such as a counter must satisfy the same bound after each assignment.
- **When not to:** Hooks require PHP 8.4+, and complex cross-field validation or client authorization needs other mechanisms.

Before:
```php
public int $count = 0;
public function updatedCount() {
    $this->count = max(0, $this->count);
}
```
After:
```php
public int $count = 0 {
    set { $this->count = max(0, $value); }
}
```

### Reservable models (Situational)

- **Smell:** Workers can select the same available model before either marks it as in use.
- **Technique:** Extract reservation behavior into a reusable model trait that claims resources atomically and records reservation timestamps. Query available resources through scopes and release or expire reservations deliberately.
- **When to use:** Several workers compete for scarce resources represented by database rows.
- **When not to:** Reservations need expiry and recovery; use a simple transaction when a claim does not outlive the transaction.

Before:
```php
$resource = Resource::whereNull("reserved_at")->first();
$resource->update(["reserved_at" => now()]);
```
After:
```php
// Application-defined reservation trait
class Resource extends Model { use Reservable; }
$resource = Resource::reserveFirstAvailable();
try { useResource($resource); }
finally { $resource->releaseReservation(); }
```

### Scoped creation defaults (Situational)

- **Smell:** A relationship or scope filters a category but callers must separately repeat its attribute values when creating records.
- **Technique:** Use withAttributes on the Eloquent relationship or scope to apply both query constraints and creation defaults. New models created through that builder inherit the same attributes.
- **When to use:** Records created through a filtered relationship must belong to that category.
- **When not to:** Use ordinary where clauses when a query filter should not influence new records.

Before:
```php
public function publishedPosts() {
    return $this->hasMany(Post::class)->where("published", true);
}
$user->publishedPosts()->create(["title" => $title, "published" => true]);
```
After:
```php
public function publishedPosts() {
    return $this->hasMany(Post::class)->withAttributes(["published" => true]);
}
$user->publishedPosts()->create(["title" => $title]);
```

### Team-owned resources (Situational)

- **Smell:** Resources attached directly to users are difficult to share or transfer when collaboration arrives.
- **Technique:** Introduce a team ownership boundary, store team_id on resources, and authorize access through team membership and capabilities. Scope resource queries to the current team rather than the individual creator.
- **When to use:** Shared workspaces, billing ownership, or transferable resources are expected.
- **When not to:** Single-user applications may not need teams; every query, job, and cache boundary must respect the team.
- **Packages:** `laravel/jetstream`

Before:
```php
$projects = auth()->user()->projects;
```
After:
```php
$team = auth()->user()->currentTeam;
$projects = $team->projects()->get();
Gate::authorize("view", $team);
```

### Custom pivot models (Situational)

- **Smell:** Multiple Filament panels duplicate branding and configuration.
- **Technique:** Move common panel configuration into a BasePanelProvider and let concrete providers add their own settings. Keep shared behavior in one place.
- **When to use:** Admin and user panels share a meaningful configuration baseline.
- **When not to:** Inheritance can couple panels that should evolve independently.
- **Packages:** `filament/filament`

Before:
```php
// AdminPanelProvider and AppPanelProvider repeat colors and fonts.
```
After:
```php
abstract class BasePanelProvider extends PanelProvider {
 protected function configureBase(Panel $panel): Panel { /* shared settings */ }
}
```

### Encapsulated domain invariants (Situational)

- **Smell:** Collection and factory helpers erase the precise types of their results.
- **Technique:** Annotate reusable PHP abstractions with PHPStan templates and class-string types. Static analysis can track concrete Laravel model types through generic helpers.
- **When to use:** Reusable containers or factories lose type information.
- **When not to:** Annotations add maintenance and should not pretend to guarantee runtime validation.
- **Packages:** `phpstan/phpstan`

Before:
```php
/** @return object */
function resolveType(string $class) { return app($class); }
```
After:
```php
/** @template T of object
 * @param class-string<T> $class
 * @return T */
function resolveType(string $class) { return app($class); }
```

### Event-sourced aggregates (Situational)

- **Smell:** Business decisions need past activity that a current database snapshot does not capture.
- **Technique:** Reconstitute an entity aggregate from stored events, apply them to internal decision state, and route commands through it. Record and persist permitted events instead of editing a projection directly.
- **When to use:** Invariants such as withdrawal limits depend on an entity history.
- **When not to:** Reconstitution, concurrency control, and event evolution add complexity; simple independent writes rarely need aggregates.
- **Packages:** `spatie/laravel-event-sourcing`

Before:
```php
if ($account->balance >= $amount) {
    $account->decrement("balance", $amount);
}
```
After:
```php
AccountAggregate::retrieve($uuid)
    ->withdraw($amount)
    ->persist();
```

### Model media collections (Situational)

- **Smell:** Controllers hand-roll model-to-file associations, storage paths, and derived image handling.
- **Technique:** Attach media to Eloquent models through named Media Library collections and define conversions on the model. Let a compatible package release manage storage metadata and queued derivatives rather than repeating file bookkeeping.
- **When to use:** Models own uploads and need multiple derived images or consistent retrieval.
- **When not to:** Simple single-file storage may only need Laravel Storage; packages add migrations, workers, and upgrade requirements.
- **Packages:** `spatie/laravel-medialibrary`

Before:
```php
$path = $request->file("avatar")->store("avatars");
$user->update(["avatar_path" => $path]);
```
After:
```php
// User implements HasMedia and uses InteractsWithMedia
$user->addMediaFromRequest("avatar")->toMediaCollection("avatars");
$url = $user->getFirstMediaUrl("avatars");
```

### Polymorphic relationships (Situational)

- **Smell:** Code manually traverses an intermediate model and assembles distant related records.
- **Technique:** Define a hasManyThrough relationship to express the path through the intermediate model. Eloquent can query the distant relation without application-level nested loops.
- **When to use:** A stable two-hop relationship should behave like a direct association.
- **When not to:** Unusual keys and constraints can complicate the relationship; arbitrary joins may be clearer.

Before:
```php
$tasks = $company->users->flatMap(fn ($user) => $user->tasks);
```
After:
```php
public function tasks(): HasManyThrough {
 return $this->hasManyThrough(Task::class, User::class);
}
```

### Rejection events (Situational)

- **Smell:** Rejected commands disappear even though repeated failures should influence later decisions.
- **Technique:** Record a rejection event without performing the rejected state change and let the aggregate use that history for later decisions. Commit the rejection before surfacing failure so an escaping transaction exception does not roll it back.
- **When to use:** Repeated limit hits or denied attempts trigger a meaningful business response.
- **When not to:** Noise-only failures can overwhelm the event store; privacy and retention of attempted actions also matter.
- **Packages:** `spatie/laravel-event-sourcing`

Before:
```php
if ($amount > $balance) {
    throw new WithdrawalDenied;
}
```
After:
```php
if ($amount > $balance) {
    $aggregate->recordRejection($amount)->persist();
    // rejection must be committed before raising the failure
    throw new WithdrawalDenied;
}
```

### Scoped conversation history (Situational)

- **Smell:** One shared chat history mixes context from different support tickets.
- **Technique:** Associate an AI conversation ID with each ticket and use RemembersConversations to maintain its history. The controller starts or resumes the conversation tied to that ticket.
- **When to use:** Users discuss separate entities with separate contextual histories.
- **When not to:** Persisted histories add storage and access-control responsibilities; one-shot tasks do not need them.
- **Packages:** `laravel/ai`

Before:
```php
$assistant->prompt($message); // no ticket-specific history
```
After:
```php
// TicketAssistant uses RemembersConversations.
$assistant->continue($ticket->ai_conversation_id, as: $user)
    ->prompt($message);
```

### Soft deletion (Situational)

- **Smell:** Accidental deletion of important records is difficult to recover.
- **Technique:** Add a deleted_at column and use SoftDeletes so ordinary deletion hides a record while preserving it for restoration.
- **When to use:** Recovery is required for normal user-driven deletion.
- **When not to:** Retained rows affect unique constraints and privacy retention; forceDelete() remains destructive.

Before:
```php
$invoice->delete(); // permanently removed
```
After:
```php
// Migration: $table->softDeletes();
class Invoice extends Model {
    use SoftDeletes;
}
$invoice->delete();
Invoice::withTrashed()->findOrFail($id)->restore();
```

### Surrogate keys with composite uniqueness (Situational)

- **Smell:** A composite business key is being forced into an Eloquent primary key or an unreliable composite-key workaround.
- **Technique:** Retain a single primary key for Eloquent identity and add a composite unique index for business identity. Laravel 13 does not support composite Eloquent primary keys.
- **When to use:** Models have a multi-column natural key but need normal Eloquent relationships and persistence.
- **When not to:** An unmanaged legacy schema may prevent adding a surrogate key; plan data cleanup and migration first.

Before:
```php
$table->primary(["package_path", "version"]);
// Eloquent cannot use this tuple as its primary key.
```
After:
```php
$table->id();
$table->string("package_path");
$table->string("version");
$table->unique(["package_path", "version"]);
```

## Data & types

### Data transfer objects (Recommended)

- **Smell:** A PHP API client and server disagree about response field names and nested shapes.
- **Technique:** Map an Eloquent model into an explicit response DTO and hydrate PHP client responses into that contract before accessing fields. Laravel Data provides transformation and hydration, while explicit validation is needed for untrusted payloads.
- **When to use:** Maintaining both an API and its PHP SDK or other typed consumers.
- **When not to:** Shared contracts couple release/version decisions; hydration alone does not establish payload validity.
- **Packages:** `api-platform/laravel`, `inertiajs/inertia-laravel`, `laravel-echo`, `laravel/reverb`, `livewire/livewire`, `spatie/laravel-data`

Before:
```php
$json = Http::get($url)->json();
$name = $json["full_name"];
```
After:
```php
// Server: explicit UserData::fromModel factory.
return UserData::fromModel($user);
// PHP client:
$data = UserData::validateAndCreate(Http::get($url)->json());
$name = $data->name;
```

### API resource projections (Recommended)

- **Smell:** API controllers manually convert application DTOs into HTTP payloads.
- **Technique:** Extract a transformer that maps a DTO to the response shape. Laravel 13 controllers can call the application action and then return the transformed array as JSON, keeping HTTP representation outside the application data object.
- **When to use:** API versions or different clients need distinct representations of the same application data.
- **When not to:** A tiny stable response may be clearer inline; JsonResource can provide the same boundary for suitable data.
- **Packages:** `inertiajs/inertia-laravel`, `spatie/laravel-data`

Before:
```php
$data = $action->run($input);
return response()->json(["id" => $data->id, "name" => $data->name]);
```
After:
```php
$data = $action->run($input);
return response()->json($transformer->transform($data));
```

### Validated data transfer objects (Recommended)

- **Smell:** Dynamic request properties hide field typos and pass loosely typed input into business code.
- **Technique:** Define typed request data with Spatie Laravel Data and retain validation when constructing it from the request. Alternatively, map a Form Request's validated fields into a plain DTO; ordinary array-based Data::from() is not automatic validation.
- **When to use:** Application operations need a typed input contract beyond the HTTP layer.
- **When not to:** DTO conversion adds mapping; complex authorization or validation may be better kept in a separate Form Request.
- **Packages:** `spatie/laravel-data`, `wendelladriel/laravel-validated-dto`

Before:
```php
$service->create($request->name, $request->emial);
```
After:
```php
class CreateUserData extends Data {
    public function __construct(
        public string $name, #[Email] public string $email
    ) {}
}
$data = CreateUserData::validateAndCreate($request->all());
$service->create($data);
```

### Immutable dates (Recommended)

- **Smell:** Mutating a shared date while calculating a deadline silently changes the original value.
- **Technique:** Configure Laravel Date to use CarbonImmutable so date arithmetic returns a new instance. Assign the returned value explicitly and use immutable date casts for model attributes where the same behavior is required.
- **When to use:** Date values are passed between operations and accidental in-place mutation causes bugs.
- **When not to:** Existing code that relies on mutating dates needs adjustment; immutability alone does not address timezone or calendar semantics.

Before:
```php
$deadline = $startedAt->addDays(7);
// Mutable $startedAt was also changed
```
After:
```php
Date::use(CarbonImmutable::class);
$startedAt = Date::now();
$deadline = $startedAt->addDays(7);
```

### Generic type annotations (Recommended)

- **Smell:** Bare collection and relationship return types lose the concrete related model during analysis.
- **Technique:** Supplement native types with generic PHPDoc key/model parameters on collections and related/declaring-model parameters on relationships. Laravel 13 Larastan can then retain model information through iteration and query chains.
- **When to use:** Callers need reliable model inference across Eloquent API boundaries.
- **When not to:** Wrong annotations mislead the analyzer and add maintenance; actual keys and relationship models must be accurate.
- **Packages:** `larastan/larastan`, `phpstan/phpstan`

Before:
```php
public function posts(): HasMany {
    return $this->hasMany(Post::class);
}
```
After:
```php
/** @return HasMany<Post, $this> */
public function posts(): HasMany {
    return $this->hasMany(Post::class);
}
```

### Collection transformation pipelines (Recommended)

- **Smell:** Code repeatedly converts collections into arrays for intermediate transformations.
- **Technique:** Keep transformations in the collection with map, pluck, and related methods, converting only at the required output boundary. In Laravel 13, preserve keys and nested conversion behavior when replacing array functions.
- **When to use:** Intermediate conversions obscure a fluent data transformation.
- **When not to:** A required array API or memory-sensitive workload may justify earlier conversion or streaming.

Before:
```php
$names = array_map(fn ($user) => $user["name"], $users->toArray());
return $names;
```
After:
```php
return $users->pluck("name")->all();
```

### Backed enums (Recommended)

- **Smell:** Lifecycle state is inferred from combinations of flags and timestamps that permit contradictory combinations.
- **Technique:** Persist one explicit current status with a defined initial value and model permitted transitions as a state machine. Keep independent facts and audit timestamps separate from mutually exclusive lifecycle states.
- **When to use:** A model moves through a defined workflow with constrained actions.
- **When not to:** Independent booleans should not be forced into one state, and a trivial two-state workflow may need little machinery.
- **Packages:** `spatie/laravel-data`

Before:
```php
if ($invoice->paid_at === null && !$invoice->cancelled) {
    // Maybe payable.
}
```
After:
```php
$invoice->status = "pending";
$invoice->save();
// Workflow methods own permitted status changes.
```

### Eloquent attribute casts (Recommended)

- **Smell:** Eloquent cast declarations need helper expressions that cannot be used in property initializers.
- **Technique:** Move cast definitions into protected casts(): array, allowing expressive declarations such as AsEnumCollection::of(Status::class). Laravel 13 applies those definitions when hydrating and persisting model attributes.
- **When to use:** Cast configuration benefits from factory methods or other expressions.
- **When not to:** A static cast property may already be sufficient; method syntax alone does not improve an unsuitable data model.

Before:
```php
protected $casts = ["statuses" => /* expression needed */];
```
After:
```php
protected function casts(): array {
    return ["statuses" => AsEnumCollection::of(Status::class)];
}
```

### Generated frontend types (Recommended)

- **Smell:** PHP DTOs and enums drift from separately maintained frontend type declarations.
- **Technique:** Use a TypeScript transformer to generate frontend definitions from PHP data classes and enums during the build. Make the PHP declarations the authoritative shape and import the generated types into Inertia or other frontend code.
- **When to use:** Laravel and TypeScript clients share many structured payloads.
- **When not to:** Generated types do not validate runtime network data; the build gains a dependency and unsupported PHP types need mapping.
- **Packages:** `spatie/laravel-data`, `spatie/laravel-typescript-transformer`, `spatie/typescript-transformer`

Before:
```php
// PHP: public int $amount;
// TS maintained manually: amount: string;
```
After:
```php
// PHP DTO is the source of truth
class InvoiceData { public int $amount; }
// Build: php artisan typescript:transform
// Generated TS: type InvoiceData = { amount: number };
```

### Lazy and on-demand response properties (Recommended)

- **Smell:** Components eagerly retrieve models that some render paths never use.
- **Technique:** Wrap model retrieval in a native PHP lazy proxy so the query executes when the model is actually accessed. Laravel 13 runs on PHP versions supporting lazy objects, allowing this pattern without relying on a proposed helper API.
- **When to use:** An expensive object lookup is only needed on some component paths.
- **When not to:** Deferred failures and implicit query timing complicate debugging; test serialization and interactions with Eloquent carefully.
- **Packages:** `@inertiajs/vue3`, `inertiajs/inertia-laravel`, `spatie/laravel-data`

Before:
```php
$user = User::findOrFail($id);
// Some render paths never access $user.
```
After:
```php
$user = (new ReflectionClass(User::class))->newLazyProxy(
    fn () => User::findOrFail($id)
);
// The query runs when an initializing access occurs.
```

### Immutable value objects (Useful)

- **Smell:** Value-like objects can be reassigned after construction and violate their intended semantics.
- **Technique:** Use typed readonly classes for value objects and Pest architecture checks to protect agreed conventions. Readonly prevents property reassignment but does not make nested referenced objects deeply immutable.
- **When to use:** Values should retain their construction-time meaning.
- **When not to:** Mutable entities and framework models need a different design.
- **Packages:** `pestphp/pest`

Before:
```php
class Money { public int $cents; }
$money->cents = 0;
```
After:
```php
readonly class Money {
    public function __construct(public int $cents) {}
}
arch()->expect("App\ValueObjects")->toBeReadonly();
```

### Value object casts (Useful)

- **Smell:** Structured address or money data is passed around as loosely shaped arrays.
- **Technique:** Introduce typed value objects and register an Eloquent custom cast or compatible Laravel Data cast to convert them to and from storage. Use a casts() method and implement serialization where the object is returned as JSON.
- **When to use:** A structured model attribute needs a stable shape and typed access.
- **When not to:** Custom conversion and serialization add work; simple scalar attributes do not need objects.
- **Packages:** `propaganistas/laravel-phone`, `spatie/laravel-data`

Before:
```php
$user->address = ["street" => "Main", "postcode" => "1000"];
$street = $user->address["street"];
```
After:
```php
protected function casts(): array {
    return ["address" => AddressCast::class];
}
$user->address = new Address("Main", "1000");
$street = $user->address->street;
```

### Value objects (Useful)

- **Smell:** Primitive arguments hide domain meaning and allow invalid quantities to enter business operations.
- **Technique:** Replace ambiguous numbers with domain types such as Duration and Discount, validating their invariants during construction. Eloquent custom casts can reconstruct these types when persistence needs them.
- **When to use:** Units or value constraints carry business meaning across several callers.
- **When not to:** Wrapping every primitive increases class count and conversion overhead without always improving clarity.
- **Packages:** `laravel/scout`

Before:
```php
$invoice->applyDiscount(0.25);
$runner->wait(5);
```
After:
```php
$invoice->applyDiscount(Discount::percentOff(25));
$runner->wait(Duration::seconds(5));
```

### Encrypted attribute casts (Useful)

- **Smell:** Scattered encryption calls allow some model read/write paths to miss encryption.
- **Technique:** Declare encrypted fields on the Eloquent model so ordinary attribute access passes through the encryption layer. Laravel 13 built-in encrypted casts handle app-key encryption; recipient wrapping needs a separate compatible implementation.
- **When to use:** All ordinary model access to a field should consistently encrypt and decrypt.
- **When not to:** Bulk database writes bypass model processing and encrypted values cannot be searched normally.

Before:
```php
$patient->notes = Crypt::encryptString($notes);
$plain = Crypt::decryptString($patient->notes);
```
After:
```php
protected function casts(): array {
    return ["notes" => "encrypted"];
}
$patient->notes = $notes;
$plain = $patient->notes;
```

### Query-time casts (Useful)

- **Smell:** SQL projections and aggregate aliases need manual conversion but should not become permanent model attributes.
- **Technique:** Use withCasts on the Eloquent query to type only the values returned by that query. Keep expressive SQL projections while exposing their results as appropriate PHP values.
- **When to use:** An aggregate or subquery returns a computed date or other castable value.
- **When not to:** A type used on every model query is better declared in the model casts.

Before:
```php
$user = User::selectRaw("users.*, MAX(posts.created_at) AS last_posted_at")
    ->join("posts", "posts.user_id", "=", "users.id")->groupBy("users.id")->first();
$date = Carbon::parse($user->last_posted_at);
```
After:
```php
$user = User::select("users.*")->addSelect([
    "last_posted_at" => Post::selectRaw("MAX(created_at)")->whereColumn("user_id", "users.id"),
])->withCasts(["last_posted_at" => "immutable_datetime"])->first();
$date = $user->last_posted_at;
```

### Shared API response contracts (Useful)

- **Smell:** Resource documents and query conventions differ across endpoints.
- **Technique:** Adopt JSON:API progressively: implement its mandatory contract and add optional capabilities as needed. Laravel 13 JsonApiResource supplies resource document serialization; request handling still needs explicit implementation.
- **When to use:** An API benefits from a standard shared with clients.
- **When not to:** A narrow internal API may not justify full protocol handling.

Before:
```php
return ["post" => $post->toArray()];
```
After:
```php
class PostResource extends JsonApiResource {
    public $attributes = ["title"];
    public $relationships = ["author"];
}
return new PostResource($post);
```

### Typed collection casts (Useful)

- **Smell:** Each consumer manually maps JSON attribute items into objects.
- **Technique:** Declare an AsCollection::of(...) cast on the model to map each JSON item into an application object automatically. Give the item class compatible construction and array/JSON serialization so it can also be saved.
- **When to use:** A JSON array contains repeated structured items with behavior or a stable shape.
- **When not to:** Simple scalar lists do not need object hydration; large collections incur allocation costs.

Before:
```php
$options = collect(json_decode($model->getRawOriginal("options"), true))
    ->mapInto(Option::class);
```
After:
```php
protected function casts(): array {
    return ["options" => AsCollection::of(Option::class)];
}
$options = $model->options;
```

### Hashed attribute casts (Useful)

- **Smell:** Password hashing is duplicated in controllers, factories, and import paths or forgotten by a writer.
- **Technique:** Declare a hashed Eloquent cast for the password attribute and assign plain input through the model. The cast centralizes hashing and avoids rehashing values already recognized as appropriate hashes.
- **When to use:** Several model write paths set a password-like attribute.
- **When not to:** Direct query-builder writes bypass casts; never serialize or log the plaintext and do not use this for recoverable secrets.

Before:
```php
$user->password = Hash::make($password);
$user->save();
```
After:
```php
// User model
protected function casts(): array {
    return ["password" => "hashed"];
}
$user->password = $password;
$user->save();
```

### Database JSON schema constraints (Situational)

- **Smell:** A JSON column accumulates incompatible structures because only some application paths validate it.
- **Technique:** Add a MySQL CHECK constraint using JSON_SCHEMA_VALID with the expected JSON schema. Keep Laravel boundary validation for helpful errors while the database enforces the stored shape for every writer.
- **When to use:** Semi-structured persisted data still has a stable required shape and several writers.
- **When not to:** Database support and schema evolution require care; frequently queried relational attributes may belong in regular columns.

Before:
```php
DB::table("profiles")->insert(["data" => json_encode($unvalidated)]);
```
After:
```php
$schema = json_encode(["type" => "object", "required" => ["name"]]);
// Migration helper safely quotes the static schema literal
addJsonSchemaCheck("profiles", "data", $schema);
// CHECK (JSON_SCHEMA_VALID(schema_literal, data))
```

### Derived TypeScript types (Situational)

- **Smell:** Inline TypeScript prop types make frontend components verbose and duplicate shared shapes.
- **Technique:** Extract named type aliases and colocate component-specific types; move shared types to importable modules as reuse appears. Laravel payload types remain explicit in the frontend.
- **When to use:** Several components share types or inline declarations obscure component behavior.
- **When not to:** A giant shared types file creates coupling; trivial local types can remain inline.

Before:
```php
function Edit({ song }: { song: { id: number; title: string } }) {}
```
After:
```php
type Song = { id: number; title: string };
type EditProps = { song: Song };
function Edit({ song }: EditProps) {}
```

### Explicit typed operation parameters (Situational)

- **Smell:** Positional calls with optional parameters obscure the meaning of values.
- **Technique:** Use PHP named arguments when parameter names clarify the call, particularly for optional application-owned APIs. Laravel 13 runs on PHP versions supporting this syntax, but framework parameter names are outside Laravel backward-compatibility guarantees.
- **When to use:** Boolean or optional positional arguments are difficult to interpret.
- **When not to:** Parameter renames break named calls and framework names may change; do not use names merely for cosmetic consistency.

Before:
```php
$exporter->export($records, false, true);
```
After:
```php
$exporter->export($records, includeHeaders: false, compress: true);
```

### Opaque public identifiers (Situational)

- **Smell:** Public URLs expose raw sequential IDs and identifier formatting is duplicated across models.
- **Technique:** Keep efficient internal integer keys and expose a prefixed encoded public identifier through a reusable trait and route-binding logic. Decode to the internal key at the boundary while applying ordinary authorization.
- **When to use:** Public identifier presentation should be stable and distinct from storage keys.
- **When not to:** Hashids are reversible and provide no access control; UUIDs may be simpler when independent ID generation is needed.
- **Packages:** `hashids/hashids`

Before:
```php
route("orders.show", $order->id);
```
After:
```php
// Application-defined trait handles encoding and binding
class Order extends Model { use HasPublicHashid; }
$url = route("orders.show", $order->publicId());
// Example public value: order_K7a2
```

### Envelope encryption and scoped key providers (Situational)

- **Smell:** One server-held decryption key grants access to all sensitive records.
- **Technique:** Encrypt each record with a random symmetric key and wrap that key for authorized recipients using public keys. Laravel 13 requires a recipient-key implementation beyond its app-wide encrypter; use library-managed primitives rather than custom cryptography.
- **When to use:** Decryption rights must be scoped to particular users or recipients.
- **When not to:** Key recovery, rotation, sharing, and revocation are substantial costs; the talk package Laravel 13 compatibility is unverified.
- **Packages:** `customd/eloquent-model-encrypt`

Before:
```php
$ciphertext = Crypt::encryptString($sensitiveData);
// APP_KEY can decrypt every record
```
After:
```php
// library-backed application boundary
$encryptedRecord = $envelopes->encryptFor(
    $sensitiveData, $recipientPublicKeys
);
```

### Named constructors for domain values (Situational)

- **Smell:** Call sites repeatedly convert percentages or time units before constructing domain values.
- **Technique:** Expose named factories that declare accepted units and normalize to a single internal representation. Use those factories at application boundaries and in casts.
- **When to use:** A value has several input units or construction modes.
- **When not to:** A factory adds little when the constructor already has one obvious representation.

Before:
```php
$discount = new Discount($percent / 100);
```
After:
```php
public static function percentOff(float $percent): self {
    return new self($percent / 100);
}
$discount = Discount::percentOff(25);
```

### Parameter objects (Situational)

- **Smell:** Remote API methods have long parameter lists and repeated query-parameter assembly.
- **Technique:** Represent an integration endpoint payload in a typed Params object with a deliberate serialization method. Pass the object to the integration request so defaults and parameter names are defined in one place.
- **When to use:** An API integration has many optional parameters or repeated payload shapes.
- **When not to:** A two-field one-off request may only need an array; do not invent a universal parameter abstraction.

Before:
```php
searchRemote($term, $page, $limit, $sort, $locale);
```
After:
```php
class SearchParams {
    public function __construct(public string $term, public int $page = 1) {}
    public function toArray(): array {
        return ["q" => $this->term, "page" => $this->page];
    }
}
Http::get($url, (new SearchParams($term))->toArray());
```

### Structured AI output (Situational)

- **Smell:** Ticket triage logic tries to extract priority and department from free-form model text.
- **Technique:** Implement HasStructuredOutput with a schema for the triage agent. Use the structured response fields to update the ticket after validating application constraints.
- **When to use:** AI output becomes typed application data.
- **When not to:** A schema controls shape rather than truth; critical decisions may need review.
- **Packages:** `laravel/ai`

Before:
```php
$ticket->priority = parsePriority($response);
```
After:
```php
$result = $triageAgent->prompt($ticket->message);
$ticket->update(["priority" => $result["priority"]]);
```

## Queries & database performance

### Cache-aside result caching (Recommended)

- **Smell:** An expensive stable derived query repeats across component requests.
- **Technique:** Cache selected computed results across requests and invalidate when their inputs change. Livewire 4 distinguishes per-component persistence from cache:true shared across instances; Laravel 13 cached object serialization may require explicit class allowance, so scalar/array results are simpler.
- **When to use:** Expensive reference data changes infrequently.
- **When not to:** Shared caches must not mix user-specific data; cached values require an invalidation policy.
- **Packages:** `livewire/livewire`, `rennokki/laravel-eloquent-query-cache`

Before:
```php
#[Computed]
public function currencies() {
    return Currency::pluck("name", "code")->all();
}
```
After:
```php
#[Computed(cache: true)]
public function currencies() {
    return Currency::pluck("name", "code")->all();
}
// Invalidate this computed cache when currencies change.
```

### Declarative query filter objects (Recommended)

- **Smell:** Endpoints duplicate request-to-filter, sort and include plumbing around Eloquent.
- **Technique:** Use Spatie Query Builder with explicitly allowed filters, sorts, includes and fields, optionally starting from an already constrained Eloquent builder. The note locates the package presentation but reconstructs API detail from its documentation; choose a release compatible with Laravel 13.
- **When to use:** Several endpoints expose structured client query options.
- **When not to:** Simple fixed queries need no extra package; request-driven options must retain authorization constraints.
- **Packages:** `api-platform/laravel`, `filament/tables`, `livewire/livewire`, `spatie/laravel-query-builder`

Before:
```php
$query = User::query();
if ($request->name) $query->where("name", $request->name);
if ($request->sort === "name") $query->orderBy("name");
```
After:
```php
return QueryBuilder::for(User::where("team_id", $teamId))
  ->allowedFilters(["name"])
  ->allowedSorts(["name"])
  ->allowedIncludes(["posts"])
  ->paginate();
```

### Query-driven indexes (Recommended)

- **Smell:** An operational query performs an expensive scan because its access path lacks an index.
- **Technique:** Inspect the expensive query and deployed schema, then add an index matching its filter or ordering in a Laravel 13 migration. The talk identifies a failed_jobs case without establishing its exact column, so the sketch uses an illustrative operational table.
- **When to use:** A measured scan dominates database work and an index fits the query shape.
- **When not to:** Indexes consume storage and slow writes; adding redundant indexes without an execution-plan reason can hurt.

Before:
```php
DB::table("workflow_runs")->where("status", "failed")
    ->orderBy("created_at")->limit(100)->get();
```
After:
```php
Schema::table("workflow_runs", function (Blueprint $table) {
    $table->index(["status", "created_at"]);
});
```

### Relationship aggregates (Recommended)

- **Smell:** Lists load entire related collections merely to count rows or sum a column.
- **Technique:** Use withCount, withSum, or related Eloquent aggregate methods to select scalar aggregates alongside each parent. Read the resulting attributes without hydrating the child models.
- **When to use:** A list or report needs related totals rather than full child data.
- **When not to:** The aggregate subqueries still need indexes; eager-load when the children themselves will also be rendered.
- **Packages:** `filament/filament`

Before:
```php
$orders = Order::with("items")->get();
$total = $orders[0]->items->sum("amount");
```
After:
```php
$orders = Order::withSum("items", "amount")->withCount("items")->get();
$total = $orders[0]->items_sum_amount;
```

### Request-scoped memoization (Recommended)

- **Smell:** Public model collections lose custom query projections during Livewire hydration.
- **Technique:** Move derived query results into Computed methods and reference them through $this in Blade. Livewire 4 evaluates them on demand and memoizes within the request, rather than hydrating custom projections as public state.
- **When to use:** Queried display data should be derived afresh from component state.
- **When not to:** Editable state still needs explicit form properties; expensive recomputation may require a deliberate cache.
- **Packages:** `inertiajs/inertia-laravel`, `livewire/livewire`, `spatie/once`

Before:
```php
public $quotes;
public function mount() {
    $this->quotes = Quote::with("contact")->get();
}
```
After:
```php
#[Computed]
public function quotes() {
    return Quote::with("contact")->get();
}
// Blade: @foreach ($this->quotes as $quote) ... @endforeach
```

### Read-model projections (Recommended)

- **Smell:** Exception detection repeatedly polls every tenant’s raw history, including idle tenants.
- **Technique:** Maintain a small per-environment/class occurrence projection with a materialized view and poll nonzero pending sums. Acknowledge the observed amount by inserting its negative so concurrently arriving counts remain pending; failure recovery and consumer coordination need separate design.
- **When to use:** High-volume exception detection should be driven by actual arrivals.
- **When not to:** Crashes or multiple consumers can lose or duplicate acknowledgments without an explicit protocol.
- **Packages:** `hirethunk/verbs`, `livewire/livewire`, `spatie/laravel-dashboard`, `spatie/laravel-event-sourcing`

Before:
```php
foreach ($environments as $env) {
    scanLastMinuteExceptions($env);
}
```
After:
```php
foreach ($pending->nonzeroCounts() as $row) {
    $alerts->process($row);
    $pending->insertDelta($row->environment, $row->class, -$row->count);
}
```

### Eager loading (Recommended)

- **Smell:** Views and model helpers execute hidden relationship queries that callers cannot optimize.
- **Technique:** Prepare endpoint-specific eager loads and constraints before passing models to views or resources. Encapsulate reusable query fragments in composable Eloquent scopes rather than helpers that execute queries internally.
- **When to use:** The request boundary knows which related data the response needs.
- **When not to:** A shared query object can be preferable when several endpoints need identical orchestration.
- **Packages:** `filament/filament`

Before:
```php
$posts = Post::paginate();
// Blade loop:
{{ $post->author->name }}
```
After:
```php
$posts = Post::with("author")->paginate();
return view("posts.index", compact("posts"));
```

### Global query scopes (Recommended)

- **Smell:** Shared analytics queries repeatedly implement tenant predicates and can omit them.
- **Technique:** Centralize tenant filtering in a shared query-building layer and lead analytical sorting with the tenant/environment key where the workload benefits. Laravel where() supplies the predicate, while enforced query construction is application/adapter code.
- **When to use:** Shared event tables serve many tenants.
- **When not to:** Raw queries can bypass the layer and tenant-first sorting does not ensure resource fairness.
- **Packages:** `glushkovds/phpclickhouse-laravel`, `spatie/laravel-multitenancy`

Before:
```php
$query = DB::connection("clickhouse")->table("events");
// Each caller is responsible for remembering environment_id.
```
After:
```php
public function events(int $environmentId) {
    return DB::connection("clickhouse")->table("events")
        ->where("environment_id", $environmentId);
}
```

### HTTP response caching (Recommended)

- **Smell:** Repeated API reads rebuild and download unchanged representations.
- **Technique:** Cache complete HTTP representations where appropriate and send Cache-Control and ETag headers for conditional requests. Laravel cache.headers supplies headers and ETags; response storage and representation-aware keys require separate logic.
- **When to use:** Stable read-heavy API representations.
- **When not to:** Personalized responses and invalidation complexity limit shared caching.
- **Packages:** `Varnish`, `spatie/laravel-responsecache`

Before:
```php
Route::get("/catalog", CatalogController::class);
```
After:
```php
Route::get("/catalog", CatalogController::class)
    ->middleware("cache.headers:public;max_age=60;etag");
```

### Server-side pagination (Recommended)

- **Smell:** Tables fetch every record and paginate only after transferring the entire dataset to the browser.
- **Technique:** Filter in the Eloquent or query-builder query and call paginate() before returning rows. Laravel applies database-level pagination instead of loading and transferring all matching records.
- **When to use:** A table displays a small page from a large dataset.
- **When not to:** Count queries and high offsets can be expensive; the notes' other pagination alternatives are documentation qualifications rather than taught techniques.

Before:
```php
return User::all(); // Browser slices to 25 rows.
```
After:
```php
return User::where("active", true)->paginate(25);
```

### Named query scopes (Recommended)

- **Smell:** The same model classification predicates are copied into different queries.
- **Technique:** Encapsulate reusable predicates in an Eloquent local scope and compose it with other builder operations. Keep the scope on the builder so filtering happens in the database.
- **When to use:** A domain category such as available or reserved is queried in multiple places.
- **When not to:** Do not create a scope for every one-off predicate or hide unrelated side effects inside it.

Before:
```php
Thing::whereNull("reserved_at")->get();
Thing::whereNull("reserved_at")->count();
```
After:
```php
// In Thing
public function scopeAvailable(Builder $query): void {
    $query->whereNull("reserved_at");
}
Thing::available()->get();
Thing::available()->count();
```

### Stale-while-revalidate caching (Recommended)

- **Smell:** Requests block on expensive regeneration whenever a cache entry expires.
- **Technique:** Use Laravel 13 Cache::flexible with a fresh interval and a longer stale interval. Requests in the stale interval receive the existing value while Laravel schedules a refresh after the response; fully expired entries recompute synchronously.
- **When to use:** Read-heavy derived data tolerates bounded staleness.
- **When not to:** Avoid for authorization, balances, or data requiring immediate consistency; refresh work can still fail or consume worker time.
- **Packages:** `Varnish`

Before:
```php
$stats = Cache::remember("stats", 60, fn () => buildStats());
```
After:
```php
$stats = Cache::flexible("stats", [60, 300], fn () => buildStats());
```

### Atomic cache locks (Recommended)

- **Smell:** Concurrent requests mutate a shared resource or all regenerate the same expensive cache entry.
- **Technique:** Use Cache::lock with a stable resource key, a bounded lease, and a shared lock-capable store. Perform the critical section inside the lock callback; for cache fills, recheck the cache after acquiring the lock.
- **When to use:** Concurrent work must be serialized across requests or workers.
- **When not to:** A cache lock does not replace database constraints; lease expiry must exceed the protected work and contention may block requests.
- **Packages:** `harris21/laravel-fuse`

Before:
```php
$value = Cache::get("report");
if ($value === null) {
    $value = buildReport();
    Cache::put("report", $value, 300);
}
```
After:
```php
$value = Cache::get("report") ??
    Cache::lock("report:fill", 30)->block(5, fn () =>
        Cache::remember("report", 300, fn () => buildReport())
    );
```

### Automatic eager loading (Recommended)

- **Smell:** Relationships accessed during iteration cause repeated lazy-load queries and relation requirements vary by consumer.
- **Technique:** Opt into Eloquent automatic relationship loading so accessing a relation can batch-load it for the collection. Prefer explicit with() where the required graph is known and assess the memory and query behavior of automatic loading.
- **When to use:** Collection consumers vary and accidental relation access creates N+1 queries.
- **When not to:** Automatic loading can fetch unexpected data and hide query intent; verify support and behavior in the deployed Laravel release.

Before:
```php
$users = User::all();
foreach ($users as $user) { echo $user->team->name; }
```
After:
```php
// Application service provider
Model::automaticallyEagerLoadRelationships();
$users = User::all();
foreach ($users as $user) { echo $user->team->name; }
```

### Query objects (Recommended)

- **Smell:** Queries repeat terminal operations such as sorting and pagination across several callers.
- **Technique:** Extract an invokable object and pass it to the query builder pipe method. Unlike tap, pipe returns the component result, so a component can execute the query and return a paginator or other value.
- **When to use:** Shared query behavior includes execution rather than just adding constraints.
- **When not to:** Simple query clauses are clearer inline; use tap when the builder must remain chainable.

Before:
```php
$users = User::query()->orderBy("name")->paginate(25);
$teams = Team::query()->orderBy("name")->paginate(25);
```
After:
```php
class PaginateByName {
    public function __invoke(Builder $query): LengthAwarePaginator {
        return $query->orderBy("name")->paginate(25);
    }
}
$users = User::query()->pipe(new PaginateByName);
```

### Inverse relationship hydration (Useful)

- **Smell:** Children query and rehydrate a parent that is already loaded in memory.
- **Technique:** Set the already loaded parent on each child with setRelation so inverse access reuses that object. Laravel 13 also offers chaperone for supported hasMany and morphMany relationships.
- **When to use:** Nested views walk from a known parent to children and back.
- **When not to:** Manual assignment must use the correct parent and only populates memory; it does not change foreign keys.

Before:
```php
$category = Category::with("products")->first();
foreach ($category->products as $product) {
    echo $product->category->name;
}
```
After:
```php
$category = Category::with("products")->first();
$category->products->each->setRelation("category", $category);
foreach ($category->products as $product) {
    echo $product->category->name;
}
```

### Chunked and streaming processing (Useful)

- **Smell:** An import loads and parses a large file entirely before processing, exhausting memory.
- **Technique:** Read and parse the input incrementally, yielding records through a generator or Laravel LazyCollection and processing bounded chunks. Streaming must start at input reading; making an already loaded array lazy does not remove its memory cost.
- **When to use:** Input files or datasets can exceed comfortable memory limits.
- **When not to:** The parser must support incremental reading and record boundaries; some operations still require global state.

Before:
```php
$lines = file($path);
foreach ($lines as $line) importRecord(str_getcsv($line));
```
After:
```php
LazyCollection::make(function () use ($path) {
    $handle = fopen($path, "r");
    try { while (($row = fgetcsv($handle)) !== false) yield $row; }
    finally { fclose($handle); }
})->each(fn ($row) => importRecord($row));
```

### Custom Eloquent builders (Useful)

- **Smell:** Model-specific query conditions are duplicated across callers and accumulate as model scopes.
- **Technique:** Move named query operations into an Eloquent Builder subclass and return it from the model newEloquentBuilder method. Callers gain a fluent model-specific query API while the model stays focused on its data behavior.
- **When to use:** A model has many composable domain queries.
- **When not to:** A few local scopes are simpler; avoid a custom builder that just renames standard methods.

Before:
```php
Invoice::where("status", "unpaid")->where("due_at", "<", now())->get();
```
After:
```php
class InvoiceBuilder extends Builder {
    public function overdue(): static {
        return $this->where("status", "unpaid")->where("due_at", "<", now());
    }
}
// Invoice model:
public function newEloquentBuilder($query): InvoiceBuilder { return new InvoiceBuilder($query); }
Invoice::query()->overdue()->get();
```

### Incremental analytical rollups (Useful)

- **Smell:** Analytics repeatedly rescans enormous event histories for the same time-bucket summaries.
- **Technique:** Serve repeated analytics from TimescaleDB continuous aggregates instead of grouping all raw events per request. Configure refresh and real-time freshness behavior deliberately; Laravel queries the resulting view through its PostgreSQL connection.
- **When to use:** Repeated time-window analytics over large event streams.
- **When not to:** Materialization costs storage and refresh work; unusual ad hoc queries or strict freshness may still require raw data.
- **Packages:** `TimescaleDB`, `laravel/pulse`

Before:
```php
SELECT date_trunc('hour', occurred_at), avg(value)
FROM events GROUP BY 1;
```
After:
```php
CREATE MATERIALIZED VIEW hourly_events
WITH (timescaledb.continuous) AS
SELECT time_bucket('1 hour', occurred_at) AS bucket, avg(value) AS value
FROM events GROUP BY 1 WITH NO DATA;
-- Configure refresh policy before relying on this view.
-- Laravel: DB::table("hourly_events")->whereBetween("bucket", $range)->get();
```

### Indexed generated columns (Useful)

- **Smell:** Queries repeatedly extract or transform values that cannot use the existing index.
- **Technique:** Create a generated database column for the expression and index that column in a migration. Query the generated value through Eloquent or the query builder instead of repeating an unindexed expression.
- **When to use:** Frequently queried JSON paths or derived values need an efficient index.
- **When not to:** Expression support differs by database and can increase write cost; measure whether the query warrants a schema change.

Before:
```php
DB::table("users")->whereRaw("JSON_UNQUOTE(JSON_EXTRACT(profile, '$.email')) = ?", [$email])->get();
```
After:
```php
Schema::table("users", function (Blueprint $table) {
    $table->string("profile_email")
        ->storedAs("JSON_UNQUOTE(JSON_EXTRACT(profile, '$.email'))");
    $table->index("profile_email");
});
User::where("profile_email", $email)->get();
```

### Scheduled cache warming (Useful)

- **Smell:** Predictable cache expiration makes foreground requests rebuild expensive data.
- **Technique:** Refresh suitable caches from a scheduled Laravel command before users need them. Optimize the underlying computation and choose intervals that match tolerated freshness and refresh cost.
- **When to use:** Read-heavy data has predictable expiry and refresh timing.
- **When not to:** Unneeded warming wastes work and coordinated refreshes can increase database load.

Before:
```php
Cache::remember("dashboard", 600, fn () => calculateStats());
```
After:
```php
Schedule::call(fn () =>
    Cache::put("dashboard", calculateStats(), 600)
)->everyFiveMinutes();
```

### Selective column and scalar loading (Useful)

- **Smell:** Scalar-only exports hydrate full Eloquent models and then extract one field.
- **Technique:** Use pluck to select the required scalar directly instead of constructing whole models. For outputs too large to materialize, use chunked or lazy retrieval as a further implementation choice.
- **When to use:** A list or export requires column values rather than model behavior.
- **When not to:** Pluck still materializes a collection and does not expose model methods or relationships.

Before:
```php
$emails = User::all()->map(fn ($user) => $user->email);
```
After:
```php
$emails = User::query()->pluck("email");
```

### Constrained eager loading (Useful)

- **Smell:** Loading every child for each parent wastes memory when only recent children are displayed.
- **Technique:** Constrain an eager-loaded relationship with ordering and a per-parent limit. Laravel 13 supports those limits without an external eager-limit package.
- **When to use:** A parent list needs only its latest few related records.
- **When not to:** Still incurs relationship query work; limiting results is wrong if downstream code expects the full relationship.

Before:
```php
User::with("posts")->paginate(5);
```
After:
```php
User::with(["posts" => fn ($q) => $q->latest()->limit(5)])
    ->paginate(5);
```

### Denormalized counters (Useful)

- **Smell:** Repeated full-table counts dominate a page while exact freshness is unnecessary.
- **Technique:** Maintain a compact summary count and read it instead of recounting the source table for every request. Update or reconcile it on a defined schedule.
- **When to use:** Large-table counts can tolerate bounded staleness.
- **When not to:** Counters drift without reconciliation and complicate writes; use a normal count for small tables.

Before:
```php
$total = Order::query()->count();
```
After:
```php
$total = DB::table("counters")->where("name", "orders")
    ->value("total");
```

### Primary reads for consistency (Useful)

- **Smell:** A redirect or job expects a recent write but reads a stale replica.
- **Technique:** Route dependent reads explicitly to the primary across execution boundaries. Laravel 13 sticky only covers the current request; use a write-connection query or a deliberately selected connection where freshness is required.
- **When to use:** Subsequent requests, jobs, or events must immediately observe a write.
- **When not to:** Primary reads consume writer capacity; tolerate replica lag where business requirements allow it.

Before:
```php
// in a later request, sticky state is gone
$order = Order::findOrFail($id);
```
After:
```php
$order = Order::query()->useWritePdo()->findOrFail($id);
```

### Read/write splitting with consistency controls (Useful)

- **Smell:** Read-heavy traffic overloads a single database primary.
- **Technique:** Configure Laravel 13 read hosts as replicas and the write host as the primary. Eloquent and query-builder calls use these connection roles while consistency-sensitive reads can explicitly use the primary.
- **When to use:** Read throughput is the bottleneck after query optimization.
- **When not to:** Replication lag complicates read-after-write and replicas do not scale primary writes.

Before:
```php
"host" => "primary",
// all reads and writes share one database host
```
After:
```php
"read" => ["host" => ["replica-a", "replica-b"]],
"write" => ["host" => ["primary"]],
"sticky" => true,
```

### Scalar subquery projections (Useful)

- **Smell:** A list loads every child row or adds bulky joins just to get the latest related value.
- **Technique:** Use addSelect with a correlated, ordered subquery limited to one row to fetch the needed scalar alongside each parent. Order or filter using the derived value when supported and index the relation key plus ordering column.
- **When to use:** A parent list needs a latest date or other single related scalar.
- **When not to:** Subqueries may be slower than joins for some plans; inspect query plans and use eager loading when full child models are needed.

Before:
```php
$users = User::with("payments")->get();
foreach ($users as $user) {
    $last = $user->payments->max("created_at");
}
```
After:
```php
$users = User::addSelect(["last_paid_at" => Payment::select("created_at")
    ->whereColumn("user_id", "users.id")
    ->orderByDesc("created_at")->limit(1)
])->orderByDesc("last_paid_at")->get();
```

### Search engine abstraction (Scout) (Useful)

- **Smell:** Large catalog search, filtering, sorting, or aggregation outgrows relational discovery queries.
- **Technique:** Move product discovery into a dedicated search index while retaining authoritative transactional data in the database. Laravel 13 can use a suitable search integration; Elasticsearch requires its own integration or custom engine rather than a bundled Scout Elasticsearch driver.
- **When to use:** Measured catalog discovery needs exceed the database query approach.
- **When not to:** An extra index adds synchronization lag and operational cost; SQL search may suffice for smaller catalogs.
- **Packages:** `laravel/scout`, `typesense/typesense-php`

Before:
```php
$products = Product::where("name", "like", "%{$query}%")->get();
```
After:
```php
// application search adapter backed by the dedicated index
$ids = $catalogSearch->search($query, $filters);
$products = Product::whereIn("id", $ids)->get();
```

### Bundled SQLite read models (Situational)

- **Smell:** Static content still requires a production database service and network queries.
- **Technique:** Build a SQLite content database locally or in CI and ship it with the application as a read-only artifact. Laravel can query it through Eloquent.
- **When to use:** Content changes through deployments and can be rebuilt from source.
- **When not to:** Unsuitable for production writes or data that must update without a deployment.

Before:
```php
$articles = DB::connection("mysql")->table("articles")->get();
```
After:
```php
// config/database.php content connection:
"content" => [
    "driver" => "sqlite",
    "database" => base_path("content.sqlite"),
],
$articles = DB::connection("content")->table("articles")->get();
```

### Cache failover (Situational)

- **Smell:** A cache backend outage interrupts requests that could proceed with a fallback cache.
- **Technique:** Configure a failover cache store with an ordered list of backing stores. Laravel 13 tries the next store when an operation on the primary store fails.
- **When to use:** Ordinary cached data can safely be retrieved or rebuilt through another store.
- **When not to:** Different stores can have divergent data; do not assume fallback stores preserve distributed-lock or coordination semantics.

Before:
```php
$value = Cache::store("redis")->get($key);
```
After:
```php
// config/cache.php stores:
"failover" => [
    "driver" => "failover",
    "stores" => ["redis", "database"],
],
$value = Cache::store("failover")->get($key);
```

### Compact domain-specific schemas (Situational)

- **Smell:** Migration defaults allocate generic columns even though the domain is bounded and well understood.
- **Technique:** Refactor schema definitions to types and lengths that safely represent the actual domain. Laravel’s migration builder exposes string lengths and integer sizes; measure the target engine’s storage and indexing effects.
- **When to use:** Large tables amplify unnecessary row and index width.
- **When not to:** Future legitimate values may exceed an assumed bound; narrowing existing data requires a safe migration.

Before:
```php
$table->string("country_code");
$table->integer("rating");
```
After:
```php
$table->string("country_code", 2);
$table->unsignedTinyInteger("rating");
```

### Cursor pagination (Situational)

- **Smell:** Increasing offsets make sequential browsing slower.
- **Technique:** Use cursorPaginate() with deterministic ordering and a unique tie-breaker so each page seeks beyond the last row.
- **When to use:** Users browse a large result set forward or backward.
- **When not to:** Cursor pagination does not support arbitrary numbered page jumps.

Before:
```php
return Post::query()->orderBy("id")->paginate(20);
```
After:
```php
return Post::query()->orderBy("id")->cursorPaginate(20);
```

### Database TTL retention (Situational)

- **Smell:** Scheduled deletion jobs scan large event tables to enforce retention.
- **Technique:** Declare timestamp-based ClickHouse TTL and optionally partition by date with ttl_only_drop_parts enabled. Whole expired parts can be removed efficiently, with asynchronous expiration rather than exact per-row deadlines.
- **When to use:** Append-oriented analytical events have bounded retention.
- **When not to:** Whole-part expiry can retain some expired rows longer and is inappropriate for exact deletion deadlines.

Before:
```php
DELETE FROM events WHERE timestamp < now() - INTERVAL 90 DAY;
```
After:
```php
CREATE TABLE events (...) ENGINE = MergeTree
PARTITION BY toDate(timestamp)
ORDER BY (environment_id, timestamp)
TTL timestamp + INTERVAL 90 DAY
SETTINGS ttl_only_drop_parts = 1;
```

### Dedicated analytical read stores (Situational)

- **Smell:** Large reporting aggregations compete with transactional workloads.
- **Technique:** Use a column-oriented analytical store such as ClickHouse for the reporting workload while retaining relational persistence where required. Integrate it through an external Laravel-compatible client/adapter.
- **When to use:** Large scans and aggregates dominate reporting costs.
- **When not to:** Additional storage and ingestion increase operational work; ClickHouse is not a native Laravel driver.
- **Packages:** `glushkovds/phpclickhouse-laravel`

Before:
```php
$totals = DB::table("events")->selectRaw("kind, count(*) AS n")
    ->groupBy("kind")->get();
```
After:
```php
// A separate analytical store receives event batches.
$totals = $analytics->query(
    "SELECT kind, count() AS n FROM events GROUP BY kind"
);
```

### Deferred join pagination (Situational)

- **Smell:** Deep offset pagination scans or retrieves wide rows before discarding most of them.
- **Technique:** Page an indexed, narrow subquery containing only IDs, then join those IDs back to the full table. In Laravel use a query-builder joinSub or equivalent SQL and preserve the same deterministic ordering in both stages.
- **When to use:** Numbered pages with large offsets must be retained on a large, wide table.
- **When not to:** Cursor pagination is simpler when numbered pages are unnecessary; verify indexes and the database execution plan.

Before:
```php
$rows = DB::table("posts")->orderBy("id")
    ->offset(100000)->limit(20)->get();
```
After:
```php
$page = DB::table("posts")->select("id")
    ->orderBy("id")->offset(100000)->limit(20);
$rows = DB::table("posts as p")
    ->joinSub($page, "page", "p.id", "=", "page.id")
    ->orderBy("p.id")->select("p.*")->get();
```

### Full-text search (Situational)

- **Smell:** LIKE queries become slow or too limited for application search.
- **Technique:** Replace substring scans with an appropriate full-text index or a dedicated search engine through Laravel Scout. Scout with Typesense synchronizes searchable models and provides a search API while separating search indexing from ordinary relational queries.
- **When to use:** Search needs ranking, typo tolerance, or better performance than unindexed wildcard queries.
- **When not to:** An external engine adds deployment and index synchronization costs; database full-text search may suffice.
- **Packages:** `laravel/scout`, `typesense/typesense-php`

Before:
```php
Post::where("title", "like", "%{$term}%")->get();
```
After:
```php
class Post extends Model { use Searchable; }
// Configure the Typesense Scout driver and import the index.
$posts = Post::search($term)->get();
```

### Grouped query filters (Situational)

- **Smell:** An OR search condition escapes mandatory status, ownership, or tenant predicates.
- **Technique:** Encapsulate multi-column OR predicates inside a nested where closure, including custom allowed filters in Spatie Query Builder. The resulting SQL groups the alternatives under the mandatory constraints.
- **When to use:** A reusable URL search filter spans several columns within an already-constrained query.
- **When not to:** Grouping fixes precedence but does not make arbitrary user-selected columns safe; simple one-column filters need no group.
- **Packages:** `spatie/laravel-query-builder`

Before:
```php
$query->where("tenant_id", $tenantId)
    ->where("name", "like", $term)
    ->orWhere("email", "like", $term);
```
After:
```php
$query->where("tenant_id", $tenantId)->where(function ($q) use ($term) {
    $q->where("name", "like", $term)
      ->orWhere("email", "like", $term);
});
```

### Mutable analytical dimensions (Situational)

- **Smell:** Updating user metadata embedded in historical events rewrites large analytical datasets.
- **Technique:** Store identifiers on immutable events and mutable details in a separate ReplacingMergeTree dimension table. Insert replacement versions and read with FINAL where current deduplicated values are required.
- **When to use:** Analytics needs current user details alongside immutable history.
- **When not to:** Joins and version resolution cost work and replacement identity must remain stable.

Before:
```php
-- Repeatedly rewrite user names on every historical event.
```
After:
```php
CREATE TABLE users (id UInt64, version UInt64, name String)
ENGINE = ReplacingMergeTree(version) ORDER BY id;
-- Events store user_id; mutable details live in users.
SELECT * FROM users FINAL WHERE id = 7;
```

### One-of-many relationships (Situational)

- **Smell:** A loop queries the latest invoice separately for every customer.
- **Technique:** Define a hasOne relationship using latestOfMany and eager-load that relationship. Laravel 13 selects one related row per parent without a separate lookup for each customer.
- **When to use:** A list needs each parent latest related record.
- **When not to:** latestOfMany defaults to ordering by the primary key; use ofMany with an appropriate column when latest means a different order.

Before:
```php
foreach (Customer::all() as $customer) {
    $invoice = $customer->invoices()->latest("id")->first();
}
```
After:
```php
public function latestInvoice(): HasOne {
    return $this->hasOne(Invoice::class)->latestOfMany();
}
foreach (Customer::with("latestInvoice")->get() as $customer) {
    $invoice = $customer->latestInvoice;
}
```

### Recursive CTE gap detection (Situational)

- **Smell:** PHP loads an entire identifier range or loops through many queries to locate missing records.
- **Technique:** Generate the expected sequence with a recursive SQL CTE and left join actual records to find gaps. Execute the bounded query through Laravel DB bindings, respecting the database recursion limit.
- **When to use:** An ordered numeric sequence needs gap detection without materializing it in PHP.
- **When not to:** Not suitable for UUIDs or intentionally nonconsecutive IDs; large ranges may exceed recursion limits.

Before:
```php
foreach (range($start, $end) as $id) {
    if (!Order::whereKey($id)->exists()) { $missing[] = $id; }
}
```
After:
```php
$missing = DB::select("WITH RECURSIVE ids AS (
 SELECT ? AS id UNION ALL
 SELECT id + 1 FROM ids WHERE id < ?
) SELECT ids.id FROM ids
LEFT JOIN orders ON orders.id = ids.id
WHERE orders.id IS NULL", [$start, $end]);
```

### Tagged cache invalidation (Situational)

- **Smell:** Team-related cached values use scattered keys that are hard to invalidate together.
- **Technique:** Group entries under a team-specific cache tag and flush that tag when the team settings change. Laravel 13 supports tags with suitable stores such as Redis.
- **When to use:** A related set of cache entries shares one invalidation boundary.
- **When not to:** Database, file, and DynamoDB stores do not support tags; broad invalidation can trigger expensive rebuilds.

Before:
```php
Cache::forget("team:{$id}:settings");
Cache::forget("team:{$id}:dashboard");
// Other keys can be missed.
```
After:
```php
Cache::store("redis")->tags(["team:{$id}"])->put("dashboard", $data, 300);
// On a relevant settings change:
Cache::store("redis")->tags(["team:{$id}"])->flush();
```

### Versioned analytical records (Situational)

- **Smell:** Frequent in-place updates are a poor fit for append-oriented analytical storage.
- **Technique:** Insert new versions of logical rows into ReplacingMergeTree with stable ORDER BY identity. Use FINAL when queries need current deduplicated values rather than relying on eventual background merges.
- **When to use:** Mutable dimensions must live in a ClickHouse analytical store.
- **When not to:** Version resolution adds query cost and does not provide relational transaction semantics.

Before:
```php
ALTER TABLE users UPDATE name = ? WHERE id = ?;
```
After:
```php
CREATE TABLE users (id UInt64, version UInt64, name String)
ENGINE = ReplacingMergeTree(version) ORDER BY id;
INSERT INTO users VALUES (7, 2, 'New name');
SELECT * FROM users FINAL WHERE id = 7;
```

### Blind indexes (Situational)

- **Smell:** Encrypted sensitive fields need selected equality lookups without decrypting a whole table.
- **Technique:** Use a library-managed keyed blind index for necessary lookups while retaining encrypted values. Compound indexes can reduce leakage for low-diversity fields; Laravel 13 core encrypted casts do not provide searchable encryption.
- **When to use:** The application must support specific equality lookups over encrypted attributes.
- **When not to:** Indexes leak information and may introduce false positives; avoid unnecessary indexes and assess low-entropy inputs.

Before:
```php
$matches = Patient::all()->filter(
    fn ($patient) => $patient->surname === $surname
);
```
After:
```php
// application search service backed by a keyed compound index
$matches = $encryptedSearch->bySurnameAndBirthDate($surname, $dob);
```

### Conditional SQL aggregates (Situational)

- **Smell:** Status dashboards load every model or issue one count query per status.
- **Technique:** Calculate multiple status totals in a single SQL aggregate query using conditional expressions through selectRaw. Bind variable values rather than interpolating them.
- **When to use:** A dashboard needs totals rather than individual records.
- **When not to:** Database-specific expressions reduce portability and complicated conditions need careful maintenance.

Before:
```php
$open = Ticket::where("status", "open")->count();
$closed = Ticket::where("status", "closed")->count();
```
After:
```php
$totals = Ticket::selectRaw(
    "COUNT(CASE WHEN status = ? THEN 1 END) AS open, ".
    "COUNT(CASE WHEN status = ? THEN 1 END) AS closed",
    ["open", "closed"]
)->first();
```

### Custom database transport adapters (Situational)

- **Smell:** Supporting a remote SQLite-compatible database would otherwise require replacing application Eloquent queries with HTTP calls.
- **Technique:** Implement a Laravel connection and a PDO-style transport adapter at the database boundary, reusing SQLite grammar and result processing where appropriate. Register the driver so Eloquent callers use a configured connection while SQL and bindings stay separate.
- **When to use:** Building a database integration whose protocol differs from native PDO transports.
- **When not to:** Transaction, schema, and binding semantics require careful verification; the notes do not establish Litebase Laravel 13 package compatibility.

Before:
```php
$result = Http::post($endpoint, ["sql" => "select * from users"]);
```
After:
```php
// Custom driver registered by the application integration.
DB::extend("remote-sqlite", fn ($config, $name) =>
    new RemoteSqliteConnection($config, $name)
);
$users = DB::connection("remote-sqlite")->table("users")->get();
```

### Database business-key uniqueness constraints (Situational)

- **Smell:** Application-only duplicate checks allow concurrent inserts of the same business-key combination.
- **Technique:** Encode each independent uniqueness invariant as its own multi-column unique constraint in a Laravel migration. Application validation can improve feedback but the database enforces the invariant.
- **When to use:** Uniqueness belongs to a pair or tuple of required business columns.
- **When not to:** A single tuple constraint does not express independent pairwise invariants; nullable-column semantics vary by engine.

Before:
```php
if (!Participation::where($attributes)->exists()) {
    Participation::create($attributes);
}
```
After:
```php
$table->unique(["user_id", "game_id"]);
$table->unique(["game_id", "country_id"]);
```

### Database sharding and partitioning (Situational)

- **Smell:** An ordinary join or relationship query assumes records on different shards share one database.
- **Technique:** Fetch the necessary data from each shard and assemble it deliberately in the application, or use middleware designed for cross-shard queries. Laravel 13 ordinary relationships do not transparently implement cross-database relationship-existence queries.
- **When to use:** A known query must span a partition boundary.
- **When not to:** Network fan-out, pagination, ordering, and consistency become expensive; frequent cross-shard work may indicate a poor partition key.

Before:
```php
// users and orders are on different shard databases
$rows = DB::table("users")->join("orders", "users.id", "=", "orders.user_id")->get();
```
After:
```php
$users = DB::connection("users_shard")->table("users")->get();
$orders = DB::connection("orders_shard")->table("orders")
    ->whereIn("user_id", $users->pluck("id"))->get()->groupBy("user_id");
$rows = $users->map(fn ($user) => [
    "user" => $user, "orders" => $orders->get($user->id, collect()),
]);
```

### Database transactions (Situational)

- **Smell:** An operation updates multiple models and partial failure leaves inconsistent records.
- **Technique:** Wrap the related database writes in DB::transaction so Laravel 13 commits them together or rolls them back. Keep the transaction short and schedule external side effects after it commits.
- **When to use:** Several writes form one business invariant, especially in payments.
- **When not to:** Transactions cannot roll back remote API calls and do not by themselves prevent concurrent lost updates.

Before:
```php
$account->decrement("balance", $amount);
$invoice->update(["paid" => true]);
```
After:
```php
DB::transaction(function () use ($account, $invoice, $amount) {
    $account->decrement("balance", $amount);
    $invoice->update(["paid" => true]);
});
```

### Dependency-aware cache invalidation (Situational)

- **Smell:** Changed models leave response caches stale or cause broad invalidation.
- **Technique:** Cache responses and reusable components with dependency tags, propagating dependencies into parent cache entries. Laravel tag-capable stores supply invalidation primitives; automatic dependency collection and bounded lookup fan-out require application logic.
- **When to use:** Many cached representations depend on shared entities.
- **When not to:** Missing dependencies cause stale results and large tag sets can add network cost.

Before:
```php
Cache::remember("catalog", 600, fn () => buildCatalog());
```
After:
```php
Cache::tags(["product:$id"])->remember(
    "product-view:$id", 600, fn () => buildProductView($id)
);
Cache::tags(["product:$id"])->flush();
```

### Durable shard directories (Situational)

- **Smell:** Entity-to-shard mappings exist only in a disposable cache and vanish on eviction.
- **Technique:** Store shard assignments in durable directory data and resolve the Laravel 13 connection before accessing the entity. A cache may accelerate lookup but must not be the only authority for the mapping.
- **When to use:** Tenant or entity sharding needs stable routing and future reassignment.
- **When not to:** Directory availability becomes critical and reassignment requires coordinated routing changes.

Before:
```php
$connection = Cache::get("shop-shard:$shopId");
// eviction loses the only shard assignment
```
After:
```php
$connection = Cache::remember("shop-shard:$shopId", 300,
    fn () => ShardAssignment::where("shop_id", $shopId)->value("connection")
);
$orders = DB::connection($connection)->table("orders")->get();
```

### Early conditional-response middleware (Situational)

- **Smell:** Unchanged pages are fully rendered before deciding to return HTTP 304.
- **Technique:** Keep a representation validator that tracks content and variants, then check conditional request headers in middleware before expensive rendering. Use the response’s conditional-request handling to construct a correct 304 response; cache.headers alone generates its ETag after rendering.
- **When to use:** Expensive public representations have cheaply retrievable, correctly invalidated version metadata.
- **When not to:** Stale or incompletely scoped validators can validate the wrong representation; cheap pages do not need this extra state.

Before:
```php
$response = $next($request);
$response->setEtag($contentHash);
$response->isNotModified($request);
```
After:
```php
// Middleware: validatorFor includes host, path and variants.
$response = response()->setEtag(validatorFor($request));
if ($response->isNotModified($request)) {
    return $response;
}
return $next($request);
```

### Independently cacheable page fragments (Situational)

- **Smell:** A small personalized or frequently changing region prevents caching an otherwise public page.
- **Technique:** Split the page into a cacheable public shell and independent fragment endpoints with their own privacy and freshness. Compose with AJAX or proxy-configured ESI and prevent downstream shared caching of personalized assembled output.
- **When to use:** Page regions have materially different freshness or personalization needs.
- **When not to:** Multiple requests and ESI integration add complexity; highly personalized pages may have little reusable shell.
- **Packages:** `Varnish`

Before:
```php
<header>{{ auth()->user()->name }}</header>
@include("catalog")
```
After:
```php
{{-- Public cacheable shell --}}
<header id="account"></header>
@include("catalog")
<script>
fetch("/account-fragment", {credentials: "same-origin"})
  .then(r => r.text()).then(html => account.innerHTML = html);
</script>
{{-- Account endpoint emits Cache-Control: private, no-store. --}}
```

### Pessimistic row locking (Situational)

- **Smell:** Concurrent multi-model updates race or leave balances and related records inconsistent.
- **Technique:** Wrap the operation in DB::transaction and select the rows being changed with lockForUpdate before checking and updating their state. Acquire locks in a consistent order and dispatch dependent effects after commit.
- **When to use:** Concurrent payment or inventory operations update shared rows and enforce cross-row invariants.
- **When not to:** Long transactions and inconsistent lock order cause contention or deadlocks; do not perform remote calls while holding locks.

Before:
```php
$account = Account::findOrFail($id);
$account->decrement("balance", $amount);
Payment::create($data);
SendReceipt::dispatch($account);
```
After:
```php
DB::transaction(function () use ($id, $amount, $data) {
    $account = Account::whereKey($id)->lockForUpdate()->firstOrFail();
    if ($account->balance < $amount) { throw new InsufficientBalance(); }
    $account->decrement("balance", $amount);
    $payment = Payment::create($data);
    SendReceipt::dispatch($payment)->afterCommit();
});
```

### Request-scoped cache memoization (Situational)

- **Smell:** Multiple operations in one request repeatedly fetch the same cache key from the backend.
- **Technique:** Use Laravel Cache::memo to decorate cache access with execution-local memoization. Repeated reads reuse the value without another backend call while writes through that repository invalidate its memoized entry.
- **When to use:** Remote cache round trips repeat for the same keys during a request or job.
- **When not to:** Memoization may not observe writes made by other processes during the execution; avoid when live freshness is required.

Before:
```php
$a = Cache::get("settings");
$b = Cache::get("settings");
```
After:
```php
$cache = Cache::memo();
$a = $cache->get("settings");
$b = $cache->get("settings");
```

### SQL business ranking and ordering (Situational)

- **Smell:** Business-defined status or activity order is calculated after loading the entire collection.
- **Technique:** Express status precedence with a SQL CASE expression or select aggregate activity scores, then order before pagination. Keep ranking expressions application-owned and bind dynamic values.
- **When to use:** Product ranking differs from alphabetical or chronological order.
- **When not to:** Large rankings can be expensive; persisted scores may fit heavier workloads.

Before:
```php
$tickets = Ticket::all()->sortBy(
    fn ($ticket) => $ranks[$ticket->status]
);
```
After:
```php
$tickets = Ticket::orderByRaw(
    "CASE status WHEN ? THEN 0 WHEN ? THEN 1 ELSE 2 END",
    ["urgent", "open"]
)->paginate();
```

### SQL ordering by related data (Situational)

- **Smell:** A paginated parent list is sorted in PHP by a relationship, producing wrong global order or excessive loading.
- **Technique:** Order the parent query using a join or correlated subquery before pagination. Keep one output row per parent and index the relationship lookup.
- **When to use:** Related values determine the ordering of a paginated list.
- **When not to:** Joins can duplicate parents and correlated queries need measurement on the actual database.

Before:
```php
$users = User::with("logins")->get()
    ->sortByDesc(fn ($user) => $user->logins->max("created_at"));
```
After:
```php
$users = User::orderByDesc(
    Login::select("created_at")->whereColumn("user_id", "users.id")
        ->latest("created_at")->limit(1)
)->paginate();
```

### Subquery-selected relationships (Situational)

- **Smell:** A list eagerly loads every historical child just to display the latest child model.
- **Technique:** Select the required child ID with an ordered correlated subquery, then eager load a belongsTo relation through that computed attribute. Ordinary latest or oldest child cases can instead use Laravel 13 one-of-many relationships.
- **When to use:** The desired related model has specialized selection logic or historical collections are large.
- **When not to:** The computed-key relationship requires its selecting query; one-of-many is simpler for common cases.

Before:
```php
$users = User::with("logins")->get();
$last = $users[0]->logins->sortByDesc("created_at")->first();
```
After:
```php
$users = User::addSelect(["last_login_id" =>
    Login::select("id")->whereColumn("user_id", "users.id")
        ->latest("created_at")->limit(1)
])->with("lastLogin")->get();
// User::lastLogin(): belongsTo(Login::class, "last_login_id")
```

### Time-series hypertables (Situational)

- **Smell:** A very large timestamped-event table becomes expensive to organize and query as one ordinary table.
- **Technique:** Evaluate TimescaleDB hypertables on PostgreSQL to partition timestamped events into time-based chunks. Laravel 13 can issue extension-specific SQL through its PostgreSQL connection; verify TimescaleDB setup and do not assume third-party schema helpers are compatible.
- **When to use:** High-volume time-series data has clear timestamp-based access and retention patterns.
- **When not to:** Database portability and operational complexity increase; ordinary indexed tables can serve smaller or non-time-series datasets.
- **Packages:** `TimescaleDB`

Before:
```php
CREATE TABLE events (occurred_at timestamptz NOT NULL, value double precision);
```
After:
```php
-- PostgreSQL with the TimescaleDB extension; existing table has a time column.
SELECT create_hypertable('events', 'occurred_at');
-- Execute version-appropriate SQL from a Laravel migration.
```

## Validation & requests

### Form Requests (boundary validation) (Recommended)

- **Smell:** Generated controller input handling repeats validation and lacks a dedicated request boundary.
- **Technique:** Declare Blueprint's controller validate statement to generate a Form Request from the model definition. The request validates before the controller runs; review inferred rules and authorization before using its validated() data.
- **When to use:** Blueprint-generated CRUD needs dedicated validation classes.
- **When not to:** Column types cannot determine permissions or all business constraints; package documentation supplies the concrete mechanism in these notes.
- **Packages:** `api-platform/laravel`, `laravel-shift/blueprint`, `livewire/livewire`

Before:
```php
$data = $request->validate(["title" => "required|string"]);
```
After:
```php
// draft.yaml controller statement:
// validate: title, content
public function store(StorePostRequest $request) {
    Post::create($request->validated());
}
```

### Scoped route model binding (Recommended)

- **Smell:** Nested routes independently resolve the parent and child, allowing unrelated child models to reach the controller.
- **Technique:** Enable scopeBindings on nested routes so Laravel 13 resolves the child through the parent relationship. The controller receives a child belonging to that parent, while authorization remains a separate concern.
- **When to use:** A nested resource must belong to the parent named in the URL.
- **When not to:** Relationship naming must match binding conventions or be customized; scoping is not user authorization.

Before:
```php
$project = Project::findOrFail($projectId);
$task = Task::findOrFail($taskId);
abort_unless($task->project_id === $project->id, 404);
```
After:
```php
Route::get("projects/{project}/tasks/{task}", ShowTaskController::class)
    ->scopeBindings();
```

### Route model binding (Recommended)

- **Smell:** Page templates manually resolve URL identifiers and implement missing-record handling.
- **Technique:** Name Folio pages with model placeholders such as [Post:slug].blade.php so the package supplies the bound model. Laravel model binding handles missing records; authorization remains separate.
- **When to use:** A Folio page corresponds directly to an Eloquent model.
- **When not to:** Binding does not grant access and package filename conventions require onboarding; use the Windows-compatible separator when needed.
- **Packages:** `laravel/folio`

Before:
```php
$post = Post::where("slug", $slug)->firstOrFail();
```
After:
```php
// resources/views/pages/posts/[Post:slug].blade.php
<h1>{{ $post->title }}</h1>
```

### Precognitive validation (Useful)

- **Smell:** Frontend live validation duplicates Laravel rules and drifts from submission validation.
- **Technique:** Apply HandlePrecognitiveRequests middleware to the existing endpoint and use a Precognition-capable client to request validation without executing the controller operation. Keep the authoritative rules in the endpoint Form Request and return normal validation feedback.
- **When to use:** Forms need live validation using the same rules as final submission.
- **When not to:** Live checks generate extra requests; authorization and validation must remain side-effect-free and expensive rules may need selective handling.
- **Packages:** `laravel-precognition-vue`, `laravel/precognition`

Before:
```php
// JS duplicates rules
if (!email.includes("@")) { errors.email = "Invalid"; }
```
After:
```php
Route::post("/users", [UserController::class, "store"])
    ->middleware(HandlePrecognitiveRequests::class);
// Client
form.validate("email");
```

### Boundary rate limiting (Useful)

- **Smell:** Repeated login or invalid automation requests reach expensive processing.
- **Technique:** Define appropriately partitioned named rate limiters and apply throttle middleware ahead of expensive handling. Laravel produces 429 responses when the budget is exceeded.
- **When to use:** Authentication or token-processing endpoints receive abusive repeated traffic.
- **When not to:** Poor partition keys penalize legitimate users or let attackers evade limits.

Before:
```php
Route::post("/login", LoginController::class);
```
After:
```php
RateLimiter::for("login", fn (Request $r) =>
    Limit::perMinute(10)->by($r->ip()));
Route::post("/login", LoginController::class)->middleware("throttle:login");
```

### Composable after-validation checks (Situational)

- **Smell:** A validator after callback grows into several unrelated cross-field checks.
- **Technique:** Extract checks into invokable classes and pass an array of them to Validator::after or return them from a FormRequest after method. Laravel 13 runs each check with the validator, allowing focused reuse and tests.
- **When to use:** Cross-field or contextual validation needs several independent checks.
- **When not to:** Simple field constraints should remain ordinary validation rules; avoid repeated expensive queries across checks.

Before:
```php
$validator->after(function ($validator) {
    // Check schedule conflicts.
    // Check account eligibility.
});
```
After:
```php
$validator->after([
    new CheckScheduleConflicts,
    new CheckAccountEligibility,
]);
```

### Form Request input normalization (Situational)

- **Smell:** Recognizable user input is rejected because equivalent formatting is not normalized consistently.
- **Technique:** Normalize accepted representations in prepareForValidation before applying rules. The same Form Request hook covers both Precognition previews and final submission.
- **When to use:** Domain policy can safely map multiple input forms to one canonical form.
- **When not to:** Ambiguous or destructive transformations should require explicit choices; normalization is not trust validation.

Before:
```php
"website" => ["required", "url"],
// User enters example.com and is rejected.
```
After:
```php
protected function prepareForValidation(): void {
    $website = trim((string) $this->input("website"));
    if ($website !== "" && !str_contains($website, "://")) {
        $website = "https://".$website;
    }
    $this->merge(["website" => $website]);
}
```

### Reusable validation rules (Situational)

- **Smell:** A complex validation condition is copied as closures or intertwined with request handlers.
- **Technique:** Extract the condition into a class implementing Laravel 13 ValidationRule with validate(string $attribute, mixed $value, Closure $fail), then reuse it in Form Requests or validators. Modernize older invokable-rule examples to the current contract.
- **When to use:** Several fields or endpoints share a domain-specific input constraint.
- **When not to:** Use built-in rules for simple constraints; expensive rules still need careful database and network behavior.

Before:
```php
$request->validate(["code" => [function ($a, $v, $fail) {
    if (!validCode($v)) { $fail("Invalid code"); }
}]]);
```
After:
```php
class ValidCode implements ValidationRule {
    public function validate(string $attribute, mixed $value, Closure $fail): void {
        if (!validCode($value)) { $fail("Invalid code"); }
    }
}
$request->validate(["code" => [new ValidCode()]]);
```

### Enum route binding (Situational)

- **Smell:** Controllers manually validate and convert a route parameter against a fixed list of strings.
- **Technique:** Type-hint a string-backed enum in the route handler so Laravel implicit enum binding converts valid values and rejects invalid values with a 404. Reuse the enum in application code to keep the fixed value set centralized.
- **When to use:** A URL parameter represents a closed set such as a category or mode.
- **When not to:** A dynamic set should use database-backed binding; 404 semantics may not fit every API validation contract.

Before:
```php
if (!in_array($category, ["news", "sports"], true)) { abort(404); }
```
After:
```php
enum Category: string { case News = "news"; case Sports = "sports"; }
Route::get("/articles/{category}", function (Category $category) {
    return Article::where("category", $category->value)->get();
});
```

## Events, jobs & side effects

### Events and listeners (Recommended)

- **Smell:** UI methods and backend mutations use inconsistent operation vocabulary and duplicate workflow logic.
- **Technique:** Give the user operation a domain name and let a Livewire action invoke the corresponding accepted domain event. In a Verbs implementation, validation and authorization precede event recording, while event processing owns state and projection changes.
- **When to use:** A feature already uses event-sourced domain operations across entry points.
- **When not to:** Straightforward CRUD can stay in a Livewire action; native browser events are not durable domain events.
- **Packages:** `harris21/laravel-fuse`, `hirethunk/verbs`, `laravel/cashier`, `laravel/slack-notification-channel`, `livewire/livewire`, `spatie/laravel-url-ai-transformer`

Before:
```php
public function startTrial(): void {
  $this->customer->update(["trial_started_at" => now()]);
}
```
After:
```php
public function startTrial(): void {
  Gate::authorize("startTrial", $this->customer);
  CustomerBeganTrial::fire(customer_id: $this->customer->id);
}
```

### Queued side effects (Recommended)

- **Smell:** Deployment execution is tied to an HTTP process rather than a serialized worker task.
- **Technique:** Encapsulate execution in a queued job handle method and pass durable identifiers or serializable inputs. Laravel 13 workers reload model identifiers and inject execution dependencies, so queued models are not dispatch-time snapshots.
- **When to use:** Work must be handed to a worker and remain meaningful after the request ends.
- **When not to:** Queueing adds latency, retries, and operational dependencies; immediate response-critical work must still finish synchronously.
- **Packages:** `laravel/nova`

Before:
```php
$deployer->deploy($site);
return back();
```
After:
```php
DeploySite::dispatch($site->id);
// DeploySite implements ShouldQueue
public function handle(Deployer $deployer): void {
    $site = Site::findOrFail($this->siteId);
    $deployer->deploy($site);
}
```

### Response-deferred side effects (Recommended)

- **Smell:** The HTTP response is delayed by work whose result is not required by the user.
- **Technique:** Register noncritical callbacks with Laravel defer(), or use Concurrency::defer for independent work that benefits from process execution. Keep payloads small and use a durable queued job when the work must survive process failure.
- **When to use:** Best-effort metrics or cache warming can happen after the response.
- **When not to:** Deferred callbacks are not durable queue delivery and still consume server resources; essential effects need durable processing.

Before:
```php
recordMetric($event);
return response()->json($result);
```
After:
```php
defer(fn () => recordMetric($event));
return response()->json($result);
```

### Idempotency keys (Recommended)

- **Smell:** Client retries create duplicate orders, payments, or updates.
- **Technique:** Accept a stable Idempotency-Key, scope it to the caller and operation, and atomically store the request fingerprint and original result. Repeated matching requests replay that result rather than execute the operation again; use compatible middleware or an application implementation.
- **When to use:** Retryable mutation endpoints have expensive or irreversible side effects.
- **When not to:** Requires concurrency handling, expiry, and mismatch rejection; it does not replace provider-side payment idempotency.
- **Packages:** `wendelladriel/laravel-idempotency`

Before:
```php
Route::post("/orders", fn (Request $r) => Order::create($r->all()));
```
After:
```php
// Application middleware: reserve key and replay stored response
Route::post("/orders", [OrderController::class, "store"])
    ->middleware(IdempotentRequest::class);
// Client preserves the same Idempotency-Key across retries
```

### Model observers (Recommended)

- **Smell:** Related model lifecycle reactions are scattered across callers or model boot closures.
- **Technique:** Group model event reactions in an observer and register it with Model::observe() or ObservedBy. Use after-commit handling when side effects require committed data, and remember that bulk update/delete queries bypass per-model events.
- **When to use:** Behavior consistently follows a model lifecycle transition.
- **When not to:** Observers hide side effects; explicit use cases are clearer for workflow-specific reactions and bulk writes need another path.
- **Packages:** `laravel/cashier`

Before:
```php
User::created(fn ($user) => sendWelcome($user));
User::updated(fn ($user) => syncProfile($user));
```
After:
```php
class UserObserver {
    public function created(User $user): void { sendWelcome($user); }
    public function updated(User $user): void { syncProfile($user); }
}
User::observe(UserObserver::class);
```

### Queue workload partitioning and scaling (Recommended)

- **Smell:** Long-running background jobs compete with urgent mail or web-request capacity.
- **Technique:** Place jobs on queues separated by urgency and runtime, and allocate dedicated workers to those queues. Run background workers on separate compute when they compete with web requests.
- **When to use:** Mixed background workloads require different latency or resource guarantees.
- **When not to:** Multiple queues and worker pools add tuning costs; priority ordering alone can starve low-priority work.
- **Packages:** `laravel/horizon`

Before:
```php
GenerateReport::dispatch($reportId);
SendReceipt::dispatch($orderId);
```
After:
```php
GenerateReport::dispatch($reportId)->onQueue("reports");
SendReceipt::dispatch($orderId)->onQueue("mail");
# Separate supervised workers:
php artisan queue:work --queue=mail
php artisan queue:work --queue=reports
```

### Buffered bulk writes and delivery (Recommended)

- **Smell:** Bulk updates write an activity entry with a separate database round trip for each change.
- **Technique:** Use the activity-log package batching or buffering facility to collect audit writes and flush them together. Keep the audit contents and transaction boundaries explicit so batching does not silently drop required records.
- **When to use:** A bulk workflow produces many audit entries and log insertion dominates query count.
- **When not to:** Buffering trades immediate persistence for fewer writes; confirm the chosen package release supports Laravel 13 and the required flush behavior.
- **Packages:** `glushkovds/phpclickhouse-laravel`, `laravel/pulse`, `spatie/laravel-activitylog`

Before:
```php
foreach ($items as $item) {
    $item->update($changes);
    activity()->performedOn($item)->log("updated");
}
```
After:
```php
// Enable buffering in config/activitylog.php
foreach ($items as $item) {
    $item->update($changes);
    activity()->performedOn($item)->log("updated");
}
// Package flushes the activities together after the response
```

### WebSocket push updates (Useful)

- **Smell:** Frequent short polling produces repeated HTTP work and delays live updates.
- **Technique:** Broadcast server events over a persistent WebSocket connection and subscribe with Echo. Laravel 13’s Reverb integration supplies the socket server while Laravel broadcasting publishes application updates.
- **When to use:** Frequent updates justify persistent connections and immediate feedback.
- **When not to:** Infrequent updates may be simpler with polling; socket infrastructure and queue workers add operations.
- **Packages:** `laravel-echo`, `laravel/reverb`, `livewire/livewire`

Before:
```php
setInterval(() => fetch("/messages"), 1000);
```
After:
```php
// Laravel broadcast event implements ShouldBroadcast.
broadcast(new MessagePosted($message));
// Browser
Echo.private(`chat.${chatId}`).listen("MessagePosted", showMessage);
```

### Bounded transient retries (Useful)

- **Smell:** Transient external failures immediately discard an operation that could recover after a short delay.
- **Technique:** Use Laravel retry() or an HTTP-specific retry helper with limited attempts and delays around recoverable failures. Distinguish terminal errors and define exhaustion behavior; HTTP responses need explicit status handling when using generic retry().
- **When to use:** A transient dependency failure is safe to retry.
- **When not to:** Retries increase latency and load; non-idempotent operations can duplicate side effects and persistent failures need another policy.

Before:
```php
$result = $gateway->fetchReport();
```
After:
```php
$result = retry(3, fn () => $gateway->fetchReport(), 200,
    fn ($exception) => $exception instanceof TemporaryFailure
);
```

### Execution context propagation (Useful)

- **Smell:** Trace IDs and request metadata are threaded through unrelated method signatures or disappear at queue boundaries.
- **Technique:** Place execution metadata in Laravel Context at the request boundary and retrieve it where needed. Context integrates with logging and queue propagation, keeping cross-cutting metadata out of business method parameters.
- **When to use:** Logs and asynchronous work need consistent correlation or initiator metadata.
- **When not to:** Do not use ambient context for essential domain inputs or authorization decisions; hidden dependencies make isolated testing harder.

Before:
```php
createInvoice($data, $traceId);
sendReceipt($invoice, $traceId);
```
After:
```php
Context::add("trace_id", (string) Str::uuid());
createInvoice($data);
Log::info("Invoice created");
SendReceipt::dispatch($invoice);
```

### Idempotent handlers (Useful)

- **Smell:** Retried or duplicate jobs perform the same business operation more than once.
- **Technique:** Give each operation a stable business key and atomically record or check its completion under a database constraint or lock. Retry transient failures without repeating already completed effects, and pass the same key to external providers when supported.
- **When to use:** Queue delivery or network retries can repeat a charge, fulfillment, or other mutation.
- **When not to:** A read-then-write existence check alone races; database atomicity cannot make an external API call atomic.

Before:
```php
function handle() {
    Payment::create(["order_id" => $this->orderId]);
}
```
After:
```php
function handle() {
    Payment::firstOrCreate(
        ["order_id" => $this->orderId],
        ["status" => "pending"]
    );
}
// Migration: unique index on payments.order_id
```

### Queued webhook processing (Useful)

- **Smell:** Webhook controllers do slow business processing before acknowledging receipt.
- **Technique:** Verify the incoming webhook, durably dispatch a queued processing job, and promptly return the expected acknowledgement. Keep processing in the job and make it idempotent because providers can redeliver.
- **When to use:** Webhook processing exceeds provider timeouts or needs retries independent of the request.
- **When not to:** Do not acknowledge before the payload is durably accepted; queues require workers and duplicate handling.
- **Packages:** `spatie/laravel-webhook-client`

Before:
```php
verifyWebhook($request);
updateOrders($request->all());
sendNotifications();
return response()->noContent();
```
After:
```php
verifyWebhook($request);
ProcessWebhook::dispatch($request->all());
return response()->noContent();
```

### Unique queued jobs (Useful)

- **Smell:** Bursts of model updates enqueue redundant rebuilds for the same entity.
- **Technique:** Give recomputation jobs a stable uniqueId and implement ShouldBeUniqueUntilProcessing to suppress duplicate pending work. Laravel 13 requires shared lock-capable cache infrastructure and does not apply uniqueness inside batches.
- **When to use:** Repeated invalidations request the same rebuild.
- **When not to:** Pending-only uniqueness allows another enqueue during processing; do not use it if each event must execute separately.

Before:
```php
foreach ($updates as $update) RebuildProduct::dispatch($update->product_id);
```
After:
```php
class RebuildProduct implements ShouldQueue, ShouldBeUniqueUntilProcessing {
    public function __construct(public int $productId) {}
    public function uniqueId(): string {
        return (string) $this->productId;
    }
}
```

### After-commit side effects (Useful)

- **Smell:** File writes, notifications, or queued work happen even when their database transaction rolls back.
- **Technique:** Register DB::afterCommit callbacks or dispatch jobs with afterCommit so side effects start only after the outer transaction succeeds. Keep the business writes inside the transaction and ensure the eventual effect tolerates retry.
- **When to use:** A side effect depends on committed database state.
- **When not to:** A post-commit callback can still fail after the database commits; use durable jobs or an outbox for stronger delivery guarantees.

Before:
```php
DB::transaction(function () use ($data) {
    $invoice = Invoice::create($data);
    Storage::put("invoice.txt", $invoice->id);
});
```
After:
```php
DB::transaction(function () use ($data) {
    $invoice = Invoice::create($data);
    DB::afterCommit(fn () => WriteInvoiceFile::dispatch($invoice));
});
```

### Durable queued workflows (Useful)

- **Smell:** A slow report or onboarding workflow holds the initiating HTTP request open and loses visible status after navigation.
- **Technique:** Dispatch long work to Laravel queued jobs and persist a status or batch identifier that the UI can inspect later. Return promptly and show recoverable progress rather than requiring the original page to stay open.
- **When to use:** Work takes longer than users should wait on an initiating request.
- **When not to:** Needs running workers and durable status; queueing does not remove job timeouts or suit operations whose result the current request requires.

Before:
```php
$report = $generator->build($filters);
return response()->download($report);
```
After:
```php
$run = ReportRun::create(["status" => "pending"]);
GenerateReport::dispatch($run->id, $filters);
return redirect()->route("reports.progress", $run);
```

### Job batches (Useful)

- **Smell:** Independent local video jobs lack persisted progress and completion coordination.
- **Technique:** Use Bus::batch for a group of independent queued jobs and its persisted completion tracking. Track externally launched child work separately because Laravel 13 batch completion measures local jobs, not remote provider completion.
- **When to use:** Independent queued work needs shared progress, cancellation, or completion callbacks.
- **When not to:** A batch is not a remote workflow state machine and requires compatible connection and queue settings.

Before:
```php
foreach ($languages as $language) {
    TranslateVideo::dispatch($id, $language);
}
// no shared local progress counter
```
After:
```php
$jobs = collect($languages)->map(
    fn ($language) => new TranslateVideo($id, $language)
);
$batch = Bus::batch($jobs)->dispatch();
$run->update(["batch_id" => $batch->id]);
```

### Job chains with batches (Useful)

- **Smell:** Custom workflow plumbing manually starts dependent steps and parallel tasks.
- **Technique:** Use Bus::chain() for ordered dependent jobs and Bus::batch() for groups whose completion drives the next step. Parallel execution needs worker capacity and batches need metadata storage.
- **When to use:** A workflow consists of queueable sequential or parallel tasks.
- **When not to:** Complex durable workflows can require more than chains and batches.

Before:
```php
Import::dispatch($file);
// Manual completion tracking starts reconciliation and indexing.
```
After:
```php
Bus::chain([
    new Import($file), new Reconcile($file), new Reindex($file),
])->dispatch();
Bus::batch($independentJobs)->dispatch();
```

### Replay-safe event-sourcing reactors (Useful)

- **Smell:** Rebuilding projections sends emails, charges payments or fires duplicate downstream events.
- **Technique:** Separate replayable state/projection updates from one-time work and guard the latter with Verbs::unlessReplaying() or a supported once hook. Replay suppression is distinct from transaction after-commit timing or retry idempotency.
- **When to use:** Event handlers can run again during replay.
- **When not to:** An ordinary Laravel listener without event-store replay does not need this package guard; retries still require separate protection.
- **Packages:** `hirethunk/verbs`, `spatie/laravel-event-sourcing`

Before:
```php
public function handle(): void {
  $this->updateProjection();
  $this->chargeCard();
}
```
After:
```php
public function handle(): void {
  $this->updateProjection();
  Verbs::unlessReplaying(fn () => $this->chargeCard());
}
```

### Deferred HTTP batches (Situational)

- **Smell:** Best-effort outbound calls hold up a response even though their results are unused.
- **Technique:** Group outbound requests in Http::batch and defer sending the batch until after the response. Configure failure callbacks as needed; move essential delivery to durable queued jobs.
- **When to use:** Several noncritical integrations can run after acknowledgement.
- **When not to:** Process termination can lose the batch; fire-and-forget is unsuitable for payments or required notifications.

Before:
```php
Http::post($analyticsUrl, $event);
Http::post($metricsUrl, $event);
return response()->noContent();
```
After:
```php
Http::batch(fn (Batch $batch) => [
    $batch->post($analyticsUrl, $event),
    $batch->post($metricsUrl, $event),
])->defer();
return response()->noContent();
```

### Desired-state reconciliation (Situational)

- **Smell:** Imperative provisioning steps drift or fail midway, leaving resources inconsistent.
- **Technique:** Persist the desired configuration and repeatedly compare actual resources with that target, applying changes until they converge. Laravel scheduled commands or queued jobs can host application-level reconciliation; its model and convergence logic are application code.
- **When to use:** Provisioning or integrations must recover from partial changes and drift.
- **When not to:** An eventually convergent loop is excessive for a simple synchronous CRUD operation.

Before:
```php
$api->createServer();
$api->attachNetwork(); // failure leaves partial state
```
After:
```php
// Application reconciliation abstraction
$desired = DesiredServer::findOrFail($id);
$actual = $provider->inspect($desired->providerId);
$reconciler->converge($desired, $actual);
```

### Model lifecycle broadcasting (Situational)

- **Smell:** Every model lifecycle operation manually dispatches an equivalent broadcast event.
- **Technique:** Use Eloquent model broadcasting and define the channels for lifecycle broadcasts. Connected users learn about creates or updates without duplicating dispatch calls in each entry point.
- **When to use:** Multiple paths mutate a model that clients observe.
- **When not to:** Broad lifecycle broadcasts can reveal data or create noise; explicit domain events fit selective notifications.
- **Packages:** `laravel/reverb`

Before:
```php
Channel::create($data);
event(new ChannelCreated($channel));
```
After:
```php
class Channel extends Model {
 use BroadcastsEvents;
 public function broadcastOn(string $event): array { return [$this]; }
}
```

### Model-derived broadcast channels (Situational)

- **Smell:** Several events repeat the channel name for the same Eloquent model.
- **Technique:** Return the model from an event broadcastOn method so Laravel 13 derives the private channel consistently. Customize the model broadcastChannel method if the application needs a different name.
- **When to use:** Several broadcast events target the same model-specific channel.
- **When not to:** Model-derived names do not replace channel authorization and may need a stable public naming contract.

Before:
```php
public function broadcastOn(): array {
    return [new PrivateChannel("App.Models.Order.".$this->order->id)];
}
```
After:
```php
public function broadcastOn(): array {
    return [$this->order];
}
```

### Persist-and-reconcile queue delivery (Situational)

- **Smell:** A persisted order can be lost between saving it and successfully dispatching work.
- **Technique:** Persist the order before dispatch and periodically scan unfinished orders to redispatch missing work. Make processing idempotent.
- **When to use:** Database writes and queue delivery can fail independently.
- **When not to:** Polling adds delay and load; duplicate delivery must be safe.

Before:
```php
FulfillOrder::dispatch($order);
$order->save();
```
After:
```php
$order->save();
FulfillOrder::dispatch($order);
// Periodic reconciliation:
Order::whereNull("fulfilled_at")->each(
    fn ($order) => FulfillOrder::dispatch($order)
);
```

### Queue failover (Situational)

- **Smell:** A primary queue connection outage prevents the application from dispatching jobs.
- **Technique:** Configure the Laravel 13 failover queue driver with an ordered list of connections. A failed enqueue attempt falls back to the next connection, which can be another queue or synchronous execution.
- **When to use:** Enqueue availability is important and a suitable backup connection exists.
- **When not to:** Synchronous fallback increases response time; failover concerns enqueueing and does not recover jobs already stranded on a failed backend.

Before:
```php
SendReceipt::dispatch($invoice)->onConnection("redis");
```
After:
```php
// config/queue.php connections:
"failover" => [
    "driver" => "failover",
    "connections" => ["redis", "database", "sync"],
],
// Select this connection when dispatching:
SendReceipt::dispatch($invoice)->onConnection("failover");
```

### Queue payload enrichment (Situational)

- **Smell:** Every job constructor repeats the initiator or tracing fields needed for operational tracking.
- **Technique:** Register Queue::createPayloadUsing in a service provider to attach metadata to dispatched job payloads. Keep the callback execution-safe in both HTTP and console contexts, and read the metadata in queue tooling or middleware.
- **When to use:** All jobs need consistent operational metadata independent of their domain input.
- **When not to:** Payload metadata is not automatically a domain property; avoid secrets and dependence on an authenticated HTTP user always existing.

Before:
```php
new ExportJob($data, auth()->id());
new SyncJob($data, auth()->id());
```
After:
```php
Queue::createPayloadUsing(function ($connection, $queue, $payload) {
    return ["initiator_id" => auth()->id()];
});
ExportJob::dispatch($data);
```

### Redis-backed broadcast fan-out (Situational)

- **Smell:** Subscribers on different Reverb servers miss messages emitted on another instance.
- **Technique:** Enable Reverb scaling and connect each instance to central Redis pub/sub behind a load balancer. Redis propagates messages across servers, allowing local subscriptions to receive global broadcasts.
- **When to use:** One Reverb instance cannot meet measured connection or message demand.
- **When not to:** A single instance may suffice; distributed operation adds Redis dependency and operational coordination.
- **Packages:** `laravel/reverb`

Before:
```php
// Two independent servers behind a load balancer.
REVERB_SCALING_ENABLED=false
```
After:
```php
REVERB_SCALING_ENABLED=true
// Configure the same central Redis connection on every instance.
```

### Resilient noncritical broadcasting (Situational)

- **Smell:** A broadcasting outage causes an otherwise successful user operation to fail.
- **Technique:** Have the broadcast event implement ShouldRescue alongside its broadcast contract. Laravel 13 reports broadcasting exceptions without allowing them to interrupt the core flow.
- **When to use:** A broadcast update is best-effort and clients can recover through normal reads.
- **When not to:** Do not suppress failure of events required for business correctness; logging is not delivery or retry.

Before:
```php
class DashboardUpdated implements ShouldBroadcastNow {
    // Broadcasting errors bubble into the request.
}
```
After:
```php
class DashboardUpdated implements ShouldBroadcastNow, ShouldRescue {
    // Implement the usual broadcast event methods.
}
```

### Resource locking job middleware (Situational)

- **Smell:** Different queued jobs concurrently modify the same resource.
- **Technique:** Attach WithoutOverlapping middleware keyed by the resource, with shared() when different job classes share the lock. Set release and expiry windows and retry budgets deliberately.
- **When to use:** Concurrent resource mutations must be serialized.
- **When not to:** Requires a shared atomic-lock cache; releases consume attempts and expiry must exceed normal execution.

Before:
```php
public function handle(): void {
    $this->account->recalculate();
}
```
After:
```php
public function middleware(): array {
    return [(new WithoutOverlapping("account:{$this->account->id}"))
        ->shared()->releaseAfter(10)->expireAfter(120)];
}
```

### Runtime budget enforcement (Situational)

- **Smell:** Heavy tenants monopolize queue workers and delay other tenants.
- **Technique:** Measure runtime by tenant/job type, charge it to a time-window budget, and route over-budget tenants to a separate worker queue. Laravel RateLimiter::increment(amount: ...) can record measured units; budgeting and rerouting remain application logic.
- **When to use:** Shared queues suffer noisy-neighbor effects.
- **When not to:** Runtime measurement and budget policy add complexity; a separate queue needs dedicated workers.
- **Packages:** `laravel/ai`

Before:
```php
RunTenantTask::dispatch($tenantId); // everyone shares one queue
```
After:
```php
RateLimiter::increment($key, decaySeconds: 60, amount: $elapsedMs);
$queue = $budget->exceeded($tenantId) ? "heavy-tenants" : "default";
RunTenantTask::dispatch($tenantId)->onQueue($queue);
```

### Socket command listeners (Situational)

- **Smell:** A realtime client sends an extra HTTP request to persist a command alongside its open socket.
- **Technique:** Handle selected Reverb MessageReceived events through a custom application listener. Parse and validate the raw command and authorize its operation before delegating to persistence; Reverb’s event hook does not supply a complete secure command protocol.
- **When to use:** A measured high-frequency path benefits from eliminating an extra HTTP hop.
- **When not to:** Custom authentication, validation, error responses and protocol handling are substantial work; normal Laravel HTTP actions are usually simpler.
- **Packages:** `laravel/reverb`

Before:
```php
await fetch("/messages", { method: "POST", body: JSON.stringify(data) });
```
After:
```php
// Custom listener registered for Reverb MessageReceived.
public function handle(MessageReceived $event): void {
  $command = $this->parseAndValidate($event->message);
  $this->authorizeCommand($event->connection, $command);
  $this->storeMessage($command);
}
```

### Telemetry sampling (Situational)

- **Smell:** Full capture of high-volume metrics remains too expensive after aggregation improvements.
- **Technique:** Configure a lower Pulse recorder sample rate and interpret scaled dashboard values as estimates. Sampling reduces recorded events while preserving useful aggregate trends for high-volume telemetry.
- **When to use:** Approximate performance metrics are sufficient.
- **When not to:** Sparse metrics lose usefulness, and sampling is inappropriate for financial totals or authoritative business ledgers.
- **Packages:** `laravel/pulse`

Before:
```php
// Recorder configuration
"sample_rate" => 1,
```
After:
```php
// Recorder configuration: record approximately one in ten.
"sample_rate" => 0.1,
```

### Traceable replayable event processing (Situational)

- **Smell:** Important downstream processing is hard to audit or rerun for one business entity.
- **Technique:** Persist application-specific processing history and expose deliberate entity-scoped reprocessing. The talk describes auction event traceability and reruns without claiming full event sourcing; Laravel events alone do not implement history or replay.
- **When to use:** Critical workflows need an audit trail and targeted recovery.
- **When not to:** Replay can duplicate emails, payments, and other effects unless handlers use deduplication or explicit replay policies.

Before:
```php
event(new AuctionEnded($auction));
// No processing history or targeted recovery.
```
After:
```php
// Application-defined event processing service:
$processor->recordAndDispatch(new AuctionEnded($auction));
$processor->rerunFor($auction->id);
// Handlers distinguish safe replay from irreversible effects.
```

### Circuit breaker (Situational)

- **Smell:** Jobs repeatedly call an unavailable external dependency, tying up workers in network timeouts.
- **Technique:** Guard the dependency with a closed/open/half-open circuit: count relevant failures, stop calls while open and admit a recovery probe after a cooldown. Laravel job middleware and shared cache can implement it; ordinary exception throttling is a simpler alternative without the full state machine.
- **When to use:** An external outage would otherwise consume worker capacity across many jobs.
- **When not to:** Adds state, tuning and distributed coordination; excessive complexity for rare inexpensive failures, and not a substitute for network timeouts.
- **Packages:** `harris21/laravel-fuse`

Before:
```php
public function handle(Gateway $gateway) {
    $gateway->charge($this->invoice); // Repeated during outages.
}
```
After:
```php
// Application breaker middleware wraps this job:
if ($breaker->isOpen("payments")) {
    $job->release(60);
    return;
}
$breaker->run("payments", fn () => $next($job));
```

### Completion watchdog (Situational)

- **Smell:** A missing job or integration run raises no exception, so silent absence escapes monitoring.
- **Technique:** Record successful completion heartbeats for expected operations and have an independent watchdog alert when the deadline passes. Queue events and scheduler success hooks can supply signals, but business completion needs its own definition.
- **When to use:** Important periodic or queued workflows must run within a known interval.
- **When not to:** Independent checking adds infrastructure; a heartbeat written before useful work finishes gives false confidence.

Before:
```php
// Only failed jobs are monitored.
```
After:
```php
// After successful business completion:
Cache::put("last-sync", now(), now()->addDay());
// Independent watchdog:
$last = Cache::get("last-sync");
if (!$last || $last->lt(now()->subMinutes(10))) alertMissingSync();
```

### Job chains (Situational)

- **Smell:** Dependent queued onboarding steps are dispatched separately and can run before their prerequisites complete.
- **Technique:** Use Bus::chain() to run each queued step only after the preceding job succeeds. A failed step prevents subsequent steps; design retries and failure recovery for the whole workflow.
- **When to use:** Account, wallet or device setup steps depend on earlier success.
- **When not to:** Chains do not provide a cross-job transaction or automatic compensation; independent work can use a batch.

Before:
```php
CreateAccount::dispatch($userId);
CreateWallet::dispatch($userId);
ProvisionDevice::dispatch($userId);
```
After:
```php
Bus::chain([
    new CreateAccount($userId),
    new CreateWallet($userId),
    new ProvisionDevice($userId),
])->dispatch();
```

### Model audit trails (Situational)

- **Smell:** Manual audit writes are duplicated around model changes.
- **Technique:** Attach LogsActivity to the model and configure the attributes and changes to record centrally.
- **When to use:** You need a consistent record of Eloquent changes.
- **When not to:** Bulk updates bypass model events; audit data adds storage and privacy obligations.
- **Packages:** `spatie/laravel-activitylog`

Before:
```php
$user->update($data);
Audit::create(["subject_id" => $user->id, "changes" => $data]);
```
After:
```php
class User extends Model {
    use LogsActivity;
    public function getActivitylogOptions(): LogOptions {
        return LogOptions::defaults()->logOnly(["name"])->logOnlyDirty();
    }
}
```

### Queue rate-limiting middleware (Situational)

- **Smell:** Each import worker independently calls a provider, collectively exceeding its API quota.
- **Technique:** Use a shared Redis-backed limiter keyed to the provider's actual quota scope and apply it to all affected jobs. Laravel RateLimitedWithRedis middleware can release throttled jobs; account for releases consuming attempts.
- **When to use:** Multiple workers share one API account or provider limit.
- **When not to:** Requires shared state, a correct quota key and retry budgeting; it does not make the provider immune to outages.

Before:
```php
public function handle() { $provider->import($this->row); }
```
After:
```php
// Register a named limiter with a shared provider key.
public function middleware(): array {
    return [new RateLimitedWithRedis("provider-import")];
}
```

### Queued job debouncing (Situational)

- **Smell:** Repeated edits dispatch redundant rebuild or synchronization jobs.
- **Technique:** Use Laravel 13’s DebounceFor attribute and a per-resource debounceId() so the latest dispatch supersedes earlier jobs within the window. Set maxWait when continual edits must not postpone execution indefinitely.
- **When to use:** Only the latest state needs processing.
- **When not to:** Discarded jobs must carry no required independent effects; shared cache is needed and eviction can permit duplicates.

Before:
```php
RebuildIndex::dispatch($document->id);
RebuildIndex::dispatch($document->id);
```
After:
```php
#[DebounceFor(30, maxWait: 120)]
class RebuildIndex implements ShouldQueue {
    use Dispatchable, Queueable;
    public function __construct(public int $documentId) {}
    public function debounceId(): string { return (string) $this->documentId; }
    public function handle(): void { SearchIndex::rebuild($this->documentId); }
}
```

### Relation-free job serialization (Situational)

- **Smell:** Queued model arguments include loaded relationships that are reloaded unnecessarily by workers.
- **Technique:** Use WithoutRelations on promoted model properties or call withoutRelations() before dispatch, then load exactly what the job requires.
- **When to use:** Loaded relation graphs inflate queue work or lose their original constraints on restoration.
- **When not to:** Jobs needing relation data must explicitly query it again.

Before:
```php
ProcessOrder::dispatch($order->load("items"));
```
After:
```php
public function __construct(
    #[WithoutRelations] public Order $order,
) {}
public function handle(): void {
    $this->order->load("items");
}
```

### Saga compensation (Situational)

- **Smell:** A workflow rolls back SQL but leaves completed remote API effects behind.
- **Technique:** Register compensating actions for completed external effects in the pipeline payload and run appropriate recovery if a later stage fails. Laravel database transactions protect database writes but cannot undo remote API calls; durable compensation needs retry and idempotency design.
- **When to use:** A multistep workflow combines database changes with reversible remote effects.
- **When not to:** Simple database-only workflows need transactions; compensation can itself fail and is not atomic rollback.

Before:
```php
DB::transaction(function () use ($lock) {
  $lock->createCode();
  Booking::create($data); // Failure leaves remote code behind.
});
```
After:
```php
$payload->code = $lock->createCode();
$payload->compensations[] = fn () => $lock->deleteCode($payload->code);
try {
  Pipeline::send($payload)->through($remainingPipes)->thenReturn();
} catch (Throwable $e) {
  foreach (array_reverse($payload->compensations) as $undo) $undo();
  throw $e;
}
```

### State-transition and coalesced broadcasts (Situational)

- **Smell:** A high-volume dashboard broadcasts every counter change faster than the frontend can render.
- **Technique:** Continue recording backend events and retain the latest aggregate, but send snapshots at a bounded cadence using shared cache coordination. Arrange a trailing update if necessary; a quickly released lock does not itself enforce the throttle interval.
- **When to use:** The UI needs current aggregates rather than every raw event.
- **When not to:** Snapshots omit intermediate states and need explicit final-delivery behavior; not suitable for a complete event stream.

Before:
```php
event(new DashboardUpdated($latest)); // Every incoming metric.
```
After:
```php
Cache::put("dashboard:latest", $latest, 60);
if (Cache::add("dashboard:broadcast-window", true, 2)) {
    event(new DashboardUpdated($latest));
}
// Arrange a trailing flush to send changes suppressed in the window.
```

## Frontend (Blade/Livewire/Inertia)

### Reusable Blade components (Recommended)

- **Smell:** Repeated input or layout markup drifts across templates and mixes styling with page content.
- **Technique:** Extract shared markup into Blade components with explicit props and slots, using named slots for variable regions. Forward the attribute bag so Livewire bindings and other caller attributes work without duplicating templates.
- **When to use:** Several views share the same visual and behavioral structure.
- **When not to:** One-off markup may be clearer inline; excessive tiny components make templates harder to follow.
- **Packages:** `alpinejs`, `livewire/flux`, `livewire/livewire`, `spatie/laravel-dashboard`, `spatie/laravel-support-bubble`, `vue`

Before:
```php
<label>Email</label>
<input name="email" class="border rounded" wire:model="email">
```
After:
```php
// resources/views/components/input.blade.php
@props(["label"])
<label>{{ $label }}</label>
<input {{ $attributes->class(["border rounded"]) }}>
// Caller
<x-input label="Email" name="email" wire:model="email" />
```

### Client-local UI state (Alpine) (Recommended)

- **Smell:** A purely client-side interaction makes an unnecessary server round trip.
- **Technique:** Keep the browser action with its Livewire component through a Js method returning JavaScript. Current Livewire on Laravel 13 executes this action in the browser when called through $wire.
- **When to use:** An interaction only clears fields or changes local browser state.
- **When not to:** Server data, persistence, validation, and authorization require server execution.
- **Packages:** `alpinejs`, `filament/filament`, `livewire/livewire`, `vue`

Before:
```php
public function clear(): void {
    $this->query = "";
}
// browser calls a server action just to clear the field
```
After:
```php
#[Js]
public function clear(): string {
    return '$wire.query = ""';
}
```

### Generated typed route helpers (Recommended)

- **Smell:** Frontend HTTP calls repeat URL strings and drift when Laravel routes or controller signatures change.
- **Technique:** Use Wayfinder to generate typed frontend functions from Laravel controller actions and routes. Pass their generated URL and method data to Axios or fetch instead of hand-maintaining endpoint strings.
- **When to use:** A TypeScript frontend calls many Laravel endpoints and benefits from shared route definitions.
- **When not to:** Generation must run when routes change; this does not validate runtime response payloads.
- **Packages:** `@laravel/vite-plugin-wayfinder`, `laravel/wayfinder`, `tightenco/ziggy`

Before:
```php
await axios.get(`/users/${id}`);
```
After:
```php
import { show } from "@/actions/App/Http/Controllers/UserController";
await axios.get(show(id).url);
```

### Component file colocation (Recommended)

- **Smell:** A very small class component spreads its presentation across an unnecessary extra template file.
- **Technique:** Return fixed Blade markup from the component render method so a small component can live in one file. Laravel 13 supports inline component views; keep runtime values as Blade expressions rather than interpolating render-closure data into template source.
- **When to use:** A class-based component has a tiny fixed template.
- **When not to:** Larger templates become hard to edit inside PHP strings and anonymous components may be simpler when no class behavior is needed.
- **Packages:** `livewire/livewire`, `react`

Before:
```php
public function render() { return view("components.badge"); }
// a separate template contains one span
```
After:
```php
public function render(): string {
    return '<span {{ $attributes }}>{{ $slot }}</span>';
}
```

### Full-page Livewire components (Recommended)

- **Smell:** A controller and wrapper view exist only to host a Livewire component.
- **Technique:** Route directly to the full-page Livewire component and let it own its page layout. With Livewire 4 use its page routing entry point, preserving hydration rather than routing to render directly.
- **When to use:** The page is already entirely driven by one Livewire component.
- **When not to:** A controller with meaningful HTTP orchestration or a mixed page may still justify the wrapper.
- **Packages:** `livewire/livewire`

Before:
```php
Route::get("/quotes", fn () => view("quotes-wrapper"));
// Wrapper contains only <livewire:quotes-page />.
```
After:
```php
Route::livewire("/quotes", "pages::quotes");
```

### Custom form control components (Recommended)

- **Smell:** Div-based overlays are clipped by overflow and need fragile z-index/focus workarounds.
- **Technique:** Use the browser’s popover top layer for non-modal overlays and modal dialog behavior for blocking interactions. Wrap those primitives with Blade components for reusable triggers, positioning and styling.
- **When to use:** Menus, popovers and dialogs suffer from layering or focus complexity.
- **When not to:** Browser support and integration lifecycle need checking; popovers are non-modal.
- **Packages:** `alpinejs`, `chart.js`, `filament/filament`, `laravel/nova`, `livewire/flux`, `livewire/livewire`

Before:
```php
<div class="absolute z-50" x-show="open">Menu</div>
```
After:
```php
<button popovertarget="menu">Menu</button>
<div id="menu" popover>Actions</div>
<dialog id="confirm">Confirm action</dialog>
```

### Independent rendering islands (Recommended)

- **Smell:** Unrelated interactions rerender expensive independent sections and repeat their queries.
- **Technique:** Wrap an independent region in a Livewire 4 island and access expensive data through an on-demand computed property inside it. The island shares component state while isolating rendering, avoiding a child component created only for performance isolation.
- **When to use:** Independent dashboard sections have expensive rendering.
- **When not to:** Reusable features with their own lifecycle merit child components; island placement and shared-state concurrency have constraints.
- **Packages:** `filament/forms`, `livewire/livewire`

Before:
```php
<div>{{ $this->revenue }}</div>
<button wire:click="increment">Increment</button>
```
After:
```php
@island(name: "revenue")
  <div>{{ $this->revenue }}</div>
  <button wire:click="$refresh">Refresh</button>
@endisland
<button wire:click="increment">Increment</button>
```

### Persistent Inertia layouts (Recommended)

- **Smell:** Navigation recreates a video player or local counter and loses its client state.
- **Technique:** Wrap matching layout DOM elements in Livewire @persist keys so wire:navigate reuses them across pages. Place persisted elements outside Livewire components; this preserves browser state rather than creating a server cache.
- **When to use:** Media or similar client state should survive navigation.
- **When not to:** Matching keys and layout placement require care; use a new key or normal rendering when state should reset.
- **Packages:** `@inertiajs/vue3`, `inertiajs/inertia-laravel`, `livewire/livewire`

Before:
```php
<video controls src="/movie.mp4"></video>
```
After:
```php
@persist("movie-player")
    <video controls src="/movie.mp4"></video>
@endpersist
```

### Debounced input synchronization (Recommended)

- **Smell:** Form inputs send server updates on each edit although the server only needs them at submission.
- **Technique:** Use ordinary deferred wire:model bindings and submit with wire:submit so input state travels with the action. Validate on the server and report rejected credentials through the component error bag.
- **When to use:** A form needs submit-time validation rather than per-keystroke feedback.
- **When not to:** Immediate server validation or dynamic dependent fields may need live updates instead.
- **Packages:** `livewire/livewire`

Before:
```php
<input wire:model.live="email">
```
After:
```php
<form wire:submit="login">
    <input wire:model="email">
    <button type="submit">Login</button>
</form>
// Action begins with $this->validate().
```

### Visibility-aware polling (Recommended)

- **Smell:** Hidden or off-screen widgets poll frequently even though nobody needs their updates.
- **Technique:** Adjust Livewire polling intervals to acceptable freshness and use the visible modifier for off-screen regions. Add application conditions to stop polling when the useful work ends.
- **When to use:** Periodic updates are sufficient and only useful while their region is being viewed.
- **When not to:** Delayed freshness is a tradeoff; critical background updates may need a different delivery mechanism.
- **Packages:** `@inertiajs/react`, `@inertiajs/vue3`, `inertiajs/inertia-laravel`, `livewire/livewire`

Before:
```php
<div wire:poll.1s="refreshStatus">...</div>
```
After:
```php
<div wire:poll.10s.visible="refreshStatus">...</div>
```

### Form state abstraction (Recommended)

- **Smell:** Autocomplete or similar interactions trigger page visits or duplicate custom loading and error state handling.
- **Technique:** Use Inertia useHttp for standalone data requests that do not navigate or mutate page props. Keep Laravel endpoints returning JSON and consume the parsed response returned by the helper while it tracks loading and errors.
- **When to use:** An interaction needs JSON data while staying on the current page.
- **When not to:** Navigation should retain normal Inertia visits; useHttp has separate data state from page props.
- **Packages:** `@inertiajs/react`, `@inertiajs/vue3`, `inertiajs/inertia-laravel`

Before:
```php
router.get("/suggestions", { q: query });
```
After:
```php
const suggestions = useHttp({ q: "" });
suggestions.q = query;
const results = await suggestions.get("/suggestions");
// Render results; suggestions tracks processing and errors.
```

### Lazy component loading (Recommended)

- **Smell:** Slow modal initialization delays displaying the container and gives no loading feedback.
- **Technique:** Lazy-load expensive Livewire child content and provide a placeholder while it initializes. Livewire 4 supports placeholder() on class-based components; use defer when loading should start after initial rendering regardless of visibility.
- **When to use:** Modal content has expensive initialization and can load after the shell appears.
- **When not to:** Lazy loading adds a request and postpones errors; cheap or immediately required content may be better rendered normally.
- **Packages:** `livewire/livewire`

Before:
```php
<livewire:expensive-report />
```
After:
```php
<livewire:expensive-report lazy />
// Component:
public function placeholder() {
    return "<div>Loading report...</div>";
}
```

### Reusable accessible modal components (Recommended)

- **Smell:** Create/edit dialogs lose their context on navigation and cannot be linked directly.
- **Technique:** Give a modal a Laravel route and compose its foreground view over a retained background page in the Inertia frontend. The 2021 API was an unreleased preview, so implement the route/overlay pattern with a current verified integration rather than copying that API.
- **When to use:** A create/edit dialog should have a URL while preserving a listing behind it.
- **When not to:** Routing, history and direct-entry fallbacks add complexity; a local modal can suffice.
- **Packages:** `@alpinejs/focus`, `alpinejs`, `inertiajs/inertia-laravel`, `livewire/livewire`

Before:
```php
<button @click="showCreate = true">Create</button>
```
After:
```php
// Conceptual route plus client overlay contract.
Route::get("/users/create", fn () => Inertia::render("Users/Index", [
  "users" => User::paginate(), "modal" => "CreateUser",
]));
```

### Semantic component events (Recommended)

- **Smell:** A tightly coupled child component emits an event solely to ask its parent to execute one action.
- **Technique:** Call the containing Livewire component through $parent from the child view to replace one-off event/listener plumbing. This deliberately exposes the parent action contract rather than making the child independently reusable.
- **When to use:** A child is specific to one parent workflow.
- **When not to:** Reusable children are better served by modelable values or events; direct calls couple them to parent method names.
- **Packages:** `@inertiajs/vue3`, `alpinejs`, `inertiajs/inertia-laravel`, `livewire/livewire`

Before:
```php
// Child emits; parent listener forwards to add().
$this->dispatch("add-requested");
```
After:
```php
<button wire:click="$parent.add()">Add</button>
```

### Client whispers for ephemeral state (Useful)

- **Smell:** Typing notifications make server requests or database writes for every keystroke.
- **Technique:** Use Echo whispers on authorized channels for transient client-to-client state. Throttle repeated start notifications and expire local typing state after inactivity; whispers bypass application persistence and should not carry trusted business commands.
- **When to use:** Typing indicators or similar disposable presence hints.
- **When not to:** Messages requiring validation, durable storage, or authoritative business effects must go through Laravel.
- **Packages:** `laravel-echo`, `laravel/reverb`

Before:
```php
input.addEventListener("input", () => fetch("/typing", {method:"POST"}));
```
After:
```php
room.whisper("typing", { name: currentUser.name });
room.listenForWhisper("typing", e => {
    showTyping(e.name);
    clearTimeout(typingTimer);
    typingTimer = setTimeout(hideTyping, 2000);
});
// Throttle sends; the timeout is an application choice.
```

### Optimistic UI (Useful)

- **Smell:** Simple interactions wait for the server response before showing a visible change.
- **Technique:** Use Inertia optimistic updates to apply the expected local change immediately and reconcile it with the server result or roll it back on failure. Preserve unchanged nested values when constructing the update.
- **When to use:** Interactions such as likes or bookmarks are predictable and reversible.
- **When not to:** Complex irreversible operations or frequent failures make optimistic state misleading and reconciliation harder.
- **Packages:** `@inertiajs/vue3`, `inertiajs/inertia-laravel`, `livewire/livewire`

Before:
```php
router.post(`/posts/${post.id}/like`);
// Count changes only after the response.
```
After:
```php
router.post(`/posts/${post.id}/like`, {}, {
  optimistic: props => ({
    post: { ...props.post, likes: props.post.likes + 1 }
  })
});
```

### Presence channels (Useful)

- **Smell:** Online-user state is maintained through repeated polling or ad hoc database flags.
- **Technique:** Join a workspace-scoped authorized presence channel and initialize the roster with here, updating it with joining and leaving. Presence authorization returns safe user information; the roster represents channel subscription membership.
- **When to use:** Connected participants need online status within a workspace or room.
- **When not to:** Subscription presence is not proof of human activity, and reconnects or multiple tabs need appropriate UI handling.
- **Packages:** `laravel-echo`, `laravel/reverb`

Before:
```php
setInterval(() => fetch("/workspace/1/online"), 1000);
```
After:
```php
Echo.join("workspace.1")
    .here(users => online = users)
    .joining(user => online.push(user))
    .leaving(user => online = online.filter(u => u.id !== user.id));
```

### Component model contracts and attribute forwarding (Useful)

- **Smell:** Parents know a custom control’s internal Alpine state and manually synchronize it.
- **Technique:** Encapsulate the control implementation in a Blade component and expose its value through Alpine x-modelable. Forward binding attributes so callers can use x-model as with a native input; verify separate Livewire integrations rather than assuming all modifiers work.
- **When to use:** A custom toggle or picker should be reusable in different surrounding forms.
- **When not to:** Modelable state must be JSON-compatible; native inputs are simpler when sufficient.
- **Packages:** `alpinejs`, `livewire/flux`, `livewire/livewire`, `vue`

Before:
```php
<x-toggle @changed="selected = $event.detail.value" :selected="selected" />
```
After:
```php
{{-- toggle.blade.php --}}
<div x-data="{ value: false }" x-modelable="value" {{ $attributes }}>
  <button type="button" @click="value = !value">Toggle</button>
</div>
{{-- Caller --}}
<x-toggle x-model="selected" />
```

### Custom Blade directives and compiler extensions (Useful)

- **Smell:** Standard authentication directives check the wrong authentication system for an application.
- **Technique:** Register custom Blade authentication directives that compile conditions against the chosen authentication service. Laravel 13 permits deliberate built-in overrides, though a guard argument or separately named Blade condition is clearer when it serves the same purpose.
- **When to use:** A genuinely separate authentication system must integrate with template conditions.
- **When not to:** Overriding familiar semantics surprises maintainers and template conditions do not authorize backend operations.
- **Packages:** `livewire/livewire`

Before:
```php
@auth
    // Laravel guard checked while application uses another auth system
@endauth
```
After:
```php
Blade::directive("auth", fn () =>
    '<?php if (app(ExternalAuth::class)->check()): ?>'
);
Blade::directive("endauth", fn () => '<?php endif; ?>');
```

### Progressive navigation (Useful)

- **Smell:** Navigation waits for the server before showing the destination page, making slow visits feel unresponsive.
- **Technique:** Use an Inertia instant visit to show the destination component immediately while its server request runs in the background. Design the component to render with shared props and loading placeholders until page-specific props arrive.
- **When to use:** The target component is known and can render a useful intermediate state.
- **When not to:** It does not reduce server work, and components that require complete page data cannot render immediately without adaptation.
- **Packages:** `@inertiajs/vue3`, `inertiajs/inertia-laravel`, `livewire/livewire`

Before:
```php
<Link href="/dashboard">Dashboard</Link>
```
After:
```php
<Link href="/dashboard" component="Dashboard">Dashboard</Link>
<!-- Dashboard renders placeholders until its real props arrive. -->
```

### Slot-based component composition (Useful)

- **Smell:** String props cannot express rich labels and optional markup causes a growing set of flags.
- **Technique:** Accept a named Blade slot for rich content and forward its slot attribute bag to the rendered element. Slots allow caller-supplied icons or buttons without expanding the component’s prop vocabulary.
- **When to use:** Reusable presentation needs custom markup at known insertion points.
- **When not to:** A simple text-only label rarely needs a named-slot API; strings do not have slot attributes.
- **Packages:** `livewire/flux`, `livewire/livewire`, `vue`

Before:
```php
<x-field label="Email" has-icon icon="mail" />
```
After:
```php
<x-field>
  <x-slot:label class="font-bold"><x-icon.mail /> Email</x-slot:label>
  <x-input.text name="email" />
</x-field>
{{-- Field label output --}}
<label {{ $label->attributes }}>{{ $label }}</label>
```

### Streaming responses (Useful)

- **Smell:** An actively waiting user sees no feedback until a long AI text response finishes.
- **Technique:** Stream text fragments into the UI as they become available using Laravel response streaming or Livewire wire:stream. When users do not need to wait, run the task asynchronously and notify them later.
- **When to use:** Interactive chat or similar text generation benefits from visible incremental output.
- **When not to:** Streaming holds an active connection; Livewire wire:stream is incompatible with Octane and unattended work may be better queued.
- **Packages:** `laravel/ai`, `livewire/livewire`

Before:
```php
$this->answer = $agent->complete($prompt);
```
After:
```php
foreach ($agent->textChunks($prompt) as $chunk) {
    $this->stream(to: "answer", content: $chunk);
}
// Blade:
<div wire:stream="answer"></div>
```

### Framework render-hook extensions (Useful)

- **Smell:** Panel UI customization requires editing vendor templates to insert an avatar or custom element.
- **Technique:** Register a Filament render hook at a documented panel extension point and return a Blade view for the custom markup. Hook placement extends the existing panel without copying its full layout.
- **When to use:** A panel needs targeted inserted UI near existing regions.
- **When not to:** Available hook positions may not express a full redesign; verify hook identifiers for the installed version.
- **Packages:** `filament/filament`, `livewire/livewire`

Before:
```php
// Patch vendor sidebar markup to add an avatar.
@include("panel.avatar")
```
After:
```php
FilamentView::registerRenderHook(
  PanelsRenderHook::SIDEBAR_NAV_START,
  fn () => view("panel.avatar"),
);
```

### Incremental append pagination (Useful)

- **Smell:** Load-more increases the query limit and rerenders all previously loaded rows.
- **Technique:** Query only the next bounded batch and append its named island using Livewire 4’s wire:island.append. An intersection sentinel can trigger the next batch; use stable ordering and keys to avoid duplicates or omissions.
- **When to use:** Long feeds or load-more lists accumulate many records.
- **When not to:** Random-access pagination or frequent reordering may be simpler with ordinary pagination; appended DOM still consumes memory.
- **Packages:** `inertiajs/inertia-laravel`, `livewire/livewire`, `vue`

Before:
```php
$posts = Post::latest()->limit($loadedCount)->get();
// Every click reloads every prior row.
```
After:
```php
@island(name: "posts")
  @foreach ($this->posts as $post)
    <article wire:key="post-{{ $post->id }}">{{ $post->title }}</article>
  @endforeach
@endisland
<button wire:click="nextPage" wire:island.append="posts">More</button>
{{-- computed posts queries only the current page --}}
```

### Livewire form objects (Useful)

- **Smell:** A component accumulates many form fields, rules, and model-population helpers.
- **Technique:** Move form state, validation, and population behavior to a typed Livewire Form object. Update bindings and property hooks to paths such as form.contactId, leaving the component to coordinate interactions.
- **When to use:** A substantial form obscures the component’s UI behavior or repeats across screens.
- **When not to:** A tiny form does not need another class; Livewire forms are not interchangeable with HTTP Form Requests.
- **Packages:** `livewire/livewire`

Before:
```php
public string $title = "";
public int $contactId;
protected function rules() { /* many field rules */ }
```
After:
```php
class QuoteForm extends Form {
    #[Validate("required")]
    public string $title = "";
    public ?int $contactId = null;
}
// Component: public QuoteForm $form;
// Blade: <input wire:model="form.title">
```

### Navigation and asset prefetching (Useful)

- **Smell:** Code-split Inertia pages wait for JavaScript chunks to download on every first visit.
- **Technique:** Call Vite::prefetch in a Laravel service provider to fetch app assets in the background after the initial page load. Set concurrency to control simultaneous requests and improve later navigation latency.
- **When to use:** An Inertia app uses code splitting and users commonly navigate among several pages.
- **When not to:** Prefetching uses bandwidth for pages users may never visit; large asset sets need a restrained strategy.
- **Packages:** `@inertiajs/vue3`, `inertiajs/inertia-laravel`

Before:
```php
// Only fetch a page chunk when navigating to it.
const page = await import("./Pages/Reports.vue");
```
After:
```php
// AppServiceProvider::boot():
Vite::prefetch(concurrency: 3);
```

### Standalone declarative admin components (Useful)

- **Smell:** Public pages duplicate rich form/table behavior or are forced into a full authenticated panel.
- **Technique:** Install the individual Filament component packages and render their Livewire interfaces in ordinary Blade pages. Configure the current package assets and preserve application access checks for public-facing operations.
- **When to use:** A public page needs a rich table or form while the application keeps its existing stack.
- **When not to:** Filament’s assets and component conventions are overhead for a simple static table.
- **Packages:** `filament/forms`, `filament/tables`, `livewire/livewire`

Before:
```php
<table>...</table>
// Hand-maintain public sorting, filters and pagination.
```
After:
```php
{{-- Public Blade page with configured Filament assets --}}
<livewire:public-products-table />
{{-- Inside that component’s view --}}
{{ $this->table }}
```

### URL-backed UI state (Useful)

- **Smell:** Sorting and pagination disappear on refresh and cannot be shared.
- **Technique:** Store shareable page filters, sorting, and pagination in route query parameters. Use replace for incidental changes and navigable links/history for pages through the chosen Vue or Inertia navigation layer.
- **When to use:** Users need bookmarks, refresh restoration, or back navigation.
- **When not to:** Private transient state and very frequent edits may not belong in the URL.
- **Packages:** `livewire/livewire`, `vue-router`

Before:
```php
const page = ref(1);
const sort = ref("title");
```
After:
```php
router.replace({ query: { ...route.query, sort: "title" } });
router.push({ query: { ...route.query, page: 2 } });
```

### Component render memoization (Situational)

- **Smell:** The same Blade component renders repeatedly with identical props.
- **Technique:** Opt a deterministic component into Blaze memoization so repeated render inputs reuse output.
- **When to use:** Rendering identical components consumes significant CPU.
- **When not to:** Avoid memoizing output dependent on hidden mutable state; measure whether caching pays off.
- **Packages:** `livewire/blaze`

Before:
```php
<x-price :amount="$amount" />
<x-price :amount="$amount" />
```
After:
```php
{{-- price.blade.php --}}
@blaze(memoize: true)
<span>{{ $amount }}</span>
```

### Deferred page props (Situational)

- **Smell:** Slow secondary dashboard data delays useful primary content.
- **Technique:** Use Inertia::defer() to render initial content first and automatically fetch secondary props afterward. A Deferred frontend component supplies the loading placeholder, replacing manual mounted-hook orchestration.
- **When to use:** A dashboard can present meaningful content before charts finish loading.
- **When not to:** Deferred requests complicate loading/error handling and are unsuitable for essential initial data.
- **Packages:** `inertiajs/inertia-laravel`

Before:
```php
return Inertia::render("Dashboard", [
  "summary" => $summary, "stats" => $this->slowStats(),
]);
```
After:
```php
return Inertia::render("Dashboard", [
  "summary" => $summary,
  "stats" => Inertia::defer(fn () => $this->slowStats()),
]);
```

### Enum-backed presentation (Situational)

- **Smell:** Select options, status labels and badge colors duplicate case mappings.
- **Technique:** Use a backed enum cast for the Eloquent attribute and implement Filament HasLabel/HasColor contracts on the enum. Forms and table badges reuse its options and presentation mappings.
- **When to use:** A finite category or status appears in multiple UI contexts.
- **When not to:** Embedding UI presentation in domain enums may be inappropriate across substantially different clients.
- **Packages:** `filament/filament`, `livewire/livewire`

Before:
```php
Select::make("status")->options(["delayed" => "Delayed"]);
// A separate table mapping repeats labels/colors.
```
After:
```php
Select::make("status")->options(FlightStatus::class);
TextColumn::make("status")->badge();
// FlightStatus implements HasLabel and HasColor.
protected function casts(): array {
    return ["status" => FlightStatus::class];
}
```

### Grouped and stepped forms (Situational)

- **Smell:** A large form presents all fields as one undifferentiated block.
- **Technique:** Use Filament schema sections, tabs or wizard steps to group fields by the user’s task. Preserve server validation and deliberate save behavior across the presentation groups.
- **When to use:** Large employee or multi-part forms have meaningful task boundaries.
- **When not to:** Extra steps slow simple forms and can hide cross-field context.
- **Packages:** `filament/schemas`, `livewire/livewire`

Before:
```php
return $schema->components([$name, $email, $street, $city, $salary]);
```
After:
```php
return $schema->components([
  Section::make("Contact")->schema([$name, $email]),
  Section::make("Address")->schema([$street, $city]),
  Section::make("Employment")->schema([$salary]),
]);
```

### Import aliases (Situational)

- **Smell:** Old JavaScript remains mixed with new modules and accidental dependencies slow migration.
- **Technique:** Give legacy directories explicit Vite import aliases so imports expose when a module depends on old code. Migrate incrementally and replace those imports as new modules take over.
- **When to use:** A Laravel frontend is being modernized incrementally while legacy code stays operational.
- **When not to:** An alias alone does not enforce architectural boundaries or remove legacy dependencies.
- **Packages:** `vite`

Before:
```php
import widget from "../../old/widgets/widget";
```
After:
```php
// vite.config.js resolve.alias
"@legacy-widgets": fileURLToPath(new URL("./resources/js/old/widgets", import.meta.url))
// Consumer
import widget from "@legacy-widgets/widget";
```

### Inertia partial reloads (Situational)

- **Smell:** Filtering one chart retransmits and recomputes unrelated props for the whole page.
- **Technique:** Use an Inertia partial visit requesting only affected prop keys, retaining other client props. Include dependent props that must refresh and preserve scroll where appropriate.
- **When to use:** Filtering or sorting within the same page component.
- **When not to:** Omitted props remain stale; this mechanism does not apply across different page components.
- **Packages:** `inertiajs/inertia-laravel`

Before:
```php
router.get("/dashboard", { range });
```
After:
```php
router.get("/dashboard", { range }, {
  only: ["chart", "rangeLabel"], preserveScroll: true,
});
```

### Markdown placeholder rendering (Situational)

- **Smell:** Rendering Markdown containing Blade components also interprets syntax inside fenced code examples.
- **Technique:** Extract fenced code blocks and substitute unique placeholders before rendering the remaining trusted Markdown and Blade content. Restore the separately rendered code blocks afterward so examples remain literal.
- **When to use:** Trusted documentation combines executable components and code samples.
- **When not to:** Never compile untrusted Markdown as Blade; placeholder collisions and parsing edge cases require care.

Before:
```php
$html = Blade::render(Str::markdown($markdown));
```
After:
```php
[$text, $blocks] = extractCodeBlocks($markdown);
$html = Blade::render(Str::markdown($text));
$html = restoreCodeBlocks($html, $blocks);
```

### Purpose-specific admin views and widgets (Situational)

- **Smell:** A separate card view duplicates filtering and pagination already implemented in a Filament table.
- **Technique:** Use Filament table layout components and contentGrid() to render records as responsive cards while retaining the table’s filtering and pagination. This adapts presentation without rebuilding query controls.
- **When to use:** The same records suit cards better than rows.
- **When not to:** Dense comparisons and tabular data may remain easier in ordinary rows.
- **Packages:** `filament/filament`, `filament/tables`, `laravel/nova`, `livewire/livewire`

Before:
```php
@foreach ($records as $record)
  <article>{{ $record->name }}</article>
@endforeach
{{-- Reimplement table filters and paging here. --}}
```
After:
```php
return $table
  ->columns([Stack::make([TextColumn::make("name")])])
  ->contentGrid(["md" => 2, "xl" => 3]);
```

### Renderless Livewire actions (Situational)

- **Smell:** An action re-renders a component although it changes nothing in the displayed view.
- **Technique:** Mark the action with Livewire Renderless or conditionally call skipRender(). On Laravel 13 with current Livewire this skips component rendering while still executing the server action.
- **When to use:** Logging or side-effect actions do not alter rendered state.
- **When not to:** Suppressing a necessary render leaves the UI stale; this does not eliminate the request.
- **Packages:** `livewire/livewire`

Before:
```php
public function recordDownload(): void {
    $this->auditDownload();
}
// full component render follows
```
After:
```php
#[Renderless]
public function recordDownload(): void {
    $this->auditDownload();
}
```

### Route-specific SSR middleware (Situational)

- **Smell:** A global Inertia server-rendering setting applies to pages that do not need SSR and causes wasted work.
- **Technique:** Use route middleware to choose whether Inertia server-side rendering is enabled for the current route. Apply the middleware to a route group so public pages and internal portal pages have explicit rendering policies.
- **When to use:** SEO-facing pages benefit from SSR but authenticated internal pages do not.
- **When not to:** Beware mutable global configuration in persistent workers and restore any temporary state; a uniformly rendered app needs no per-route policy.
- **Packages:** `inertiajs/inertia-laravel`

Before:
```php
// Every Inertia route uses the same SSR configuration
```
After:
```php
// Route-group middleware, with request-scoped state restoration
Route::prefix("admin")->middleware(DisableSsr::class)->group(function () {
    Route::get("/", AdminDashboard::class);
});
```

### Session and history-backed UI state (Situational)

- **Smell:** Leaving a form and returning with Back loses an unfinished draft.
- **Technique:** Use a uniquely keyed Inertia useForm or useRemember to store selected state in browser history and restore it on return. Exclude secrets such as passwords from remembered state.
- **When to use:** Users navigate away from forms and expect their draft on history return.
- **When not to:** History state is not durable draft storage across new sessions, and remembering sensitive fields is inappropriate.
- **Packages:** `@inertiajs/vue3`, `inertiajs/inertia-laravel`, `livewire/livewire`

Before:
```php
const form = useForm({ title: "" });
```
After:
```php
const form = useForm("EditPost:" + post.id, { title: post.title });
```

### Shared panel providers (Situational)

- **Smell:** A single admin interface accumulates conditionals for different application audiences.
- **Technique:** Configure separate Filament panels and providers with audience-specific resources, paths and pages. Register their providers in bootstrap/providers.php and enforce permissions independently of visual separation.
- **When to use:** Administrators and customers need distinct interfaces in one application.
- **When not to:** Multiple panels duplicate configuration and do not themselves provide authorization.
- **Packages:** `filament/filament`

Before:
```php
// One panel conditionally hides many unrelated resources.
```
After:
```php
$adminPanel->id("admin")->path("admin")->resources($adminResources);
$clientPanel->id("client")->path("client")->resources($clientResources);
```

### Wrapper UI components (Situational)

- **Smell:** A shared table owns fetching, filtering, sorting, and application-specific policy.
- **Technique:** Make the base table render rows/columns and emit sorting intent. Keep data operations in its page or an optional wrapper so the table works with local and server-driven data.
- **When to use:** Several pages need the same rendering with different data workflows.
- **When not to:** Separating tiny one-use components can add unnecessary plumbing.
- **Packages:** `vue`

Before:
```php
// BaseTable fetches /api/orders and paginates internally.
```
After:
```php
<BaseTable :rows="orders" :columns="columns"
    @sort="loadOrders" />
```

### Blade-based PDF rendering (Situational)

- **Smell:** PDF generation uses repetitive drawing commands or duplicates markup and presentation logic.
- **Technique:** Render a PDF from a Blade view and explicit data through a PDF service, selecting a Chromium-based driver when browser CSS is needed. Choose download, inline display, or persistence at the boundary.
- **When to use:** Invoices or reports benefit from normal HTML and CSS templating.
- **When not to:** Browser rendering requires runtime dependencies and may be too costly for high-volume synchronous requests.
- **Packages:** `spatie/laravel-pdf`

Before:
```php
$pdf->text(10, 10, $invoice->number);
$pdf->text(10, 20, $invoice->total);
```
After:
```php
return Pdf::view("invoices.show", ["invoice" => $invoice])
    ->name("invoice.pdf");
```

### Compile-time Blade component folding (Situational)

- **Smell:** Measured rendering cost is dominated by repeated eligible static Blade component composition.
- **Technique:** Use optional Blaze optimizations to fold eligible static components at compile time. Enable folding or memoization only where documented eligibility and data-dependence constraints are satisfied, rather than assuming every component is automatically folded.
- **When to use:** Component-heavy Blade pages remain slow after normal view compilation.
- **When not to:** Dynamic, side-effectful, or unsupported components may not qualify; profile first and verify installed package compatibility.
- **Packages:** `livewire/blaze`

Before:
```php
<x-static-decoration />
{{-- Repeated runtime composition on a measured hot path. --}}
```
After:
```php
{{-- Eligible static component template --}}
@blaze(fold: true)
<span class="decoration">...</span>
```

### Context-aware heading components (Situational)

- **Smell:** Reusable cards hard-code a heading level for styling, breaking document hierarchy in other contexts.
- **Technique:** Make the reusable component accept a heading-level decision from its caller while keeping visual styling independent. Blade props and slots can preserve context-dependent semantic markup; restrict dynamic tag choices to allowed heading levels.
- **When to use:** The same component appears at different levels in a page hierarchy.
- **When not to:** A component used in one fixed structural context may use a fixed heading; unconstrained dynamic tags are unsafe.

Before:
```php
<h2 class="card-title">{{ $title }}</h2>
```
After:
```php
@props(["title", "level" => 2])
@php($tag = "h".(in_array((int) $level, range(1, 6), true) ? (int) $level : 2))
<{{ $tag }} class="card-title">{{ $title }}</{{ $tag }}>
{{-- caller: <x-card-title :level="3" title="Details" /> --}}
```

### HTTP Early Hints (Situational)

- **Smell:** Browsers discover critical assets only after expensive page generation finishes.
- **Technique:** Send preload Link headers in an interim HTTP 103 response before slow rendering, then deliver the final Laravel response. FrankenPHP supplies headers_send; the server and proxy path must preserve the interim response.
- **When to use:** Critical assets are known early and backend rendering is measurably slow.
- **When not to:** Unsupported delivery paths or inaccurate preloads waste bandwidth; cheap responses may gain little.
- **Packages:** `FrankenPHP`

Before:
```php
$data = expensivePageData();
return view("page", compact("data"));
```
After:
```php
header("Link: </build/app.css>; rel=preload; as=style");
headers_send(103);
$data = expensivePageData();
return view("page", compact("data"));
```

### Inline local presentation logic (Situational)

- **Smell:** A one-template formatting operation is scattered into an accessor or presenter used nowhere else.
- **Technique:** Inline simple local display formatting or ordering into the Blade template when extraction adds more ceremony than reuse. Keep business decisions and expensive data loading outside the template.
- **When to use:** A transformation belongs only to one presentation and is easy to understand locally.
- **When not to:** Repeated, complex, or security-sensitive transformations need a reusable and testable boundary.

Before:
```php
// model accessor used by one template only
public function getDisplayDateAttribute() {
    return $this->created_at->format("d M Y");
}
```
After:
```php
{{ $post->created_at->format("d M Y") }}
```

### Isolated Livewire requests (Situational)

- **Smell:** A slow polling or event-listening component holds up other components bundled in the same request.
- **Technique:** Mark the slow Livewire component with Isolate so it makes an independent request. Other component updates can complete without waiting for its work.
- **When to use:** An independently updating component performs slow or frequent work.
- **When not to:** Isolation increases request count and loses the efficiency of bundling.
- **Packages:** `livewire/livewire`

Before:
```php
class SlowMetrics extends Component {
    // Slow polling shares bundled requests with other components.
}
```
After:
```php
#[Isolate]
class SlowMetrics extends Component {
    // Polling updates run in separate requests.
}
```

### Livewire JavaScript bridge (Situational)

- **Smell:** Alpine interactions duplicate AJAX endpoints and response plumbing for existing component actions.
- **Technique:** Call component properties and actions through Alpine’s $wire bridge rather than maintaining a separate transport for that component. Validate and authorize the PHP action as usual.
- **When to use:** Browser code needs to invoke existing Livewire behavior.
- **When not to:** Retain a separate API when mobile apps or external clients also consume the operation.
- **Packages:** `alpinejs`, `livewire/livewire`

Before:
```php
await fetch("/counter/increment", { method: "POST" });
```
After:
```php
<button x-on:click="await $wire.increment()">+</button>
```

### Managed scroll regions (Situational)

- **Smell:** Navigation restores document scrolling but loses position in independently scrolling panels.
- **Technique:** Mark overflow containers with scroll-region so Inertia resets them on visits and restores their positions on history navigation.
- **When to use:** An Inertia page scrolls within a panel rather than the document body.
- **When not to:** Unnecessary when the document is the only scroll container; preservation on new visits is a separate choice.
- **Packages:** `inertiajs/inertia-laravel`

Before:
```php
<main class="overflow-y-auto">...</main>
```
After:
```php
<main scroll-region class="overflow-y-auto">...</main>
```

### Progressive enhancement (Situational)

- **Smell:** Failed JavaScript, CSS or images leave a blank or unusable interface.
- **Technique:** Start from understandable server-rendered semantic HTML and add richer interaction on top. Use Blade for the baseline and provide useful notices where JavaScript is required.
- **When to use:** Core content should survive asset failures.
- **When not to:** Some collaborative or highly interactive features inherently need JavaScript.

Before:
```php
<div id="app"></div>
```
After:
```php
<main>
    <h1>{{ $article->title }}</h1>
    <p>{{ $article->summary }}</p>
</main>
<noscript>Enable JavaScript for live updates.</noscript>
```

### Reactive child props (Situational)

- **Smell:** An extracted child display is kept in sync with ad hoc parent events.
- **Technique:** Mark the child’s input property with Livewire’s #[Reactive] attribute so it tracks parent changes. Keep independent child props as the default.
- **When to use:** A child’s display must follow a changing parent value.
- **When not to:** Reactivity adds synchronization overhead and is unnecessary for stable values.
- **Packages:** `livewire/livewire`

Before:
```php
// Parent emits changes; child listener copies the value.
$this->dispatch("items-changed", items: $this->items);
```
After:
```php
// Child component
#[Reactive]
public $items;
// Parent view
<livewire:item-count :items="$items" />
```

### Relationship-aware admin form fields (Situational)

- **Smell:** Foreign-key options and related table data are assembled manually.
- **Technique:** Bind Filament selects to Eloquent relationship names and display attributes. Use relationship dot notation for columns, letting the Eloquent-backed table handle relationship loading.
- **When to use:** CRUD manages foreign keys and related labels.
- **When not to:** Complex or externally sourced relations may need custom queries and option loading.
- **Packages:** `filament/forms`, `filament/tables`

Before:
```php
$options = Aircraft::pluck("model", "id");
Select::make("aircraft_id")->options($options);
```
After:
```php
Select::make("aircraft_id")
    ->relationship(name: "aircraft", titleAttribute: "model");
TextColumn::make("aircraft.model");
```

### Shared page props (Situational)

- **Smell:** Controllers repeat the same common props and collide with page-specific names.
- **Technique:** Centralize genuinely common data in HandleInertiaRequests::share() and namespace it. Read the shared data through the frontend usePage().props API.
- **When to use:** Authentication summaries or navigation data recur across pages.
- **When not to:** Shared props appear on many requests; large page-specific data belongs in the page controller.
- **Packages:** `inertiajs/inertia-laravel`

Before:
```php
return Inertia::render("A", ["user" => auth()->user()]);
return Inertia::render("B", ["user" => auth()->user()]);
```
After:
```php
// HandleInertiaRequests::share()
return [...parent::share($request),
  "auth" => ["user" => fn () => $request->user()?->only("id", "name")],
];
```

### Vue composables (Situational)

- **Smell:** Clipboard interaction and reactive feedback are copied across components.
- **Technique:** Extract stateful Clipboard API behavior into a useClipboard composable. Each Vue component consumes the returned state and operations while the composable owns the shared implementation.
- **When to use:** Reusable behavior includes reactive state or lifecycle work.
- **When not to:** Stateless utilities need no composable; be explicit about whether state is shared or per call.
- **Packages:** `inertiajs/inertia-laravel`, `vue`

Before:
```php
const copied = ref(false);
async function copy(text) { await navigator.clipboard.writeText(text); copied.value = true; }
```
After:
```php
const { copied, copy } = useClipboard();
```

## Testing

### Selective boundary test doubles (Recommended)

- **Smell:** Global event faking suppresses fixture invariants, while late faking lets setup satisfy assertions.
- **Technique:** Intercept only relevant events with Event::fake([...]) or exceptions, ordering setup according to real fixture dependencies. Assert events from the actual action without suppressing unrelated creation hooks.
- **When to use:** Event-driven behavior needs isolated dispatch assertions.
- **When not to:** Event fakes stop listeners; use real execution when asserting listener outcomes.
- **Packages:** `mockery/mockery`

Before:
```php
Event::fake();
$user = User::factory()->create(); // UUID event may be suppressed.
```
After:
```php
Event::fake([OrderAnalyzed::class]);
$user = User::factory()->create();
$service->analyze($order);
Event::assertDispatched(OrderAnalyzed::class);
```

### Model factories and scenario builders (Recommended)

- **Smell:** Large scenario-specific graphs clutter tests or shared setUp methods.
- **Technique:** Expose a domain-named factory operation such as preparedForReport() that composes prerequisites through native states and relationships. Preserve the ordinary create()/make() interface rather than overriding factory internals.
- **When to use:** Several tests require a complex domain scenario.
- **When not to:** The builder must not hide behavior-relevant values or introduce persistence where make() was expected.
- **Packages:** `pestphp/pest`, `pestphp/pest-plugin-laravel`

Before:
```php
// Every report test repeats graph setup.
$user = User::factory()->create();
$recipe = Recipe::factory()->for($user)->create();
$recipe->participants()->attach($participant);
```
After:
```php
// RecipeFactory
public function preparedForReport(): static {
  return $this->for(User::factory(), "owner")
    ->hasAttached(User::factory()->count(3), [], "participants");
}
$recipe = Recipe::factory()->preparedForReport()->create();
```

### External service adapters and fakes (Recommended)

- **Smell:** Tests mock low-level HTTP calls throughout the application or require real external services.
- **Technique:** Put the external capability behind an interface and bind a real adapter in production and an in-memory fake in tests. Test business behavior through that contract and separately test the real adapter against expected transport semantics.
- **When to use:** Many business tests depend on an external service and need deterministic behavior.
- **When not to:** A fake can drift from the provider; keep contract or adapter tests and avoid interfaces for nonexistent variation.
- **Packages:** `laravel/ai`

Before:
```php
Http::post($providerUrl, $payload);
// Every business test mocks transport details
```
After:
```php
interface SendsPayments { public function send(int $amount): string; }
$this->app->bind(SendsPayments::class, ProviderPayments::class);
// In tests
app()->instance(SendsPayments::class, new InMemoryPayments());
$paymentId = app(SendsPayments::class)->send(100);
```

### Parameterized tests and datasets (Recommended)

- **Smell:** One invalid literal is tested or duplicated tests repeat the same signup flow.
- **Technique:** Use named Pest datasets for representative valid and invalid inputs, sharing datasets across related tests where appropriate. Laravel feature tests execute each dataset row through the same request and assertion flow.
- **When to use:** Input classes share behavior but need more than one representative.
- **When not to:** Datasets should not erase distinctions requiring different setup or expected results.
- **Packages:** `pestphp/pest`

Before:
```php
it("rejects email", fn () =>
  $this->post("/signup", ["email" => "bad"])->assertInvalid("email"));
```
After:
```php
it("rejects email", function ($email) {
  $this->post("/signup", ["email" => $email])->assertInvalid("email");
})->with(["no at" => ["bad"], "no domain" => ["a@"]]);
```

### Reusable test helpers and expectations (Recommended)

- **Smell:** Each caller test repeats all assertions about the same action or event downstream effects.
- **Technique:** Test the operation or event behavior separately, then have caller tests assert that the action was invoked or event dispatched. Laravel event fakes and container replacement for actions make the separation practical.
- **When to use:** Several entry points use the same independently testable behavior.
- **When not to:** Retain integration coverage for important wiring; overly narrow dispatch tests can miss incorrect payloads.
- **Packages:** `pestphp/pest`, `pestphp/pest-plugin-rector`, `rector/rector`

Before:
```php
// Every caller test checks receipt creation, mail, and database side effects.
```
After:
```php
Event::fake([InvoicePaid::class]);
$this->post("/invoices/1/pay")->assertSuccessful();
Event::assertDispatched(InvoicePaid::class);
// Separate tests exercise the real listener effects.
```

### Characterization tests (Useful)

- **Smell:** An untested legacy app is too daunting to cover all at once.
- **Technique:** Start with a framework-backed feature test that requests the homepage and asserts success, then expand coverage from that runnable baseline. This is the talk’s explicit answer to starting tests on an existing application.
- **When to use:** A legacy application needs a manageable entry point for incremental test adoption.
- **When not to:** A successful homepage does not cover business behavior, browser interactions, or deployed infrastructure.
- **Packages:** `pestphp/pest`, `phpunit/phpunit`

Before:
```php
// No automated regression tests.
```
After:
```php
test("homepage responds", function () {
    $this->get("/")->assertOk();
});
```

### Isolated Form Request tests (Useful)

- **Smell:** Large validation matrices are repeatedly exercised through unrelated controller behavior.
- **Technique:** Exercise a Form Request rules and authorization in focused tests with explicit payload, user, and route context. Laravel 13 tests requiring its validator or container must boot the application; retain endpoint coverage for request lifecycle behavior.
- **When to use:** A Form Request has substantial validation or authorization permutations.
- **When not to:** Direct rules tests can miss prepareForValidation, binding, middleware, and controller wiring.
- **Packages:** `livewire/livewire`

Before:
```php
// every validation case repeats the full endpoint setup
$this->post("/posts", ["title" => ""])->assertInvalid("title");
```
After:
```php
$request = new StorePostRequest;
$validator = Validator::make(["title" => ""], $request->rules());
$this->assertTrue($validator->fails());
$this->assertArrayHasKey("title", $validator->failed());
```

### Named factory states (Useful)

- **Smell:** Tests repeat low-level attribute setup for the same business scenario.
- **Technique:** Encode reusable scenario attributes in named model factory state methods. Laravel 13 factories then prepare records with domain-readable calls instead of repeated arrays.
- **When to use:** Many feature tests need the same approved, paid, or expired scenario.
- **When not to:** One-off arbitrary combinations do not need named states; excessive hidden setup makes tests hard to understand.

Before:
```php
Auction::factory()->create(["status" => "ended"]);
```
After:
```php
// AuctionFactory:
public function ended(): static {
    return $this->state(fn () => ["status" => "ended"]);
}
$auction = Auction::factory()->ended()->create();
```

### Notification and mail fakes (Useful)

- **Smell:** Feature tests send real mail or cannot detect a wrong recipient, payload or duplicate mailable.
- **Technique:** Replace mail delivery with Mail::fake(), exercise the action and assert the intended sent or queued mailable with a payload callback. Add assertOutgoingCount() when the contract permits exactly one outgoing message.
- **When to use:** Testing application mail dispatch without transport dependencies.
- **When not to:** A fake does not verify rendered content or provider delivery.

Before:
```php
$this->post("/orders", $data); // Sends mail through real transport.
```
After:
```php
Mail::fake();
$this->post("/orders", $data);
Mail::assertQueued(OrderReceipt::class, fn ($m) => $m->hasTo($buyer->email));
Mail::assertOutgoingCount(1);
```

### Architecture fitness tests (Useful)

- **Smell:** A chosen repository boundary is repeatedly bypassed by controllers.
- **Technique:** Use Pest architecture tests to restrict model usage to the intended repository namespace and inject repositories into callers. Add exemptions for legitimate relationships and binding rather than imposing a blanket Laravel-wide prohibition.
- **When to use:** The application deliberately adopts repositories.
- **When not to:** Laravel supports direct Eloquent use; enforcement can become brittle if the chosen boundary is artificial.
- **Packages:** `deptrac/deptrac`, `pestphp/pest`, `pestphp/pest-plugin-arch`

Before:
```php
return User::where("active", true)->get();
```
After:
```php
arch()->expect("App\Models")
    ->toOnlyBeUsedIn("App\Repositories");
// Controllers inject UserRepository.
```

### Higher-order tests and expectations (Useful)

- **Smell:** Class and method scaffolding obscures small behavioral tests.
- **Technique:** Express named tests as Pest closures while retaining Laravel test-case capabilities where needed. This reduces boilerplate without changing the need for meaningful assertions.
- **When to use:** Small tests benefit from compact behavior descriptions.
- **When not to:** Existing PHPUnit classes may already be clear; syntax conversion alone does not improve coverage.
- **Packages:** `pestphp/pest`

Before:
```php
class HomeTest extends TestCase {
    public function test_home(): void {
        $this->get("/")->assertOk();
    }
}
```
After:
```php
it("loads home", function () {
    $this->get("/")->assertOk();
});
```

### Mutation-guided test strengthening (Useful)

- **Smell:** Coverage is high but tests still pass when meaningful application logic is changed or removed.
- **Technique:** Run Infection against a passing PHPUnit suite, inspect surviving application-code mutations and strengthen assertions or input cases for observable faults. This targets assertion quality rather than merely executing more lines.
- **When to use:** Critical logic has reassuring coverage but uncertain fault detection.
- **When not to:** Mutation runs are costly; equivalent mutations require judgment and a score does not prove all requirements are tested.
- **Packages:** `infection/infection`, `pestphp/pest`

Before:
```php
$this->assertIsInt($calculator->shippingCost($order));
```
After:
```php
$this->assertSame(0, $calculator->shippingCost($freeOrder));
$this->assertSame(500, $calculator->shippingCost($paidOrder));
// Run Infection and inspect surviving mutations.
```

### Snapshot testing (Useful)

- **Smell:** Large expected HTML, JSON or XML strings overwhelm test bodies and diffs.
- **Technique:** Store reviewed expected output in version-controlled snapshot files and compare future output against them. Use format-specific Spatie assertions where appropriate, or Pest snapshots, and review changes before updating the baseline.
- **When to use:** Complex, reasonably stable output is difficult to assert clearly by hand.
- **When not to:** Small outputs are clearer with explicit assertions; dynamic data and incidental formatting cause brittle diffs and baseline review is essential.
- **Packages:** `pestphp/pest`, `spatie/phpunit-snapshot-assertions`

Before:
```php
$this->assertSame($hugeExpectedHtml, $renderedHtml);
```
After:
```php
$this->assertMatchesHtmlSnapshot($renderedHtml);
// Review and commit the generated snapshot.
```

### Package integration fixture application (Useful)

- **Smell:** Package tests lack a real Laravel application in which providers and framework behavior can run.
- **Technique:** Base Laravel-dependent package tests on Orchestra Testbench to boot a minimal application fixture. Load the package provider explicitly and test the actual integration using a Laravel 13-compatible Testbench release.
- **When to use:** A package depends on container bindings, routes, commands, or Eloquent behavior.
- **When not to:** Framework boot adds cost; pure PHP behavior can use ordinary unit tests and version compatibility must be resolved.
- **Packages:** `orchestra/testbench`, `orchestra/workbench`

Before:
```php
class PackageTest extends PHPUnit\Framework\TestCase {
    // no Laravel application to resolve package services
}
```
After:
```php
class PackageTest extends Orchestra\Testbench\TestCase {
    protected function getPackageProviders($app) {
        return [PackageServiceProvider::class];
    }
}
```

### Factory parent recycling (Situational)

- **Smell:** Nested factories create several parent records that should be shared.
- **Technique:** Use factory recycle() to reuse an existing parent across related model creation.
- **When to use:** A test fixture needs one tenant, team, or user across a graph.
- **When not to:** Do not recycle where independent parents are part of the behavior under test.

Before:
```php
$ticket = Ticket::factory()->create();
$comment = Comment::factory()->create();
```
After:
```php
$team = Team::factory()->create();
$ticket = Ticket::factory()->recycle($team)->create();
$comment = Comment::factory()->recycle($team)->create();
```

### Parallel test resource isolation (Situational)

- **Smell:** Parallel tests collide through shared Redis keys, temporary directories, or ambiguous fixture lookups.
- **Technique:** Partition shared cache namespaces and temporary paths by parallel worker token, and pass explicit fixture identities to the test. Laravel supplies separate test databases per process, but other shared resources need equivalent isolation.
- **When to use:** A large suite is being parallelized.
- **When not to:** Independent resources consume more capacity and require cleanup; isolation does not make order-dependent tests correct.

Before:
```php
Cache::put("fixture", $value);
$path = storage_path("tmp/export.csv");
```
After:
```php
$token = ParallelTesting::token();
Cache::put("test:".$token.":fixture", $value);
$path = storage_path("tmp/".$token."/export.csv");
```

### Cached test bootstrap (Situational)

- **Smell:** Unchanged configuration and routes are rebuilt repeatedly during tests.
- **Technique:** Use Laravel’s WithCachedConfig and WithCachedRoutes test traits where configuration/routes stay stable. Clear ordinary production configuration caches before testing so the test environment remains correct.
- **When to use:** Bootstrap work is a measured significant suite cost.
- **When not to:** Tests that mutate configuration or route registration need normal setup.

Before:
```php
class CatalogTest extends TestCase {
    // Every test repeats unchanged bootstrap work.
}
```
After:
```php
class CatalogTest extends TestCase {
    use WithCachedConfig, WithCachedRoutes;
}
```

### Direct policy tests (Situational)

- **Smell:** Authorization cases require repetitive endpoint setup that obscures the policy decision.
- **Technique:** Call policy methods directly with explicit user and resource scenarios. Laravel 13 factories can make unpersisted inputs or create persisted fixtures when database behavior is required.
- **When to use:** A policy has a substantial matrix of permission decisions.
- **When not to:** Direct calls do not verify policy registration, middleware, or endpoint authorization; keep relevant integration tests.

Before:
```php
$this->actingAs($guest)->delete("/posts/{$post->id}")
    ->assertForbidden();
// repeated endpoint setup for every policy branch
```
After:
```php
$user = User::factory()->make();
$post = Post::factory()->make(["user_id" => 999]);
$this->assertFalse((new PostPolicy)->delete($user, $post));
```

### High-fidelity workflow simulators (Situational)

- **Smell:** Isolated tests miss time-dependent interactions across a multi-system pipeline.
- **Technique:** Create a simulator that emits realistic inputs through the actual workflow over time. The talk’s vehicle-location simulator exercises ingestion and dispatch behavior; it complements ordinary feature tests and controlled fakes.
- **When to use:** Complex timed pipelines have meaningful emergent failure modes.
- **When not to:** Simulation is costly and nondeterministic without controlled inputs and timing; small synchronous workflows need simpler tests.

Before:
```php
// Each parser passes isolated fixture tests.
```
After:
```php
foreach ($simulatedRoute as $point) {
    $ingestion->receive($vehicleId, $point);
    $clock->advance($point->elapsedSeconds);
}
assertExpectedDispatchOutcome();
```

### Seeded procedural generation (Situational)

- **Smell:** Randomly generated output cannot be reproduced for debugging or sharing.
- **Technique:** Seed the pseudorandom generator and keep the random-call sequence stable so the same inputs recreate the generated layout. This is a PHP technique usable in Laravel 13 application services.
- **When to use:** Generated levels or other procedural structures need repeatability.
- **When not to:** Changing the algorithm alters output for an old seed and shared generator state can affect other code; this is not cryptographic randomness.

Before:
```php
$split = mt_rand(20, 80);
// the same level input produces different layouts
```
After:
```php
mt_srand($seed);
$split = mt_rand(20, 80);
$level = $generator->fromFirstSplit($split);
```

## Legacy & large-scale refactoring

### Automated AST refactoring (Recommended)

- **Smell:** AI-assisted refactoring drifts across a large set of repeated code changes.
- **Technique:** Provide one completed before/after diff as the refactoring template, apply it to explicit file batches and inspect and commit each batch. Re-establish context when output drifts; the strategy works for Laravel 13 code without requiring a new runtime abstraction.
- **When to use:** A mechanical refactor repeats across many similar files.
- **When not to:** A flawed template propagates mistakes; varied domain logic requires individual judgment and tests.
- **Packages:** `driftingly/rector-laravel`, `nikic/php-parser`, `pestphp/pest`, `pestphp/pest-plugin-drift`, `rector/rector`

Before:
```php
"Refactor all controllers however you think best."
```
After:
```php
"Use this reviewed before/after diff as the template."
"Apply the same transformation only to these five controllers."
# Review diff, run relevant tests, commit, then take the next batch.
```

### Remove dead code (Useful)

- **Smell:** Redundant assignments and unreachable statements accumulate.
- **Technique:** Use Rector dead-code rules to remove mechanically detectable redundant code. Review changes and regression behavior rather than assuming all dynamically invoked Laravel code is unused.
- **When to use:** Repetitive mechanical cleanup spans many files.
- **When not to:** Dynamic dispatch and framework hooks need care; broad unused-class claims exceed this technique.
- **Packages:** `larastan/larastan`, `rector/rector`

Before:
```php
return $result;
$unused = 1;
```
After:
```php
return $result;
// Configure Rector withDeadCodeLevel(0) for an initial reviewed pass.
```

### Wrap method (decorator) (Useful)

- **Smell:** Custom service behavior is duplicated across consumers or implemented by editing framework internals.
- **Technique:** Decorate an existing container service with extend and forward its original operation through an application wrapper. Resolve consumers through that binding so customization stays centralized.
- **When to use:** Cross-cutting behavior belongs around an existing service API.
- **When not to:** Decoration can obscure behavior and depends on consumers using the binding.

Before:
```php
$report = $reporter->send($data);
$audit->record($data);
```
After:
```php
$this->app->extend(Reporter::class,
    fn ($reporter, $app) => new AuditedReporter(
        $reporter, $app->make(Audit::class)
    )
);
```

### Incremental module migration (Useful)

- **Smell:** Moving tangled code into a package would freeze poor boundaries.
- **Technique:** First refactor a coherent capability inside the Laravel application. Extract it into a small Composer package only after the internal boundary and actual reuse become clear.
- **When to use:** Code is being considered for package or service extraction.
- **When not to:** Another package creates release coordination and public-API commitments.

Before:
```php
// Billing logic interleaved with controllers and persistence.
```
After:
```php
App\Billing\InvoiceCalculator
App\Billing\InvoiceCalculatorTest
// Later extract the stable calculator API into a package.
```

### Introduce a seam (Useful)

- **Smell:** A shared abstraction gains switches and special cases for unrelated callers.
- **Technique:** Separate callers into simple implementations when sharing forces incompatible requirements together. Laravel can autowire concrete classes without adding interfaces or bindings immediately.
- **When to use:** New cases continually complicate a common helper.
- **When not to:** Deliberate duplication adds maintenance cost when the behavior really is shared.

Before:
```php
$formatter->format($data, isInvoice: true, legacy: false);
```
After:
```php
$invoiceFormatter->format($invoice);
$statementFormatter->format($statement);
```

### Schema snapshot bootstrap (Useful)

- **Smell:** Test database initialization replays a long migration history before useful tests can run.
- **Technique:** Commit a schema dump for the testing connection and let Laravel load it before subsequent migrations. The speaker presents this as a candidate improvement, so measure initialization time before adopting it.
- **When to use:** Migration replay dominates test setup time.
- **When not to:** Dumps require supported database clients and do not exercise the entire historical migration path.

Before:
```php
php artisan migrate --env=testing
```
After:
```php
php artisan schema:dump --database=testing
# Commit database/schema/testing-schema.sql.
php artisan test
```

### Incremental Livewire adoption (Useful)

- **Smell:** An existing Blade app needs one interactive feature but a whole-stack rewrite is being considered.
- **Technique:** Embed a discrete Livewire component in an existing Blade page and expand adoption only where useful. The talk specifically suggests a small navbar/search component.
- **When to use:** Adding isolated interactivity to an established server-rendered application.
- **When not to:** A fragmented mixture of stacks would cost more to maintain than a consistent existing solution.
- **Packages:** `livewire/livewire`

Before:
```php
<form action="/search"><input name="q"></form>
```
After:
```php
<nav>
  <livewire:navbar-search />
</nav>
```

### Strangler fig migration (Useful)

- **Smell:** A legacy PHP application needs modernization while a complete rewrite would interrupt the business.
- **Technique:** Make Laravel the entry point and route remaining legacy functionality through an isolated fallback bridge. Add specific Laravel routes ahead of the bridge and replace legacy behavior incrementally until the fallback can be removed.
- **When to use:** Legacy endpoints can be replaced gradually while preserving existing operation.
- **When not to:** Two execution models and compatibility shims add temporary complexity; a small disposable system may be easier to replace directly.

Before:
```php
// Web entry point runs legacy.php for every request.
```
After:
```php
Route::get("/reports", ReportController::class);
Route::any("/{path?}", LegacyController::class)
    ->where("path", ".*");
```

### Incremental test-framework migration (Situational)

- **Smell:** Adopting a different test syntax appears to require rewriting an entire PHPUnit suite.
- **Technique:** Run existing PHPUnit tests with Pest and convert individual tests to descriptive closures as useful. Laravel 13 supports both runners, so shared Laravel test-case setup can remain while syntax changes incrementally.
- **When to use:** An existing suite should keep running during gradual Pest adoption.
- **When not to:** Syntax migration alone adds no coverage and Laravel-dependent closures still need the correct test case.
- **Packages:** `pestphp/pest`

Before:
```php
class HomeTest extends Tests\TestCase {
    public function test_home() {
        $this->get("/")->assertOk();
    }
}
```
After:
```php
// configure Tests\TestCase for this test directory
 test("home responds", function () {
    $this->get("/")->assertOk();
});
```

### Batched historical event backfill (Situational)

- **Smell:** Rebuilding millions of historical rows in one pass makes failures and semantic errors hard to detect.
- **Technique:** Translate historical rows into events in small checked batches, replay them and reconcile the results before increasing batch size. Use chunkById or lazyById for bounded Laravel processing and make reruns safe.
- **When to use:** A durable event migration needs to reconstruct existing records.
- **When not to:** Synthesized history may lack original detail; replay overlap can duplicate results unless guarded or rebuilt deliberately.

Before:
```php
foreach (LegacyResponse::all() as $row) backfill($row);
```
After:
```php
LegacyResponse::chunkById(100, function ($rows) {
    foreach ($rows as $row) backfillEvent($row);
    reconcileBatch($rows);
});
```

### Forward-only migration repair (Situational)

- **Smell:** Rollback migrations threaten records created after deployment or depend on obsolete assumptions.
- **Technique:** Repair a deployed schema with a new forward migration instead of relying on a destructive down migration. Use explicit query-builder operations for any necessary data transition and validate the change on representative data.
- **When to use:** A production schema change must preserve post-deployment data.
- **When not to:** Forward repair still requires backups and planning; it does not justify irreversible changes without a recovery path.

Before:
```php
// Rolling back drops data created after release
Schema::dropIfExists("invoice_notes");
```
After:
```php
// New corrective migration
Schema::table("invoice_notes", function (Blueprint $table) {
    $table->nullableMorphs("subject");
});
```

### Legacy HTTP response adapters (Situational)

- **Smell:** Raw header redirects inside legacy scripts bypass Laravel responses and tests.
- **Technique:** Translate legacy redirects into a dedicated exception carrying destination and status, then return a Laravel redirect from the bridge. Preserve the old redirect semantics deliberately while endpoints are being migrated.
- **When to use:** The bridge must retain legacy redirect paths.
- **When not to:** Exception-based flow adds temporary indirection; normal controllers should return redirect responses themselves.

Before:
```php
header("Location: /login", true, 302);
exit;
```
After:
```php
throw new LegacyRedirect("/login", 302);
// Bridge:
catch (LegacyRedirect $redirect) {
    return redirect()->to($redirect->url, $redirect->status);
}
```

### Relational schema normalization (Situational)

- **Smell:** Text-based associations and repeated answer columns make legacy data brittle to update and query.
- **Technique:** Project legacy events into separate questions, answers, responses and scores linked by stable IDs. Express the normalized associations with Eloquent hasMany and belongsTo relationships.
- **When to use:** A rewrite replaces repeated columns or associations based on mutable text.
- **When not to:** Backfill and mapping require careful reconciliation; normalization introduces joins and may not suit every read workload.

Before:
```php
Survey::create(["answer_1" => $a, "answer_2" => $b]);
```
After:
```php
Response::create([
    "user_id" => $userId, "question_id" => $questionId,
    "answer_id" => $answerId,
]);
```

### Rename for domain meaning (Situational)

- **Smell:** Generic names such as list, row, max, or datetime hide business intent.
- **Technique:** Rename variables, loop items, and fields to describe their domain meaning. In Laravel 13, column renames also require updating model, query, and validation references through an appropriate migration.
- **When to use:** The reader must reverse-engineer what a value represents.
- **When not to:** Public or persisted names require coordinated migration; cosmetic renaming creates churn without clarifying intent.

Before:
```php
foreach ($list as $row) {
    $max = $row->datetime;
}
```
After:
```php
foreach ($mapSpots as $mapSpot) {
    $lastVisitedAt = $mapSpot->visited_at;
}
```

### Shadow legacy projections (Situational)

- **Smell:** A rewrite relies on assumptions about old data transformations with no way to verify them.
- **Technique:** Project captured events into separate tables matching the legacy schema and reconcile those rows against the real legacy output. Turn discrepancies into characterization cases before serving users from the new implementation.
- **When to use:** Legacy semantics must be validated without immediately changing production reads.
- **When not to:** Duplicated storage and reconciliation cost time; comparisons need stable IDs and meaningful fields.
- **Packages:** `spatie/laravel-event-sourcing`

Before:
```php
// Rewrite assumes its interpretation is correct.
$newHandler->submit($answer);
```
After:
```php
$eventStore->append($event);
$mirrorProjector->apply($event);
$reconciler->compare($legacyRows, $mirrorRows);
```

### Static-analysis baseline (Situational)

- **Smell:** A legacy application’s existing violations prevent introducing useful analysis on new work.
- **Technique:** Generate a PHPStan baseline for existing errors, include it in analysis configuration, and enforce checks on subsequent changes. Reduce the baseline through ordinary fixes rather than regenerating it to suppress new failures.
- **When to use:** An established codebase has too much analysis debt to fix in one change.
- **When not to:** A huge baseline can hide poor configuration and suppress important existing problems.
- **Packages:** `phpstan/phpstan`

Before:
```php
# Analysis fails on hundreds of pre-existing errors.
vendor/bin/phpstan analyse
```
After:
```php
vendor/bin/phpstan analyse --generate-baseline
# phpstan.neon:
includes:
    - phpstan-baseline.neon
# CI continues running analyse.
```

## Architecture & modules

### Modular monolith (domain organization) (Recommended)

- **Smell:** Backend and Inertia frontend ownership are scattered across technical folders and domains are tightly coupled.
- **Technique:** Give each business domain an end-to-end module with its backend and frontend code, maintaining explicit boundaries within one Laravel application. Configure autoloading, providers, and frontend resolution as needed; modules still share a deployment and runtime.
- **When to use:** Growing teams and distinct business areas justify domain ownership.
- **When not to:** Small applications do not need module infrastructure; modules cannot be independently deployed or scaled like services.
- **Packages:** `livewire/livewire`, `spatie/laravel-data`

Before:
```php
app/Http/Controllers/BookingController.php
resources/js/pages/booking/create.tsx
```
After:
```php
app-modules/booking/
  src/Http/Controllers/BookingController.php
  resources/js/pages/passenger/create.tsx
app-modules/payment/
```

### Inertia server-driven application architecture (Recommended)

- **Smell:** An interactive JavaScript UI duplicates Laravel routing and requires a separate API solely to render pages.
- **Technique:** Keep routing, authentication, validation and controllers in Laravel, and use Inertia to render Vue, React or Svelte pages with server-provided props. The browser replaces page components on visits while the backend remains one application.
- **When to use:** A JavaScript UI and Laravel backend are maintained by the same team.
- **When not to:** A public API or independently deployed clients may still justify a separate backend contract.
- **Packages:** `@inertiajs/react`, `inertiajs/inertia-laravel`

Before:
```php
// Browser loads page, then fetches its API.
const users = await fetch("/api/users").then(r => r.json());
```
After:
```php
return Inertia::render("Users/Index", [
    "users" => User::paginate(),
]);
```

### External integration adapters (Recommended)

- **Smell:** Local PDF or screenshot generation requires maintaining Chrome and Node on application servers.
- **Technique:** Put browser rendering behind a Sidecar-backed Browsershot adapter so callers retain a familiar rendering interface. For Laravel 13, resolve a compatible adapter and supported runtime rather than copying the historical Node 14 setup.
- **When to use:** Browser rendering is a specialized workload whose binaries burden the application host.
- **When not to:** Remote calls add latency and AWS/runtime maintenance; exact adapter compatibility must be checked.
- **Packages:** `hammerstone/sidecar`, `phpunit/phpunit`, `spatie/browsershot`, `stefanzweifel/sidecar-browsershot`, `symfony/routing`

Before:
```php
Browsershot::url($url)->save($path);
// Chrome and Node installed on every app server
```
After:
```php
BrowsershotLambda::url($url)->save($path);
// adapter invokes the packaged remote Chrome function
```

### Feature flags and progressive rollout (Recommended)

- **Smell:** A rewrite is enabled for every user at once, increasing the impact of undiscovered problems.
- **Technique:** Route stable user cohorts through a feature flag such as Pennant, expand exposure after checking correctness and feedback, and keep a rollback override. Explicitly expand stored per-user decisions rather than assuming changed lottery odds reassign existing users.
- **When to use:** A replacement application can coexist with the old implementation.
- **When not to:** Flags add routing and cleanup work; they cannot restore compatibility unless both systems keep their data current.
- **Packages:** `laravel/pennant`

Before:
```php
return $newApplication->handle($request);
```
After:
```php
return Feature::for($user)->active("new-survey")
    ? $newApplication->handle($request)
    : $legacyApplication->handle($request);
```

### Dependency inversion (contracts) (Recommended)

- **Smell:** Consumers reach into another module's internal classes and cannot tolerate implementation changes.
- **Technique:** Expose a small public interface, bind its implementation in a service provider and inject that contract into consumers. Translate incoming data into the receiving module's own domain language behind the boundary.
- **When to use:** Modules need stable collaboration without sharing their internals.
- **When not to:** Contract versioning and translation add work; Laravel does not enforce module visibility automatically.

Before:
```php
$payment = new Modules\Billing\Internal\StripeProcessor();
$payment->charge($order);
```
After:
```php
interface Payments { public function pay(PendingPayment $payment): void; }
$this->app->bind(Payments::class, BillingPayments::class);
// Consumer depends on Payments, not Billing internals.
```

### Dependency injection (Recommended)

- **Smell:** Actions are instantiated directly, making dependencies and test replacement cumbersome.
- **Technique:** Resolve business actions through Laravel constructor or method injection and bind interchangeable implementations in the container. Tests can replace the binding with a fake while the UI remains an orchestrator.
- **When to use:** Actions have dependencies or need controlled replacement in tests or different deployments.
- **When not to:** Do not add interfaces to every trivial class without a substitution need; container resolution can hide dependencies when used as a service locator.
- **Packages:** `inertiajs/inertia-laravel`, `livewire/livewire`

Before:
```php
$action = new SendInvoice(new Mailer());
$action->execute($invoice);
```
After:
```php
public function send(SendsInvoices $action): void {
    $action->execute($this->invoice);
}
// Provider
$this->app->bind(SendsInvoices::class, SendInvoice::class);
// Test
app()->instance(SendsInvoices::class, new FakeInvoiceSender());
```

### Repository extraction (Recommended)

- **Smell:** Persistence callers need actual alternative backends rather than one fixed Eloquent store.
- **Technique:** Define an application repository contract and provide backend implementations behind container bindings. Return domain-appropriate results rather than leaking an Eloquent builder through a persistence-independent interface.
- **When to use:** There is a concrete alternate data source or backend.
- **When not to:** A repository over ordinary Eloquent adds maintenance without portability if its API still exposes Eloquent details.

Before:
```php
$products = Product::where("active", true)->get();
```
After:
```php
interface ProductCatalog { public function active(): array; }
$this->app->bind(ProductCatalog::class, EloquentProductCatalog::class);
$products = $catalog->active();
```

### Container-scoped state (Recommended)

- **Smell:** Middleware and listeners maintain separate audit records or mutable state leaks across workers.
- **Technique:** Bind an auditor contract to a shared execution-scoped collector and inject it into middleware/listeners. Laravel 13 scoped() preserves one collector per request/job and resets it for persistent workers.
- **When to use:** Several execution participants contribute to one audit.
- **When not to:** Request scope alone cannot correlate later queued executions.

Before:
```php
$middlewareAudit = new Auditor();
$listenerAudit = new Auditor();
```
After:
```php
$this->app->scoped(AuditorContract::class, DatabaseAuditor::class);
// Both middleware and listeners inject AuditorContract.
```

### Container-backed facades (Useful)

- **Smell:** A service with injected dependencies is being converted to native static methods for convenient call sites.
- **Technique:** Import the service through the Facades namespace to obtain a real-time facade. Laravel resolves the underlying instance through the container while preserving constructor injection and facade testing helpers.
- **When to use:** A concise facade API is useful for an existing instance service.
- **When not to:** Constructor injection makes dependencies more visible, especially when a class accumulates many services.

Before:
```php
MailerService::send($message);
// Native static implementation loses instance dependencies.
```
After:
```php
use Facades\App\Services\MailerService;
MailerService::send($message);
```

### Contextual dependency injection (Useful)

- **Smell:** Consumers manually choose service implementations or disks inside application logic.
- **Technique:** Declare contextual dependencies with Laravel’s supported container attributes or contextual bindings so the container supplies the correct implementation for that consumer.
- **When to use:** Different consumers need different configurations of the same dependency.
- **When not to:** Container configuration can obscure dependencies if used for simple local values.

Before:
```php
$disk = Storage::disk("s3");
```
After:
```php
public function __construct(
    #[Storage("s3")] Filesystem $disk,
) {
    $this->disk = $disk;
}
```

### Fluent configuration objects (Useful)

- **Smell:** Consumers repeat procedural import steps or must know an abstraction’s internal execution order.
- **Technique:** Let callers describe desired data and constraints while the reusable abstraction owns execution. A Laravel-compatible application is a Filament importer’s column declarations; this illustrates the talk’s reported description-over-instructions principle rather than verified stage code.
- **When to use:** Reusable workflows have a stable structure with varied schemas or constraints.
- **When not to:** A generic declaration language is costly when there is only one simple workflow.
- **Packages:** `filament/actions`

Before:
```php
foreach ($rows as $row) {
  validateEmail($row["email"]);
  $user->email = $row["email"];
  $user->save();
}
```
After:
```php
// Filament Importer subclass
public static function getColumns(): array {
  return [ImportColumn::make("email")
    ->requiredMapping()->rules(["required", "email"])];
}
```

### Code-derived API contracts and documentation (Useful)

- **Smell:** Endpoint shapes and consumer documentation drift across hand-written routes and controllers.
- **Technique:** Describe paths, inputs and responses in an OpenAPI document used as the shared API contract. Implement Laravel 13 validation and response serialization to match that contract; the specification alone does not enforce runtime behavior.
- **When to use:** Multiple API consumers need a stable, reviewable interface.
- **When not to:** Maintaining a separate specification adds synchronization work for a small private API.
- **Packages:** `knuckleswtf/scribe`

Before:
```php
// Controller and consumer disagree about fields.
return $todo->toArray();
```
After:
```php
// openapi.yaml
paths:
  /todos:
    get:
      responses:
        "200":
          description: Todo collection
```

### Hexagonal architecture (Useful)

- **Smell:** Core domain logic needs to outlive framework choices but is intertwined with HTTP, Eloquent, or external SDK calls.
- **Technique:** Put domain operations behind input and output ports and implement Laravel-facing adapters at the application boundary. Bind adapters through the container while the domain uses framework-independent contracts.
- **When to use:** There is a demonstrated need for domain independence and the team accepts the extra layers.
- **When not to:** Ports and adapters add mapping and indirection that ordinary Laravel applications may not need.

Before:
```php
class ApproveInvoice extends Controller {
    public function __invoke(Request $r) { Invoice::find($r->id)->approve(); }
}
```
After:
```php
interface InvoiceStore { public function load(string $id): DomainInvoice; public function save(DomainInvoice $invoice): void; }
class ApproveInvoice {
    public function __construct(private InvoiceStore $store) {}
    public function run(string $id): void {
        $invoice = $this->store->load($id); $invoice->approve();
        $this->store->save($invoice);
    } }
$this->app->bind(InvoiceStore::class, EloquentInvoiceStore::class);
```

### Object and service composition (Useful)

- **Smell:** A component combines product selection and quote calculation, preventing independent reuse.
- **Technique:** Separate retrieval from per-product calculation and inject the calculator into a quote-retrieval service. Resolve the composed service through the container in Livewire boot hooks, controllers or commands.
- **When to use:** The same business workflow must serve multiple entry points or evolve in independent parts.
- **When not to:** Service proliferation and indirect calls add navigation overhead when the workflow is already simple.

Before:
```php
foreach (Product::where("active", true)->get() as $product) {
    $quotes[] = /* inline calculation */;
}
```
After:
```php
class GetProductQuotes {
    public function __construct(private CreateProductQuote $quote) {}
    public function __invoke($search) {
        return Product::where("active", true)->get()
            ->map(fn ($product) => ($this->quote)($product, $search));
    }
}
```

### Serverless function extraction (Useful)

- **Smell:** A deliberate serverless migration leaves tightly coupled work in one indivisible handler.
- **Technique:** Extract autonomous functions with explicit event inputs and wire their API or stream triggers. Laravel events/jobs can define responsibilities internally, while independent cloud deployment and triggers require separate integration.
- **When to use:** Independent execution and deployment are explicit architecture goals.
- **When not to:** Distribution adds latency, contracts and shared dependency failure modes.
- **Packages:** `hammerstone/sidecar`

Before:
```php
function handleEverything($request) {
    createAccount($request);
    buildReport($request);
}
```
After:
```php
CreateAccountFunction::handle($accountEvent);
BuildReportFunction::handle($reportEvent);
// Each function has an explicit cloud trigger.
```

### CQRS (Useful)

- **Smell:** One application path handles state-changing operations and read-only reporting with different dependencies.
- **Technique:** Separate commands that change state from queries that return data. Laravel 13 can resolve command handlers and query objects independently, allowing their storage and testing needs to evolve separately.
- **When to use:** Write workflows and read projections have substantially different complexity.
- **When not to:** Simple CRUD rarely needs this separation; asynchronous projections add consistency concerns.

Before:
```php
$service->updateInvoice($id, $data);
$report = $service->invoiceReport($filters);
```
After:
```php
app(UpdateInvoice::class)->handle($id, $data);
$report = app(InvoiceReportQuery::class)->get($filters);
```

### Capability and segregated interfaces (Useful)

- **Smell:** A calculator depends on a full Eloquent model although it only needs its fee and interest rate.
- **Technique:** Define a small capability interface and have the model implement it, then accept that interface in the calculation service. Bind an implementation only if Laravel must resolve the interface itself; passing a queried model requires no binding.
- **When to use:** Domain calculations should work with substitutes that expose a small capability.
- **When not to:** An interface used by one simple consumer can add indirection; avoid abstracting the entire persistence API.

Before:
```php
function quote(Product $product) { /* needs fee and rate */ }
```
After:
```php
interface QuotableProduct {
    public function fee(): int;
    public function rate(): float;
}
class Product extends Model implements QuotableProduct { /* ... */ }
function quote(QuotableProduct $product) { /* calculate */ }
```

### Graceful dependency degradation (Useful)

- **Smell:** An optional API outage disables an otherwise usable feature or page.
- **Technique:** Catch the relevant dependency failure and serve a previously cached result when its age is acceptable, communicating reduced availability. Keep unaffected application features operational and define what happens when no suitable cached value exists.
- **When to use:** Read-only external data can tolerate a known amount of staleness.
- **When not to:** Stale results can be incorrect for critical decisions; expiry helpers alone do not guarantee outage fallback.

Before:
```php
$weather = $provider->current(); // Failure aborts the page.
```
After:
```php
try {
    $weather = $provider->current();
    Cache::put("weather:last", $weather, 3600);
} catch (ProviderUnavailable $e) {
    $weather = Cache::get("weather:last");
}
// Render unavailable state if no acceptable fallback exists.
```

### HTTP integration request objects (Useful)

- **Smell:** Controllers duplicate remote URLs, HTTP verbs, parameter encoding, and response mapping.
- **Technique:** Create endpoint-specific request classes that connect typed parameters, the HTTP method, and transformation into response DTOs. Keep transport configuration in the integration layer and return application-facing data instead of raw HTTP responses.
- **When to use:** A remote API has several endpoints and callers repeatedly interpret the same response shape.
- **When not to:** Extra classes may not pay off for a single call; preserve access to transport errors and pagination details.
- **Packages:** `saloonphp/saloon`, `spatie/laravel-data`

Before:
```php
$response = Http::get($base."/users", ["q" => $term]);
$name = $response->json("data.0.name");
```
After:
```php
class SearchUsersRequest {
    public function send(SearchParams $params): array {
        $response = Http::get(config("services.remote.url")."/users", $params->toArray());
        $response->throw();
        return array_map(fn ($row) => UserData::fromArray($row), $response->json("data"));
    }
}
```

### Metadata-driven API resource definitions (Useful)

- **Smell:** Administrative CRUD screens repeat forms, tables and persistence plumbing.
- **Technique:** Define a Filament resource around an Eloquent model with declarative schemas and table configuration. On Laravel 13 use current Filament signatures such as Schema and generated schema/table classes, adding domain validation and policies.
- **When to use:** Standard administrative model management.
- **When not to:** Highly bespoke workflows can fight the resource abstraction.
- **Packages:** `api-platform/laravel`, `filament/filament`

Before:
```php
// Repeated Blade forms, controllers and listing templates.
```
After:
```php
public static function form(Schema $schema): Schema {
    return $schema->components([
        TextInput::make("flight_number")->required(),
    ]);
}
```

### Module service providers (Useful)

- **Smell:** Module bindings, routes and views are registered in scattered application bootstrapping code.
- **Technique:** Give each module a service provider that registers its container bindings and loads routes, views and configuration. Register that provider in Laravel's provider configuration so the module actually boots.
- **When to use:** A modular monolith needs cohesive initialization per business context.
- **When not to:** Provider ordering and resource names need care; do not create modules solely to reduce a small route file.

Before:
```php
// One global provider loads every module manually.
```
After:
```php
class CommerceServiceProvider extends ServiceProvider {
    public function boot(): void {
        $this->loadRoutesFrom(__DIR__."/routes.php");
        $this->loadViewsFrom(__DIR__."/views", "commerce");
    }
}
```

### Null object (Useful)

- **Smell:** A forgotten test fake can resolve the real Twitter client and send a tweet.
- **Technique:** Implement a NullTwitter client that satisfies the same interface but performs no external work. Configure it as the safe implementation when tweeting is intentionally disabled.
- **When to use:** An optional integration needs a harmless default in tests or non-production environments.
- **When not to:** Silent no-ops can hide production misconfiguration; use the real binding deliberately in production.

Before:
```php
if ($twitter !== null) $twitter->tweet($text);
```
After:
```php
app()->bind(TwitterClientInterface::class, NullTwitter::class);
$twitter->tweet($text);
```

### Reusable package extraction (Useful)

- **Smell:** Dashboard layouts and storage helpers are copied across applications and tiles cannot be reused independently.
- **Technique:** Extract shared layout and persistence support into a small Laravel package and distribute configurable tiles separately. Providers register the package views, components, and commands.
- **When to use:** Several applications need the same dashboard capabilities.
- **When not to:** Public APIs and dependency compatibility cost more than maintaining a single application component.
- **Packages:** `livewire/livewire`, `spatie/laravel-dashboard`

Before:
```php
// App A and App B duplicate the layout and tile storage.
@include("dashboard.weather")
```
After:
```php
// Shared core supplies layout/storage; tiles supply behavior.
<x-dashboard>
    <livewire:weather-tile city="Ghent" />
</x-dashboard>
```

### Strategy pattern (driver selection) (Useful)

- **Smell:** Application code branches on storage providers and manually constructs interchangeable implementations.
- **Technique:** Use Laravel's manager and driver selection mechanisms to choose an implementation through configuration. For custom session storage, implement SessionHandlerInterface and register it with Session::extend().
- **When to use:** A framework subsystem already provides interchangeable drivers.
- **When not to:** A new driver requires testing and maintenance; this does not imply creating a factory for every application service.

Before:
```php
if ($driver === "redis") $store = new RedisSession();
else $store = new DatabaseSession();
```
After:
```php
// config/session.php
"driver" => env("SESSION_DRIVER", "database"),
// Custom driver registered once via Session::extend(...).
```

### HTTP client macros (Situational)

- **Smell:** Third-party integration calls repeat base URLs, credentials, and resilience configuration.
- **Technique:** Register a named Http macro in a service provider that returns a configured PendingRequest. Callers use the named client so timeouts, retries, authentication, and base URL are defined together.
- **When to use:** Several parts of the app call the same vendor API.
- **When not to:** Retrying non-idempotent requests can duplicate effects; a dedicated adapter is clearer for substantial domain behavior.

Before:
```php
Http::withToken(config("services.vendor.token"))
    ->baseUrl(config("services.vendor.url"))->timeout(10)->get("items");
```
After:
```php
Http::macro("vendor", fn () => Http::withToken(config("services.vendor.token"))
    ->baseUrl(config("services.vendor.url"))->timeout(10)->retry(3, 200));
$response = Http::vendor()->get("items");
```

### Macros and mixins (Situational)

- **Smell:** Repeated helper operations wrap the same macro-capable framework class.
- **Technique:** Register a reusable macro in a provider’s boot method rather than modifying framework code or repeatedly wrapping the operation. Expose Macroable on custom classes when consumers need the same extension mechanism.
- **When to use:** A coherent reusable operation belongs on a supported macro-capable API.
- **When not to:** Runtime methods reduce static discoverability and can collide with future framework methods; ordinary services may be clearer.
- **Packages:** `rector/rector`

Before:
```php
$upper = $items->map(fn ($s) => strtoupper($s));
```
After:
```php
Collection::macro("upper", function () {
    return $this->map(fn ($s) => strtoupper($s));
});
$upper = $items->upper();
```

### Provider fallback chains (Situational)

- **Smell:** A critical external operation fails outright or its exception is silently swallowed.
- **Technique:** Try independent providers in sequence, report failures, then use a local lookup or useful degraded result. Explicitly check or throw on HTTP errors because Laravel’s HTTP client does not throw for 4xx/5xx by default.
- **When to use:** A workflow can tolerate degraded data but needs to continue through external outages.
- **When not to:** Sequential fallbacks increase latency; irreversible operations need safeguards against duplicate execution.
- **Packages:** `laravel/ai`

Before:
```php
try { return $provider->reverse($lat, $lng); }
catch (Throwable $e) { return null; }
```
After:
```php
foreach ([$providerA, $providerB, $localLookup] as $provider) {
    try {
        $result = $provider->reverse($lat, $lng);
        if ($result !== null) return $result;
    } catch (Throwable $e) { report($e); }
}
return "$lat, $lng";
```

### Schema-first GraphQL APIs (Situational)

- **Smell:** Clients need an explicit nested-data contract rather than ad hoc endpoint response shapes.
- **Technique:** Define GraphQL types, fields and entry points in SDL and serve the schema from Laravel with Lighthouse. Queries select read data and mutations expose writes through declared inputs and return types.
- **When to use:** A GraphQL client benefits from typed, selectable nested responses.
- **When not to:** Adds schema and resolver complexity; a simple REST interface may be sufficient.
- **Packages:** `nuwave/lighthouse`

Before:
```php
return ["users" => User::all()->toArray()];
```
After:
```php
type User { id: ID!, name: String! }
type Query { users: [User!]! @all }
```

### Shared service instances (Situational)

- **Smell:** Repeated resolutions reconstruct an expensive service or lose intentionally shared service state.
- **Technique:** Bind the service as a singleton when one worker-wide instance is appropriate. In Laravel 13 use scoped for state that should reset each request or queue job.
- **When to use:** An expensive service is safe to share or several consumers must use the same instance.
- **When not to:** Singletons outlive requests under persistent workers and must not retain request-specific data.

Before:
```php
$this->app->bind(Catalog::class, fn () => new Catalog());
```
After:
```php
$this->app->singleton(Catalog::class, fn () => new Catalog());
// For lifecycle-local state instead:
$this->app->scoped(RequestContext::class);
```

### Vertical slices (Situational)

- **Smell:** Small teams complete frontend and backend layers separately and discover integration problems late.
- **Technique:** Deliver one user feature through its Laravel route, persistence and view before progressing to the next. This changes delivery boundaries without requiring nonstandard directories or a separate framework.
- **When to use:** A small team needs working end-to-end increments.
- **When not to:** Shared foundational changes may still span features; this is a delivery strategy rather than a prescribed folder layout.

Before:
```php
// Milestone 1: every table
// Milestone 2: every controller
// Milestone 3: every page
```
After:
```php
// Slice 1: create invoice
Route::post("/invoices", StoreInvoiceController::class);
// Ship its validation, persistence and view together.
// Slice 2: pay invoice, through the same layers.
```

### AI tool classes with bounded access (Situational)

- **Smell:** An agent’s tool schema and execution logic are mixed into its main loop.
- **Technique:** Extract each tool into a class that describes its inputs and implements its execution. Resolve the tools from Laravel and have the loop delegate calls to the appropriate tool.
- **When to use:** Several tools have independent validation, dependencies, or tests.
- **When not to:** One tiny tool may not justify a registry and class hierarchy.
- **Packages:** `laravel/ai`

Before:
```php
if ($call->name === "read_file") { /* inline filesystem logic */ }
```
After:
```php
$result = $tools[$call->name]->handle($call->arguments);
```

### API boundary for separate clients (Situational)

- **Smell:** Storefront code repeatedly handles raw amounts and currency formatting.
- **Technique:** Represent monetary values with Money objects and centralize formatting and arithmetic around those values. Pass a single monetary value through Laravel application code instead of loose amount and currency primitives.
- **When to use:** Pricing and totals need consistent currency-aware behavior.
- **When not to:** Objects and conversion boundaries add overhead for values with no monetary semantics.
- **Packages:** `nativephp/mobile`

Before:
```php
$label = $currency . " " . number_format($amount / 100, 2);
```
After:
```php
$price = new Money($amount, $currency);
$label = $formatter->format($price);
```

### Configurable package jobs (Situational)

- **Smell:** Consumers need different queue behavior but must fork the package’s hard-coded job.
- **Technique:** Expose a configurable subclass of the default processing job and dispatch that configured class. Consumers adjust queue behavior while inheriting processing logic and honoring the base constructor and execution contract.
- **When to use:** Package users need workload-specific job behavior.
- **When not to:** A simple queue-name option may suffice; a broad subclass API creates compatibility commitments.
- **Packages:** `spatie/laravel-url-ai-transformer`

Before:
```php
ProcessTransformerJob::dispatch($id);
```
After:
```php
$jobClass = config("transformer.process_job", ProcessTransformerJob::class);
$jobClass::dispatch($id);
// Consumer sets a subclass of ProcessTransformerJob.
```

### Configurable package models (Situational)

- **Smell:** Every model customization demands another package setting or a vendor-code edit.
- **Technique:** Expose the default Eloquent model class in package configuration and instantiate or query the configured class consistently. Consumers extend that model for table, connection, or behavior changes, preserving the package’s expected model contract.
- **When to use:** A reusable package needs consumer-controlled persistence details.
- **When not to:** Subclassing couples consumers to the base model; incompatible schema or lifecycle changes need a deliberate extension contract.
- **Packages:** `spatie/laravel-multitenancy`, `spatie/laravel-url-ai-transformer`

Before:
```php
TransformationResult::query()->create($data);
```
After:
```php
$model = config("transformer.model", TransformationResult::class);
$model::query()->create($data);
// Consumer configuration:
// "model" => CustomTransformationResult::class
```

### Custom-resource operators (Situational)

- **Smell:** Built-in infrastructure resources cannot express a managed service’s lifecycle.
- **Technique:** Define a Kubernetes custom resource describing the service and an operator that reconciles it. A Laravel control plane can publish desired data, while CRDs and operators remain external infrastructure components.
- **When to use:** A platform owns domain-specific infrastructure automation.
- **When not to:** Ordinary Laravel applications rarely justify custom Kubernetes operators.

Before:
```php
// Bespoke imperative scripts create and repair service resources.
```
After:
```php
apiVersion: platform.example/v1
kind: ManagedService
metadata:
  name: analytics
spec:
  replicas: 3
# Operator reconciles this resource.
```

### Data mapper (Situational)

- **Smell:** Domain objects must remain independent of persistence but Eloquent models carry both behavior and storage concerns.
- **Technique:** Keep persistence-independent domain objects and map them to database records through dedicated infrastructure code. Bind the mapper or storage port through the Laravel container.
- **When to use:** The domain has a real need to survive persistence or framework changes.
- **When not to:** Mapping duplicates schema knowledge and forfeits much of Eloquent convenience.

Before:
```php
class Invoice extends Model {
    public function approve() { /* domain logic */ }
}
```
After:
```php
class Invoice { public function approve(): void { /* domain logic */ } }
$invoice = $mapper->load($id);
$invoice->approve();
$mapper->save($invoice);
```

### Destructor-backed fluent builder (Situational)

- **Smell:** A pending operation needs several chainable configuration calls without a mandatory terminal dispatch method.
- **Technique:** Accumulate options in a pending object and execute the operation when that object is destroyed, as Laravel’s PendingDispatch does. The speaker explicitly describes reusing this design in his own projects; object retention changes execution timing.
- **When to use:** A small fluent API benefits from automatic terminal execution and its lifetime is predictable.
- **When not to:** Explicit execution is preferable for critical side effects, exception handling, or objects retained across scopes.

Before:
```php
$job = new SendReceipt($orderId);
$job->onQueue("mail");
Bus::dispatch($job);
```
After:
```php
class PendingSend {
    private string $queue = "default";
    public function onQueue(string $queue): self {
        $this->queue = $queue; return $this;
    }
    public function __destruct() { /* send using options */ }
}
(new PendingSend)->onQueue("mail");
```

### Factory and manager pattern (Situational)

- **Smell:** Consumers duplicate provider construction and conditional selection of interchangeable products.
- **Technique:** Centralize configurable product creation behind a factory or manager, leaving consumers to use a common product API. The talk uses Socialite OAuth drivers and session managers; Laravel 13 already provides these managers, so reuse them before creating an application factory.
- **When to use:** Several interchangeable providers need configuration-driven construction.
- **When not to:** One stable concrete dependency often needs only container auto-resolution; a factory adds no value without product variation.
- **Packages:** `laravel/socialite`

Before:
```php
if ($provider === "slack") {
    $client = new SlackOAuth($clientId, $secret, $callback);
}
return $client->redirect();
```
After:
```php
return Socialite::driver("slack")->redirect();
// Credentials are configured in config/services.php.
// Callback: Socialite::driver("slack")->user();
```

### Microservice extraction (Situational)

- **Smell:** A well-defined domain cannot obtain needed team autonomy while tied to the application release boundary.
- **Technique:** Extract that domain into an independently deployable service with a deliberate API or messaging contract. Keep Laravel callers behind an integration boundary and account for network and distributed failure behavior.
- **When to use:** Real team autonomy needs justify a separate deployment and ownership boundary.
- **When not to:** Distributed failures, contracts, observability, and operations are a substantial cost; speculative scale is insufficient.

Before:
```php
$shipment = $shippingModule->dispatch($order);
```
After:
```php
$shipment = $shippingGateway->dispatch($order->id);
// Gateway talks to the independently deployed shipping service.
```

### Replaceable package actions (Situational)

- **Smell:** A package workflow is monolithic and consumers copy it to customize one step.
- **Technique:** Extract workflow steps into small action classes and resolve configured implementations through Laravel container bindings or package configuration. Compose those actions in the workflow so a consumer can replace one operation without copying the rest.
- **When to use:** Package extension points or application variants need replacement of a specific business operation.
- **When not to:** Too many extension points create an unstable public API; retain concrete defaults and document input/output contracts.
- **Packages:** `spatie/laravel-url-ai-transformer`

Before:
```php
function register($data) {
    $user = User::create($data);
    // many more tightly coupled steps
}
```
After:
```php
interface CreatesUsers { public function execute(array $data): User; }
app()->bind(CreatesUsers::class, config("package.create_user_action"));
$user = app(CreatesUsers::class)->execute($data);
```

### Runtime request dependency resolution (Situational)

- **Smell:** Long-lived services retain a previous request, user, or cached user-specific value.
- **Technique:** Pass current request values into operations at runtime or resolve current dependencies through a closure rather than capturing them at worker boot. Use lifecycle-scoped bindings where appropriate.
- **When to use:** The application runs under Octane or another persistent worker.
- **When not to:** Resolver closures hide dependencies; ordinary short-lived stateless objects may need no special lifetime handling.
- **Packages:** `laravel/octane`

Before:
```php
class PriceService {
    public function __construct(private Request $request) {}
    public function quote() { return $this->request->user()->discount; }
}
```
After:
```php
class PriceService {
    public function quote(User $user) { return $user->discount; }
}
$quote = $prices->quote($request->user());
```

### Shared REST and GraphQL resource definitions (Situational)

- **Smell:** REST and GraphQL interfaces duplicate resource shapes and underlying retrieval logic.
- **Technique:** Enable API Platform’s compatible GraphQL integration to reuse existing resource definitions for field-selecting consumers. The separate GraphQL package and configuration belong to API Platform, rather than native Laravel API Resources.
- **When to use:** Consumers need both REST and GraphQL over the same resource vocabulary.
- **When not to:** GraphQL adds schema, authorization and query-cost concerns; REST-only consumers do not justify it.
- **Packages:** `api-platform/graphql`, `api-platform/laravel`

Before:
```php
// Maintain separate REST mapping and GraphQL model schema.
Route::apiResource("books", BookController::class);
```
After:
```php
composer require api-platform/graphql
# Match the API Platform package major and enable GraphQL.
# Reuse existing ApiResource definitions.
```

### Shared object storage (Situational)

- **Smell:** Uploaded files live on a container filesystem and disappear on replacement or return 404 from other replicas.
- **Technique:** Write durable uploads and generated artifacts to a configured shared S3-compatible Laravel disk. Store references to those objects so every replica can retrieve the same file.
- **When to use:** A containerized or replicated application serves durable files.
- **When not to:** Temporary processing files can remain local; object storage adds latency, cost and access configuration.
- **Packages:** `league/flysystem-aws-s3-v3`

Before:
```php
$path = $request->file("image")->store("images", "public");
```
After:
```php
$path = $request->file("image")->store("images", "s3");
$image->update(["path" => $path]);
$url = Storage::disk("s3")->temporaryUrl($path, now()->addMinutes(5));
```

### Tenant-aware runtime context (Situational)

- **Smell:** Business methods repeatedly change database, cache, and filesystem settings themselves.
- **Technique:** Register switching tasks with makeCurrent and forgetCurrent methods to prepare and restore tenant resources. Refresh resolved service instances when configuration changes so ordinary service calls use the active tenant.
- **When to use:** Several resources must change together when tenant context changes.
- **When not to:** Switching mutates process-wide configuration and does not provide simultaneous independent contexts in one process.
- **Packages:** `spatie/laravel-multitenancy`

Before:
```php
config(["database.connections.tenant.database" => $tenant->database]);
DB::purge("tenant");
// Repeated in business operations.
```
After:
```php
"switch_tenant_tasks" => [
    SwitchTenantDatabaseTask::class,
    PrefixCacheTask::class,
    TenantFilesystemTask::class,
],
$tenant->makeCurrent();
// Custom tasks also restore resources in forgetCurrent().
```

## Other

### Concurrent independent tasks (Recommended)

- **Smell:** Independent expensive tasks are serialized or exceed one application server capacity.
- **Technique:** Dispatch a bounded group of independent Lambda executions and collect results or use shared result storage. Laravel 13 needs an external integration for Lambda fan-out; its native Concurrency drivers do not distribute work to Lambda.
- **When to use:** Independent work benefits from remote workers beyond local capacity.
- **When not to:** Quotas, cold starts, result coordination, and per-invocation cost can outweigh the gain.
- **Packages:** `hammerstone/sidecar`, `laravel/octane`, `spatie/fork`

Before:
```php
foreach ($inputs as $input) {
    $results[] = $processor->run($input);
}
```
After:
```php
// application adapter manages a bounded invocation pool
$results = $remotePool->map($inputs, maxConcurrent: 8);
```

### Declarative code scaffolding (Useful)

- **Smell:** Repeated API route and controller scaffolding is manually recreated for each documented resource.
- **Technique:** Generate server scaffolding from the OpenAPI contract, then adapt its dependencies, routes and controllers to Laravel 13. Treat generated output as reviewed starting code because the historical Laravel generator does not establish Laravel 13 compatibility.
- **When to use:** A sizeable stable contract has repetitive endpoint scaffolding.
- **When not to:** Generator limitations, unsupported schema composition or regeneration overwriting custom logic may outweigh the savings.
- **Packages:** `OpenAPI Generator`, `laravel-shift/blueprint`

Before:
```php
// Hand-write another resource controller and routes.
Route::get("/todos", [TodoController::class, "index"]);
```
After:
```php
openapi-generator-cli generate \
  -i openapi.yaml -g php-laravel -o generated
# Review and adapt output to Laravel 13 before use.
```

### HTTP connection reuse (Useful)

- **Smell:** Frequent outbound calls recreate clients and repeatedly pay connection setup costs.
- **Technique:** Reuse a configured Guzzle client across a sequence of requests so its handler can reuse connections. In Laravel, encapsulate the client in an integration service; persistent-worker lifetimes must not retain per-user credentials.
- **When to use:** A process repeatedly calls the same remote host and measurements show connection overhead.
- **When not to:** A shared client with mutable tenant or user state risks leakage; one-off requests gain little.
- **Packages:** `guzzlehttp/guzzle`

Before:
```php
foreach ($urls as $url) {
    (new Client())->get($url);
}
```
After:
```php
$client = new Client(["timeout" => 10]);
foreach ($urls as $url) {
    $client->get($url);
}
```

### Persistent application workers (Useful)

- **Smell:** Repeated framework bootstrapping consumes a significant part of request time.
- **Technique:** Run Laravel Octane with FrankenPHP so application workers bootstrap once and process multiple requests. Audit request/container references, mutable globals, and memory growth because application state lives longer than a request.
- **When to use:** Profiling shows startup overhead matters for the workload.
- **When not to:** Request-isolation assumptions, leaks, or deployment constraints make persistent workers costly to adopt.
- **Packages:** `FrankenPHP`, `laravel/octane`

Before:
```php
// Conventional request starts and boots Laravel each time.
```
After:
```php
// composer require laravel/octane
// php artisan octane:install --server=frankenphp
// php artisan octane:frankenphp
```

### Cleanup stack (undo callbacks) (Situational)

- **Smell:** A workflow acquires multiple temporary resources but duplicates cleanup or misses it on exceptions.
- **Technique:** Register cleanup callbacks as each resource is acquired, then execute them in reverse order in a finally block. Put the stack in a small helper when multiple Laravel workflows share the same resource-management behavior.
- **When to use:** Files, handles, or other temporary resources have dependent cleanup steps.
- **When not to:** One resource usually needs only a simple try/finally; ensure one cleanup failure does not skip remaining cleanup.

Before:
```php
$file = createTempFile();
process($file);
unlink($file);
```
After:
```php
$cleanup = [];
try {
    $file = createTempFile();
    $cleanup[] = fn () => unlink($file);
    process($file);
} finally {
    foreach (array_reverse($cleanup) as $close) { $close(); }
}
```

### Cooperative event-loop integration (Situational)

- **Smell:** Several asynchronous device or stream integrations introduce separate blocking execution paths.
- **Technique:** Use the same active ReactPHP loop for Reverb, non-blocking UDP handlers, and child-process stream callbacks. The talk’s custom Reverb bootstrap depends on package internals, so verify those separately while retaining the shared-loop design.
- **When to use:** Specialized real-time applications multiplex device telemetry and streams.
- **When not to:** Ordinary broadcasts do not need custom server bootstrapping; blocking dependencies defeat the pattern.
- **Packages:** `laravel/reverb`, `react/event-loop`

Before:
```php
$telemetry->blockingReadLoop();
$video->blockingReadLoop();
```
After:
```php
$loop = Loop::get();
// Configure Reverb and UDP integration with this loop.
$loop->addReadStream($videoStdout, function ($stream) {
    publishFrame(fread($stream, 8192));
});
```

### Presigned multipart uploads (Situational)

- **Smell:** Large single-request uploads are fragile and must restart after a failed transfer.
- **Technique:** Use an Uppy S3 multipart uploader with Laravel endpoints that create multipart uploads, sign individual parts, and complete or abort the upload. The client sends parts directly to S3 instead of proxying the file through PHP.
- **When to use:** Files are large enough that parallel parts and retrying individual parts are valuable.
- **When not to:** Multipart uploads add lifecycle endpoints and require cleanup of abandoned parts; small files rarely need them.
- **Packages:** `@uppy/aws-s3`, `@uppy/core`, `aws/aws-sdk-php`

Before:
```php
Storage::disk("s3")->putFile("uploads", $request->file("file"));
```
After:
```php
// Laravel endpoint, using an authorized object key:
$upload = $s3->createMultipartUpload(["Bucket" => $bucket, "Key" => $key]);
// Sign UploadPart commands for the returned UploadId.
// Uppy uploads parts directly and submits their ETags.
// Laravel completes or aborts the multipart upload.
```

### Reusable project starter kits (Situational)

- **Smell:** New projects repeat the same manual scaffold customization and setup scripts.
- **Technique:** Capture recurring defaults in a maintained starter kit with an environment template and Composer project-creation hooks. Laravel 13 supports laravel new --using=vendor/starter-kit for published kits; follow a selected kit’s own instructions when it has installation limitations.
- **When to use:** Multiple projects share meaningful frontend, authentication, or application defaults.
- **When not to:** A single project does not justify template maintenance; stale templates propagate obsolete dependencies and assumptions.

Before:
```php
laravel new app
# Manually repeat the same defaults and setup every time.
```
After:
```php
laravel new app --using=acme/starter-kit
# Kit owns .env.example and Composer post-create-project-cmd.
```

### Scoped state restoration (Situational)

- **Smell:** A helper toggles a global flag and restores an assumed default, breaking nested callers.
- **Technique:** Capture the previous state before changing it and restore that exact value in a finally block. Apply the pattern to temporary framework flags or application context so exceptions and nested calls preserve the caller state.
- **When to use:** A method temporarily changes ambient state it does not own.
- **When not to:** Prefer passing state explicitly when possible; global state is especially delicate in long-lived workers.

Before:
```php
$enabled = false;
runTask();
$enabled = true;
```
After:
```php
$previous = $enabled;
$enabled = false;
try { runTask(); }
finally { $enabled = $previous; }
```

### Self-healing slug URLs (Situational)

- **Smell:** Renamed titles break slug URLs or allow multiple URLs for the same record.
- **Technique:** Include a stable record identifier alongside the descriptive slug and resolve the record by that identifier. Compare the incoming URL with the current canonical URL and redirect stale slugs to the canonical address.
- **When to use:** Public records need readable URLs that survive title changes.
- **When not to:** Extra redirects have a cost; this does not provide authorization or conceal internal IDs.

Before:
```php
Post::where("slug", $slug)->firstOrFail();
```
After:
```php
$post = Post::findOrFail($id);
$canonical = route("posts.show", [
    "id" => $post->id, "slug" => Str::slug($post->title),
]);
if (request()->url() !== $canonical) {
    return redirect($canonical, 301);
}
return view("posts.show", compact("post"));
```

### Buffered stream decoding (Situational)

- **Smell:** Network chunks are treated as complete messages and fail on partial or combined lines.
- **Technique:** For a Laravel NDJSON stream, retain incomplete text in a client buffer and parse only completed lines. Flush the remainder when the stream ends.
- **When to use:** A browser consumes line-framed streamed events.
- **When not to:** Needs limits and error handling for malformed or unbounded input.

Before:
```php
processLine(decoder.decode(value));
```
After:
```php
buffer += decoder.decode(value, { stream: true });
const lines = buffer.split("\n");
buffer = lines.pop() ?? "";
for (const line of lines) processLine(line);
// At EOF:
if (buffer.trim()) processLine(buffer);
```

### Multi-stage asset builds (Situational)

- **Smell:** A production Docker image includes Node and asset-build dependencies solely to compile frontend files.
- **Technique:** Compile Laravel frontend assets in a separate Node builder stage and copy only the generated artifacts into the PHP runtime stage. Keep runtime Node separately when application functionality actually requires it.
- **When to use:** A Laravel container serves prebuilt Vite assets and needs a smaller runtime image.
- **When not to:** SSR or other runtime Node workloads still require Node; multi-stage builds add artifact-path and build-cache coordination.

Before:
```php
FROM php:8.3-fpm
# Install Node and npm into the runtime image.
RUN npm ci && npm run build
```
After:
```php
FROM node:22 AS assets
WORKDIR /app
COPY . .
RUN npm ci && npm run build
FROM php:8.3-fpm
# Install required Laravel PHP extensions and copy prepared app.
COPY --from=assets /app/public/build /app/public/build
```
