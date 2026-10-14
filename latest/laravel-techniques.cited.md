# Laravel Techniques

**Version:** 2026-10-14 12:13 (Asia/Manila)

**Purpose:** refactoring strategies and design patterns that improve Laravel 13 code when a specific smell shows up. A reviewing agent uses this to *suggest* improvements, not to flag violations.

**Levels:** **Recommended** = score 7+ · **Useful** = 4–6 · **Situational** = 2–3 (same source tiers and points as the rules docs; each person or org counted once).

*Each technique lists its score and every independent source (tier, points). Summaries are machine-written from talk notes, lessons, posts and articles; follow the link before quoting.*

## Controllers & business logic

### Action classes (Recommended)

*Score 25:* [Nuno Maduro](https://github.com/nunomaduro/essentials) (Tier 1, 3); [James Brooks](https://x.com/jbrooksuk/status/1928824002144768458) (Tier 1, 3); [Dan Harrin](https://x.com/danjharrin/status/1501598752892669959) (Tier 2, 3); [Luke Downing](https://x.com/LukeDowning19/status/1825535110512341174) (Tier 2, 3); [Mateus Guimarães](https://www.youtube.com/watch?v=WqL6wgH1fKs) (Laracon speaker, 2); [Lucas Giovanny](https://www.youtube.com/watch?v=33CIhvkAPEY&t=1290s) (Laracon speaker, 2); [Mary Perry](https://www.youtube.com/watch?v=vsxoaTgtyjw) (Laracon speaker, 2); [Josh Hanley](https://www.youtube.com/watch?v=esD6o8HYV5Q&t=1044s) (Laracon speaker, 2); [Spatie](https://github.com/spatie/laravel-queueable-action/blob/e28cf997d9cc1b1e1fd587aad0e292fa84498e9b/README.md) (Tier 2, 2); [Jeffrey Way](https://laracasts.com/episodes/3906) (Tier 3, 1); [Simon Vrachliotis](https://laracasts.com/episodes/3667) (Tier 3, 1); [Jeremy McPeak](https://laracasts.com/episodes/4128) (Tier 3, 1); [Laravel News](https://laravel-news.com/structuring-laravel-apps) (Supporting, 0)

- **Smell:** Controllers or Livewire components contain long business workflows that other entry points duplicate.
- **Technique:** Extract a single business operation into a class with execute or __invoke, accepting plain values or DTOs. Let controllers and Livewire components validate, authorize, and manage UI responses while the action coordinates persistence and domain behavior.
- **When to use:** An operation is reused from HTTP, jobs, commands, or multiple components.
- **When not to:** A trivial CRUD operation may not justify a class; action proliferation can fragment a simple workflow.
- **Packages:** `filament/actions`, `filament/tables`, `livewire/livewire`, `spatie/laravel-queueable-action`

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

### Convention-based route discovery (Recommended)

*Score 10:* [Aaron Francis](https://aaronfrancis.com/2024/file-based-routing-in-inertiajs-c40ed2db) (Tier 1, 3); [Taylor Otwell](https://www.youtube.com/watch?v=1P3wLy49t2c&t=1740s) (Laracon speaker (employee), 3); [Spatie](https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=28080s) (Tier 2, 2); [Ryuta Hamasaki](https://www.youtube.com/watch?v=X6xPG6DJibg) (Laracon speaker, 2)

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

*Score 10:* [Nuno Maduro](https://x.com/enunomaduro/status/1976706189242347565) (Tier 1, 3); [Luke Downing](https://www.youtube.com/watch?v=GERCwd65O-8&t=660s) (Laracon speaker, 2); [Jason McCreary](https://www.youtube.com/watch?v=Jna0DOgnxds&t=1050s) (Laracon speaker, 2); [Spatie](https://github.com/spatie/guidelines-skills/blob/a9c9d1a3174c1b1693c025f82c71dac4601de4b2/resources/boost/skills/spatie-laravel-php/references/spatie-laravel-php-guidelines.md) (Tier 2, 2); [Jeffrey Way](https://laracasts.com/episodes/3879) (Tier 3, 1); [Laravel News](https://laravel-news.com/laravel-route-organization-tips) (Supporting, 0)

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

### Middleware composition (Recommended)

*Score 9:* [@laravel](https://laravel.com/docs/13.x/ai-sdk#middleware) (Tier 2, 3); [Bobby Bouwmann](https://www.youtube.com/watch?v=-jo6QjeaJXQ&t=300s) (Laracon speaker, 2); [Mary Perry](https://www.youtube.com/watch?v=w7jKcbVcI54) (Laracon speaker, 2); [Jeremy McPeak](https://laracasts.com/episodes/4046) (Tier 3, 1); [@joelclermont](https://masteringlaravel.io/daily/2026-09-02-republished-sharing-redirect-logic-between-controller-actions) (Tier 3, 1)

- **Smell:** Controllers repeat request filtering and before/after processing across endpoints.
- **Technique:** Extract reusable request behavior into Laravel middleware and compose it through the middleware stack. Each layer can reject early or call $next and process the returned response; register application middleware in bootstrap/app.php.
- **When to use:** Multiple endpoints share request/response concerns.
- **When not to:** Business operations tied to one use case may belong in that use case rather than a global request layer.
- **Packages:** `laravel/ai`, `laravel/pennant`

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

### Pipelines (Recommended)

*Score 9:* [@AshAllenDesign](https://ashallendesign.co.uk/blog/laravel-pipeline) (Tier 2, 3); [@laravel](https://laravel.com/docs/13.x/queries#query-pipes) (Tier 2, 3); [Bobby Bouwmann](https://www.youtube.com/watch?v=ru-0QSciNvU) (Laracon speaker, 2); [aschmelyun](https://x.com/aschmelyun/status/1957040459978539313) (Tier 3, 1)

- **Smell:** A long workflow mixes many ordered responsibilities in one method.
- **Technique:** Extract named, focused pipe classes and compose them with Laravel 13 Pipeline::send()->through()->thenReturn(). Continuing pipes accept the payload and continuation and return $next($payload); container resolution supports injected dependencies.
- **When to use:** Several meaningful stages are hard to maintain or reuse.
- **When not to:** One- or two-step flows are clearer directly; ordering dependencies remain even after extraction.
- **Packages:** `laravel/framework`

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

### Application services (Recommended)

*Score 7:* [@PovilasKorop](https://laraveldaily.com/post/laravel-service-classes-injection) (Tier 2, 3); [Duilio Palacios](https://www.youtube.com/watch?v=1sixoOQliZI&t=510s) (Laracon speaker, 2); [Caleb Porzio](https://laracasts.com/episodes/4089) (Tier 3, 1); [Jeffrey Way](https://laracasts.com/episodes/2306) (Tier 3, 1); [Laravel News](https://laravel-news.com/controller-refactor) (Supporting, 0)

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

### Guard clauses (Recommended)

*Score 7:* [Jason McCreary](https://www.youtube.com/watch?v=sBpzYsMgLK8&t=401s) (Laracon speaker, 2); [Duilio Palacios](https://www.youtube.com/watch?v=1sixoOQliZI&t=1410s) (Laracon speaker, 2); [Spatie](https://github.com/spatie/guidelines-skills/blob/a9c9d1a3174c1b1693c025f82c71dac4601de4b2/resources/boost/skills/spatie-laravel-php/references/spatie-laravel-php-guidelines.md) (Tier 2, 2); [Jeremy McPeak](https://laracasts.com/episodes/3999) (Tier 3, 1)

- **Smell:** Nested prerequisite checks bury the successful calculation path in if/else blocks.
- **Technique:** Return early when prerequisites are absent, using null or an empty collection as the established absence result. Keep the main calculation or rendering path flat after those checks.
- **When to use:** Optional search input or missing calculations create nested control flow.
- **When not to:** Do not use a silent empty result where missing data should be an error; consumers must understand the absence contract.

Before:
```php
if ($calculation) {
    return $quotes->get($calculation);
} else {
    return collect();
}
```
After:
```php
if (! $calculation) return collect();
return $quotes->get($calculation);
```

### HTTP-independent application operations (Useful)

*Score 5:* [Nuno Maduro](https://x.com/enunomaduro/status/1530182783079002113) (Tier 1, 3); [Paul Dragoonis](https://x.com/dr4goonis/status/1647246295914930176) (Tier 2, 2)

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

### Named exception factories (Useful)

*Score 5:* [Wendell Adriel](https://x.com/wendell_adriel/status/1948761110468804821) (Tier 1, 3); [Spatie](https://freek.dev/3091-exceptions-that-explain-themselves) (Tier 2, 2)

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

*Score 4:* [Jason McCreary](https://www.youtube.com/watch?v=sBpzYsMgLK8&t=234s) (Laracon speaker, 2); [Ryan Chandler](https://www.youtube.com/watch?v=cucIWpAenro&t=5098s) (Laracon speaker, 2)

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

### Domain exceptions (Useful)

*Score 4:* [Spatie](https://freek.dev/3091-exceptions-that-explain-themselves) (Tier 2, 2); [Steve McDougall](https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=22800s) (Laracon speaker, 2)

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

### Exception-local HTTP rendering (Situational)

*Score 3:* [Ryan Chandler](https://www.youtube.com/watch?v=cucIWpAenro&t=5098s) (Laracon speaker, 2); [Jeremy McPeak](https://laracasts.com/episodes/3212) (Tier 3, 1)

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

### Explicit dispatch tables (Situational)

*Score 3:* [@AshAllenDesign](https://ashallendesign.co.uk/blog/php-dynamic-method-call-dangers) (Tier 2, 3)

- **Smell:** Handler names are constructed at runtime, hiding usages from static analysis and refactoring tools.
- **Technique:** Enumerate supported cases with direct method calls and an explicit unsupported-case failure.
- **When to use:** An application handles a known finite event set.
- **When not to:** A framework intentionally supports an open-ended dynamically registered handler system.

Before:
```php
$this->{"handle".ucfirst($payload["event"])."Webhook"}($payload);
```
After:
```php
match ($payload["event"]) {
    "success" => $this->handleSuccessWebhook($payload),
    "failure" => $this->handleFailureWebhook($payload),
    default => throw new InvalidArgumentException("Unsupported event"),
};
```

### Extract method (Situational)

*Score 3:* [Christoph Rumpel](https://laracasts.com/episodes/2882) (Tier 1, 3)

- **Smell:** Templates or component methods repeat duration formatting details.
- **Technique:** Move duration formatting into an intention-revealing method and call it where needed. Keep the method close to the data or presentation object that owns the behavior.
- **When to use:** Repeated calculations distract from the feature’s intent.
- **When not to:** A helper for a single obvious expression can make code harder to navigate.

Before:
```php
$text = floor($seconds / 60).":".str_pad($seconds % 60, 2, "0", STR_PAD_LEFT);
```
After:
```php
$text = $video->durationForHumans();
```

### Feature policy classes (Situational)

*Score 3:* [@laravel](https://laravel.com/docs/13.x/pennant) (Tier 2, 3)

- **Smell:** Controllers duplicate conditional rollout logic across endpoints.
- **Technique:** Move initial eligibility into a Pennant feature resolve method and query the feature class from callers.
- **When to use:** A rollout policy is reused or needs injected dependencies.
- **When not to:** The condition is a permanent authorization rule rather than a rollout flag.
- **Packages:** `laravel/pennant`

Before:
```php
$enabled = $user->isInternalTeamMember()
    || $user->id % 100 === 0;
return $enabled ? $newApi() : $legacyApi();
```
After:
```php
class NewApi {
    public function resolve(User $user): mixed {
        return $user->isInternalTeamMember() ?: Lottery::odds(1, 100);
    }
}
return Feature::when(NewApi::class, $newApi, $legacyApi);
```

### Policy filters (Situational)

*Score 3:* [Stephen Rees-Carter](https://x.com/valorin/status/1836497927671349380) (Tier 2, 3)

- **Smell:** Every policy method repeats the same shared authorization check.
- **Technique:** Put the shared decision in the policy before method and return null when the method-specific checks should continue. Laravel 13 invokes this filter before the individual ability method.
- **When to use:** Several abilities share a truly universal grant or denial condition.
- **When not to:** An unconditional grant can bypass important per-resource checks; return null rather than false when continuing.

Before:
```php
public function update(User $user, Post $post): bool {
    return $user->is_admin || $user->id === $post->user_id;
}
// is_admin repeats in delete(), restore(), ...
```
After:
```php
public function before(User $user, string $ability): ?bool {
    return $user->is_admin ? true : null;
}
public function update(User $user, Post $post): bool {
    return $user->id === $post->user_id;
}
```

### Route groups and route-file modularization (Situational)

*Score 2:* [Spatie](https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=28410s) (Tier 2, 2)

- **Smell:** Related endpoints repeat common prefixes or middleware.
- **Technique:** Wrap explicit or discovered route registration in a Laravel route group. Share URI prefixes and middleware while leaving individual route behavior inside the group.
- **When to use:** A section has consistent routing metadata.
- **When not to:** Deeply nested groups obscure effective metadata; a URI prefix alone is not authorization.
- **Packages:** `spatie/laravel-route-discovery`

Before:
```php
Route::get("/admin/reports", $reports)->middleware("auth");
Route::get("/admin/stats", $stats)->middleware("auth");
```
After:
```php
Route::prefix("admin")->middleware("auth")->group(function () {
    Discover::controllers()->in(app_path("Http/Controllers/Admin"));
});
```

## Models & domain state

### State pattern (state classes) (Recommended)

*Score 9:* [@brendt_gd](https://stitcher.io/blog/php-enum-style-guide) (Tier 2, 3); [Joshua Alphonse](https://www.youtube.com/watch?v=HhcieSa52SQ) (Laracon speaker, 2); [Jake Bennett](https://www.youtube.com/watch?v=1A1xFtlDyzU) (Laracon speaker, 2); [Daniel Coulbourne](https://www.youtube.com/watch?v=-mb2L97Yq88&t=1170s) (Laracon speaker, 2)

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

### Custom Eloquent collections (Recommended)

*Score 7:* [Christoph Rumpel](https://christoph-rumpel.com/2024/9/all-laravel-php-attributes-at-your-disposal) (Tier 1, 3); [Tim MacDonald](https://x.com/timacdonald87/status/1490212109656858627) (Tier 1, 3); [laravel/boost](https://github.com/laravel/boost/blob/d165b9ea6086294a8f1999ac3dcf4e61da267b23/.ai/laravel/skill/laravel-best-practices/rules/collections.md) (Tier 3, 1)

- **Smell:** Callers repeat operations on sets of domain models using anonymous collection callbacks.
- **Technique:** Extract the set-level behavior into an Eloquent Collection subclass and return it through the model newCollection method. Domain-named collection methods make callers express intent rather than repeat filtering or aggregation.
- **When to use:** Several callers manipulate the same kind of model collection in the same way.
- **When not to:** Keep database filtering in queries for large sets; not every one-off callback needs a named method.
- **Packages:** `laravel/framework`

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

### Event sourcing (Useful)

*Score 6:* [Matt Stauffer](https://speakerdeck.com/mattstauffer/abstracting-too-early) (Laracon speaker, 2); [Daniel Coulbourne](https://www.youtube.com/watch?v=-mb2L97Yq88&t=450s) (Laracon speaker, 2); [Josh Hanley](https://www.youtube.com/watch?v=evEU7ZAROlU) (Laracon speaker, 2)

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

### Reusable bootable model traits (Useful)

*Score 6:* [Christoph Rumpel](https://x.com/christophrumpel/status/1613516027702435841) (Tier 1, 3); [Bobby Bouwmann](https://www.youtube.com/watch?v=-jo6QjeaJXQ&t=750s) (Laracon speaker, 2); [Luke Downing](https://laracasts.com/episodes/3055) (Tier 3, 1)

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

### Scoped creation defaults (Useful)

*Score 6:* [Taylor Otwell](https://x.com/taylorotwell/status/1890862781836173768) (Tier 1, 3); [@laravel](https://laravel.com/docs/13.x/eloquent-relationships#scoped-relationships) (Tier 2, 3); [Laravel News](https://laravel-news.com/laravel-13-35-0) (Supporting, 0)

- **Smell:** A relationship or scope filters a category but callers must separately repeat its attribute values when creating records.
- **Technique:** Use withAttributes on the Eloquent relationship or scope to apply both query constraints and creation defaults. New models created through that builder inherit the same attributes.
- **When to use:** Records created through a filtered relationship must belong to that category.
- **When not to:** Use ordinary where clauses when a query filter should not influence new records.
- **Packages:** `laravel/framework`

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

### Custom Eloquent relationship adapters (Useful)

*Score 5:* [Lucas Giovanny](https://www.youtube.com/watch?v=33CIhvkAPEY&t=1680s) (Laracon speaker, 2); [Chris Morrell](https://www.youtube.com/watch?v=qLC04_BPQTY) (Laracon speaker, 2); [Instructor Team](https://laracasts.com/episodes/3702) (Tier 3, 1)

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

### Custom pivot models (Useful)

*Score 5:* [@filamentphp](https://github.com/filamentphp/filament/blob/7c5c222be3e92439357213f8274d1548c8cb769e/packages/forms/docs/12-repeater.md) (Tier 2, 3); [Shruti Balasa](https://laracasts.com/episodes/4019) (Tier 3, 1); [Jonathan Reinink](https://laracasts.com/episodes/2655) (Tier 3, 1)

- **Smell:** Multiple Filament panels duplicate branding and configuration.
- **Technique:** Move common panel configuration into a BasePanelProvider and let concrete providers add their own settings. Keep shared behavior in one place.
- **When to use:** Admin and user panels share a meaningful configuration baseline.
- **When not to:** Inheritance can couple panels that should evolve independently.
- **Packages:** `filament/filament`, `filament/forms`

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

### Rich domain models (Useful)

*Score 4:* [Jason McCreary](https://laracasts.com/episodes/3637) (Tier 3, 1); [Mateus Guimarães](https://laracasts.com/episodes/3592) (Tier 3, 1); [Jeffrey Way](https://laracasts.com/episodes/2303) (Tier 3, 1); [Instructor Team](https://laracasts.com/episodes/3705) (Tier 3, 1)

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

### Array-backed Eloquent models (Situational)

*Score 3:* [Christoph Rumpel](https://laracasts.com/episodes/3062) (Tier 1, 3)

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

*Score 3:* [Taylor Otwell](https://x.com/taylorotwell/status/1757436932185825452) (Tier 1, 3)

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

### Derived time-based visibility (Situational)

*Score 3:* [@brendt_gd](https://stitcher.io/blog/reducing-code-motion) (Tier 2, 3)

- **Smell:** A cron job, console command and scheduled state exist only to make content visible at a time.
- **Technique:** Store a future publication timestamp and select published records whose timestamp has passed.
- **When to use:** The transition only changes visibility and does not need to trigger external effects.
- **When not to:** Publication must emit notifications, audit transitions, reserve scarce capacity, or run other reliable one-time work.

Before:
```php
Post::where("state", "scheduled")->where("publish_at", "<=", now())
    ->update(["state" => "published"]);
```
After:
```php
Post::create(["state" => "published", "publish_at" => $nextSlot]);
$visible = Post::where("state", "published")
    ->where("publish_at", "<=", now())->get();
```

### Encapsulated domain invariants (Situational)

*Score 3:* [Mateus Guimarães](https://laracasts.com/episodes/3599) (Tier 3, 1); [Jeffrey Way](https://laracasts.com/episodes/3413) (Tier 3, 1); [@matthiasnoback](https://matthiasnoback.nl/2022/04/ddd-entities-and-orm-entities/) (Tier 3, 1)

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

### Property hooks (Situational)

*Score 3:* [Caleb Porzio](https://www.youtube.com/watch?v=M60-nxlrePc&t=840s) (Laracon speaker, 2); [Jeffrey Way](https://laracasts.com/episodes/3434) (Tier 3, 1)

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

### Relationship-based persistence (Situational)

*Score 3:* [Povilas Korop](https://www.youtube.com/watch?v=olvqO33PBI0) (Laracon speaker, 2); [Jeffrey Way](https://laracasts.com/episodes/3885) (Tier 3, 1)

- **Smell:** Parent/child forms manually manage adding, removing and saving many repeated line-item controls.
- **Technique:** Define a Filament Repeater with a child schema and configure its HasMany relationship to load and save related records. A relationship-aware repeater centralizes the repeated form structure.
- **When to use:** Invoice lines or other modest editable child collections belong inside a parent form.
- **When not to:** Very large collections need a separate interface; BelongsToMany pivot rows require explicit pivot-model handling.
- **Packages:** `filament/forms`

Before:
```php
@foreach ($invoice->items as $item)
  <input name="items[{{ $item->id }}][description]">
@endforeach
// Manually synchronize created/updated/deleted children.
```
After:
```php
Repeater::make("items")->relationship()->schema([
  TextInput::make("description")->required(),
  TextInput::make("quantity")->numeric()->required(),
]);
```

### Reservable models (Situational)

*Score 3:* [Aaron Francis](https://aaronfrancis.com/2024/reservable-models-in-laravel-990d6e9e) (Tier 1, 3)

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

### Semantic cast comparison (Situational)

*Score 3:* [@laravel](https://laravel.com/docs/13.x/eloquent-mutators#comparing-cast-values) (Tier 2, 3)

- **Smell:** Equivalent custom-cast values trigger unnecessary database updates.
- **Technique:** Implement ComparesCastableAttributes in the cast and compare canonical representations.
- **When to use:** Domain equality differs from raw stored representation.
- **When not to:** Use strict comparison when distinct representations are meaningfully different.
- **Packages:** `laravel/framework`

Before:
```php
// Equivalent Money encodings repeatedly mark the field dirty.
$invoice->amount = $equivalentAmount;
$invoice->save();
```
After:
```php
// In a cast implementing ComparesCastableAttributes:
public function compare(Model $model, string $key, mixed $a, mixed $b): bool {
    return Money::fromStorage($a)->equals(Money::fromStorage($b));
}
```

### Team-owned resources (Situational)

*Score 3:* [Julien Bourdeau](https://x.com/julienbourdeau/status/1771398329793957944) (Tier 1, 3)

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

### Aggregate partials (Situational)

*Score 2:* [Spatie](https://github.com/spatie/laravel-event-sourcing/blob/6188940471c1a3b82e4135ce623e819cfd262a92/docs/advanced-usage/aggregate-partials.md) (Tier 2, 2)

- **Smell:** An aggregate root grows into a large class with unrelated groups of state and event-application methods.
- **Technique:** Move a cohesive group into AggregatePartial and delegate public root commands to it without splitting the event stream.
- **When to use:** One consistency boundary remains appropriate but its implementation has become hard to navigate.
- **When not to:** Independent consistency boundaries that should be separate aggregates, or a small aggregate with little internal complexity.
- **Packages:** `spatie/laravel-event-sourcing`

Before:
```php
class Cart extends AggregateRoot {
    private array $items = [];
    // Item commands and apply methods crowd the root.
}
```
After:
```php
class Cart extends AggregateRoot {
    protected CartItems $items;
    public function __construct() { $this->items = new CartItems($this); }
    public function addItem($id, $product, $amount): self {
        $this->items->addItem($id, $product, $amount);
        return $this;
    }
}
```

### Authorization relation invalidation (Situational)

*Score 2:* [Spatie](https://github.com/spatie/laravel-permission/blob/c0fba2df0376f8a7b84711da4e343d57dee0a0c0/docs/basic-usage/teams-permissions.md) (Tier 2, 2)

- **Smell:** Changing the active team leaves a user's eagerly loaded roles from the old team in memory.
- **Technique:** Switch the permission team and clear both authorization relations before evaluating capabilities.
- **When to use:** Team switches or administration of another team within one request.
- **When not to:** A system that does not enable team permissions.
- **Packages:** `spatie/laravel-permission`

Before:
```php
setPermissionsTeamId($teamId);
$allowed = $user->can('edit articles');
```
After:
```php
setPermissionsTeamId($teamId);
$user->unsetRelation('roles')->unsetRelation('permissions');
$allowed = $user->can('edit articles');
```

### Model media collections (Situational)

*Score 2:* [Spatie](https://spatie.be/blog/five-levels-of-handling-images-in-laravel) (Tier 2, 2)

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

*Score 2:* [Luke Downing](https://laracasts.com/episodes/3097) (Tier 3, 1); [Jeremy McPeak](https://laracasts.com/episodes/3764) (Tier 3, 1)

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

### Projection replay (Situational)

*Score 2:* [Daniel Coulbourne](https://www.youtube.com/watch?v=-mb2L97Yq88&t=570s) (Laracon speaker, 2)

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

### Scoped conversation history (Situational)

*Score 2:* [Jeffrey Way](https://laracasts.com/episodes/4077) (Tier 3, 1); [Jeremy McPeak](https://laracasts.com/episodes/4038) (Tier 3, 1)

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

*Score 2:* [Edward](https://x.com/MaybeEdward/status/1695244065720123396) (Tier 2, 2)

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

### State-transition graphs (Situational)

*Score 2:* [Spatie](https://github.com/spatie/laravel-model-states/blob/8507168651fb4229f8cb71e1b7c457f82988d95a/docs/working-with-transitions/01-configuring-transitions.md) (Tier 2, 2)

- **Smell:** Different callers assign status strings directly and allow invalid lifecycle moves.
- **Technique:** Cast the state to a State class, configure allowed edges in StateConfig, and request transitions through transitionTo.
- **When to use:** Business entities with nontrivial lifecycle restrictions.
- **When not to:** A static label with no transition rules or state-specific behavior; an enum is usually sufficient.
- **Packages:** `spatie/laravel-model-states`

Before:
```php
if ($payment->status === 'pending') {
    $payment->update(['status' => 'paid']);
}
```
After:
```php
// In PaymentState::config():
return parent::config()
    ->allowTransition(Pending::class, Paid::class);
// At the application boundary:
$payment->state->transitionTo(Paid::class);
```

### State-transition objects (Situational)

*Score 2:* [Spatie](https://github.com/spatie/laravel-model-states/blob/5fe3c004c3d33c8984db07b208e30e1f40c42c0e/docs/working-with-transitions/02-custom-transition-classes.md) (Tier 2, 2)

- **Smell:** Repeated controllers change state, failure timestamps, and failure messages independently and drift apart.
- **Technique:** Centralize the complete state change in a registered transition class and pass contextual values through transitionTo.
- **When to use:** A lifecycle move owns additional persistent changes.
- **When not to:** A state change consisting only of assigning the next state; use the default transition.
- **Packages:** `spatie/laravel-model-states`

Before:
```php
$payment->state = new Failed($payment);
$payment->failed_at = now();
$payment->error_message = $reason;
$payment->save();
```
After:
```php
// Register PendingToFailed for the Pending -> Failed edge.
$payment->state->transitionTo(Failed::class, $reason);
// PendingToFailed::handle() owns all related field updates.
```

### Translated model attributes (Situational)

*Score 2:* [Spatie](https://github.com/spatie/laravel-translatable/blob/1ad5f0680ed31fcf0023ef81aa55e2af9debb62f/docs/installation-setup.md) (Tier 2, 2)

- **Smell:** Adding each language requires a schema change and repeated title_en/title_fr selection logic.
- **Technique:** Store a locale map in one JSON attribute and use HasTranslations for locale-aware access.
- **When to use:** A bounded set of model attributes needs translations and JSON storage fits the query model.
- **When not to:** A translation domain requiring independent approval workflows, large searchable text documents, or relational reporting per translation.
- **Packages:** `spatie/laravel-translatable`

Before:
```php
// Schema and access grow with every language.
$title = $locale === 'fr' ? $post->title_fr : $post->title_en;
```
After:
```php
class Post extends Model {
    use HasTranslations;
    public array $translatable = ['title'];
}
$post->getTranslation('title', $locale);
```

## Data & types

### Data transfer objects (Recommended)

*Score 31:* [Patricio](https://x.com/PatricioOnCode/status/1933089793161789660) (Tier 2, 3); [Joe Dixon](https://www.youtube.com/watch?v=60g-Z-dAWNk&t=1710s) (Laracon speaker (employee), 3); [@AshAllenDesign](https://ashallendesign.co.uk/blog/data-transfer-objects-dtos-in-php) (Tier 2, 3); [@brendt_gd](https://stitcher.io/blog/readonly-classes-in-php-82) (Tier 2, 3); [Spatie](https://freek.dev/3090-keeping-laravel-and-typescript-in-sync-with-data-objects) (Tier 2, 2); [Mateus Guimarães](https://www.youtube.com/watch?v=WqL6wgH1fKs) (Laracon speaker, 2); [Steve McDougall](https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=23370s) (Laracon speaker, 2); [Ryuta Hamasaki](https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=5310s) (Laracon speaker, 2); [Kévin Dunglas](https://www.youtube.com/watch?v=DJJt_2V26V8&t=1470s) (Laracon speaker, 2); [Bobby Bouwmann](https://www.youtube.com/watch?v=ru-0QSciNvU) (Laracon speaker, 2); [Duilio Palacios](https://www.youtube.com/watch?v=1sixoOQliZI&t=810s) (Laracon speaker, 2); [Craig Morris](https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=7860s) (Laracon speaker, 2); [Philo](https://www.youtube.com/watch?v=MbynDaPaBt4&t=630s) (Laracon speaker, 2); [Jeffrey Way](https://laracasts.com/episodes/3399) (Tier 3, 1); [Laravel News](https://laravel-news.com/use-inertia-js-like-a-boss) (Supporting, 0)

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

*Score 16:* [Jason Beggs](https://laravel.com/blog/cutting-laravel-cloud-load-times-by-60-with-nightwatch) (Tier 1, 3); [Spatie](https://www.youtube.com/watch?v=CrO_7Df1cBc&t=750s) (Tier 2, 2); [Diana Scharf](https://www.youtube.com/watch?v=50uRIFaUWqg&t=720s) (Laracon speaker, 2); [Paul Dragoonis](https://x.com/dr4goonis/status/1647246295914930176) (Tier 2, 2); [Stephen Rees-Carter](https://securinglaravel.com/security-tip-fix-your-leaky-apis) (Tier 3, 1); [Luke Downing](https://laracasts.com/episodes/3009) (Tier 3, 1); [Jeffrey Way](https://laracasts.com/episodes/2292) (Tier 3, 1); [Jeremy McPeak](https://laracasts.com/episodes/3196) (Tier 3, 1); [Matt Stauffer](https://laracasts.com/episodes/3313) (Tier 3, 1); [Simon Vrachliotis](https://laracasts.com/episodes/3652) (Tier 3, 1); [Instructor Team](https://laracasts.com/episodes/3840) (Tier 3, 1)

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

### Backed enums (Recommended)

*Score 12:* [Dan Harrin](https://x.com/danjharrin/status/1685624702876741632) (Tier 2, 3); [@PovilasKorop](https://laraveldaily.com/post/my-cursor-rules-for-laravel) (Tier 2, 3); [Spatie](https://www.youtube.com/watch?v=CrO_7Df1cBc&t=1200s) (Tier 2, 2); [Jake Bennett](https://www.youtube.com/watch?v=1A1xFtlDyzU) (Laracon speaker, 2); [Jeffrey Way](https://laracasts.com/episodes/3893) (Tier 3, 1); [Jeremy McPeak](https://laracasts.com/episodes/3430) (Tier 3, 1)

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

### Value object casts (Recommended)

*Score 12:* [Christoph Rumpel](https://laracasts.com/episodes/3071) (Tier 1, 3); [@AshAllenDesign](https://ashallendesign.co.uk/blog/custom-laravel-casts) (Tier 2, 3); [@laravel](https://laravel.com/docs/13.x/eloquent-mutators#defining-an-accessor) (Tier 2, 3); [Craig Morris](https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=7620s) (Laracon speaker, 2); [Mateus Guimarães](https://laracasts.com/episodes/3595) (Tier 3, 1)

- **Smell:** Structured address or money data is passed around as loosely shaped arrays.
- **Technique:** Introduce typed value objects and register an Eloquent custom cast or compatible Laravel Data cast to convert them to and from storage. Use a casts() method and implement serialization where the object is returned as JSON.
- **When to use:** A structured model attribute needs a stable shape and typed access.
- **When not to:** Custom conversion and serialization add work; simple scalar attributes do not need objects.
- **Packages:** `laravel/framework`, `propaganistas/laravel-phone`, `spatie/laravel-data`

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

### Validated data transfer objects (Recommended)

*Score 11:* [Wendell Adriel](https://x.com/wendell_adriel/status/1601276500300685314) (Tier 1, 3); [Christoph Rumpel](https://laracasts.com/episodes/3069) (Tier 1, 3); [Spatie](https://www.youtube.com/watch?v=CrO_7Df1cBc&t=540s) (Tier 2, 2); [Craig Morris](https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=7440s) (Laracon speaker, 2); [wendelladriel/laravel-validated-dto](https://github.com/WendellAdriel/laravel-validated-dto) (Tier 3, 1)

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

*Score 8:* [Nuno Maduro](https://github.com/nunomaduro/essentials) (Tier 1, 3); [Tighten](https://tighten.com/insights/building-a-calendar-with-carbon) (Tier 2, 2); [wendelladriel/laravel-exa](https://github.com/WendellAdriel/laravel-exa) (Tier 3, 1); [Mateus Guimarães](https://laracasts.com/episodes/3590) (Tier 3, 1); [mateusjatenee](https://x.com/mateusjatenee/status/1881004658174595469) (Tier 3, 1)

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

### Typed collection casts (Recommended)

*Score 8:* [Taylor Otwell](https://x.com/taylorotwell/status/1914683428886171934) (Tier 1, 3); [@laravel](https://laravel.com/docs/13.x/eloquent-mutators#array-object-and-collection-casting) (Tier 2, 3); [Bobby Bouwmann](https://www.youtube.com/watch?v=-jo6QjeaJXQ&t=1230s) (Laracon speaker, 2)

- **Smell:** Each consumer manually maps JSON attribute items into objects.
- **Technique:** Declare an AsCollection::of(...) cast on the model to map each JSON item into an application object automatically. Give the item class compatible construction and array/JSON serialization so it can also be saved.
- **When to use:** A JSON array contains repeated structured items with behavior or a stable shape.
- **When not to:** Simple scalar lists do not need object hydration; large collections incur allocation costs.
- **Packages:** `laravel/framework`

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

### Eloquent attribute casts (Recommended)

*Score 7:* [Taylor Otwell](https://www.youtube.com/watch?v=1P3wLy49t2c&t=1260s) (Laracon speaker (employee), 3); [Aaron Francis](https://x.com/aarondfrancis/status/1666124062538514433) (Tier 1, 3); [Jeremy McPeak](https://laracasts.com/episodes/4133) (Tier 3, 1)

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

*Score 7:* [Joe Tannenbaum](https://laracasts.com/episodes/3865) (Tier 1, 3); [Spatie](https://freek.dev/3090-keeping-laravel-and-typescript-in-sync-with-data-objects) (Tier 2, 2); [Craig Morris](https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=8130s) (Laracon speaker, 2)

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

### Value objects (Recommended)

*Score 7:* [Kai Sassnowski](https://www.youtube.com/watch?v=f4QShF42c6E&t=874s) (Laracon speaker, 2); [Jason McCreary](https://laracasts.com/episodes/3636) (Tier 3, 1); [Luke Downing](https://laracasts.com/episodes/3258) (Tier 3, 1); [Jeremy McPeak](https://laracasts.com/episodes/3998) (Tier 3, 1); [Mateus Guimarães](https://laracasts.com/episodes/3594) (Tier 3, 1); [@matthiasnoback](https://matthiasnoback.nl/2022/09/is-it-a-dto-or-a-value-object/) (Tier 3, 1)

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

### Generic type annotations (Useful)

*Score 6:* [Lucas Giovanny](https://www.youtube.com/watch?v=PWV5b0sL5Wo&t=720s) (Laracon speaker, 2); [Craig Morris](https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=7560s) (Laracon speaker, 2); [Mateus Guimarães](https://laracasts.com/episodes/3597) (Tier 3, 1); [Luke Downing](https://laracasts.com/episodes/2983) (Tier 3, 1)

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

### Immutable value objects (Useful)

*Score 6:* [Nuno Maduro](https://www.youtube.com/watch?v=AkDMDHAs09U&t=1560s) (Laracon speaker (employee), 3); [Kai Sassnowski](https://www.youtube.com/watch?v=f4QShF42c6E&t=874s) (Laracon speaker, 2); [Mateus Guimarães](https://laracasts.com/episodes/3593) (Tier 3, 1)

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

### Encrypted attribute casts (Useful)

*Score 5:* [Wendell Adriel](https://x.com/wendell_adriel/status/1972988790374371626) (Tier 1, 3); [Sam Sehnert](https://www.youtube.com/watch?v=32788JRtEnQ&t=1260s) (Laracon speaker, 2)

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

### Explicit typed operation parameters (Useful)

*Score 5:* [Dan Harrin](https://x.com/danjharrin/status/1501598772656316416) (Tier 2, 3); [Jason McCreary](https://www.youtube.com/watch?v=sBpzYsMgLK8&t=2016s) (Laracon speaker, 2)

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

### Collection transformation pipelines (Useful)

*Score 4:* [Jason McCreary](https://www.youtube.com/watch?v=sBpzYsMgLK8&t=1261s) (Laracon speaker, 2); [Mary Perry](https://www.youtube.com/watch?v=w7jKcbVcI54) (Laracon speaker, 2)

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

### Hashed attribute casts (Useful)

*Score 4:* [laravel/framework PRs & discussions](https://github.com/laravel/framework/pull/46947) (Tier 2, 2); [Spatie](https://spatie.be/guidelines/secure-coding) (Tier 2, 2)

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

*Score 3:* [Aaron Francis](https://x.com/aarondfrancis/status/1600568712205041664) (Tier 1, 3)

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

*Score 3:* [Joe Tannenbaum](https://laracasts.com/episodes/3859) (Tier 1, 3)

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

### Immutable fluent processing pipelines (Situational)

*Score 3:* [@laravel](https://laravel.com/docs/13.x/images#manipulating-images) (Tier 2, 3)

- **Smell:** Manual decode-transform-encode steps duplicate processing and invite discarded transformations.
- **Technique:** Chain image transformations and encode/store only after the full transformation pipeline.
- **When to use:** Uploaded or stored images need several transformations.
- **When not to:** Large processing workloads should be dispatched to a queued job.
- **Packages:** `intervention/image:^4.0`, `laravel/framework`

Before:
```php
$image = Image::fromStorage('photo.jpg');
$image->cover(400, 400);
$path = $image->toWebp()->store('avatars');
```
After:
```php
$path = Image::fromStorage('photo.jpg')
    ->cover(400, 400)
    ->toWebp()
    ->store('avatars');
```

### Opaque public identifiers (Situational)

*Score 3:* [Julien Bourdeau](https://www.julienbourdeau.com/laravel-hashid) (Tier 1, 3)

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

### Query-time casts (Situational)

*Score 3:* [Aaron Francis](https://x.com/aarondfrancis/status/1666124062538514433) (Tier 1, 3)

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

### Stable feature identifiers (Situational)

*Score 3:* [@laravel](https://laravel.com/docs/13.x/pennant) (Tier 2, 3)

- **Smell:** Moving or renaming a feature class silently changes its stored flag identity.
- **Technique:** Attach a stable Pennant Name attribute independent of the PHP class name.
- **When to use:** Feature classes are likely to move while existing assignments must be retained.
- **When not to:** The rename intentionally creates a different feature and rollout.
- **Packages:** `laravel/pennant`

Before:
```php
class NewApi {
    // Stored by fully qualified class name.
}
```
After:
```php
#[\Laravel\Pennant\Attributes\Name('new-api')]
class NewApi {
    // Namespace changes preserve the stored feature name.
}
```

### Structured AI output (Situational)

*Score 3:* [Jeffrey Way](https://laracasts.com/episodes/4081) (Tier 3, 1); [Jeremy McPeak](https://laracasts.com/episodes/4036) (Tier 3, 1); [@prism-php](https://github.com/prism-php/prism/blob/bcd71254685cab5969fa2aaf28ac791c11579482/docs/core-concepts/structured-output.md) (Tier 3, 1)

- **Smell:** Ticket triage logic tries to extract priority and department from free-form model text.
- **Technique:** Implement HasStructuredOutput with a schema for the triage agent. Use the structured response fields to update the ticket after validating application constraints.
- **When to use:** AI output becomes typed application data.
- **When not to:** A schema controls shape rather than truth; critical decisions may need review.
- **Packages:** `laravel/ai`, `prism-php/prism`

Before:
```php
$ticket->priority = parsePriority($response);
```
After:
```php
$result = $triageAgent->prompt($ticket->message);
$ticket->update(["priority" => $result["priority"]]);
```

### DTO construction pipelines (Situational)

*Score 2:* [Spatie](https://github.com/spatie/laravel-data/blob/1af068ca8bfa81d0fc63bcae8b111709fedd8f9e/docs/advanced-usage/pipeline.md) (Tier 2, 2)

- **Smell:** Payload normalization is duplicated in controllers and custom constructors, or a replacement pipeline omits validation.
- **Technique:** Insert a reusable DataPipe into parent::pipeline() rather than rebuilding the complete pipeline.
- **When to use:** Several DTOs need the same normalized input convention.
- **When not to:** One trivial field cast; a dedicated cast or property mapper may be simpler.
- **Packages:** `spatie/laravel-data`

Before:
```php
$payload = normalizeKeys($request->all());
return SongData::from($payload);
```
After:
```php
public static function pipeline(): DataPipeline
{
    return parent::pipeline()
        ->firstThrough(NormalizeKeysPipe::class);
}
```

### Data object casts (Situational)

*Score 2:* [Spatie](https://github.com/spatie/laravel-data/blob/0d5e4d0764d6a5ad472bf36470ad1b976ce7dff5/docs/advanced-usage/eloquent-casting.md) (Tier 2, 2)

- **Smell:** Many callers manually decode JSON and use string array keys for the same nested structure.
- **Technique:** Define a Data class and assign it as an Eloquent cast for the JSON attribute.
- **When to use:** A stable structured value belongs to one model rather than a separate entity table.
- **When not to:** Independently queried or related entities that need relational constraints and indexing.
- **Packages:** `spatie/laravel-data`

Before:
```php
$artist = json_decode($song->artist, true);
$name = $artist['name'];
```
After:
```php
// In Song::casts():
return ['artist' => ArtistData::class];
// At the call site:
$name = $song->artist->name;
```

### Envelope encryption and scoped key providers (Situational)

*Score 2:* [Sam Sehnert](https://www.youtube.com/watch?v=32788JRtEnQ&t=810s) (Laracon speaker, 2)

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

*Score 2:* [Kai Sassnowski](https://www.youtube.com/watch?v=f4QShF42c6E&t=874s) (Laracon speaker, 2)

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

### Optional-field sentinels (Situational)

*Score 2:* [Spatie](https://github.com/spatie/laravel-data/blob/a298eccc5d6d5a02cde5e8530e661d24fa90e385/docs/as-a-data-transfer-object/optional-properties.md) (Tier 2, 2)

- **Smell:** A nullable update DTO cannot distinguish a missing field from an explicit request to clear it.
- **Technique:** Union the property with Optional and pass the transformed DTO array to the update so absent fields are omitted.
- **When to use:** Partial updates with different omitted and null semantics.
- **When not to:** Create-only payloads with mandatory fields, or contracts where missing and null intentionally mean the same thing.
- **Packages:** `spatie/laravel-data`

Before:
```php
class ProfileData extends Data {
    public function __construct(public ?string $bio = null) {}
}
$user->update(ProfileData::from($payload)->toArray());
```
After:
```php
class ProfileData extends Data {
    public function __construct(
        public string|null|Optional $bio,
    ) {}
}
$user->update(ProfileData::from($payload)->toArray());
```

### Parameter objects (Situational)

*Score 2:* [Spatie](https://spatie.be/blog/utilizing-claude-skills-in-client-projects) (Tier 2, 2)

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

## Queries & database performance

### Cache-aside result caching (Recommended)

*Score 20:* [Christoph Rumpel](https://laracasts.com/episodes/3073) (Tier 1, 3); [Nuno Maduro](https://x.com/enunomaduro/status/1555137144062107648) (Tier 1, 3); [Luke Downing](https://www.youtube.com/watch?v=GERCwd65O-8&t=960s) (Laracon speaker, 2); [Tobias Petry](https://www.youtube.com/watch?v=QbsmLXmpYXo&t=300s) (Laracon speaker, 2); [Matt Stauffer](https://www.youtube.com/watch?v=KBigS5vLwZk&t=720s) (Laracon speaker, 2); [Tim](https://www.youtube.com/watch?v=NcucthcnGY0&t=1710s) (Laracon speaker, 2); [Marcel Pociot](https://www.youtube.com/watch?v=rk29Hyk59JY) (Laracon speaker, 2); [Josh Hanley](https://www.youtube.com/watch?v=esD6o8HYV5Q&t=619s) (Laracon speaker, 2); [Jeremy McPeak](https://laracasts.com/episodes/3995) (Tier 3, 1); [laravel/boost](https://github.com/laravel/boost/blob/d165b9ea6086294a8f1999ac3dcf4e61da267b23/.ai/laravel/skill/laravel-best-practices/rules/caching.md) (Tier 3, 1)

- **Smell:** An expensive stable derived query repeats across component requests.
- **Technique:** Cache selected computed results across requests and invalidate when their inputs change. Livewire 4 distinguishes per-component persistence from cache:true shared across instances; Laravel 13 cached object serialization may require explicit class allowance, so scalar/array results are simpler.
- **When to use:** Expensive reference data changes infrequently.
- **When not to:** Shared caches must not mix user-specific data; cached values require an invalidation policy.
- **Packages:** `laravel/framework`, `livewire/livewire`, `rennokki/laravel-eloquent-query-cache`

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

### Request-scoped memoization (Recommended)

*Score 18:* [Tim MacDonald](https://x.com/timacdonald87/status/1850776003376730405) (Tier 1, 3); [Taylor Otwell](https://x.com/taylorotwell/status/1555573050954977280) (Tier 1, 3); [@AshAllenDesign](https://ashallendesign.co.uk/blog/laravel-memoisation-once-helper) (Tier 2, 3); [laravel/framework PRs & discussions](https://github.com/laravel/framework/pull/49744) (Tier 2, 2); [Josh Hanley](https://www.youtube.com/watch?v=esD6o8HYV5Q&t=407s) (Laracon speaker, 2); [Caleb Porzio](https://www.youtube.com/watch?v=M60-nxlrePc&t=960s) (Laracon speaker, 2); [Jeremy McPeak](https://laracasts.com/episodes/3372) (Tier 3, 1); [Luke Downing](https://laracasts.com/episodes/3111) (Tier 3, 1); [laravel/boost](https://github.com/laravel/boost/blob/d165b9ea6086294a8f1999ac3dcf4e61da267b23/.ai/laravel/skill/laravel-best-practices/rules/caching.md) (Tier 3, 1)

- **Smell:** Public model collections lose custom query projections during Livewire hydration.
- **Technique:** Move derived query results into Computed methods and reference them through $this in Blade. Livewire 4 evaluates them on demand and memoizes within the request, rather than hydrating custom projections as public state.
- **When to use:** Queried display data should be derived afresh from component state.
- **When not to:** Editable state still needs explicit form properties; expensive recomputation may require a deliberate cache.
- **Packages:** `inertiajs/inertia-laravel`, `laravel/framework`, `livewire/livewire`, `spatie/once`

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

### Declarative query filter objects (Recommended)

*Score 17:* [Nuno Maduro](https://x.com/enunomaduro/status/1801593918305235251) (Tier 1, 3); [@laravel](https://laravel.com/docs/13.x/queries#reusable-query-components) (Tier 2, 3); [Lucas Giovanny](https://www.youtube.com/watch?v=33CIhvkAPEY&t=1230s) (Laracon speaker, 2); [Jigar Dhulla](https://www.youtube.com/watch?v=dP88NZfVjnY) (Laracon speaker, 2); [Spatie](https://www.youtube.com/watch?v=Boqfw4ZCk7g&t=960s) (Tier 2, 2); [Kévin Dunglas](https://www.youtube.com/watch?v=DJJt_2V26V8&t=1260s) (Laracon speaker, 2); [Bobby Bouwmann](https://www.youtube.com/watch?v=ru-0QSciNvU) (Laracon speaker, 2); [Jeremy McPeak](https://laracasts.com/episodes/3199) (Tier 3, 1)

- **Smell:** Endpoints duplicate request-to-filter, sort and include plumbing around Eloquent.
- **Technique:** Use Spatie Query Builder with explicitly allowed filters, sorts, includes and fields, optionally starting from an already constrained Eloquent builder. The note locates the package presentation but reconstructs API detail from its documentation; choose a release compatible with Laravel 13.
- **When to use:** Several endpoints expose structured client query options.
- **When not to:** Simple fixed queries need no extra package; request-driven options must retain authorization constraints.
- **Packages:** `api-platform/laravel`, `filament/tables`, `laravel/framework`, `livewire/livewire`, `spatie/laravel-query-builder`

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

### Relationship aggregates (Recommended)

*Score 17:* [Wendell Adriel](https://x.com/wendell_adriel/status/1973351116537528601) (Tier 1, 3); [Christoph Rumpel](https://laracasts.com/episodes/2875) (Tier 1, 3); [@laravel](https://laravel.com/docs/13.x/eloquent-resources#conditional-relationship-counts) (Tier 2, 3); [Tim](https://www.youtube.com/watch?v=NcucthcnGY0&t=840s) (Laracon speaker, 2); [Marcel Pociot](https://www.youtube.com/watch?v=rk29Hyk59JY) (Laracon speaker, 2); [Shruti Balasa](https://laracasts.com/episodes/4021) (Tier 3, 1); [Jeremy McPeak](https://laracasts.com/episodes/3767) (Tier 3, 1); [@benjamincrozat](https://benjamincrozat.com/laravel-debugbar) (Tier 3, 1); [laravel/boost](https://github.com/laravel/boost/blob/d165b9ea6086294a8f1999ac3dcf4e61da267b23/.ai/laravel/skill/laravel-best-practices/rules/db-performance.md) (Tier 3, 1)

- **Smell:** Lists load entire related collections merely to count rows or sum a column.
- **Technique:** Use withCount, withSum, or related Eloquent aggregate methods to select scalar aggregates alongside each parent. Read the resulting attributes without hydrating the child models.
- **When to use:** A list or report needs related totals rather than full child data.
- **When not to:** The aggregate subqueries still need indexes; eager-load when the children themselves will also be rendered.
- **Packages:** `filament/filament`, `laravel/framework`

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

### Query-driven indexes (Recommended)

*Score 13:* [Devon Garbalosa](https://www.youtube.com/watch?v=toSMnEINXAo&t=319s) (Laracon speaker (employee), 3); [Jess Archer](https://www.youtube.com/watch?v=G9Toicww8BU&t=720s) (Laracon speaker (employee), 3); [Aaron Francis](https://www.youtube.com/watch?v=f4QShF42c6E&t=21743s) (Laracon speaker (employee), 3); [Tobias Petry](https://www.youtube.com/watch?v=QbsmLXmpYXo&t=240s) (Laracon speaker, 2); [Jonathan Reinink](https://laracasts.com/episodes/2563) (Tier 3, 1); [Sabatino Masala](https://x.com/SabatinoMasala/status/2003069291222982658) (Tier 3, 1)

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

### Stale-while-revalidate caching (Recommended)

*Score 13:* [Jess Archer](https://www.youtube.com/watch?v=0j6z_oq4cZU&t=360s) (Laracon speaker (employee), 3); [@laravel](https://laravel.com/docs/13.x/cache#swr) (Tier 2, 3); [Spatie](https://freek.dev/2597-adding-stale-while-revalidate-functionality-to-laravels-cache) (Tier 2, 2); [Tim](https://www.youtube.com/watch?v=NcucthcnGY0&t=1200s) (Laracon speaker, 2); [Thijs Feryn](https://www.youtube.com/watch?v=VFTeGtPLlS0) (Laracon speaker, 2); [laravel/boost](https://github.com/laravel/boost/blob/d165b9ea6086294a8f1999ac3dcf4e61da267b23/.ai/laravel/skill/laravel-best-practices/rules/caching.md) (Tier 3, 1); [Laravel News](https://laravel-news.com/boosting-app-speed-with-flexible-caching-laravel-in-practice-ep8) (Supporting, 0)

- **Smell:** Requests block on expensive regeneration whenever a cache entry expires.
- **Technique:** Use Laravel 13 Cache::flexible with a fresh interval and a longer stale interval. Requests in the stale interval receive the existing value while Laravel schedules a refresh after the response; fully expired entries recompute synchronously.
- **When to use:** Read-heavy derived data tolerates bounded staleness.
- **When not to:** Avoid for authorization, balances, or data requiring immediate consistency; refresh work can still fail or consume worker time.
- **Packages:** `Varnish`, `laravel/framework`, `spatie/laravel-responsecache`

Before:
```php
$stats = Cache::remember("stats", 60, fn () => buildStats());
```
After:
```php
$stats = Cache::flexible("stats", [60, 300], fn () => buildStats());
```

### Named query scopes (Recommended)

*Score 12:* [Aaron Francis](https://aaronfrancis.com/2024/reservable-models-in-laravel-990d6e9e) (Tier 1, 3); [Christoph Rumpel](https://laracasts.com/episodes/2870) (Tier 1, 3); [@laravel](https://laravel.com/docs/13.x/eloquent#local-scopes) (Tier 2, 3); [laravel/framework PRs & discussions](https://github.com/laravel/framework/pull/54450) (Tier 2, 2); [Jeffrey Way](https://laracasts.com/episodes/3898) (Tier 3, 1)

- **Smell:** The same model classification predicates are copied into different queries.
- **Technique:** Encapsulate reusable predicates in an Eloquent local scope and compose it with other builder operations. Keep the scope on the builder so filtering happens in the database.
- **When to use:** A domain category such as available or reserved is queried in multiple places.
- **When not to:** Do not create a scope for every one-off predicate or hide unrelated side effects inside it.
- **Packages:** `laravel/framework`

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

### Incremental analytical rollups (Recommended)

*Score 11:* [Jess Archer](https://www.youtube.com/watch?v=G9Toicww8BU&t=900s) (Laracon speaker (employee), 3); [@brendt_gd](https://stitcher.io/blog/11-million-rows-in-seconds) (Tier 2, 3); [@laravel](https://laravel.com/docs/13.x/pulse) (Tier 2, 3); [Tobias Petry](https://www.youtube.com/watch?v=cucIWpAenro&t=10520s) (Laracon speaker, 2)

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

### Read-model projections (Recommended)

*Score 11:* [Jess Archer](https://www.youtube.com/watch?v=G9Toicww8BU&t=1530s) (Laracon speaker (employee), 3); [Devansh Bawari](https://www.youtube.com/watch?v=eJJsGhu9gtI) (Laracon speaker, 2); [Seb Armand](https://www.youtube.com/watch?v=9mDkf4popb8&t=990s) (Laracon speaker, 2); [Daniel Coulbourne](https://www.linkedin.com/posts/coulbourne_yesterday-i-gave-my-talk-at-laracon-eu-called-activity-7160575816014155776-JI0u) (Laracon speaker, 2); [Mateus Guimarães](https://www.youtube.com/watch?v=WqL6wgH1fKs) (Laracon speaker, 2)

- **Smell:** Exception detection repeatedly polls every tenant’s raw history, including idle tenants.
- **Technique:** Maintain a small per-environment/class occurrence projection with a materialized view and poll nonzero pending sums. Acknowledge the observed amount by inserting its negative so concurrently arriving counts remain pending; failure recovery and consumer coordination need separate design.
- **When to use:** High-volume exception detection should be driven by actual arrivals.
- **When not to:** Crashes or multiple consumers can lose or duplicate acknowledgments without an explicit protocol.
- **Packages:** `hirethunk/verbs`, `spatie/laravel-event-sourcing`

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

*Score 10:* [Jason Beggs](https://laravel.com/blog/cutting-laravel-cloud-load-times-by-60-with-nightwatch) (Tier 1, 3); [Seb Armand](https://www.youtube.com/watch?v=9mDkf4popb8&t=420s) (Laracon speaker, 2); [Marcel Pociot](https://www.youtube.com/watch?v=rk29Hyk59JY) (Laracon speaker, 2); [Shruti Balasa](https://laracasts.com/episodes/4030) (Tier 3, 1); [Simon Vrachliotis](https://laracasts.com/episodes/3660) (Tier 3, 1); [Jeremy McPeak](https://laracasts.com/episodes/4125) (Tier 3, 1); [Laravel News](https://laravel-news.com/laravel-model-tips) (Supporting, 0)

- **Smell:** Rendering a collection issues one related-model query per parent.
- **Technique:** Declare the relationships needed by the consumer with Eloquent with() before retrieving the models. Select only needed relations and columns so loading avoids N+1 queries without hydrating excessive data.
- **When to use:** A list or view will access the same relationship on many models.
- **When not to:** Loading large unused relations wastes memory; use aggregates when only counts or totals are needed.
- **Packages:** `filament/filament`

Before:
```php
$posts = Post::all();
foreach ($posts as $post) { echo $post->author->name; }
```
After:
```php
$posts = Post::with("author")->get();
foreach ($posts as $post) { echo $post->author->name; }
```

### Global query scopes (Recommended)

*Score 10:* [Christoph Rumpel](https://christoph-rumpel.com/2024/9/all-laravel-php-attributes-at-your-disposal) (Tier 1, 3); [Jess Archer](https://www.youtube.com/watch?v=G9Toicww8BU&t=1380s) (Laracon speaker (employee), 3); [laravel/framework PRs & discussions](https://github.com/laravel/framework/pull/50034) (Tier 2, 2); [Spatie](https://www.youtube.com/watch?v=JmEmrLtMB_A&t=1560s) (Tier 2, 2)

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

### Inverse relationship hydration (Recommended)

*Score 10:* [@laravel](https://laravel.com/docs/13.x/eloquent-relationships#one-to-many) (Tier 2, 3); [Spatie](https://freek.dev/2400-a-recap-of-2022) (Tier 2, 2); [laravel/framework PRs & discussions](https://github.com/laravel/framework/pull/51582) (Tier 2, 2); [Jonathan Reinink](https://laracasts.com/episodes/2546) (Tier 3, 1); [laravel/boost](https://github.com/laravel/boost/blob/d165b9ea6086294a8f1999ac3dcf4e61da267b23/.ai/laravel/skill/laravel-best-practices/rules/advanced-queries.md) (Tier 3, 1); [laravel/framework](https://github.com/laravel/framework/pull/61152) (Tier 3, 1)

- **Smell:** Children re-query a parent model that the caller already has.
- **Technique:** Use setRelation() to attach the existing parent instance to each child before accessing the inverse relationship.
- **When to use:** A parent is already loaded and children refer to that exact instance.
- **When not to:** Incorrect relation assignment can hide stale data; do not attach a parent that does not match the foreign key.
- **Packages:** `laravel/framework`

Before:
```php
$team->sites->each(fn ($site) => inspect($site->team));
```
After:
```php
$team->sites->each(function ($site) use ($team) {
    $site->setRelation("team", $team);
    inspect($site->team);
});
```

### Server-side pagination (Recommended)

*Score 10:* [James Brooks](https://www.youtube.com/watch?v=he2-4Dbc-y0&t=1047s) (Laracon speaker (employee), 3); [Jess Archer](https://www.youtube.com/watch?v=uZLkGuylXDQ&t=2334s) (Laracon speaker (employee), 3); [Jordan Havard](https://www.youtube.com/watch?v=riJBa0Bxo-8&t=1575s) (Laracon speaker, 2); [Luke Downing](https://laracasts.com/episodes/3010) (Tier 3, 1); [Simon Vrachliotis](https://laracasts.com/episodes/3661) (Tier 3, 1)

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

### Automatic eager loading (Recommended)

*Score 9:* [Nuno Maduro](https://github.com/nunomaduro/essentials) (Tier 1, 3); [@laravel](https://laravel.com/docs/13.x/eloquent-relationships#automatic-eager-loading) (Tier 2, 3); [laravel/framework PRs & discussions](https://github.com/laravel/framework/pull/53655) (Tier 2, 2); [wendelladriel/laravel-exa](https://github.com/WendellAdriel/laravel-exa) (Tier 3, 1)

- **Smell:** Relationships accessed during iteration cause repeated lazy-load queries and relation requirements vary by consumer.
- **Technique:** Opt into Eloquent automatic relationship loading so accessing a relation can batch-load it for the collection. Prefer explicit with() where the required graph is known and assess the memory and query behavior of automatic loading.
- **When to use:** Collection consumers vary and accidental relation access creates N+1 queries.
- **When not to:** Automatic loading can fetch unexpected data and hide query intent; verify support and behavior in the deployed Laravel release.
- **Packages:** `laravel/framework`

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

### Lazy and on-demand response properties (Recommended)

*Score 9:* [Caleb Porzio](https://x.com/calebporzio/status/1986549709733876217) (Tier 2, 3); [Spatie](https://www.youtube.com/watch?v=CrO_7Df1cBc&t=1470s) (Tier 2, 2); [Diana Scharf](https://www.youtube.com/watch?v=50uRIFaUWqg&t=1140s) (Laracon speaker, 2); [Jeffrey Way](https://laracasts.com/episodes/3443) (Tier 3, 1); [Jeremy McPeak](https://laracasts.com/episodes/3198) (Tier 3, 1)

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

### Request-scoped cache memoization (Recommended)

*Score 9:* [@AshAllenDesign](https://ashallendesign.co.uk/blog/laravel-cache-memo) (Tier 2, 3); [@laravel](https://laravel.com/docs/13.x/cache#cache-memoization) (Tier 2, 3); [laravel/framework PRs & discussions](https://github.com/laravel/framework/pull/55304) (Tier 2, 2); [laravel/boost](https://github.com/laravel/boost/blob/d165b9ea6086294a8f1999ac3dcf4e61da267b23/.ai/laravel/skill/laravel-best-practices/rules/caching.md) (Tier 3, 1); [Laravel News](https://laravel-news.com/optimizing-queries-with-memoization-laravel-in-practice-ep9) (Supporting, 0)

- **Smell:** Multiple operations in one request repeatedly fetch the same cache key from the backend.
- **Technique:** Use Laravel Cache::memo to decorate cache access with execution-local memoization. Repeated reads reuse the value without another backend call while writes through that repository invalidate its memoized entry.
- **When to use:** Remote cache round trips repeat for the same keys during a request or job.
- **When not to:** Memoization may not observe writes made by other processes during the execution; avoid when live freshness is required.
- **Packages:** `laravel/framework`

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

### Atomic cache locks (Recommended)

*Score 8:* [Aaron Francis](https://aaronfrancis.com/2023/year-in-review-2023-5431fc1f) (Tier 1, 3); [Wendell Adriel](https://x.com/wendell_adriel/status/1970094482486948113) (Tier 1, 3); [Harris Raftopoulos](https://www.youtube.com/watch?v=E9t2lVn0z8A&t=570s) (Laracon speaker, 2)

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

### HTTP response caching (Recommended)

*Score 8:* [Devansh Bawari](https://www.youtube.com/watch?v=eJJsGhu9gtI) (Laracon speaker, 2); [Seb Armand](https://www.youtube.com/watch?v=9mDkf4popb8&t=870s) (Laracon speaker, 2); [Spatie](https://www.youtube.com/watch?v=Boqfw4ZCk7g&t=780s) (Tier 2, 2); [Thijs Feryn](https://www.youtube.com/watch?v=VFTeGtPLlS0) (Laracon speaker, 2)

- **Smell:** Anonymous storefront requests repeatedly produce identical expensive responses.
- **Technique:** Cache reusable data or anonymous page output with explicit freshness and invalidation boundaries. Laravel 13 supports shared data caching; full-page caching or Varnish needs an additional integration with personalization excluded.
- **When to use:** High-volume anonymous reads share output and tolerate defined freshness.
- **When not to:** Personalized or authorization-dependent output risks incorrect sharing and cache invalidation adds maintenance.
- **Packages:** `Varnish`, `spatie/laravel-responsecache`

Before:
```php
return $storefront->renderAnonymousCatalog($category);
```
After:
```php
$key = "catalog:{$category->id}:{$locale}";
return Cache::remember($key, 60,
    fn () => $storefront->renderAnonymousCatalog($category)
);
// authenticated/personalized traffic follows its own path
```

### Database transactions (Useful)

*Score 6:* [@laravel](https://laravel.com/docs/13.x/helpers#pipeline) (Tier 2, 3); [Mateus Guimarães](https://laracasts.com/episodes/3037) (Tier 3, 1); [Sabatino Masala](https://x.com/SabatinoMasala/status/1989280428977721704) (Tier 3, 1); [@benjamincrozat](https://benjamincrozat.com/database-transactions-laravel) (Tier 3, 1)

- **Smell:** An operation updates multiple models and partial failure leaves inconsistent records.
- **Technique:** Wrap the related database writes in DB::transaction so Laravel 13 commits them together or rolls them back. Keep the transaction short and schedule external side effects after it commits.
- **When to use:** Several writes form one business invariant, especially in payments.
- **When not to:** Transactions cannot roll back remote API calls and do not by themselves prevent concurrent lost updates.
- **Packages:** `laravel/framework`

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

### Dedicated analytical read stores (Useful)

*Score 6:* [Jess Archer](https://www.youtube.com/watch?v=_jjvaFWWKqg) (Laracon speaker (employee), 3); [@laravel](https://laravel.com/docs/13.x/pulse) (Tier 2, 3)

- **Smell:** Large reporting aggregations compete with transactional workloads.
- **Technique:** Use a column-oriented analytical store such as ClickHouse for the reporting workload while retaining relational persistence where required. Integrate it through an external Laravel-compatible client/adapter.
- **When to use:** Large scans and aggregates dominate reporting costs.
- **When not to:** Additional storage and ingestion increase operational work; ClickHouse is not a native Laravel driver.
- **Packages:** `glushkovds/phpclickhouse-laravel`, `laravel/pulse`

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

### Full-text search (Useful)

*Score 6:* [@laravel](https://laravel.com/docs/13.x/search#full-text-search) (Tier 2, 3); [Tighten](https://tighten.com/insights/blazing-fast-full-text-search-in-laravel-from-mysql-to-typesense) (Tier 2, 2); [Jonathan Reinink](https://laracasts.com/episodes/2632) (Tier 3, 1)

- **Smell:** LIKE queries become slow or too limited for application search.
- **Technique:** Replace substring scans with an appropriate full-text index or a dedicated search engine through Laravel Scout. Scout with Typesense synchronizes searchable models and provides a search API while separating search indexing from ordinary relational queries.
- **When to use:** Search needs ranking, typo tolerance, or better performance than unindexed wildcard queries.
- **When not to:** An external engine adds deployment and index synchronization costs; database full-text search may suffice.
- **Packages:** `laravel/framework`, `laravel/scout`, `typesense/typesense-php`

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

### Scalar subquery projections (Useful)

*Score 6:* [Wendell Adriel](https://x.com/wendell_adriel/status/1959962874106748987) (Tier 1, 3); [Jonathan Reinink](https://laracasts.com/episodes/2543) (Tier 3, 1); [@benjamincrozat](https://benjamincrozat.com/laravel-subquery) (Tier 3, 1); [laravel/boost](https://github.com/laravel/boost/blob/d165b9ea6086294a8f1999ac3dcf4e61da267b23/.ai/laravel/skill/laravel-best-practices/rules/advanced-queries.md) (Tier 3, 1)

- **Smell:** A list loads every child row or adds bulky joins just to get the latest related value.
- **Technique:** Use addSelect with a correlated, ordered subquery limited to one row to fetch the needed scalar alongside each parent. Order or filter using the derived value when supported and index the relation key plus ordering column.
- **When to use:** A parent list needs a latest date or other single related scalar.
- **When not to:** Subqueries may be slower than joins for some plans; inspect query plans and use eager loading when full child models are needed.
- **Packages:** `laravel/framework`

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

### Custom Eloquent builders (Useful)

*Score 5:* [Tim MacDonald](https://x.com/timacdonald87/status/1490212109656858627) (Tier 1, 3); [laravel/framework PRs & discussions](https://github.com/laravel/framework/pull/56025) (Tier 2, 2)

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

### Indexed generated columns (Useful)

*Score 5:* [Aaron Francis](https://x.com/aarondfrancis/status/1615753303899373574) (Tier 1, 3); [Spatie](https://spatie.be/blog/speeding-up-database-searches-using-an-invisible-column) (Tier 2, 2)

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

### Query objects (Useful)

*Score 5:* [Tim MacDonald](https://x.com/timacdonald87/status/1904713292284911677) (Tier 1, 3); [wendelladriel/eloquent-query-classes](https://github.com/WendellAdriel/eloquent-query-classes) (Tier 3, 1); [Instructor Team](https://laracasts.com/episodes/3743) (Tier 3, 1); [Laravel News](https://laravel-news.com/effective-eloquent) (Supporting, 0)

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

### Read/write splitting with consistency controls (Useful)

*Score 5:* [Tobias Petry](https://www.youtube.com/watch?v=QbsmLXmpYXo&t=630s) (Laracon speaker, 2); [Guus Leeuw](https://www.youtube.com/watch?v=rpvhD-twpY4) (Laracon speaker, 2); [laravel/framework](https://github.com/laravel/framework/pull/61915) (Tier 3, 1)

- **Smell:** Read-heavy traffic overloads a single database primary.
- **Technique:** Configure Laravel 13 read hosts as replicas and the write host as the primary. Eloquent and query-builder calls use these connection roles while consistency-sensitive reads can explicitly use the primary.
- **When to use:** Read throughput is the bottleneck after query optimization.
- **When not to:** Replication lag complicates read-after-write and replicas do not scale primary writes.
- **Packages:** `laravel/framework`

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

### Scheduled cache warming (Useful)

*Score 5:* [Jess Archer](https://www.youtube.com/watch?v=0j6z_oq4cZU&t=360s) (Laracon speaker (employee), 3); [Tim](https://www.youtube.com/watch?v=NcucthcnGY0&t=1320s) (Laracon speaker, 2)

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

*Score 5:* [Mohamed Said](https://laracasts.com/episodes/2780) (Tier 1, 3); [Marcel Pociot](https://www.youtube.com/watch?v=rk29Hyk59JY) (Laracon speaker, 2)

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

*Score 4:* [laravel/framework PRs & discussions](https://github.com/laravel/framework/pull/49695) (Tier 2, 2); [Jeremy McPeak](https://laracasts.com/episodes/3766) (Tier 3, 1); [Luke Downing](https://laracasts.com/episodes/3110) (Tier 3, 1)

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

*Score 4:* [Aaron Francis](https://aaronfrancis.com/2022/is-count-slow-in-mysql-aabe5c35) (Tier 1, 3); [Luke Downing](https://laracasts.com/episodes/3251) (Tier 3, 1)

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

### Dependency-aware cache invalidation (Useful)

*Score 4:* [Seb Armand](https://www.youtube.com/watch?v=9mDkf4popb8&t=480s) (Laracon speaker, 2); [Tighten](https://tighten.com/insights/overview-of-caching-strategies-in-laravel/) (Tier 2, 2)

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

### Primary reads for consistency (Useful)

*Score 4:* [Philo Hermans](https://www.youtube.com/watch?v=857ean0JIHE) (Laracon speaker, 2); [Tobias Petry](https://www.youtube.com/watch?v=QbsmLXmpYXo&t=690s) (Laracon speaker, 2)

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

### Search engine abstraction (Scout) (Useful)

*Score 4:* [Devansh Bawari](https://www.youtube.com/watch?v=eJJsGhu9gtI) (Laracon speaker, 2); [Luke Downing](https://laracasts.com/episodes/3252) (Tier 3, 1); [Jeffrey Way](https://laracasts.com/episodes/3571) (Tier 3, 1)

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

### Batched feature-flag loading (Situational)

*Score 3:* [@laravel](https://laravel.com/docs/13.x/pennant#eager-loading) (Tier 2, 3)

- **Smell:** Checking a feature per user causes one database lookup for each user.
- **Technique:** Load the feature for the whole user collection before the loop; subsequent checks reuse Pennant's in-memory cache.
- **When to use:** The database feature driver serves repeated checks over many scopes.
- **When not to:** A single scope or an already warmed feature cache needs no eager-load step.
- **Packages:** `laravel/pennant`

Before:
```php
foreach ($users as $user) {
    if (Feature::for($user)->active('notifications-beta')) $user->notify($notice);
}
```
After:
```php
Feature::for($users)->load(['notifications-beta']);
foreach ($users as $user) {
    if (Feature::for($user)->active('notifications-beta')) $user->notify($notice);
}
```

### Bundled SQLite read models (Situational)

*Score 3:* [Aaron Francis](https://aaronfrancis.com/2023/shipping-sqlite-to-lambda-with-laravel-vapor-fd8b6bd5) (Tier 1, 3)

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

*Score 3:* [Taylor Otwell](https://x.com/taylorotwell/status/1980664560492376378) (Tier 1, 3)

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

*Score 3:* [Aaron Francis](https://www.youtube.com/watch?v=f4QShF42c6E&t=21743s) (Laracon speaker (employee), 3)

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

*Score 3:* [Aaron Francis](https://aaronfrancis.com/2022/efficient-mysql-pagination-using-deferred-joins-15d0de14) (Tier 1, 3)

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

*Score 3:* [Jess Archer](https://www.youtube.com/watch?v=G9Toicww8BU&t=1440s) (Laracon speaker (employee), 3)

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

### Deferred join pagination (Situational)

*Score 3:* [Aaron Francis](https://aaronfrancis.com/2022/efficient-mysql-pagination-using-deferred-joins-15d0de14) (Tier 1, 3)

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

### Grouped query filters (Situational)

*Score 3:* [Spatie](https://freek.dev/3118-searching-multiple-columns-with-one-url-parameter-in-laravel-query-builder) (Tier 2, 2); [Stephen Rees-Carter](https://securinglaravel.com/security-tip-scoping-orwhere-can-be-disastrous) (Tier 3, 1)

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

*Score 3:* [Jess Archer](https://www.youtube.com/watch?v=G9Toicww8BU&t=1170s) (Laracon speaker (employee), 3)

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

*Score 3:* [Wendell Adriel](https://x.com/wendell_adriel/status/1962479188109058525) (Tier 1, 3)

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

### Online index creation (Situational)

*Score 3:* [@laravel](https://laravel.com/docs/13.x/migrations#online-index-creation) (Tier 2, 3)

- **Smell:** Creating an index blocks application reads or writes on a large table.
- **Technique:** Use online on PostgreSQL or SQL Server index definitions; choose MySQL inplace and compatible lock modes for that driver.
- **When to use:** A supported database must keep serving traffic while building an index.
- **When not to:** Driver, database version, and operation must support the requested algorithm; PostgreSQL concurrent indexes cannot run inside transactions.
- **Packages:** `laravel/framework`

Before:
```php
$table->unique('email');
```
After:
```php
// PostgreSQL or SQL Server:
$table->unique('email')->online();
// MySQL, for compatible operations:
$table->index('email')->inplace()->lock('none');
```

### Pessimistic row locking (Situational)

*Score 3:* [Sabatino Masala](https://x.com/SabatinoMasala/status/1989280428977721704) (Tier 3, 1); [Jeremy McPeak](https://laracasts.com/episodes/3992) (Tier 3, 1); [laravel/boost](https://github.com/laravel/boost/blob/75c269a8a53e931c7252e7cd562e1a209356a7c6/.ai/laravel/skill/laravel-best-practices/rules/architecture.md) (Tier 3, 1)

- **Smell:** Concurrent multi-model updates race or leave balances and related records inconsistent.
- **Technique:** Wrap the operation in DB::transaction and select the rows being changed with lockForUpdate before checking and updating their state. Acquire locks in a consistent order and dispatch dependent effects after commit.
- **When to use:** Concurrent payment or inventory operations update shared rows and enforce cross-row invariants.
- **When not to:** Long transactions and inconsistent lock order cause contention or deadlocks; do not perform remote calls while holding locks.
- **Packages:** `laravel/framework`

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

### Recursive CTE gap detection (Situational)

*Score 3:* [Aaron Francis](https://aaronfrancis.com/2024/finding-missing-orders-with-a-recursive-cte-bf59e6de) (Tier 1, 3)

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

### Retrieve-and-rerank search (Situational)

*Score 3:* [@laravel](https://laravel.com/docs/13.x/search#reranking-results) (Tier 2, 3)

- **Smell:** Semantic ranking every record is slow and costly.
- **Technique:** Retrieve a limited candidate set with indexed full-text search, then rerank that collection using the AI SDK.
- **When to use:** A fast first pass needs improved semantic relevance.
- **When not to:** Candidate retrieval misses relevant records or provider cost is unacceptable.
- **Packages:** `laravel/ai`, `laravel/framework`

Before:
```php
$articles = Article::all()->rerank('body', $query);
```
After:
```php
$articles = Article::whereFullText('body', $query)
    ->limit(50)->get()
    ->rerank('body', $query);
```

### Session-scoped cache (Situational)

*Score 3:* [@laravel](https://laravel.com/docs/13.x/session#session-cache) (Tier 2, 3)

- **Smell:** User-specific temporary values use manually constructed global cache keys.
- **Technique:** Store temporary results through the request session cache so they share the session lifecycle.
- **When to use:** Ephemeral calculations or API results belong to one user session.
- **When not to:** Permanent data or data shared across sessions belongs in a database or application cache.
- **Packages:** `laravel/framework`

Before:
```php
Cache::put('quote:'.session()->getId(), $quote, 60);
```
After:
```php
$request->session()->cache()->put('quote', $quote, 60);
```

### Subquery membership filtering (Situational)

*Score 3:* [@AshAllenDesign](https://ashallendesign.co.uk/blog/laravel-wherein-reduce-queries) (Tier 2, 3)

- **Smell:** A query materializes an ID list only to feed another query, adding round trips and PHP memory.
- **Technique:** Pass the unexecuted single-column builder to whereIn.
- **When to use:** Both tables are queryable in the same database and the planner handles the subquery efficiently.
- **When not to:** IDs come from another connection or must be reused in application logic; some databases restrict LIMIT in IN subqueries.

Before:
```php
$ids = Article::where("published", true)->pluck("user_id");
$users = User::whereIn("id", $ids)->get();
```
After:
```php
$ids = Article::where("published", true)->select("user_id");
$users = User::whereIn("id", $ids)->get();
```

### Tagged cache invalidation (Situational)

*Score 3:* [Wendell Adriel](https://x.com/wendell_adriel/status/1971180406717210806) (Tier 1, 3)

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

*Score 3:* [Jess Archer](https://www.youtube.com/watch?v=_jjvaFWWKqg) (Laracon speaker (employee), 3)

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

*Score 2:* [Sam Sehnert](https://www.youtube.com/watch?v=32788JRtEnQ&t=1410s) (Laracon speaker, 2)

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

*Score 2:* [Jonathan Reinink](https://laracasts.com/episodes/2545) (Tier 3, 1); [laravel/boost](https://github.com/laravel/boost/blob/d165b9ea6086294a8f1999ac3dcf4e61da267b23/.ai/laravel/skill/laravel-best-practices/rules/advanced-queries.md) (Tier 3, 1)

- **Smell:** Each parent needs one full child model, but eager loading all children consumes excessive memory.
- **Technique:** Select the desired child ID with a correlated subquery and use it as the foreign key of a belongsTo relationship. Eager load that relationship to hydrate only the selected children.
- **When to use:** Listings need the latest or otherwise selected related object per parent.
- **When not to:** Computed foreign keys add query coupling and require the subquery whenever the relationship is used.
- **Packages:** `laravel/framework`

Before:
```php
$users = User::with("logins")->get();
```
After:
```php
$users = User::addSelect(["last_login_id" =>
 Login::select("id")->whereColumn("user_id", "users.id")
 ->latest()->limit(1)])->with("lastLogin")->get();
// lastLogin(): belongsTo(Login::class, "last_login_id")
```

### Custom database transport adapters (Situational)

*Score 2:* [Thiery Laverdure](https://www.youtube.com/watch?v=E4_QSEfInrc&t=1440s) (Laracon speaker, 2)

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

### Database sharding and partitioning (Situational)

*Score 2:* [Tobias Petry](https://www.youtube.com/watch?v=QbsmLXmpYXo&t=1020s) (Laracon speaker, 2)

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

### Durable shard directories (Situational)

*Score 2:* [Tobias Petry](https://www.youtube.com/watch?v=QbsmLXmpYXo&t=1080s) (Laracon speaker, 2)

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

*Score 2:* [Thijs Feryn](https://www.youtube.com/watch?v=VFTeGtPLlS0) (Laracon speaker, 2)

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

*Score 2:* [Thijs Feryn](https://www.youtube.com/watch?v=VFTeGtPLlS0) (Laracon speaker, 2)

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

### Query-level authorization (Situational)

*Score 2:* [Jonathan Reinink](https://laracasts.com/episodes/2558) (Tier 3, 1); [laravel/livewire-starter-kit](https://github.com/laravel/livewire-starter-kit/blob/0f62a26c4e4b401c1300930f47d72a497add8cce/resources/views/pages/settings/security.blade.php) (Tier 3, 1)

- **Smell:** OR-heavy search branches prevent efficient independent index use.
- **Technique:** Express branches as separate queries joined by UNION and fetch the combined result. SQL can plan branches independently while the application makes one request.
- **When to use:** Independent search predicates benefit from distinct indexes.
- **When not to:** Union deduplication and global ordering can be expensive.
- **Packages:** `laravel/fortify`, `laravel/framework`, `livewire/livewire`

Before:
```php
$users = User::where("name", $term)->orWhere("email", $term)->get();
```
After:
```php
$users = User::where("name", $term)
 ->union(User::where("email", $term))->get();
```

### SQL ordering by related data (Situational)

*Score 2:* [Jonathan Reinink](https://laracasts.com/episodes/2566) (Tier 3, 1); [laravel/boost](https://github.com/laravel/boost/blob/d165b9ea6086294a8f1999ac3dcf4e61da267b23/.ai/laravel/skill/laravel-best-practices/rules/advanced-queries.md) (Tier 3, 1)

- **Smell:** Filtering and ordering a large dataset causes costly sorting or scans.
- **Technique:** Create a composite index aligned with the query’s filter and ordering columns. Check the execution plan rather than indexing columns independently without regard to the query.
- **When to use:** Frequently executed list queries have stable filter/sort combinations.
- **When not to:** Indexes cost storage and slow writes; column order matters.
- **Packages:** `laravel/framework`

Before:
```php
$posts = Post::where("user_id", $id)->orderBy("created_at")->get();
```
After:
```php
Schema::table("posts", function (Blueprint $table) {
 $table->index(["user_id", "created_at"]);
});
```

### Subquery-selected relationships (Situational)

*Score 2:* [Jonathan Reinink](https://laracasts.com/episodes/2544) (Tier 3, 1); [laravel/boost](https://github.com/laravel/boost/blob/d165b9ea6086294a8f1999ac3dcf4e61da267b23/.ai/laravel/skill/laravel-best-practices/rules/advanced-queries.md) (Tier 3, 1)

- **Smell:** A listing loads every related record just to display one value.
- **Technique:** Use a correlated subquery in addSelect to fetch the desired related value, ordered and limited in SQL. Avoid hydrating the whole has-many collection.
- **When to use:** Each parent needs one scalar from its latest or otherwise selected child.
- **When not to:** A full related object requires a relationship or another query; subqueries need suitable indexes.
- **Packages:** `laravel/framework`

Before:
```php
$users = User::with("logins")->get();
$date = $users[0]->logins->max("created_at");
```
After:
```php
$users = User::addSelect(["last_login_at" =>
 Login::select("created_at")->whereColumn("user_id", "users.id")
 ->latest()->limit(1)])->get();
```

### Time-series hypertables (Situational)

*Score 2:* [Tobias Petry](https://www.youtube.com/watch?v=cucIWpAenro&t=10520s) (Laracon speaker, 2)

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

*Score 20:* [Nuno Maduro](https://x.com/enunomaduro/status/2071213127840960842) (Tier 1, 3); [Dan Harrin](https://x.com/danjharrin/status/1501598785092435969) (Tier 2, 3); [Taylor Otwell](https://x.com/taylorotwell/status/1555573050954977280) (Tier 1, 3); [Spatie](https://freek.dev/2441-returning-native-types-from-the-laravel-request-class) (Tier 2, 2); [Jason McCreary](https://www.youtube.com/watch?v=Jna0DOgnxds&t=510s) (Laracon speaker, 2); [Kévin Dunglas](https://www.youtube.com/watch?v=DJJt_2V26V8&t=990s) (Laracon speaker, 2); [Tighten](https://tighten.com/insights/request-level-validation/) (Tier 2, 2); [Jeffrey Way](https://laracasts.com/episodes/3881) (Tier 3, 1); [Instructor Team](https://laracasts.com/episodes/3737) (Tier 3, 1); [Jeremy McPeak](https://laracasts.com/episodes/4128) (Tier 3, 1); [Laravel News](https://laravel-news.com/controller-refactor) (Supporting, 0)

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

### Authorization policies (Recommended)

*Score 12:* [@AshAllenDesign](https://ashallendesign.co.uk/blog/unauthorised-access-404) (Tier 2, 3); [Kévin Dunglas](https://www.youtube.com/watch?v=DJJt_2V26V8&t=1380s) (Laracon speaker, 2); [Stephen Rees-Carter](https://securinglaravel.com/in-depth-policy-objects) (Tier 3, 1); [Luke Downing](https://laracasts.com/episodes/3023) (Tier 3, 1); [Jeremy McPeak](https://laracasts.com/episodes/3991) (Tier 3, 1); [Kevin McKee](https://laracasts.com/episodes/4004) (Tier 3, 1); [Jeffrey Way](https://laracasts.com/episodes/3887) (Tier 3, 1); [Matt Stauffer](https://laracasts.com/episodes/3309) (Tier 3, 1); [Simon Vrachliotis](https://laracasts.com/episodes/3673) (Tier 3, 1)

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

### Scoped route model binding (Recommended)

*Score 8:* [Wendell Adriel](https://x.com/wendell_adriel/status/1947608079689760944) (Tier 1, 3); [Taylor Otwell](https://www.youtube.com/watch?v=1P3wLy49t2c&t=2220s) (Laracon speaker (employee), 3); [Stephen Rees-Carter](https://securinglaravel.com/security-tip-scoping-bindings) (Tier 3, 1); [Instructor Team](https://laracasts.com/episodes/3715) (Tier 3, 1)

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

### Reusable validation rules (Recommended)

*Score 7:* [Tim MacDonald](https://x.com/timacdonald87/status/1546308230686867457) (Tier 1, 3); [@laravel](https://laravel.com/docs/13.x/validation#using-rule-objects) (Tier 2, 3); [@joelclermont](https://masteringlaravel.io/daily/2026-01-22-rule-exists-doesnt-work-with-scopes) (Tier 3, 1)

- **Smell:** A complex validation condition is copied as closures or intertwined with request handlers.
- **Technique:** Extract the condition into a class implementing Laravel 13 ValidationRule with validate(string $attribute, mixed $value, Closure $fail), then reuse it in Form Requests or validators. Modernize older invokable-rule examples to the current contract.
- **When to use:** Several fields or endpoints share a domain-specific input constraint.
- **When not to:** Use built-in rules for simple constraints; expensive rules still need careful database and network behavior.
- **Packages:** `laravel/framework`

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

### Composable after-validation checks (Useful)

*Score 6:* [Taylor Otwell](https://x.com/taylorotwell/status/1648339921982357504) (Tier 1, 3); [@laravel](https://laravel.com/docs/13.x/validation#performing-additional-validation-on-form-requests) (Tier 2, 3)

- **Smell:** A validator after callback grows into several unrelated cross-field checks.
- **Technique:** Extract checks into invokable classes and pass an array of them to Validator::after or return them from a FormRequest after method. Laravel 13 runs each check with the validator, allowing focused reuse and tests.
- **When to use:** Cross-field or contextual validation needs several independent checks.
- **When not to:** Simple field constraints should remain ordinary validation rules; avoid repeated expensive queries across checks.
- **Packages:** `laravel/framework`

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

### Precognitive validation (Useful)

*Score 6:* [Tim MacDonald](https://x.com/timacdonald87/status/1571787552256065537) (Tier 1, 3); [Taylor Otwell](https://www.youtube.com/watch?v=1P3wLy49t2c&t=180s) (Laracon speaker (employee), 3)

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

*Score 5:* [Jess Archer](https://www.youtube.com/watch?v=0j6z_oq4cZU&t=1680s) (Laracon speaker (employee), 3); [Tim](https://www.youtube.com/watch?v=NcucthcnGY0&t=1650s) (Laracon speaker, 2)

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

### Route model binding (Useful)

*Score 5:* [Taylor Otwell](https://www.youtube.com/watch?v=1P3wLy49t2c&t=1950s) (Laracon speaker (employee), 3); [Jason McCreary](https://laracasts.com/episodes/3638) (Tier 3, 1); [Jeffrey Way](https://laracasts.com/episodes/3878) (Tier 3, 1)

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

### Fluent validation rule builders (Useful)

*Score 4:* [@laravel](https://laravel.com/docs/13.x/validation#rule-builders) (Tier 2, 3); [laravel/framework](https://github.com/laravel/framework/pull/61833) (Tier 3, 1)

- **Smell:** A large validation array repeats hardcoded parameterized rule strings that are hard to discover in an IDE.
- **Technique:** Replace applicable string rules with the new Rule builders while retaining the existing validation semantics.
- **When to use:** Laravel 13.36.0+ and the team benefits from autocomplete and navigation.
- **When not to:** A short, clear string rule already follows project conventions or the installed minor lacks the builder.
- **Packages:** `laravel/framework`

Before:
```php
'tags.*' => ['distinct:strict'],
'password' => ['confirmed:password_repeat'],
```
After:
```php
'tags.*' => [Rule::distinct()->strict()],
'password' => [Rule::confirmed()->customField('password_repeat')],
```

### AI tool argument validation (Situational)

*Score 3:* [@laravel](https://laravel.com/docs/13.x/ai-sdk#validating-tool-arguments) (Tier 2, 3)

- **Smell:** Model-supplied arguments reach application services without validation.
- **Technique:** Validate the tool Request and use validated values; failure messages are returned to the model for correction.
- **When to use:** Tools accept generated arguments with domain limits.
- **When not to:** Validation does not replace authorization to perform the requested operation.
- **Packages:** `laravel/ai`

Before:
```php
return $this->forecast($request['city'], $request['days']);
```
After:
```php
$data = $request->validate([
    'city' => 'required|string',
    'days' => 'required|integer|max:7',
]);
return $this->forecast($data['city'], $data['days']);
```

### Disjunctive validation rulesets (Situational)

*Score 3:* [@laravel](https://laravel.com/docs/13.x/validation#rule-anyof) (Tier 2, 3)

- **Smell:** Handwritten validation branches accept different formats for the same field.
- **Technique:** Compose alternative rulesets through Rule::anyOf while keeping shared requirements outside it.
- **When to use:** One field allows an email or another well-defined identifier format.
- **When not to:** All constraints must hold together; use a normal rules array.
- **Packages:** `laravel/framework`

Before:
```php
if (! filter_var($name, FILTER_VALIDATE_EMAIL)
    && ! preg_match('/^[a-z0-9_-]{6,}$/i', $name)) abort(422);
```
After:
```php
'username' => ['required', Rule::anyOf([
    ['string', 'email'],
    ['string', 'alpha_dash:ascii', 'min:6'],
])],
```

### Form Request input normalization (Situational)

*Score 3:* [Tim MacDonald](https://www.youtube.com/watch?v=MMc2TzBY6l4&t=750s) (Laracon speaker (employee), 3)

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

### Per-element validation rule factories (Situational)

*Score 3:* [@laravel](https://laravel.com/docs/13.x/validation#accessing-nested-array-data) (Tier 2, 3)

- **Smell:** Manual loops build validators for each nested item and assemble their errors.
- **Technique:** Use Rule::forEach to derive an array of rules from each value and expanded attribute name.
- **When to use:** Nested items need value-dependent rules or permissions.
- **When not to:** A wildcard rule is simpler when every element has identical constraints.
- **Packages:** `laravel/framework`

Before:
```php
foreach ($request->input('companies') as $row) {
    validator($row, ['id' => [new HasPermission('manage-company', $row['id'])]])->validate();
}
```
After:
```php
'companies.*.id' => Rule::forEach(fn ($value, $attribute) => [
    Rule::exists(Company::class, 'id'),
    new HasPermission('manage-company', $value),
]),
```

### Shared password rule defaults (Situational)

*Score 3:* [Stephen Rees-Carter](https://x.com/valorin/status/1846644787840950516) (Tier 2, 3)

- **Smell:** Registration, password reset, and profile forms duplicate password complexity rules.
- **Technique:** Configure Password::defaults once in a service provider, then use Password::defaults in each form. This centralizes the application password validation policy without duplicating each fluent rule chain.
- **When to use:** Multiple endpoints accept new passwords.
- **When not to:** Different security contexts may require explicit overrides; shared validation does not replace secure password hashing.

Before:
```php
"password" => [Password::min(12)->mixedCase()->numbers()],
// Repeated in several forms.
```
After:
```php
Password::defaults(fn () => Password::min(12)->mixedCase()->numbers());
// Each form:
$rules = ["password" => ["required", Password::defaults()]];
```

### Tenant-scoped validation (Situational)

*Score 3:* [@filamentphp](https://github.com/filamentphp/filament/blob/7c5c222be3e92439357213f8274d1548c8cb769e/docs/07-users/03-tenancy.md) (Tier 2, 3)

- **Smell:** A globally unique lookup rejects a valid value owned by another tenant or accepts a related record from another tenant.
- **Technique:** Use scopedUnique/scopedExists so validation queries include model scopes in the current tenant-aware context.
- **When to use:** Filament fields whose uniqueness or foreign-key choices are tenant-local.
- **When not to:** Globally unique identities or contexts where the model has no correct tenant scope.
- **Packages:** `filament/filament`, `filament/forms`

Before:
```php
TextInput::make('email')->unique();
Select::make('author_id')->exists();
```
After:
```php
TextInput::make('email')->scopedUnique();
Select::make('author_id')->scopedExists();
```

### Enum route binding (Situational)

*Score 2:* [laravel/framework PRs & discussions](https://github.com/laravel/framework/pull/40281) (Tier 2, 2)

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

### Concurrent independent tasks (Recommended)

*Score 19:* [Wendell Adriel](https://x.com/wendell_adriel/status/1950522528138412216) (Tier 1, 3); [Mohamed Said](https://laracasts.com/episodes/2808) (Tier 1, 3); [Aaron Francis](https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=13170s) (Laracon speaker (employee), 3); [@laravel](https://laravel.com/docs/13.x/concurrency#task-timeouts) (Tier 2, 3); [Spatie](https://www.youtube.com/watch?v=klMO4vn540A) (Tier 2, 2); [Mateus Guimarães](https://www.youtube.com/watch?v=dOXXL-98Zbs) (Laracon speaker, 2); [Tim](https://www.youtube.com/watch?v=NcucthcnGY0&t=90s) (Laracon speaker, 2); [laravel/boost](https://github.com/laravel/boost/blob/d165b9ea6086294a8f1999ac3dcf4e61da267b23/.ai/laravel/skill/laravel-best-practices/rules/http-client.md) (Tier 3, 1)

- **Smell:** Independent expensive tasks are serialized or exceed one application server capacity.
- **Technique:** Dispatch a bounded group of independent Lambda executions and collect results or use shared result storage. Laravel 13 needs an external integration for Lambda fan-out; its native Concurrency drivers do not distribute work to Lambda.
- **When to use:** Independent work benefits from remote workers beyond local capacity.
- **When not to:** Quotas, cold starts, result coordination, and per-invocation cost can outweigh the gain.
- **Packages:** `hammerstone/sidecar`, `laravel/framework`, `laravel/octane`, `spatie/fork`

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

### Events and listeners (Recommended)

*Score 19:* [Ashley Hindle](https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=33480s) (Laracon speaker (employee), 3); [Spatie](https://freek.dev/2442-strategies-for-making-laravel-packages-customizable) (Tier 2, 2); [Mateus Guimarães](https://www.youtube.com/watch?v=WqL6wgH1fKs) (Laracon speaker, 2); [Bobby Bouwmann](https://www.youtube.com/watch?v=-jo6QjeaJXQ&t=900s) (Laracon speaker, 2); [Ryuta Hamasaki](https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=5580s) (Laracon speaker, 2); [Mary Perry](https://www.youtube.com/watch?v=vsxoaTgtyjw) (Laracon speaker, 2); [Jason McCreary](https://www.youtube.com/watch?v=LaKEFjA25r4&t=870s) (Laracon speaker, 2); [Josh Hanley](https://www.youtube.com/watch?v=evEU7ZAROlU) (Laracon speaker, 2); [Harris Raftopoulos](https://www.youtube.com/watch?v=E9t2lVn0z8A&t=690s) (Laracon speaker, 2); [Laravel News](https://laravel-news.com/working-with-laravel-model-events) (Supporting, 0)

- **Smell:** UI methods and backend mutations use inconsistent operation vocabulary and duplicate workflow logic.
- **Technique:** Give the user operation a domain name and let a Livewire action invoke the corresponding accepted domain event. In a Verbs implementation, validation and authorization precede event recording, while event processing owns state and projection changes.
- **When to use:** A feature already uses event-sourced domain operations across entry points.
- **When not to:** Straightforward CRUD can stay in a Livewire action; native browser events are not durable domain events.
- **Packages:** `harris21/laravel-fuse`, `hirethunk/verbs`, `laravel/slack-notification-channel`, `livewire/livewire`, `spatie/laravel-url-ai-transformer`

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

*Score 13:* [Josh Cirre](https://laracasts.com/episodes/3331) (Tier 1, 3); [Joe Tannenbaum](https://www.youtube.com/watch?v=v7spVsbN_Qc) (Laracon speaker (employee), 3); [Devansh Bawari](https://www.youtube.com/watch?v=eJJsGhu9gtI) (Laracon speaker, 2); [Marcel Pociot](https://www.youtube.com/watch?v=rk29Hyk59JY) (Laracon speaker, 2); [Jeffrey Way](https://laracasts.com/episodes/3890) (Tier 3, 1); [Luke Downing](https://laracasts.com/episodes/2855) (Tier 3, 1); [Jeremy McPeak](https://laracasts.com/episodes/4126) (Tier 3, 1); [Laravel News](https://laravel-news.com/controller-refactor) (Supporting, 0)

- **Smell:** Catalog updates wait synchronously for expensive price, inventory, or search calculations.
- **Technique:** Extract expensive derivation into queued jobs with an asynchronous connection and active workers. Laravel 13 after-commit dispatch ensures jobs depending on transactional writes see committed data.
- **When to use:** Derived work can complete after the HTTP response without violating business correctness.
- **When not to:** Queues add lag and retry coordination; immediate authoritative inventory checks may still need synchronous transactions.
- **Packages:** `laravel/nova`

Before:
```php
$product->save();
$indexer->rebuildPrice($product);
$indexer->rebuildSearch($product);
```
After:
```php
$product->save();
RebuildProductIndexes::dispatch($product->id)->afterCommit();
// asynchronous connection, with queue workers running
```

### Response-deferred side effects (Recommended)

*Score 10:* [Wendell Adriel](https://x.com/wendell_adriel/status/1978070012842635426) (Tier 1, 3); [Taylor Otwell](https://x.com/taylorotwell/status/1980664560492376378) (Tier 1, 3); [laravel/framework PRs & discussions](https://github.com/laravel/framework/pull/52801) (Tier 2, 2); [Tim](https://www.youtube.com/watch?v=NcucthcnGY0&t=480s) (Laracon speaker, 2)

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

### Dependency injection (Recommended)

*Score 9:* [Dan Harrin](https://x.com/danjharrin/status/1501598779056734213) (Tier 2, 3); [Pascal Baljet](https://x.com/pascalbaljet/status/1760794330455818526) (Tier 1, 3); [Mary Perry](https://www.youtube.com/watch?v=vsxoaTgtyjw) (Laracon speaker, 2); [Mastering Laravel / Aaron Saray](https://masteringlaravel.io/daily/2026-09-15-the-right-place-to-inject-a-service-into-a-queued-job) (Tier 3, 1)

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

### Execution context propagation (Recommended)

*Score 9:* [@laravel](https://laravel.com/docs/13.x/context#how-it-works) (Tier 2, 3); [Spatie](https://freek.dev/3203-laravel-context-the-hidden-information-that-follows-your-code) (Tier 2, 2); [laravel/framework PRs & discussions](https://github.com/laravel/framework/pull/49730) (Tier 2, 2); [Jeremy McPeak](https://laracasts.com/episodes/3262) (Tier 3, 1); [laravel/boost](https://github.com/laravel/boost/blob/75c269a8a53e931c7252e7cd562e1a209356a7c6/.ai/laravel/skill/laravel-best-practices/rules/architecture.md) (Tier 3, 1)

- **Smell:** Trace IDs and request metadata are threaded through unrelated method signatures or disappear at queue boundaries.
- **Technique:** Place execution metadata in Laravel Context at the request boundary and retrieve it where needed. Context integrates with logging and queue propagation, keeping cross-cutting metadata out of business method parameters.
- **When to use:** Logs and asynchronous work need consistent correlation or initiator metadata.
- **When not to:** Do not use ambient context for essential domain inputs or authorization decisions; hidden dependencies make isolated testing harder.
- **Packages:** `laravel/framework`, `laravel/nightwatch`, `laravel/pennant`

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

### Bounded transient retries (Recommended)

*Score 8:* [@laravel](https://laravel.com/docs/13.x/horizon) (Tier 2, 3); [Joshua Alphonse](https://www.youtube.com/watch?v=HhcieSa52SQ) (Laracon speaker, 2); [Zuzana Kunckova](https://www.youtube.com/watch?v=Emm559sI6tI&t=150s) (Laracon speaker, 2); [Luke Downing](https://laracasts.com/episodes/3114) (Tier 3, 1)

- **Smell:** Transient external failures immediately discard an operation that could recover after a short delay.
- **Technique:** Use Laravel retry() or an HTTP-specific retry helper with limited attempts and delays around recoverable failures. Distinguish terminal errors and define exhaustion behavior; HTTP responses need explicit status handling when using generic retry().
- **When to use:** A transient dependency failure is safe to retry.
- **When not to:** Retries increase latency and load; non-idempotent operations can duplicate side effects and persistent failures need another policy.
- **Packages:** `laravel/horizon`

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

### Idempotency keys (Recommended)

*Score 8:* [Wendell Adriel](https://x.com/wendell_adriel/status/1968643415861608767) (Tier 1, 3); [Joshua Alphonse](https://www.youtube.com/watch?v=HhcieSa52SQ) (Laracon speaker, 2); [wendelladriel/laravel-idempotency](https://github.com/WendellAdriel/laravel-idempotency) (Tier 3, 1); [Simon Vrachliotis](https://laracasts.com/episodes/3656) (Tier 3, 1); [Instructor Team](https://laracasts.com/episodes/3708) (Tier 3, 1)

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

### Queue workload partitioning and scaling (Recommended)

*Score 8:* [Devon Garbalosa](https://www.youtube.com/watch?v=toSMnEINXAo&t=319s) (Laracon speaker (employee), 3); [@laravel](https://laravel.com/docs/13.x/horizon) (Tier 2, 3); [Guus Leeuw](https://www.youtube.com/watch?v=rpvhD-twpY4) (Laracon speaker, 2)

- **Smell:** Long-running background jobs compete with urgent mail or web-request capacity.
- **Technique:** Place jobs on queues separated by urgency and runtime, and allocate dedicated workers to those queues. Run background workers on separate compute when they compete with web requests.
- **When to use:** Mixed background workloads require different latency or resource guarantees.
- **When not to:** Multiple queues and worker pools add tuning costs; priority ordering alone can starve low-priority work.
- **Packages:** `aws/aws-sdk-php`, `laravel/horizon`

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

*Score 7:* [Jess Archer](https://www.youtube.com/watch?v=_jjvaFWWKqg) (Laracon speaker (employee), 3); [Spatie](https://freek.dev/3058-whats-new-in-laravel-activitylog-v5) (Tier 2, 2); [Seb Armand](https://www.youtube.com/watch?v=9mDkf4popb8&t=1470s) (Laracon speaker, 2)

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

### Signed URLs (temporary access links) (Recommended)

*Score 7:* [Wendell Adriel](https://x.com/wendell_adriel/status/1970463766492983566) (Tier 1, 3); [@laravel](https://docs.vapor.build/projects/development) (Tier 2, 3); [Stephen Rees-Carter](https://securinglaravel.com/security-tip-restricting-local-file) (Tier 3, 1)

- **Smell:** Emailed downloads require custom token tables or expose permanent unauthenticated file URLs.
- **Technique:** Create a temporarySignedRoute for a download endpoint and apply signed middleware before serving a private file. Bind the link to a recipient or add authorization where downloads require identity checks.
- **When to use:** An expiring bearer link is an appropriate access model for an emailed file.
- **When not to:** Anyone possessing the link can use it unless identity is checked; revocation needs additional state.
- **Packages:** `laravel/vapor-core`, `league/flysystem-aws-s3-v3`

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

### WebSocket push updates (Useful)

*Score 6:* [Joe Dixon](https://laracasts.com/episodes/3403) (Tier 1, 3); [Josh Cirre](https://laracasts.com/episodes/3342) (Tier 1, 3)

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

### Idempotent handlers (Useful)

*Score 5:* [Mohamed Said](https://themsaid.com/defensive-programming-the-byzantine-generals-problem) (Tier 1, 3); [Chris Fidao](https://www.youtube.com/watch?v=fymjefGMUas) (Laracon speaker, 2)

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

### Model observers (Useful)

*Score 5:* [Christoph Rumpel](https://christoph-rumpel.com/2024/9/all-laravel-php-attributes-at-your-disposal) (Tier 1, 3); [Mary Perry](https://www.youtube.com/watch?v=w7jKcbVcI54) (Laracon speaker, 2); [Laravel News](https://laravel-news.com/working-with-laravel-model-events) (Supporting, 0)

- **Smell:** Related model lifecycle reactions are scattered across callers or model boot closures.
- **Technique:** Group model event reactions in an observer and register it with Model::observe() or ObservedBy. Use after-commit handling when side effects require committed data, and remember that bulk update/delete queries bypass per-model events.
- **When to use:** Behavior consistently follows a model lifecycle transition.
- **When not to:** Observers hide side effects; explicit use cases are clearer for workflow-specific reactions and bulk writes need another path.

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

### Queued webhook processing (Useful)

*Score 5:* [Christoph Rumpel](https://laracasts.com/episodes/2895) (Tier 1, 3); [Tighten](https://tighten.com/insights/bol-ep4-ben-holmen-the-boutique-hub) (Tier 2, 2)

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

### Tenant-aware runtime context (Useful)

*Score 5:* [@laravel](https://laravel.com/docs/13.x/ai-sdk#on-demand-providers) (Tier 2, 3); [Spatie](https://www.youtube.com/watch?v=JmEmrLtMB_A&t=570s) (Tier 2, 2)

- **Smell:** Business methods repeatedly change database, cache, and filesystem settings themselves.
- **Technique:** Register switching tasks with makeCurrent and forgetCurrent methods to prepare and restore tenant resources. Refresh resolved service instances when configuration changes so ordinary service calls use the active tenant.
- **When to use:** Several resources must change together when tenant context changes.
- **When not to:** Switching mutates process-wide configuration and does not provide simultaneous independent contexts in one process.
- **Packages:** `laravel/ai`, `spatie/laravel-multitenancy`

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

### Unique queued jobs (Useful)

*Score 5:* [Tim MacDonald](https://tim.macdonald.au/unique-jobs-and-reserving-resources-on-the-queue) (Tier 1, 3); [Seb Armand](https://www.youtube.com/watch?v=9mDkf4popb8&t=1350s) (Laracon speaker, 2)

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

*Score 4:* [Wendell Adriel](https://x.com/wendell_adriel/status/1973719258354901440) (Tier 1, 3); [Sabatino Masala](https://x.com/SabatinoMasala/status/1989280428977721704) (Tier 3, 1)

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

*Score 4:* [Joshua Alphonse](https://www.youtube.com/watch?v=HhcieSa52SQ) (Laracon speaker, 2); [Bert De Swaef](https://www.youtube.com/watch?v=xxAv6s5045Y) (Laracon speaker, 2)

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

*Score 4:* [Joshua Alphonse](https://www.youtube.com/watch?v=HhcieSa52SQ) (Laracon speaker, 2); [Bert De Swaef](https://www.youtube.com/watch?v=xxAv6s5045Y) (Laracon speaker, 2)

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

*Score 4:* [laravel/framework PRs & discussions](https://github.com/laravel/framework/pull/48633) (Tier 2, 2); [Seb Armand](https://www.youtube.com/watch?v=9mDkf4popb8&t=1410s) (Laracon speaker, 2)

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

### Bounded HTTP batches (Situational)

*Score 3:* [@laravel](https://laravel.com/docs/13.x/http-client#request-batching) (Tier 2, 3)

- **Smell:** A large fan-out issues sequential requests or overwhelms a service with unlimited parallel calls.
- **Technique:** Build an Http batch and cap concurrency before sending it.
- **When to use:** Independent HTTP calls need bounded parallelism and optional completion callbacks.
- **When not to:** One request or dependent request chains gain nothing from batching.
- **Packages:** `laravel/framework`

Before:
```php
$a = Http::get($firstUrl);
$b = Http::get($secondUrl);
```
After:
```php
$responses = Http::batch(fn (Batch $batch) => [
    $batch->as('first')->get($firstUrl),
    $batch->as('second')->get($secondUrl),
])->concurrency(2)->send();
```

### Conditional job middleware (Situational)

*Score 3:* [@laravel](https://laravel.com/docs/13.x/queues#skipping-jobs) (Tier 2, 3)

- **Smell:** Repeated precondition branches obscure job processing logic.
- **Technique:** Return Skip::when or Skip::unless from middleware to delete jobs whose work is unnecessary.
- **When to use:** The same skip rule recurs or deserves separate composition.
- **When not to:** Skipping would bypass required cleanup or failure reporting.
- **Packages:** `laravel/framework`

Before:
```php
public function handle(): void {
    if ($this->order->cancelled) return;
    $this->process();
}
```
After:
```php
public function middleware(): array {
    return [Skip::when(fn () => $this->order->cancelled)];
}
public function handle(): void { $this->process(); }
```

### Deferred HTTP batches (Situational)

*Score 3:* [Wendell Adriel](https://x.com/wendell_adriel/status/1978427159153668577) (Tier 1, 3)

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

*Score 3:* [Chris Fidao](https://www.youtube.com/watch?v=fymjefGMUas) (Laracon speaker, 2); [Simon Vrachliotis](https://laracasts.com/episodes/3670) (Tier 3, 1)

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

### Distributed concurrency funnels (Situational)

*Score 3:* [@laravel](https://laravel.com/docs/13.x/cache#concurrency-limiting) (Tier 2, 3)

- **Smell:** Unbounded simultaneous work overloads a shared external resource.
- **Technique:** Use a named cache funnel with a slot limit, expiry and acquisition timeout.
- **When to use:** Multiple processes need a shared concurrency ceiling.
- **When not to:** A rate limit counts events over time; a funnel controls simultaneous work and needs lock support.
- **Packages:** `laravel/framework`

Before:
```php
$service->generateReport();
```
After:
```php
Cache::funnel('reports')
    ->limit(3)->releaseAfter(60)->block(10)
    ->then(fn () => $service->generateReport());
```

### Encrypted queue payloads (Situational)

*Score 3:* [@AshAllenDesign](https://ashallendesign.co.uk/blog/encrypting-queued-classes-in-laravel) (Tier 2, 3)

- **Smell:** A queued OTP notification leaves its secret visible in the serialized command stored by the queue backend.
- **Technique:** Implement the ShouldBeEncrypted marker alongside ShouldQueue.
- **When to use:** Queued serialized data contains secrets and workers share the required application encryption key.
- **When not to:** This does not hide all queue metadata or protect a host where the application key is compromised.

Before:
```php
class SendOtp implements ShouldQueue {
    public function __construct(public string $otp) {}
}
```
After:
```php
class SendOtp implements ShouldQueue, ShouldBeEncrypted {
    public function __construct(public string $otp) {}
}
```

### Event-based job profiling (Situational)

*Score 3:* [@laravel](https://laravel.com/cloud/docs/knowledge-base/sizing-queue-worker-memory) (Tier 2, 3)

- **Smell:** Worker-level memory output does not identify which job class determines allocation.
- **Technique:** Attach a temporary JobAttempted listener that records the class and peak memory, run representative jobs, then remove it.
- **When to use:** Choosing a managed-queue worker size from evidence.
- **When not to:** Persistent production instrumentation is needed; use supported monitoring instead.

Before:
```php
// Process-wide reading alone:
php artisan queue:work -v --queue=exports
```
After:
```php
Event::listen(function (JobAttempted $event) {
    Log::info('Job peak memory', [
        'job' => $event->job->resolveName(),
        'peak_mb' => round(memory_get_peak_usage(true) / 1024 / 1024, 1),
    ]);
});
```

### Explicit queue connection routing (Situational)

*Score 3:* [@laravel](https://laravel.com/cloud/docs/queues) (Tier 2, 3)

- **Smell:** Introducing a Cloud managed queue silently reroutes previously implicit database or Redis jobs.
- **Technique:** Keep the existing default connection and mark only the migrated jobs with onConnection cloud.
- **When to use:** A staged migration must run both existing and managed drivers.
- **When not to:** All jobs intentionally move to the managed queue together.
- **Packages:** `aws/aws-sdk-php`

Before:
```php
// Formerly routed to the default database connection:
SendInvoice::dispatch($invoice);
```
After:
```php
# Environment: preserve the existing default
QUEUE_CONNECTION=database
// PHP: explicitly route the migrated job
SendInvoice::dispatch($invoice)->onConnection('cloud');
```

### Job cancellation middleware (Situational)

*Score 3:* [@laravel](https://laravel.com/docs/13.x/queues#cancelling-batches) (Tier 2, 3)

- **Smell:** Every batch job repeats an early return for batch cancellation.
- **Technique:** Attach SkipIfBatchCancelled middleware and remove the repeated cancellation branch from handle.
- **When to use:** Many jobs share the same batch cancellation behavior.
- **When not to:** A job must do cleanup even after cancellation or is not part of a batch.
- **Packages:** `laravel/framework`

Before:
```php
public function handle(): void {
    if ($this->batch()?->cancelled()) return;
    $this->process();
}
```
After:
```php
public function middleware(): array {
    return [new SkipIfBatchCancelled];
}
public function handle(): void { $this->process(); }
```

### Logical queue name mapping (Situational)

*Score 3:* [@laravel](https://docs.vapor.build/resources/queues) (Tier 2, 3)

- **Smell:** Dispatch calls embed production or staging suffixes and require conditional environment-specific names.
- **Technique:** Put the environment suffix in SQS_SUFFIX and queue connection configuration, then dispatch to stable logical names.
- **When to use:** Vapor environments need unique physical SQS names but shared application code.
- **When not to:** The application intentionally selects queues in another environment or has incompatible naming conventions.
- **Packages:** `laravel/vapor-core`

Before:
```php
SendInvoice::dispatch($invoice)
    ->onQueue(app()->isProduction() ? 'invoices-production' : 'invoices-staging');
```
After:
```php
# Environment
SQS_SUFFIX=-production
// config/queue.php sqs connection:
'suffix' => env('SQS_SUFFIX'),
// Dispatch:
SendInvoice::dispatch($invoice)->onQueue('invoices');
```

### Model lifecycle broadcasting (Situational)

*Score 3:* [Joe Dixon](https://laracasts.com/episodes/3408) (Tier 1, 3)

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

*Score 3:* [Pascal Baljet](https://x.com/pascalbaljet/status/1496073019340513285) (Tier 1, 3)

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

### Named concurrent task results (Situational)

*Score 3:* [@laravel](https://laravel.com/docs/13.x/concurrency#named-results) (Tier 2, 3)

- **Smell:** Positional result destructuring becomes fragile as tasks are reordered.
- **Technique:** Use an associative array of closures so results retain task names.
- **When to use:** Several independent tasks return values with different meanings.
- **When not to:** A tiny fixed pair of results is already clear through destructuring.
- **Packages:** `laravel/framework`

Before:
```php
[$users, $orders] = Concurrency::run([$countUsers, $countOrders]);
```
After:
```php
$counts = Concurrency::run([
    'users' => $countUsers,
    'orders' => $countOrders,
]);
$users = $counts['users'];
```

### Opaque upload storage names (Situational)

*Score 3:* [@filamentphp](https://github.com/filamentphp/filament/blob/7c5c222be3e92439357213f8274d1548c8cb769e/packages/forms/docs/09-file-upload.md) (Tier 2, 3)

- **Smell:** Original client filenames become executable paths or overwrite other users' files.
- **Technique:** Store files under generated names and retain original names in a separate model attribute.
- **When to use:** User uploads where original names matter to the UI.
- **When not to:** Controlled trusted import storage with a deliberately managed naming scheme.
- **Packages:** `filament/forms`

Before:
```php
FileUpload::make('attachment')->preserveFilenames();
```
After:
```php
FileUpload::make('attachment')
    ->storeFileNamesIn('attachment_original_name')
    ->preventFilePathTampering();
```

### Persist-and-reconcile queue delivery (Situational)

*Score 3:* [Mohamed Said](https://themsaid.com/defensive-programming-the-byzantine-generals-problem) (Tier 1, 3)

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

*Score 3:* [Taylor Otwell](https://x.com/taylorotwell/status/1978107436297794004) (Tier 1, 3)

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

*Score 3:* [James Brooks](https://james.brooks.page/blog/injecting-additional-data-into-laravel-queued-jobs) (Tier 1, 3)

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

*Score 3:* [Joe Dixon](https://www.youtube.com/watch?v=60g-Z-dAWNk&t=960s) (Laracon speaker (employee), 3)

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

### Replay-safe event-sourcing reactors (Situational)

*Score 3:* [Daniel Coulbourne](https://www.youtube.com/watch?v=-mb2L97Yq88&t=1770s) (Laracon speaker, 2); [@hirethunk](https://github.com/hirethunk/verbs/blob/13ee038c0a/docs/replaying-events.md) (Tier 3, 1)

- **Smell:** Rebuilding projections sends emails, charges payments or fires duplicate downstream events.
- **Technique:** Separate replayable state/projection updates from one-time work and guard the latter with Verbs::unlessReplaying() or a supported once hook. Replay suppression is distinct from transaction after-commit timing or retry idempotency.
- **When to use:** Event handlers can run again during replay.
- **When not to:** An ordinary Laravel listener without event-store replay does not need this package guard; retries still require separate protection.
- **Packages:** `hirethunk/verbs`

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

### Resilient noncritical broadcasting (Situational)

*Score 3:* [Taylor Otwell](https://x.com/taylorotwell/status/1932810338262765569) (Tier 1, 3)

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

*Score 3:* [Tim MacDonald](https://tim.macdonald.au/unique-jobs-and-reserving-resources-on-the-queue) (Tier 1, 3)

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

*Score 3:* [Seb Armand](https://www.youtube.com/watch?v=9mDkf4popb8&t=1530s) (Laracon speaker, 2); [Jeremy McPeak](https://laracasts.com/episodes/4045) (Tier 3, 1)

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

### Semantic queue tags (Situational)

*Score 3:* [@laravel](https://laravel.com/docs/13.x/horizon) (Tier 2, 3)

- **Smell:** Operational searches cannot correlate jobs or queued listeners with business entities.
- **Technique:** Implement tags on the queued object; for listeners accept the event argument and derive tags from its data.
- **When to use:** Operators need to find work for a tenant, model, or business operation.
- **When not to:** Existing Eloquent automatic tags already provide the necessary correlation.
- **Packages:** `laravel/horizon`

Before:
```php
class RenderVideo implements ShouldQueue {
    // No domain-specific tags.
}
```
After:
```php
class RenderVideo implements ShouldQueue {
    public function tags(): array {
        return ['render', 'video:'.$this->video->id];
    }
}
```

### Single-pass multi-projector replay (Situational)

*Score 3:* [@brendt_gd](https://stitcher.io/blog/processing-11-million-rows) (Tier 2, 3)

- **Smell:** Every projector rereads the complete event table.
- **Technique:** Read each event batch once and pass it to all initialized projectors.
- **When to use:** Multiple projectors share the same ordered immutable event stream.
- **When not to:** Projectors require different data selections or cannot safely share the same replay lifecycle.

Before:
```php
foreach ($projectors as $projector) {
    StoredEvent::chunkById(500, fn ($events) => $projector->replayBatch($events));
}
```
After:
```php
StoredEvent::chunkById(500, function ($events) use ($projectors) {
    foreach ($projectors as $projector) $projector->replayBatch($events);
});
```

### Socket command listeners (Situational)

*Score 3:* [Joe Dixon](https://www.youtube.com/watch?v=yrL5eCMpqtc) (Laracon speaker (employee), 3)

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

*Score 3:* [Jess Archer](https://www.youtube.com/watch?v=uZLkGuylXDQ&t=2232s) (Laracon speaker (employee), 3)

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

*Score 3:* [Ashley Hindle](https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=33480s) (Laracon speaker (employee), 3)

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

*Score 2:* [Harris Raftopoulos](https://www.youtube.com/watch?v=E9t2lVn0z8A&t=210s) (Laracon speaker, 2)

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

*Score 2:* [Ryan Lovett](https://www.youtube.com/watch?v=AJ8MOFoDeP0&t=1320s) (Laracon speaker, 2)

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

*Score 2:* [Bert De Swaef](https://www.youtube.com/watch?v=xxAv6s5045Y) (Laracon speaker, 2)

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

*Score 2:* [Spatie](https://freek.dev/3058-whats-new-in-laravel-activitylog-v5) (Tier 2, 2)

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

*Score 2:* [Daniel Coulbourne](https://www.youtube.com/watch?v=YJmuKPk3d9M&t=20462s) (Laracon speaker, 2); [Laravel News](https://laravel-news.com/managing-api-rate-limits-in-laravel-through-job-throttling) (Supporting, 0)

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

*Score 2:* [laravel/framework PRs & discussions](https://github.com/laravel/framework/pull/59507) (Tier 2, 2)

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

*Score 2:* [laravel/framework PRs & discussions](https://github.com/laravel/framework/pull/47989) (Tier 2, 2)

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

*Score 2:* [Bobby Bouwmann](https://www.youtube.com/watch?v=ru-0QSciNvU) (Laracon speaker, 2)

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

### Stable stored event aliases (Situational)

*Score 2:* [Spatie](https://github.com/spatie/laravel-event-sourcing/blob/2964c03412d4378cc899e7eadbb1569f572a1133/docs/advanced-usage/using-aliases-for-stored-event-classes.md) (Tier 2, 2)

- **Smell:** Renaming an event class makes historical rows point to a class that no longer exists.
- **Technique:** Store stable event aliases through event_class_map and keep the map synchronized with class moves.
- **When to use:** Long-lived event streams whose classes may be refactored.
- **When not to:** Do not assume adding a map automatically migrates old rows already storing FQCNs.
- **Packages:** `spatie/laravel-event-sourcing`

Before:
```php
// Stored event_class couples history to a PHP namespace.
'App\Events\MoneyAddedEvent'
```
After:
```php
// config/event-sourcing.php
'event_class_map' => ['money_added' => MoneyAddedEvent::class],
'enforce_event_class_map' => true,
```

### State-transition and coalesced broadcasts (Situational)

*Score 2:* [Bert De Swaef](https://www.youtube.com/watch?v=xxAv6s5045Y) (Laracon speaker, 2)

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

*Score 13:* [Tighten](https://tighten.com/insights/extensible-blade-components) (Tier 2, 2); [Spatie](https://freek.dev/2640-reusable-input-components-without-livewire-with-livewire) (Tier 2, 2); [Caleb Porzio](https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=24420s) (Laracon speaker, 2); [Duilio Palacios](https://www.youtube.com/watch?v=1sixoOQliZI&t=540s) (Laracon speaker, 2); [Andre Madarang](https://laracasts.com/episodes/2529) (Tier 3, 1); [Jeffrey Way](https://laracasts.com/episodes/3873) (Tier 3, 1); [Jeremy McPeak](https://laracasts.com/episodes/3360) (Tier 3, 1); [Instructor Team](https://laracasts.com/episodes/3696) (Tier 3, 1); [@benjamincrozat](https://benjamincrozat.com/laravel-blade-components-layouts-props-slots) (Tier 3, 1)

- **Smell:** Repeated input or layout markup drifts across templates and mixes styling with page content.
- **Technique:** Extract shared markup into Blade components with explicit props and slots, using named slots for variable regions. Forward the attribute bag so Livewire bindings and other caller attributes work without duplicating templates.
- **When to use:** Several views share the same visual and behavioral structure.
- **When not to:** One-off markup may be clearer inline; excessive tiny components make templates harder to follow.
- **Packages:** `livewire/flux`, `livewire/livewire`, `spatie/laravel-support-bubble`

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

### Independent rendering islands (Recommended)

*Score 10:* [Taylor Otwell](https://www.youtube.com/watch?v=1P3wLy49t2c&t=3120s) (Laracon speaker (employee), 3); [Caleb Porzio](https://www.youtube.com/watch?v=M60-nxlrePc&t=2100s) (Laracon speaker, 2); [Lucas Giovanny](https://www.youtube.com/watch?v=33CIhvkAPEY&t=1590s) (Laracon speaker, 2); [Bert De Swaef](https://www.youtube.com/watch?v=xxAv6s5045Y) (Laracon speaker, 2); [@livewire](https://github.com/livewire/livewire/blob/6c7be5381c960bbaa51684b16f35a38bebfbb63b/docs/nesting.md) (Tier 3, 1)

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

### Generated typed route helpers (Recommended)

*Score 9:* [Joe Tannenbaum](https://laracasts.com/episodes/3866) (Tier 1, 3); [Tim MacDonald](https://x.com/timacdonald87/status/1908263115088945286) (Tier 1, 3); [Spatie](https://www.youtube.com/watch?v=CrO_7Df1cBc&t=1290s) (Tier 2, 2); [Instructor Team](https://laracasts.com/episodes/3838) (Tier 3, 1)

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

### Client-local UI state (Alpine) (Recommended)

*Score 8:* [Wendell Adriel](https://x.com/wendell_adriel/status/1693951526312554803) (Tier 1, 3); [Caleb Porzio](https://www.youtube.com/watch?v=U-N8Qqq02b0) (Laracon speaker, 2); [Philo](https://www.youtube.com/watch?v=MbynDaPaBt4&t=150s) (Laracon speaker, 2); [Kevin McKee](https://laracasts.com/episodes/4005) (Tier 3, 1)

- **Smell:** A purely client-side interaction makes an unnecessary server round trip.
- **Technique:** Keep the browser action with its Livewire component through a Js method returning JavaScript. Current Livewire on Laravel 13 executes this action in the browser when called through $wire.
- **When to use:** An interaction only clears fields or changes local browser state.
- **When not to:** Server data, persistence, validation, and authorization require server execution.
- **Packages:** `alpinejs`, `filament/filament`, `livewire/livewire`

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

### Component file colocation (Recommended)

*Score 8:* [Taylor Otwell](https://www.youtube.com/watch?v=1P3wLy49t2c&t=2460s) (Laracon speaker (employee), 3); [Caleb Porzio](https://www.youtube.com/watch?v=M60-nxlrePc&t=270s) (Laracon speaker, 2); [Simon Vrachliotis](https://www.youtube.com/watch?v=vZC-rVjuiLA&t=492s) (Laracon speaker, 2); [Jeremy McPeak](https://laracasts.com/episodes/3767) (Tier 3, 1)

- **Smell:** A change to one component requires searching disconnected PHP, template, and JavaScript folders.
- **Technique:** Use Livewire 4 single-file components for compact units or collocated multi-file components when complexity grows. Keep component-specific scripts and tests adjacent so their lifecycle and responsibility remain clear.
- **When to use:** Component changes frequently span PHP, Blade, and JavaScript.
- **When not to:** Existing team conventions or shared assets may favor separate folders; large single files can become harder to navigate.
- **Packages:** `livewire/livewire`, `react`

Before:
```php
app/Livewire/Counter.php
resources/views/livewire/counter.blade.php
resources/js/counter.js
```
After:
```php
resources/views/components/counter/
  counter.php
  counter.blade.php
  counter.js
  counter.test.php
```

### Debounced input synchronization (Recommended)

*Score 8:* [Taylor Otwell](https://www.youtube.com/watch?v=1P3wLy49t2c&t=3240s) (Laracon speaker (employee), 3); [Caleb Porzio](https://youtu.be/f4QShF42c6E?t=6018) (Laracon speaker, 2); [Philo](https://www.youtube.com/watch?v=MbynDaPaBt4&t=840s) (Laracon speaker, 2); [Jeffrey Way](https://laracasts.com/episodes/3564) (Tier 3, 1)

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

### Form state abstraction (Recommended)

*Score 8:* [Joe Tannenbaum](https://www.youtube.com/watch?v=YJmuKPk3d9M&t=2329s) (Laracon speaker (employee), 3); [@filamentphp](https://github.com/filamentphp/filament/blob/7c5c222be3e92439357213f8274d1548c8cb769e/docs/12-components/02-form.md) (Tier 2, 3); [Simon Vrachliotis](https://laracasts.com/episodes/3658) (Tier 3, 1); [Instructor Team](https://laracasts.com/episodes/3841) (Tier 3, 1)

- **Smell:** Autocomplete or similar interactions trigger page visits or duplicate custom loading and error state handling.
- **Technique:** Use Inertia useHttp for standalone data requests that do not navigate or mutate page props. Keep Laravel endpoints returning JSON and consume the parsed response returned by the helper while it tracks loading and errors.
- **When to use:** An interaction needs JSON data while staying on the current page.
- **When not to:** Navigation should retain normal Inertia visits; useHttp has separate data state from page props.
- **Packages:** `@inertiajs/react`, `@inertiajs/vue3`, `filament/forms`, `filament/schemas`, `inertiajs/inertia-laravel`

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

### Full-page Livewire components (Recommended)

*Score 8:* [Taylor Otwell](https://www.youtube.com/watch?v=1P3wLy49t2c&t=450s) (Laracon speaker (employee), 3); [Jigar Dhulla](https://www.youtube.com/watch?v=dP88NZfVjnY) (Laracon speaker, 2); [Josh Hanley](https://www.youtube.com/watch?v=esD6o8HYV5Q&t=128s) (Laracon speaker, 2); [Shruti Balasa](https://laracasts.com/episodes/3757) (Tier 3, 1)

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

### Lazy component loading (Recommended)

*Score 8:* [Caleb Porzio](https://youtu.be/f4QShF42c6E?t=6018) (Laracon speaker, 2); [Philo](https://www.youtube.com/watch?v=MbynDaPaBt4&t=1080s) (Laracon speaker, 2); [Bert De Swaef](https://www.youtube.com/watch?v=xxAv6s5045Y) (Laracon speaker, 2); [Jeremy McPeak](https://laracasts.com/episodes/3369) (Tier 3, 1); [@livewire](https://github.com/livewire/livewire/blob/80373fa09bec6532cb28572ff80dd7bf94ea4487/docs/lazy.md) (Tier 3, 1)

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

### Persistent Inertia layouts (Recommended)

*Score 8:* [Aaron Francis](https://x.com/aarondfrancis/status/1813937201291682134) (Tier 1, 3); [Caleb Porzio](https://x.com/calebporzio/status/1729946261787214203) (Tier 2, 3); [Bert De Swaef](https://www.youtube.com/watch?v=xxAv6s5045Y) (Laracon speaker, 2)

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

### Livewire form objects (Recommended)

*Score 7:* [Caleb Porzio](https://x.com/calebporzio/status/1735734917781143992) (Tier 2, 3); [Josh Hanley](https://www.youtube.com/watch?v=esD6o8HYV5Q&t=925s) (Laracon speaker, 2); [Jeremy McPeak](https://laracasts.com/episodes/3365) (Tier 3, 1); [@livewire](https://github.com/livewire/livewire/blob/ed482026c93839bf50a90d2da60b4cbd345d242f/docs/forms.md) (Tier 3, 1)

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

### Semantic component events (Recommended)

*Score 7:* [Caleb Porzio](https://youtu.be/f4QShF42c6E?t=6018) (Laracon speaker, 2); [Philo](https://www.youtube.com/watch?v=MbynDaPaBt4&t=960s) (Laracon speaker, 2); [Jeremy McPeak](https://laracasts.com/episodes/3361) (Tier 3, 1); [Shruti Balasa](https://laracasts.com/episodes/4136) (Tier 3, 1); [Instructor Team](https://laracasts.com/episodes/3848) (Tier 3, 1)

- **Smell:** A tightly coupled child component emits an event solely to ask its parent to execute one action.
- **Technique:** Call the containing Livewire component through $parent from the child view to replace one-off event/listener plumbing. This deliberately exposes the parent action contract rather than making the child independently reusable.
- **When to use:** A child is specific to one parent workflow.
- **When not to:** Reusable children are better served by modelable values or events; direct calls couple them to parent method names.
- **Packages:** `@inertiajs/vue3`, `inertiajs/inertia-laravel`, `livewire/livewire`

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

*Score 6:* [Joe Dixon](https://laracasts.com/episodes/3405) (Tier 1, 3); [Joe Tannenbaum](https://www.youtube.com/watch?v=v7spVsbN_Qc) (Laracon speaker (employee), 3)

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

### Custom form control components (Useful)

*Score 6:* [Caleb Porzio](https://www.youtube.com/watch?v=31pBMi0UdYE&t=1620s) (Laracon speaker, 2); [Shruti Balasa](https://laracasts.com/episodes/4022) (Tier 3, 1); [Jeffrey Way](https://laracasts.com/episodes/2300) (Tier 3, 1); [Andre Madarang](https://laracasts.com/episodes/2523) (Tier 3, 1); [Luke Downing](https://laracasts.com/episodes/2860) (Tier 3, 1)

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

### Deferred page props (Useful)

*Score 6:* [@inertiajs](https://github.com/inertiajs/docs/blob/d76a154d697f7af7ff9b17c85fe67044c3c36bd6/v3/data-props/deferred-props.mdx) (Tier 2, 3); [Diana Scharf](https://www.youtube.com/watch?v=50uRIFaUWqg&t=1440s) (Laracon speaker, 2); [Jeffrey Way](https://laracasts.com/episodes/3439) (Tier 3, 1)

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

### Enum-backed presentation (Useful)

*Score 6:* [@brendt_gd](https://stitcher.io/blog/php-enum-style-guide) (Tier 2, 3); [Lucas Giovanny](https://www.youtube.com/watch?v=33CIhvkAPEY&t=750s) (Laracon speaker, 2); [Caleb Porzio](https://laracasts.com/episodes/3937) (Tier 3, 1)

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

### Inertia partial reloads (Useful)

*Score 6:* [@inertiajs](https://github.com/inertiajs/docs/blob/387c5129df358a758432eabed053625f24e01f9b/v3/data-props/partial-reloads.mdx) (Tier 2, 3); [Diana Scharf](https://www.youtube.com/watch?v=50uRIFaUWqg&t=1080s) (Laracon speaker, 2); [Luke Downing](https://laracasts.com/episodes/3011) (Tier 3, 1)

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

### Optimistic UI (Useful)

*Score 6:* [Joe Tannenbaum](https://www.youtube.com/watch?v=YJmuKPk3d9M&t=2329s) (Laracon speaker (employee), 3); [Philo Hermans](https://www.youtube.com/watch?v=857ean0JIHE) (Laracon speaker, 2); [Caleb Porzio](https://laracasts.com/episodes/3924) (Tier 3, 1)

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

*Score 6:* [Joe Tannenbaum](https://www.youtube.com/watch?v=v7spVsbN_Qc) (Laracon speaker (employee), 3); [Joe Dixon](https://www.youtube.com/watch?v=yrL5eCMpqtc) (Laracon speaker (employee), 3)

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

### Visibility-aware polling (Useful)

*Score 6:* [Joe Tannenbaum](https://www.youtube.com/watch?v=v7spVsbN_Qc) (Laracon speaker (employee), 3); [Bert De Swaef](https://www.youtube.com/watch?v=xxAv6s5045Y) (Laracon speaker, 2); [Jeffrey Way](https://laracasts.com/episodes/3437) (Tier 3, 1); [Laravel News](https://laravel-news.com/inertia-3-9-0) (Supporting, 0)

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

### Progressive navigation (Useful)

*Score 5:* [Joe Tannenbaum](https://www.youtube.com/watch?v=YJmuKPk3d9M&t=2329s) (Laracon speaker (employee), 3); [Caleb Porzio](https://youtu.be/f4QShF42c6E?t=6018) (Laracon speaker, 2)

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

### Reusable accessible modal components (Useful)

*Score 5:* [Caleb Porzio](https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=26640s) (Laracon speaker, 2); [Philo](https://www.youtube.com/watch?v=MbynDaPaBt4&t=270s) (Laracon speaker, 2); [Jeffrey Way](https://laracasts.com/episodes/3900) (Tier 3, 1)

- **Smell:** Every form repeats its own modal shell, transitions and open/close wiring.
- **Technique:** Mount one modal host in the layout and dispatch a component name plus arguments to it. Render the selected Livewire child with a distinct key, validating allowed content before rendering.
- **When to use:** Several forms share the same modal presentation and lifecycle.
- **When not to:** Dynamic selection requires allowlisting, authorization and key management; one fixed modal may not justify a host.
- **Packages:** `@alpinejs/focus`, `alpinejs`, `livewire/livewire`

Before:
```php
<x-modal><livewire:login /></x-modal>
<x-modal><livewire:register /></x-modal>
```
After:
```php
<livewire:modal-host />
// Host view:
<livewire:dynamic-component :is="$component" :key="$component" />
// Trigger:
<button @click="$dispatch('open-modal', {component: 'login'})">Login</button>
```

### Streaming responses (Useful)

*Score 5:* [Spatie](https://freek.dev/3092-streaming-ai-replies-from-laravel-to-react) (Tier 2, 2); [Bert De Swaef](https://www.youtube.com/watch?v=xxAv6s5045Y) (Laracon speaker, 2); [Jeremy McPeak](https://laracasts.com/episodes/4040) (Tier 3, 1)

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

### Component render memoization (Useful)

*Score 4:* [Tighten](https://tighten.com/insights/blaze-under-the-hood) (Tier 2, 2); [Caleb Porzio](https://laracasts.com/episodes/4093) (Tier 3, 1); [@livewire](https://github.com/livewire/livewire/blob/4ec01c9c7841c107d1a84fc15adcca5977d6f759/docs/computed-properties.md) (Tier 3, 1)

- **Smell:** The same Blade component renders repeatedly with identical props.
- **Technique:** Opt a deterministic component into Blaze memoization so repeated render inputs reuse output.
- **When to use:** Rendering identical components consumes significant CPU.
- **When not to:** Avoid memoizing output dependent on hidden mutable state; measure whether caching pays off.
- **Packages:** `livewire/blaze`, `livewire/livewire`

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

### Framework render-hook extensions (Useful)

*Score 4:* [Povilas Korop](https://www.youtube.com/watch?v=olvqO33PBI0) (Laracon speaker, 2); [Shruti Balasa](https://laracasts.com/episodes/4025) (Tier 3, 1); [Katerina Trajchevska](https://laracasts.com/episodes/3151) (Tier 3, 1)

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

*Score 4:* [Caleb Porzio](https://www.youtube.com/watch?v=M60-nxlrePc&t=2310s) (Laracon speaker, 2); [Jeffrey Way](https://laracasts.com/episodes/3444) (Tier 3, 1); [Luke Downing](https://laracasts.com/episodes/2762) (Tier 3, 1)

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

### Isolated Livewire requests (Useful)

*Score 4:* [Caleb Porzio](https://x.com/calebporzio/status/1741876004816027993) (Tier 2, 3); [Jeremy McPeak](https://laracasts.com/episodes/3378) (Tier 3, 1)

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

### Navigation and asset prefetching (Useful)

*Score 4:* [Tim MacDonald](https://x.com/timacdonald87/status/1840542827924861423) (Tier 1, 3); [Jeffrey Way](https://laracasts.com/episodes/3438) (Tier 3, 1)

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

*Score 4:* [Lucas Giovanny](https://www.youtube.com/watch?v=33CIhvkAPEY&t=120s) (Laracon speaker, 2); [Povilas Korop](https://www.youtube.com/watch?v=olvqO33PBI0) (Laracon speaker, 2)

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

### Authorization capability props (Situational)

*Score 3:* [@inertiajs](https://github.com/inertiajs/docs/blob/387c5129df358a758432eabed053625f24e01f9b/v3/security/authorization.mdx) (Tier 2, 3)

- **Smell:** Frontend role comparisons duplicate backend permissions and drift as policy logic evolves.
- **Technique:** Evaluate capabilities server-side, expose booleans for presentation, and keep write endpoints protected by the policy.
- **When to use:** An Inertia page needs conditional controls for authorized operations.
- **When not to:** Do not use client booleans as authorization for server mutations.
- **Packages:** `inertiajs/inertia-laravel`

Before:
```php
// Client duplicates authorization rules.
const canEdit = user.role === 'admin';
```
After:
```php
// Controller prop, with policy enforcement on the write endpoint.
'can' => ['update' => $request->user()->can('update', $post)],
// Client renders its edit control from props.can.update.
```

### Centralized rich-content rendering (Situational)

*Score 3:* [@filamentphp](https://github.com/filamentphp/filament/blob/7c5c222be3e92439357213f8274d1548c8cb769e/packages/forms/docs/10-rich-editor.md) (Tier 2, 3)

- **Smell:** Custom Blade output prints editor HTML directly and cannot safely resolve private image URLs.
- **Technique:** Route rich content through RichContentRenderer with matching attachment settings.
- **When to use:** Rich text is authored by users or contains private attachment references.
- **When not to:** Plain text that should be escaped instead of interpreted as HTML.
- **Packages:** `filament/forms`

Before:
```php
{!! $record->content !!}
```
After:
```php
{{ RichContentRenderer::make($record->content)
    ->fileAttachmentsDisk('s3')
    ->fileAttachmentsVisibility('private') }}
```

### Component-boundary record authorization (Situational)

*Score 3:* [@filamentphp](https://github.com/filamentphp/filament/blob/7c5c222be3e92439357213f8274d1548c8cb769e/docs/09-advanced/06-security.md) (Tier 2, 3)

- **Smell:** A child component retains a model that was originally tenant-scoped, then uses native restoration after access changes.
- **Technique:** Keep a locked scalar ID and use a scoped resolver that authorizes the intended operation on each request.
- **When to use:** Custom Filament or Livewire model properties outside built-in resource-page binding.
- **When not to:** Do not rely on a cached authorization result across requests or assume a locked ID establishes record access.
- **Packages:** `filament/filament`, `livewire/livewire`

Before:
```php
public Post $post;
public function save() {
    $this->post->update($this->data);
}
```
After:
```php
#[Locked]
public int $postId;
public function save() {
    $post = Post::where('tenant_id', tenant()->id)->findOrFail($this->postId);
    Gate::authorize('update', $post);
    $post->update($this->form->getState());
}
```

### Component-scoped Livewire scripts (Situational)

*Score 3:* [Caleb Porzio](https://x.com/calebporzio/status/1726612011331494387) (Tier 2, 3)

- **Smell:** Component scripts repeatedly register document listeners and assets are included once for every component instance.
- **Technique:** Use Livewire @assets for resources that should load once and @script for per-instance initialization with access to $wire. This associates initialization with the component lifecycle, including lazy-loaded components.
- **When to use:** A reusable Livewire component depends on JavaScript initialization or third-party assets.
- **When not to:** A shared application module may be clearer for substantial global JavaScript; avoid duplicating global side effects in per-instance scripts.
- **Packages:** `livewire/livewire`

Before:
```php
<script src="/chart.js"></script>
<script>document.addEventListener("livewire:init", () => { /* init */ });</script>
```
After:
```php
@assets
<script src="/chart.js"></script>
@endassets
@script
<script>initChart($wire);</script>
@endscript
```

### Custom Blade directives and compiler extensions (Situational)

*Score 3:* [Philo](https://www.youtube.com/watch?v=MbynDaPaBt4&t=1200s) (Laracon speaker, 2); [Jeremy McPeak](https://laracasts.com/episodes/3418) (Tier 3, 1)

- **Smell:** Many buttons repeat the same verbose modal event payload and modifier handling.
- **Technique:** Extract the trigger behavior into an application-defined wire:modal directive using Livewire.directive(). Interpret the expression and modifiers, dispatch the shared opening event and register cleanup for attached listeners.
- **When to use:** A repeated UI interaction needs one consistent declarative syntax.
- **When not to:** Adds JavaScript extension code and lifecycle maintenance; a few simple triggers can remain inline.
- **Packages:** `livewire/livewire`

Before:
```php
<button @click="$dispatch('open-modal', {component: 'login', lazy: true})">Login</button>
```
After:
```php
// Register the application directive with Livewire.directive().
<button wire:modal.lazy="login">Login</button>
```

### Grouped and stepped forms (Situational)

*Score 3:* [Povilas Korop](https://www.youtube.com/watch?v=olvqO33PBI0) (Laracon speaker, 2); [Shruti Balasa](https://laracasts.com/episodes/4137) (Tier 3, 1)

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

### Inertia once props (Situational)

*Score 3:* [@inertiajs](https://github.com/inertiajs/docs/blob/387c5129df358a758432eabed053625f24e01f9b/v3/data-props/once-props.mdx) (Tier 2, 3)

- **Smell:** Every Inertia response repeatedly computes and transmits a large stable reference list.
- **Technique:** Use Inertia::once with a bounded freshness policy and an explicit reload after changes.
- **When to use:** Reference data shared across related pages that can tolerate a defined stale window.
- **When not to:** Rapidly changing or authorization-critical values that must be checked on every server request.
- **Packages:** `inertiajs/inertia-laravel`

Before:
```php
'countries' => Country::all(),
```
After:
```php
'countries' => Inertia::once(fn () => Country::all())
    ->until(now()->addDay()),
```

### Markdown placeholder rendering (Situational)

*Score 3:* [Aaron Francis](https://aaronfrancis.com/2023/rendering-blade-components-in-markdown-e2e74e55) (Tier 1, 3)

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

*Score 3:* [Povilas Korop](https://www.youtube.com/watch?v=olvqO33PBI0) (Laracon speaker, 2); [Luke Downing](https://laracasts.com/episodes/2849) (Tier 3, 1)

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

*Score 3:* [Philo Hermans](https://www.youtube.com/watch?v=857ean0JIHE) (Laracon speaker, 2); [Caleb Porzio](https://laracasts.com/episodes/3930) (Tier 3, 1)

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

*Score 3:* [Tim MacDonald](https://x.com/timacdonald87/status/1556801179073490945) (Tier 1, 3)

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

### Shared panel providers (Situational)

*Score 3:* [Lucas Giovanny](https://www.youtube.com/watch?v=33CIhvkAPEY&t=240s) (Laracon speaker, 2); [Luke Downing](https://laracasts.com/episodes/3392) (Tier 3, 1)

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

### Blade-based PDF rendering (Situational)

*Score 2:* [Spatie](https://www.youtube.com/watch?v=Boqfw4ZCk7g&t=1590s) (Tier 2, 2)

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

*Score 2:* [Caleb Porzio](https://www.youtube.com/watch?v=M60-nxlrePc&t=1830s) (Laracon speaker, 2)

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

### Component model contracts and attribute forwarding (Situational)

*Score 2:* [Caleb Porzio](https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=24660s) (Laracon speaker, 2)

- **Smell:** Parents know a custom control’s internal Alpine state and manually synchronize it.
- **Technique:** Encapsulate the control implementation in a Blade component and expose its value through Alpine x-modelable. Forward binding attributes so callers can use x-model as with a native input; verify separate Livewire integrations rather than assuming all modifiers work.
- **When to use:** A custom toggle or picker should be reusable in different surrounding forms.
- **When not to:** Modelable state must be JSON-compatible; native inputs are simpler when sufficient.
- **Packages:** `alpinejs`, `livewire/flux`, `livewire/livewire`

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

### Context-aware heading components (Situational)

*Score 2:* [Charlie Morris](https://www.youtube.com/watch?v=y7yV-f8jDFQ&t=1180s) (Laracon speaker, 2)

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

*Score 2:* [Kévin Dunglas](https://www.youtube.com/watch?v=PzOLyz1vr5I&t=1140s) (Laracon speaker, 2)

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

### Import aliases (Situational)

*Score 2:* [Spatie](https://spatie.be/blog/expose-legacy-javascript-code-with-import-aliases) (Tier 2, 2)

- **Smell:** Old JavaScript remains mixed with new modules and accidental dependencies slow migration.
- **Technique:** Give legacy directories explicit Vite import aliases so imports expose when a module depends on old code. Migrate incrementally and replace those imports as new modules take over.
- **When to use:** A Laravel frontend is being modernized incrementally while legacy code stays operational.
- **When not to:** An alias alone does not enforce architectural boundaries or remove legacy dependencies.

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

### Progressive enhancement (Situational)

*Score 2:* [Zuzana Kunckova](https://www.youtube.com/watch?v=Emm559sI6tI&t=240s) (Laracon speaker, 2)

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

*Score 2:* [Caleb Porzio](https://youtu.be/f4QShF42c6E?t=6018) (Laracon speaker, 2)

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

*Score 2:* [Lucas Giovanny](https://www.youtube.com/watch?v=33CIhvkAPEY&t=990s) (Laracon speaker, 2)

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

*Score 2:* [Diana Scharf](https://www.youtube.com/watch?v=50uRIFaUWqg&t=900s) (Laracon speaker, 2)

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

### Slot-based component composition (Situational)

*Score 2:* [Caleb Porzio](https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=24840s) (Laracon speaker, 2)

- **Smell:** String props cannot express rich labels and optional markup causes a growing set of flags.
- **Technique:** Accept a named Blade slot for rich content and forward its slot attribute bag to the rendered element. Slots allow caller-supplied icons or buttons without expanding the component’s prop vocabulary.
- **When to use:** Reusable presentation needs custom markup at known insertion points.
- **When not to:** A simple text-only label rarely needs a named-slot API; strings do not have slot attributes.
- **Packages:** `livewire/flux`, `livewire/livewire`

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

### Vue composables (Situational)

*Score 2:* [Jeffrey Way](https://laracasts.com/episodes/2301) (Tier 3, 1); [Luke Downing](https://laracasts.com/episodes/2763) (Tier 3, 1)

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

### External service adapters and fakes (Recommended)

*Score 20:* [@mateusjatenee](https://x.com/mateusjatenee/status/1799167431736172614) (Tier 2, 3); [Christoph Rumpel](https://laracasts.com/episodes/2903) (Tier 1, 3); [@ryangjchandler](https://ryangjchandler.co.uk/posts/writing-code-that-feels-native-to-laravel) (Tier 2, 3); [@laravel](https://laravel.com/docs/13.x/processes#preventing-stray-processes) (Tier 2, 3); [Ryuta Hamasaki](https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=5730s) (Laracon speaker, 2); [Steve McDougall](https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=22530s) (Laracon speaker, 2); [Mateus Guimarães](https://laracasts.com/episodes/3047) (Tier 3, 1); [Jeremy McPeak](https://laracasts.com/episodes/4047) (Tier 3, 1); [mateusjatenee](https://x.com/mateusjatenee/status/1799167431736172614) (Tier 3, 1); [@prism-php](https://github.com/prism-php/prism/blob/7a7c7896236afe82f34764554cccf654a9eff3c4/docs/core-concepts/testing.md) (Tier 3, 1)

- **Smell:** Tests mock low-level HTTP calls throughout the application or require real external services.
- **Technique:** Put the external capability behind an interface and bind a real adapter in production and an in-memory fake in tests. Test business behavior through that contract and separately test the real adapter against expected transport semantics.
- **When to use:** Many business tests depend on an external service and need deterministic behavior.
- **When not to:** A fake can drift from the provider; keep contract or adapter tests and avoid interfaces for nonexistent variation.
- **Packages:** `laravel/ai`, `laravel/framework`, `prism-php/prism`

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

### Selective boundary test doubles (Recommended)

*Score 14:* [@AshAllenDesign](https://ashallendesign.co.uk/blog/testing-saloon-api-integrations) (Tier 2, 3); [Luke Downing](https://www.youtube.com/watch?v=GERCwd65O-8&t=1170s) (Laracon speaker, 2); [Steve McDougall](https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=23610s) (Laracon speaker, 2); [Kath Young](https://www.youtube.com/watch?v=pRaV4BG8zvg&t=347s) (Laracon speaker, 2); [Mary Perry](https://www.youtube.com/watch?v=vsxoaTgtyjw) (Laracon speaker, 2); [Jason McCreary](https://www.youtube.com/watch?v=LaKEFjA25r4&t=630s) (Laracon speaker, 2); [laravel/boost](https://github.com/laravel/boost/blob/0555ee4665bb375af13f272852cf3f860d1f9637/.ai/laravel/skill/testing-best-practices/rules/isolation.blade.php) (Tier 3, 1)

- **Smell:** Global event faking suppresses fixture invariants, while late faking lets setup satisfy assertions.
- **Technique:** Intercept only relevant events with Event::fake([...]) or exceptions, ordering setup according to real fixture dependencies. Assert events from the actual action without suppressing unrelated creation hooks.
- **When to use:** Event-driven behavior needs isolated dispatch assertions.
- **When not to:** Event fakes stop listeners; use real execution when asserting listener outcomes.
- **Packages:** `laravel/framework`, `mockery/mockery`, `saloonphp/laravel-plugin`, `saloonphp/saloon`

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

### Parameterized tests and datasets (Recommended)

*Score 13:* [Nuno Maduro](https://www.youtube.com/watch?v=9EGPo_enEc8&t=450s) (Laracon speaker (employee), 3); [@brendt_gd](https://stitcher.io/blog/testing-patterns) (Tier 2, 3); [@pestphp](https://github.com/pestphp/docs/blob/c8fbc86501e63254899e8b90315821294456433c/datasets.md) (Tier 2, 3); [Luke Downing](https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=3450s) (Laracon speaker, 2); [Duilio Palacios](https://www.youtube.com/watch?v=1sixoOQliZI&t=1140s) (Laracon speaker, 2)

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

### Model factories and scenario builders (Recommended)

*Score 12:* [Christoph Rumpel](https://laracasts.com/episodes/2878) (Tier 1, 3); [Nuno Maduro](https://www.youtube.com/watch?v=AkDMDHAs09U&t=900s) (Laracon speaker (employee), 3); [Kath Young](https://www.youtube.com/watch?v=pRaV4BG8zvg&t=232s) (Laracon speaker, 2); [Jason McCreary](https://www.youtube.com/watch?v=LaKEFjA25r4&t=960s) (Laracon speaker, 2); [Matt Stauffer](https://www.youtube.com/watch?v=1H4QN4_WbQE) (Laracon speaker, 2)

- **Smell:** Development and tests repeatedly assemble sample records manually and lack useful reproducible data.
- **Technique:** Build Eloquent factories and compose them through seeders to generate realistic application data. Keep fixture construction separate from feature assertions so the same data setup can support development and tests.
- **When to use:** The application needs repeatable populated states for development or verification.
- **When not to:** Large default seeders can slow tests and hide dependencies; the talk does not prescribe specific factory states.
- **Packages:** `pestphp/pest`, `pestphp/pest-plugin-laravel`

Before:
```php
User::create(["name" => "Test", "email" => "test@example.com"]);
// Repeat explicit sample data for related records.
```
After:
```php
User::factory()->count(20)->create();
// DatabaseSeeder composes the required factories.
```

### Architecture fitness tests (Recommended)

*Score 11:* [Nuno Maduro](https://www.youtube.com/watch?v=9EGPo_enEc8&t=780s) (Laracon speaker (employee), 3); [@AshAllenDesign](https://ashallendesign.co.uk/blog/architecture-testing-in-laravel-with-pest) (Tier 2, 3); [@pestphp](https://github.com/pestphp/docs/blob/378cad16c9822850a6b8ec0f7c742c6c02b1f39f/arch-testing.md) (Tier 2, 3); [Ryuta Hamasaki](https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=5880s) (Laracon speaker, 2)

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

### Reusable test helpers and expectations (Recommended)

*Score 11:* [Nuno Maduro](https://www.youtube.com/watch?v=71bMyZcDlM4) (Laracon speaker (employee), 3); [Christoph Rumpel](https://x.com/christophrumpel/status/1985307348659732858) (Tier 1, 3); [@pestphp](https://github.com/pestphp/docs/blob/378cad16c9822850a6b8ec0f7c742c6c02b1f39f/custom-expectations.md) (Tier 2, 3); [Luke Downing](https://laracasts.com/episodes/3020) (Tier 3, 1); [laravel/framework](https://github.com/laravel/framework/pull/61415) (Tier 3, 1); [Laravel News](https://laravel-news.com/laravel-fake-assertion-property-arrays) (Supporting, 0)

- **Smell:** Each caller test repeats all assertions about the same action or event downstream effects.
- **Technique:** Test the operation or event behavior separately, then have caller tests assert that the action was invoked or event dispatched. Laravel event fakes and container replacement for actions make the separation practical.
- **When to use:** Several entry points use the same independently testable behavior.
- **When not to:** Retain integration coverage for important wiring; overly narrow dispatch tests can miss incorrect payloads.
- **Packages:** `laravel/framework`, `pestphp/pest`, `pestphp/pest-plugin-rector`, `rector/rector`

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

### Named factory states (Recommended)

*Score 8:* [Christoph Rumpel](https://laracasts.com/episodes/2870) (Tier 1, 3); [Ashley Hindle](https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=33510s) (Laracon speaker (employee), 3); [@joelclermont](https://masteringlaravel.io/daily/2026-09-18-republished-a-good-rule-for-when-to-create-a-factory-state) (Tier 3, 1); [Mastering Laravel / Aaron Saray](https://masteringlaravel.io/daily/2026-08-20-factory-states-can-accept-arguments) (Tier 3, 1)

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

### Characterization tests (Useful)

*Score 6:* [Nuno Maduro](https://www.youtube.com/watch?v=IH4-bkDSqyk&t=1890s) (Laracon speaker (employee), 3); [Jordan Havard](https://www.youtube.com/watch?v=riJBa0Bxo-8&t=1230s) (Laracon speaker, 2); [Katerina Trajchevska](https://laracasts.com/episodes/3173) (Tier 3, 1)

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

### Factory parent recycling (Useful)

*Score 4:* [Jess Archer](https://x.com/jessarcher/status/1569598451486453763) (Tier 1, 3); [laravel/boost](https://github.com/laravel/boost/blob/0555ee4665bb375af13f272852cf3f860d1f9637/.ai/laravel/skill/testing-best-practices/rules/test-data.blade.php) (Tier 3, 1)

- **Smell:** Nested factories create several parent records that should be shared.
- **Technique:** Use factory recycle() to reuse an existing parent across related model creation.
- **When to use:** A test fixture needs one tenant, team, or user across a graph.
- **When not to:** Do not recycle where independent parents are part of the behavior under test.
- **Packages:** `laravel/framework`

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

### Package integration fixture application (Useful)

*Score 4:* [Spatie](https://freek.dev/2558-workbench-app-for-packages) (Tier 2, 2); [Kath Young](https://www.youtube.com/watch?v=blUswpYpmIs&t=725s) (Laracon speaker, 2)

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

### Mutation-guided test strengthening (Situational)

*Score 3:* [Nuno Maduro](https://www.youtube.com/watch?v=BNhbgcNJyAk&t=1260s) (Laracon speaker (employee), 3)

- **Smell:** Tests report full coverage but pass after persistence or validation logic is removed.
- **Technique:** Run Pest mutation testing and inspect surviving mutations, then add assertions for actual stored state and rejection behavior. Repeat against the relevant scope to confirm meaningful changes are detected.
- **When to use:** Coverage does not provide confidence that assertions catch regressions.
- **When not to:** Extra test executions cost time; scores depend on scope and generated mutations and do not prove complete correctness.
- **Packages:** `pestphp/pest`

Before:
```php
$component->call("updatePassword")->assertHasNoErrors();
```
After:
```php
$component->call("updatePassword")->assertHasNoErrors();
expect(Hash::check($newPassword, $user->fresh()->password))->toBeTrue();
// vendor/bin/pest --mutate
```

### Notification and mail fakes (Situational)

*Score 3:* [Jason McCreary](https://www.youtube.com/watch?v=LaKEFjA25r4&t=360s) (Laracon speaker, 2); [Jeffrey Way](https://laracasts.com/episodes/2308) (Tier 3, 1)

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

### Observable behavior assertions (Situational)

*Score 3:* [@pestphp](https://github.com/pestphp/docs/blob/6daaf1678d3ecd33af75e845f7a907df69fb98a4/mutation-testing.md) (Tier 2, 3)

- **Smell:** An endpoint test passes even when its response body is replaced by an empty result.
- **Technique:** Seed a representative record and assert the expected response content so a behavior-changing mutant fails.
- **When to use:** Mutation feedback exposes executed but weakly asserted code.
- **When not to:** Do not add redundant assertions merely to maximize a score when the mutation is equivalent.
- **Packages:** `pestphp/pest`

Before:
```php
$this->getJson('/todos')->assertOk();
```
After:
```php
Todo::factory()->create(['name' => 'Buy milk']);
$this->getJson('/todos')->assertOk()
    ->assertJsonFragment(['name' => 'Buy milk']);
```

### Parallel test resource isolation (Situational)

*Score 3:* [Christoph Rumpel](https://www.youtube.com/watch?v=tDqPe8LbWEI&t=1025s) (Laracon speaker (employee), 3)

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

### Snapshot testing (Situational)

*Score 3:* [Nuno Maduro](https://www.youtube.com/watch?v=AkDMDHAs09U&t=480s) (Laracon speaker (employee), 3)

- **Smell:** A newly exposed model field can escape narrow serialization assertions.
- **Technique:** Snapshot the keys of model toArray() output to detect unexpected exposure without random factory values. Cover API resource contracts separately when they serialize a different shape.
- **When to use:** Sensitive model serialization fields must remain deliberate.
- **When not to:** Loaded relationships change the key set; a reviewed baseline is essential.
- **Packages:** `pestphp/pest`

Before:
```php
expect($user->toArray())->toHaveKey("name");
```
After:
```php
expect(array_keys($user->toArray()))->toMatchSnapshot();
```

### Cached test bootstrap (Situational)

*Score 2:* [Kath Young](https://www.youtube.com/watch?v=pRaV4BG8zvg&t=178s) (Laracon speaker, 2)

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

*Score 2:* [Jason McCreary](https://www.youtube.com/watch?v=sBpzYsMgLK8&t=1618s) (Laracon speaker, 2)

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

*Score 2:* [Ryan Lovett](https://www.youtube.com/watch?v=AJ8MOFoDeP0&t=1650s) (Laracon speaker, 2)

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

### Higher-order tests and expectations (Situational)

*Score 2:* [Luke Downing](https://www.youtube.com/watch?v=b3ybZlxrZZY) (Laracon speaker, 2)

- **Smell:** Related assertions repeatedly name the same model and obscure the result being specified.
- **Technique:** Group related model attributes and relationship checks into a chained Pest expectation. Use the comparison method that matches the intended value or object semantics.
- **When to use:** Several checks describe the same returned model graph.
- **When not to:** Long chains across unrelated behaviors become hard to diagnose.
- **Packages:** `pestphp/pest`

Before:
```php
expect($discount->country)->toBe("GB");
expect($discount->referrer->id)->toBe($referrer->id);
```
After:
```php
expect($discount)
  ->country->toBe("GB")
  ->referrer->id->toBe($referrer->id);
```

### Independent test oracles (Situational)

*Score 2:* [Mastering Laravel / Aaron Saray](https://masteringlaravel.io/daily/2026-10-12-is-your-test-just-copying-your-code) (Tier 3, 1); [laravel/boost](https://github.com/laravel/boost/blob/75c269a8a53e931c7252e7cd562e1a209356a7c6/.ai/laravel/skill/testing-best-practices/rules/assertions.blade.php) (Tier 3, 1)

- **Smell:** A test reproduces the same date arithmetic as production, so both can be wrong together.
- **Technique:** Freeze time to a known instant and assert a literal expected result.
- **When to use:** Testing date boundaries or derived timestamps.
- **When not to:** The exact value is intentionally nondeterministic and only an invariant is specified.
- **Packages:** `laravel/framework`

Before:
```php
$expected = now()->subHours(24)->toIso8601String();
$this->assertSame($expected, $service->cutoff());
```
After:
```php
$this->travelTo('2026-01-02 00:00:00');
$this->assertSame('2026-01-01T00:00:00+00:00', $service->cutoff());
```

### Isolated Form Request tests (Situational)

*Score 2:* [Jigar Dhulla](https://www.youtube.com/watch?v=dP88NZfVjnY) (Laracon speaker, 2)

- **Smell:** Filter behavior is difficult to test without exercising a full browser journey.
- **Technique:** Use Livewire PHP component tests with matching and excluded records, set the filter property, and assert both rendered inclusion and exclusion. Keep browser coverage for interactions these component tests cannot exercise.
- **When to use:** Server-owned filtering needs repeatable behavioral tests.
- **When not to:** Component tests do not verify browser wiring or complex JavaScript interaction.
- **Packages:** `livewire/livewire`

Before:
```php
// Manual browser check that remote jobs appear.
```
After:
```php
Job::factory()->create(["title" => "Remote role", "location" => "remote"]);
Job::factory()->create(["title" => "Office role", "location" => "NY"]);
Livewire::test(JobList::class)->set("location", "remote")
    ->assertSee("Remote role")->assertDontSee("Office role");
```

### Route-discovery smoke tests (Situational)

*Score 2:* [Spatie](https://freek.dev/2739-how-to-test-all-routes-in-your-laravel-app-by-writing-just-a-single-pest-test) (Tier 2, 2)

- **Smell:** Simple new pages are added without any test because each needs repetitive smoke-test boilerplate.
- **Technique:** Discover included GET routes with a Pest route-testing dataset and supply binding/auth setup where needed.
- **When to use:** Many simple pages share the same successful-response invariant.
- **When not to:** Endpoint-specific semantics, authorization edge cases or side effects need richer individual tests.
- **Packages:** `spatie/pest-plugin-route-testing`

Before:
```php
it("renders home", fn () => $this->get("/")->assertSuccessful());
it("renders about", fn () => $this->get("/about")->assertSuccessful());
```
After:
```php
use function Spatie\RouteTesting\routeTesting;
routeTesting("public GET pages")
    ->exclude("admin*")
    ->assertSuccessful();
```

### Seeded procedural generation (Situational)

*Score 2:* [Will King](https://www.youtube.com/watch?v=LN3heGluQ7A&t=330s) (Laracon speaker, 2)

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

### Chunked and streaming processing (Recommended)

*Score 12:* [@brendt_gd](https://stitcher.io/blog/processing-11-million-rows) (Tier 2, 3); [@laravel](https://laravel.com/cloud/docs/knowledge-base/migrate-upstash-to-valkey) (Tier 2, 3); [Devansh Bawari](https://www.youtube.com/watch?v=eJJsGhu9gtI) (Laracon speaker, 2); [Daniel Coulbourne](https://www.youtube.com/watch?v=YJmuKPk3d9M&t=20462s) (Laracon speaker, 2); [Jeremy McPeak](https://laracasts.com/episodes/3994) (Tier 3, 1); [laravel/boost](https://github.com/laravel/boost/blob/d165b9ea6086294a8f1999ac3dcf4e61da267b23/.ai/laravel/skill/laravel-best-practices/rules/db-performance.md) (Tier 3, 1); [Laravel News](https://laravel-news.com/18-tips-to-optimize-your-laravel-database-queries) (Supporting, 0)

- **Smell:** An import loads and parses a large file entirely before processing, exhausting memory.
- **Technique:** Read and parse the input incrementally, yielding records through a generator or Laravel LazyCollection and processing bounded chunks. Streaming must start at input reading; making an already loaded array lazy does not remove its memory cost.
- **When to use:** Input files or datasets can exceed comfortable memory limits.
- **When not to:** The parser must support incremental reading and record boundaries; some operations still require global state.
- **Packages:** `laravel/framework`

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

### Automated AST refactoring (Recommended)

*Score 9:* [Nuno Maduro](https://www.youtube.com/watch?v=AkDMDHAs09U&t=2040s) (Laracon speaker (employee), 3); [Tomas Votruba](https://www.youtube.com/watch?v=qGrsvcWdERc) (Laracon speaker, 2); [Colin DeCarlo](https://www.youtube.com/watch?v=Xz4QOv5Xe_s) (Laracon speaker, 2); [Jason McCreary](https://laracasts.com/episodes/2711) (Tier 3, 1); [Mateus Guimarães](https://laracasts.com/episodes/3602) (Tier 3, 1)

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

*Score 6:* [Jason McCreary](https://www.youtube.com/watch?v=sBpzYsMgLK8&t=62s) (Laracon speaker, 2); [Tomas Votruba](https://www.youtube.com/watch?v=qGrsvcWdERc) (Laracon speaker, 2); [Ryan Chandler](https://www.youtube.com/watch?v=bZMgPoC2n_Y) (Laracon speaker, 2)

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

### Forward-only migration repair (Useful)

*Score 5:* [@PovilasKorop](https://laraveldaily.com/post/laravel-migrations-table-already-exists-after-foreign-key-failed) (Tier 2, 3); [Spatie](https://freek.dev/2900-why-i-dont-use-down-migrations) (Tier 2, 2)

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

### Incremental module migration (Useful)

*Score 5:* [Wendell Adriel](https://wendelladriel.com/blog/modular-monoliths-creating-real-boundaries-before-reaching-for-microservices) (Tier 1, 3); [Matt Stauffer](https://speakerdeck.com/mattstauffer/abstracting-too-early) (Laracon speaker, 2)

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

*Score 5:* [Matt Stauffer](https://speakerdeck.com/mattstauffer/abstracting-too-early) (Laracon speaker, 2); [Kath Young](https://www.youtube.com/watch?v=pRaV4BG8zvg&t=867s) (Laracon speaker, 2); [Katerina Trajchevska](https://laracasts.com/episodes/3220) (Tier 3, 1)

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

*Score 5:* [Christoph Rumpel](https://www.youtube.com/watch?v=tDqPe8LbWEI&t=1303s) (Laracon speaker (employee), 3); [Sam Sehnert](https://www.youtube.com/watch?v=IPbxscS0xiY&t=1050s) (Laracon speaker, 2)

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

### Strangler fig migration (Useful)

*Score 4:* [John Drexler](https://www.youtube.com/watch?v=YJmuKPk3d9M&t=11255s) (Laracon speaker, 2); [Jordan Havard](https://www.youtube.com/watch?v=riJBa0Bxo-8&t=935s) (Laracon speaker, 2)

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

### Dual-store queue drain migration (Situational)

*Score 3:* [@laravel](https://laravel.com/cloud/docs/knowledge-base/migrate-upstash-to-valkey) (Tier 2, 3)

- **Smell:** Changing Redis connections abandons pending or delayed jobs in the previous store.
- **Technique:** Keep an explicitly named source queue connection and run a temporary worker on it while new dispatches use the destination.
- **When to use:** Queued work must survive a cache or queue-provider migration without copying mutable queue data.
- **When not to:** The old queue is verified empty or jobs can be discarded.

Before:
```php
# Cutover without a source drain:
QUEUE_CONNECTION=redis
```
After:
```php
# After attaching the destination, keep an upstash queue connection.
php artisan queue:work upstash --queue=default,emails
# Verify both names, including delayed and reserved jobs:
php artisan queue:monitor upstash:default,upstash:emails
# Resolve failed jobs before removing the source connection.
```

### Incremental test-framework migration (Situational)

*Score 3:* [Nuno Maduro](https://www.youtube.com/watch?v=9EGPo_enEc8&t=180s) (Laracon speaker (employee), 3)

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

### Read-through session migration (Situational)

*Score 3:* [@laravel](https://laravel.com/cloud/docs/knowledge-base/migrate-upstash-to-valkey) (Tier 2, 3)

- **Smell:** A cache-provider cutover signs out all users because their sessions remain in the old store.
- **Technique:** Wrap old and new session handlers: read destination then source, write destination, and destroy both; remove the adapter after one session lifetime.
- **When to use:** Sessions must survive a Redis-compatible store migration.
- **When not to:** Losing sessions is acceptable or old and new serialization or key prefixes differ.

Before:
```php
public function read($id): string|false {
    return $this->new->read($id);
}
```
After:
```php
public function read($id): string|false {
    return $this->new->read($id) ?: $this->old->read($id);
}
public function write($id, $data): bool { return $this->new->write($id, $data); }
public function destroy($id): bool {
    $this->new->destroy($id); $this->old->destroy($id);
    return true;
}
```

### Wrap method (decorator) (Situational)

*Score 3:* [Luke Downing](https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=2880s) (Laracon speaker, 2); [Katerina Trajchevska](https://laracasts.com/episodes/3221) (Tier 3, 1)

- **Smell:** An optional audit trail leads to duplicated registration code or a growing conditional inside the base action.
- **Technique:** Implement the same contract in a decorator that delegates to the original action, then records the audit trail. Bind the decorator where the additional behavior is needed.
- **When to use:** Selected consumers need additional behavior around a shared action.
- **When not to:** Multiple decorator layers complicate tracing, and audit failures need a clear transaction policy.

Before:
```php
$user = User::create($data);
if ($audited) { $audit->record($user); }
```
After:
```php
class AuditedRegistration implements RegistersUser {
    public function __construct(private RegisterUser $inner, private Audit $audit) {}
    public function __invoke(array $data): User {
        $user = ($this->inner)($data);
        $this->audit->record($user);
        return $user;
    }
}
```

### Batched historical event backfill (Situational)

*Score 2:* [Daniel Coulbourne](https://www.youtube.com/watch?v=OpCT3s4wTI0&t=1020s) (Laracon speaker, 2)

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

### Incremental Livewire adoption (Situational)

*Score 2:* [Spatie](https://spatie.be/blog/livewire-and-inertia-how-we-love-and-use-both) (Tier 2, 2)

- **Smell:** A legacy Blade application needs interactive features but a full frontend rewrite is costly.
- **Technique:** Replace selected page fragments with Livewire components while retaining the surrounding Laravel application. Migrate additional interactions incrementally.
- **When to use:** Existing server-rendered pages need focused interactivity.
- **When not to:** Complex client-owned state or an established SPA architecture may fit a JavaScript frontend better.
- **Packages:** `livewire/livewire`

Before:
```php
<form action="/search" method="get">
    <input name="q">
</form>
```
After:
```php
<livewire:customer-search />
{{-- Component view --}}
<input wire:model.live.debounce.300ms="query">
```

### Legacy HTTP response adapters (Situational)

*Score 2:* [Jordan Havard](https://www.youtube.com/watch?v=riJBa0Bxo-8&t=1035s) (Laracon speaker, 2)

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

*Score 2:* [Daniel Coulbourne](https://www.youtube.com/watch?v=OpCT3s4wTI0&t=540s) (Laracon speaker, 2)

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

### Shadow legacy projections (Situational)

*Score 2:* [Daniel Coulbourne](https://www.youtube.com/watch?v=OpCT3s4wTI0&t=840s) (Laracon speaker, 2)

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

*Score 2:* [Sam Sehnert](https://www.youtube.com/watch?v=IPbxscS0xiY&t=960s) (Laracon speaker, 2)

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

*Score 20:* [Wendell Adriel](https://wendelladriel.com/blog/modular-monoliths-creating-real-boundaries-before-reaching-for-microservices) (Tier 1, 3); [Ashley Hindle](https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=33420s) (Laracon speaker (employee), 3); [Mateus Guimarães](https://www.youtube.com/watch?v=WqL6wgH1fKs) (Laracon speaker, 2); [Ryuta Hamasaki](https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=4470s) (Laracon speaker, 2); [Spatie](https://www.youtube.com/watch?v=CrO_7Df1cBc&t=1080s) (Tier 2, 2); [Bobby Bouwmann](https://www.youtube.com/watch?v=jX3nWXyLank) (Laracon speaker, 2); [Mary Perry](https://www.youtube.com/watch?v=vsxoaTgtyjw) (Laracon speaker, 2); [Josh Hanley](https://www.youtube.com/watch?v=esD6o8HYV5Q&t=325s) (Laracon speaker, 2); [wendelladriel/laravel-exa](https://github.com/WendellAdriel/laravel-exa) (Tier 3, 1); [@benjamincrozat](https://benjamincrozat.com/laravel-architecture-best-practices) (Tier 3, 1)

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

### Dependency inversion (contracts) (Recommended)

*Score 14:* [Dan Harrin](https://x.com/danjharrin/status/1501598754360729603) (Tier 2, 3); [@laravel](https://laravel.com/docs/13.x/container#bind-attribute) (Tier 2, 3); [Mateus Guimarães](https://www.youtube.com/watch?v=WqL6wgH1fKs) (Laracon speaker, 2); [Ryuta Hamasaki](https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=4950s) (Laracon speaker, 2); [Luke Downing](https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=2850s) (Laracon speaker, 2); [Jeffrey Way](https://laracasts.com/episodes/3400) (Tier 3, 1); [Katerina Trajchevska](https://laracasts.com/episodes/3144) (Tier 3, 1)

- **Smell:** Consumers reach into another module's internal classes and cannot tolerate implementation changes.
- **Technique:** Expose a small public interface, bind its implementation in a service provider and inject that contract into consumers. Translate incoming data into the receiving module's own domain language behind the boundary.
- **When to use:** Modules need stable collaboration without sharing their internals.
- **When not to:** Contract versioning and translation add work; Laravel does not enforce module visibility automatically.
- **Packages:** `laravel/framework`

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

### External integration adapters (Recommended)

*Score 13:* [Aaron Francis](https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=12570s) (Laracon speaker (employee), 3); [@laravel](https://laravel.com/docs/13.x/pennant#identifying-scope) (Tier 2, 3); [Matt Stauffer](https://www.youtube.com/watch?v=KBigS5vLwZk&t=1320s) (Laracon speaker, 2); [Steve McDougall](https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=22500s) (Laracon speaker, 2); [Sebastian Bergmann](https://www.youtube.com/watch?v=CpkHHxE5g7c&t=1440s) (Laracon speaker, 2); [Jeffrey Way](https://laracasts.com/episodes/2293) (Tier 3, 1)

- **Smell:** Local PDF or screenshot generation requires maintaining Chrome and Node on application servers.
- **Technique:** Put browser rendering behind a Sidecar-backed Browsershot adapter so callers retain a familiar rendering interface. For Laravel 13, resolve a compatible adapter and supported runtime rather than copying the historical Node 14 setup.
- **When to use:** Browser rendering is a specialized workload whose binaries burden the application host.
- **When not to:** Remote calls add latency and AWS/runtime maintenance; exact adapter compatibility must be checked.
- **Packages:** `hammerstone/sidecar`, `laravel/pennant`, `phpunit/phpunit`, `resend/resend-php`, `spatie/browsershot`, `stefanzweifel/sidecar-browsershot`

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

*Score 13:* [James Brooks](https://www.youtube.com/watch?v=he2-4Dbc-y0&t=720s) (Laracon speaker (employee), 3); [Edward](https://x.com/MaybeEdward/status/1765326244504018997) (Tier 2, 2); [James Mooring](https://www.youtube.com/watch?v=hpUrWWlBKCY&t=1164s) (Laracon speaker, 2); [John Drexler](https://www.youtube.com/watch?v=YJmuKPk3d9M&t=11255s) (Laracon speaker, 2); [Sam Sehnert](https://www.youtube.com/watch?v=IPbxscS0xiY&t=1260s) (Laracon speaker, 2); [Daniel Coulbourne](https://www.youtube.com/watch?v=OpCT3s4wTI0&t=1080s) (Laracon speaker, 2)

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

### Inertia server-driven application architecture (Recommended)

*Score 11:* [Taylor Otwell](https://www.youtube.com/watch?v=1P3wLy49t2c&t=450s) (Laracon speaker (employee), 3); [Simon Vrachliotis](https://www.youtube.com/watch?v=vZC-rVjuiLA&t=1254s) (Laracon speaker, 2); [Seb Armand](https://www.youtube.com/watch?v=9mDkf4popb8&t=810s) (Laracon speaker, 2); [Mary Perry](https://www.youtube.com/watch?v=vsxoaTgtyjw) (Laracon speaker, 2); [Diana Scharf](https://www.youtube.com/watch?v=50uRIFaUWqg&t=270s) (Laracon speaker, 2)

- **Smell:** A tightly coupled React frontend duplicates backend routing and data coordination.
- **Technique:** Use Inertia to connect React pages to Laravel routes, controllers and authentication within one application. Laravel sends page props through Inertia rather than requiring a separate SPA backend contract for those pages.
- **When to use:** A Laravel-owned React application benefits from a shared release and routing model.
- **When not to:** Independent API consumers or independently deployed frontends may still need a separate API.
- **Packages:** `@inertiajs/react`, `inertiajs/inertia-laravel`

Before:
```php
// Separate React router and Laravel API for the same page.
fetch("/api/puppies").then(setPuppies);
```
After:
```php
return Inertia::render("Puppies", [
    "puppies" => Puppy::all(),
]);
```

### Container-scoped state (Recommended)

*Score 10:* [Tim MacDonald](https://x.com/timacdonald87/status/1549943424450580480) (Tier 1, 3); [@laravel](https://laravel.com/docs/13.x/container#scoped-attribute) (Tier 2, 3); [Bobby Bouwmann](https://www.youtube.com/watch?v=-jo6QjeaJXQ&t=480s) (Laracon speaker, 2); [Jilesh Patadiya](https://www.youtube.com/watch?v=MBr5DxfEGLA&t=960s) (Laracon speaker, 2)

- **Smell:** Middleware and listeners maintain separate audit records or mutable state leaks across workers.
- **Technique:** Bind an auditor contract to a shared execution-scoped collector and inject it into middleware/listeners. Laravel 13 scoped() preserves one collector per request/job and resets it for persistent workers.
- **When to use:** Several execution participants contribute to one audit.
- **When not to:** Request scope alone cannot correlate later queued executions.
- **Packages:** `laravel/framework`

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

### Strategy pattern (driver selection) (Recommended)

*Score 10:* [@brendt_gd](https://stitcher.io/blog/strategies) (Tier 2, 3); [@laravel](https://laravel.com/cloud/docs/knowledge-base/generating-pdfs) (Tier 2, 3); [Mary Perry](https://www.youtube.com/watch?v=w7jKcbVcI54) (Laracon speaker, 2); [Jeffrey Way](https://laracasts.com/episodes/3436) (Tier 3, 1); [Katerina Trajchevska](https://laracasts.com/episodes/3220) (Tier 3, 1)

- **Smell:** Application code branches on storage providers and manually constructs interchangeable implementations.
- **Technique:** Use Laravel's manager and driver selection mechanisms to choose an implementation through configuration. For custom session storage, implement SessionHandlerInterface and register it with Session::extend().
- **When to use:** A framework subsystem already provides interchangeable drivers.
- **When not to:** A new driver requires testing and maintenance; this does not imply creating a factory for every application service.
- **Packages:** `spatie/laravel-pdf`

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

### Contextual dependency injection (Recommended)

*Score 9:* [Christoph Rumpel](https://christoph-rumpel.com/2024/9/all-laravel-php-attributes-at-your-disposal) (Tier 1, 3); [@laravel](https://laravel.com/docs/13.x/container#contextual-attributes) (Tier 2, 3); [laravel/framework PRs & discussions](https://github.com/laravel/framework/pull/55904) (Tier 2, 2); [Jeremy McPeak](https://laracasts.com/episodes/4127) (Tier 3, 1)

- **Smell:** Consumers manually choose service implementations or disks inside application logic.
- **Technique:** Declare contextual dependencies with Laravel’s supported container attributes or contextual bindings so the container supplies the correct implementation for that consumer.
- **When to use:** Different consumers need different configurations of the same dependency.
- **When not to:** Container configuration can obscure dependencies if used for simple local values.
- **Packages:** `laravel/framework`

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

### Repository extraction (Recommended)

*Score 8:* [Nuno Maduro](https://www.youtube.com/watch?v=9EGPo_enEc8&t=900s) (Laracon speaker (employee), 3); [Matt Stauffer](https://speakerdeck.com/mattstauffer/abstracting-too-early) (Laracon speaker, 2); [Mary Perry](https://www.youtube.com/watch?v=vsxoaTgtyjw) (Laracon speaker, 2); [Katerina Trajchevska](https://laracasts.com/episodes/3140) (Tier 3, 1)

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

### Object and service composition (Recommended)

*Score 7:* [Luke Downing](https://x.com/LukeDowning19/status/1825535110512341174) (Tier 2, 3); [Duilio Palacios](https://www.youtube.com/watch?v=1sixoOQliZI&t=960s) (Laracon speaker, 2); [Jeffrey Way](https://laracasts.com/episodes/3435) (Tier 3, 1); [@matthiasnoback](https://matthiasnoback.nl/2022/07/when-to-use-a-trait/) (Tier 3, 1)

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

### Hexagonal architecture (Useful)

*Score 5:* [Mary Perry](https://www.youtube.com/watch?v=vsxoaTgtyjw) (Laracon speaker, 2); [Paul Dragoonis](https://x.com/dr4goonis/status/1668213589696565250) (Tier 2, 2); [Jeremy McPeak](https://laracasts.com/episodes/4132) (Tier 3, 1)

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

### Runtime request dependency resolution (Useful)

*Score 5:* [@laravel](https://laravel.com/docs/13.x/octane) (Tier 2, 3); [Mateus Guimarães](https://www.youtube.com/watch?v=dOXXL-98Zbs) (Laracon speaker, 2)

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

### Serverless function extraction (Useful)

*Score 5:* [Aaron Francis](https://aaronfrancis.com/2022/adding-a-microservice-to-your-monolith-f35f5d4d) (Tier 1, 3); [Jilesh Patadiya](https://www.youtube.com/watch?v=MBr5DxfEGLA&t=870s) (Laracon speaker, 2)

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

*Score 4:* [Mary Perry](https://www.youtube.com/watch?v=vsxoaTgtyjw) (Laracon speaker, 2); [Paul Dragoonis](https://x.com/dr4goonis/status/1668213589696565250) (Tier 2, 2)

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

*Score 4:* [Duilio Palacios](https://www.youtube.com/watch?v=1sixoOQliZI&t=1320s) (Laracon speaker, 2); [Jeffrey Way](https://laracasts.com/episodes/3412) (Tier 3, 1); [Katerina Trajchevska](https://laracasts.com/episodes/3139) (Tier 3, 1)

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

### Container-backed facades (Useful)

*Score 4:* [Jason McCreary](https://www.youtube.com/watch?v=LaKEFjA25r4&t=1500s) (Laracon speaker, 2); [Mary Perry](https://www.youtube.com/watch?v=w7jKcbVcI54) (Laracon speaker, 2)

- **Smell:** Application code wants facade-style service calls without losing container substitution and mockability.
- **Technique:** Import an application service through the Facades namespace to obtain a real-time facade backed by Laravel’s container. Tests can mock that service through facade expectation helpers.
- **When to use:** Facade syntax suits an application’s conventions and the service is container-resolvable.
- **When not to:** Constructor or method injection makes dependencies more visible and can be clearer.

Before:
```php
(new Analyzer)->analyze($order);
```
After:
```php
use Facades\App\Services\Analyzer;
Analyzer::analyze($order);
// Test
Analyzer::expects("analyze")->with($order)->andReturn(3);
```

### Fluent configuration objects (Useful)

*Score 4:* [Spatie](https://freek.dev/2640-reusable-input-components-without-livewire-with-livewire) (Tier 2, 2); [Dan Harrin](https://www.youtube.com/watch?v=cucIWpAenro&t=3257s) (Laracon speaker, 2)

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

### Graceful dependency degradation (Useful)

*Score 4:* [Zuzana Kunckova](https://www.youtube.com/watch?v=Emm559sI6tI&t=180s) (Laracon speaker, 2); [Harris Raftopoulos](https://www.youtube.com/watch?v=E9t2lVn0z8A&t=1170s) (Laracon speaker, 2)

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

### HTTP client macros (Useful)

*Score 4:* [Wendell Adriel](https://x.com/wendell_adriel/status/1971548035646636514) (Tier 1, 3); [laravel/boost](https://github.com/laravel/boost/blob/d165b9ea6086294a8f1999ac3dcf4e61da267b23/.ai/laravel/skill/laravel-best-practices/rules/http-client.md) (Tier 3, 1); [Laravel News](https://laravel-news.com/laravel-http-client-tips) (Supporting, 0)

- **Smell:** Third-party integration calls repeat base URLs, credentials, and resilience configuration.
- **Technique:** Register a named Http macro in a service provider that returns a configured PendingRequest. Callers use the named client so timeouts, retries, authentication, and base URL are defined together.
- **When to use:** Several parts of the app call the same vendor API.
- **When not to:** Retrying non-idempotent requests can duplicate effects; a dedicated adapter is clearer for substantial domain behavior.
- **Packages:** `laravel/framework`

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

### HTTP integration request objects (Useful)

*Score 4:* [Spatie](https://spatie.be/blog/utilizing-claude-skills-in-client-projects) (Tier 2, 2); [Steve McDougall](https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=22680s) (Laracon speaker, 2)

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

*Score 4:* [Lucas Giovanny](https://www.youtube.com/watch?v=33CIhvkAPEY&t=420s) (Laracon speaker, 2); [Kévin Dunglas](https://www.youtube.com/watch?v=DJJt_2V26V8&t=120s) (Laracon speaker, 2)

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

*Score 4:* [Mateus Guimarães](https://www.youtube.com/watch?v=WqL6wgH1fKs) (Laracon speaker, 2); [Ryuta Hamasaki](https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=4770s) (Laracon speaker, 2)

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

*Score 4:* [Christoph Rumpel](https://laracasts.com/episodes/2904) (Tier 1, 3); [Jason McCreary](https://laracasts.com/episodes/3642) (Tier 3, 1)

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

*Score 4:* [Matt Stauffer](https://speakerdeck.com/mattstauffer/abstracting-too-early) (Laracon speaker, 2); [Spatie](https://www.youtube.com/watch?v=Boqfw4ZCk7g&t=120s) (Tier 2, 2)

- **Smell:** Multiple applications copy a coherent capability and fixes drift between copies.
- **Technique:** Extract a small stable Composer or NPM package once actual cross-project reuse is established. Keep its public API minimal and modularize the code internally before extraction.
- **When to use:** Several real consumers need the same coherent behavior.
- **When not to:** Package releases and consumer coordination add cost; speculative reuse is insufficient.

Before:
```php
// Project A and Project B each maintain a copied calculator.
```
After:
```php
// Extract the stable calculator into a shared Composer package.
$calculator = new Shared\Billing\InvoiceCalculator();
$total = $calculator->total($lines);
```

### AI tool classes with bounded access (Situational)

*Score 3:* [Jeffrey Way](https://laracasts.com/episodes/4079) (Tier 3, 1); [Jeremy McPeak](https://laracasts.com/episodes/4041) (Tier 3, 1); [@prism-php](https://github.com/prism-php/prism/blob/a207a10fef846fa2573c2a690bd06540e729bcef/docs/core-concepts/tools-function-calling.md) (Tier 3, 1)

- **Smell:** An agent’s tool schema and execution logic are mixed into its main loop.
- **Technique:** Extract each tool into a class that describes its inputs and implements its execution. Resolve the tools from Laravel and have the loop delegate calls to the appropriate tool.
- **When to use:** Several tools have independent validation, dependencies, or tests.
- **When not to:** One tiny tool may not justify a registry and class hierarchy.
- **Packages:** `laravel/ai`, `prism-php/prism`

Before:
```php
if ($call->name === "read_file") { /* inline filesystem logic */ }
```
After:
```php
$result = $tools[$call->name]->handle($call->arguments);
```

### Code-derived API contracts and documentation (Situational)

*Score 3:* [James Brooks](https://www.youtube.com/watch?v=K-g-t99mKZU&t=480s) (Laracon speaker (employee), 3)

- **Smell:** Hand-maintained endpoint documentation drifts from validation and serialization code.
- **Technique:** Generate documentation with Scribe using Laravel routes, real validation rules, API resources and supplemental metadata beside the code. Regenerate after changes and review examples the extractor cannot infer.
- **When to use:** An API contract needs maintained developer documentation.
- **When not to:** Generation needs deliberate examples and does not prove runtime compliance.
- **Packages:** `knuckleswtf/scribe`

Before:
```php
// Separate manually edited endpoint document.
```
After:
```php
/** @group Messages */
public function store(StoreMessageRequest $request) {
    return new MessageResource(Message::create($request->validated()));
}
// Build: php artisan scribe:generate
```

### Method-local execution state (Situational)

*Score 3:* [@laravel](https://laravel.com/docs/13.x/octane) (Tier 2, 3)

- **Smell:** Each request appends to a static array, causing unbounded Octane worker memory growth.
- **Technique:** Keep per-request intermediate data local and return it rather than retaining it globally.
- **When to use:** The accumulated data has no intentional cross-request purpose.
- **When not to:** Data must be shared or retained; use bounded external storage with explicit expiry.
- **Packages:** `laravel/octane`

Before:
```php
Service::$data[] = Str::random(10);
return Service::$data;
```
After:
```php
$data = [Str::random(10)];
return $data;
```

### Provider fallback chains (Situational)

*Score 3:* [Ryan Lovett](https://www.youtube.com/watch?v=AJ8MOFoDeP0&t=1050s) (Laracon speaker, 2); [Jeremy McPeak](https://laracasts.com/episodes/4044) (Tier 3, 1)

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

### Transient dependency bindings (Situational)

*Score 3:* [@laravel](https://laravel.com/docs/13.x/octane) (Tier 2, 3)

- **Smell:** A singleton holds a request object across worker requests.
- **Technique:** Change the service-container registration to bind so each resolution receives the current dependency.
- **When to use:** The service does not require singleton identity or expensive retained state.
- **When not to:** The service is retained by another long-lived object; use runtime inputs or a resolver instead.
- **Packages:** `laravel/octane`

Before:
```php
$this->app->singleton(Service::class,
    fn ($app) => new Service($app['request']));
```
After:
```php
$this->app->bind(Service::class,
    fn ($app) => new Service($app['request']));
```

### Vertical slices (Situational)

*Score 3:* [Jo Minney](https://www.youtube.com/watch?v=seX4HqMIAMk&t=940s) (Laracon speaker, 2); [Mateus Guimarães](https://laracasts.com/episodes/3045) (Tier 3, 1)

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

### API boundary for separate clients (Situational)

*Score 2:* [Philo Hermans](https://laracasts.com/episodes/3099) (Tier 3, 1); [Simon Hamp](https://laracasts.com/episodes/3973) (Tier 3, 1)

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

*Score 2:* [Spatie](https://www.youtube.com/watch?v=sizOaNo6YWI) (Tier 2, 2)

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

*Score 2:* [Spatie](https://freek.dev/2442-strategies-for-making-laravel-packages-customizable) (Tier 2, 2)

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

*Score 2:* [Chris Fidao](https://www.youtube.com/watch?v=fymjefGMUas) (Laracon speaker, 2)

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

*Score 2:* [Mary Perry](https://www.youtube.com/watch?v=vsxoaTgtyjw) (Laracon speaker, 2)

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

### Factory and manager pattern (Situational)

*Score 2:* [Mary Perry](https://www.youtube.com/watch?v=KroiagwnV0s&t=300s) (Laracon speaker, 2)

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

*Score 2:* [Mary Perry](https://www.youtube.com/watch?v=vsxoaTgtyjw) (Laracon speaker, 2)

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

*Score 2:* [Spatie](https://freek.dev/2442-strategies-for-making-laravel-packages-customizable) (Tier 2, 2)

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

### Shared REST and GraphQL resource definitions (Situational)

*Score 2:* [Kévin Dunglas](https://www.youtube.com/watch?v=DJJt_2V26V8&t=1170s) (Laracon speaker, 2)

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

*Score 2:* [Justin Reasoner](https://www.youtube.com/watch?v=DdMFP_K0gv4&t=1050s) (Laracon speaker, 2)

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

## Other

### Scoped state restoration (Recommended)

*Score 7:* [Aaron Francis](https://aaronfrancis.com/2024/dont-undo-what-you-havent-done-430ac590) (Tier 1, 3); [@laravel](https://laravel.com/docs/13.x/database#running-analytical-queries-on-vitess) (Tier 2, 3); [laravel/framework](https://github.com/laravel/framework/pull/61973) (Tier 3, 1)

- **Smell:** A helper toggles a global flag and restores an assumed default, breaking nested callers.
- **Technique:** Capture the previous state before changing it and restore that exact value in a finally block. Apply the pattern to temporary framework flags or application context so exceptions and nested calls preserve the caller state.
- **When to use:** A method temporarily changes ambient state it does not own.
- **When not to:** Prefer passing state explicitly when possible; global state is especially delicate in long-lived workers.
- **Packages:** `laravel/framework`, `laravel/nightwatch`

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

### HTTP connection reuse (Useful)

*Score 5:* [Aaron Francis](https://aaronfrancis.com/2024/make-5x-faster-outbound-requests-in-laravel-192e8e98) (Tier 1, 3); [Thiery Laverdure](https://www.youtube.com/watch?v=E4_QSEfInrc&t=1500s) (Laracon speaker, 2)

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

*Score 4:* [Mateus Guimarães](https://www.youtube.com/watch?v=dOXXL-98Zbs) (Laracon speaker, 2); [Kévin Dunglas](https://www.youtube.com/watch?v=PzOLyz1vr5I&t=780s) (Laracon speaker, 2)

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

*Score 3:* [Aaron Francis](https://aaronfrancis.com/2024/clean-up-after-yourself-618236cb) (Tier 1, 3)

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

### Cohort-preserving feature kill switches (Situational)

*Score 3:* [@laravel](https://laravel.com/docs/13.x/pennant) (Tier 2, 3)

- **Smell:** Disabling a feature by overwriting or purging stored assignments destroys the existing rollout cohort.
- **Technique:** Place the emergency override in a class-based before method and return null when normal stored behavior should resume.
- **When to use:** A temporary outage needs an immediate flag override while preserving user assignments.
- **When not to:** The goal is permanently changing assignments for all existing users.
- **Packages:** `laravel/pennant`

Before:
```php
// Erases the enabled cohort:
Feature::deactivateForEveryone(NewApi::class);
```
After:
```php
public function before(User $user): mixed {
    return config('features.new-api.disabled')
        ? $user->isInternalTeamMember()
        : null;
}
```

### Cooperative event-loop integration (Situational)

*Score 3:* [Joe Dixon](https://www.youtube.com/watch?v=60g-Z-dAWNk&t=540s) (Laracon speaker (employee), 3)

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

### Credential provider chains (Situational)

*Score 3:* [@laravel](https://laravel.com/cloud/docs/private-cloud/iam-roles) (Tier 2, 3)

- **Smell:** AWS clients embed static keys and require manual rotation across code and environments.
- **Technique:** Attach a scoped Private Cloud IAM role, remove static AWS key variables, and let the SDK default provider resolve rotating workload credentials.
- **When to use:** The app runs on Private Cloud and accesses AWS services supported by its role.
- **When not to:** The destination is Cloudflare R2 or another service requiring independent explicit credentials.
- **Packages:** `aws/aws-sdk-php`

Before:
```php
$s3 = new \Aws\S3\S3Client([
    'region' => 'us-east-1', 'version' => 'latest',
    'credentials' => ['key' => $accessKey, 'secret' => $secretKey],
]);
```
After:
```php
// Attach the IAM role, remove static AWS key variables, then deploy.
$s3 = new \Aws\S3\S3Client([
    'region' => config('services.aws.region'),
    'version' => 'latest',
]);
```

### HTTP client tracing middleware (Situational)

*Score 3:* [@laravel](https://nightwatch.laravel.com/docs/outgoing-requests) (Tier 2, 3)

- **Smell:** Standalone Guzzle calls escape automatic Laravel HTTP telemetry or repeat manual tracing wrappers.
- **Technique:** Attach Nightwatch middleware once to the Guzzle HandlerStack and reuse that client.
- **When to use:** A package or service needs direct Guzzle rather than the Laravel Http facade.
- **When not to:** Automatic Laravel Http instrumentation already captures all calls.
- **Packages:** `guzzlehttp/guzzle`, `laravel/nightwatch`

Before:
```php
$client = new \GuzzleHttp\Client();
$client->get($url);
```
After:
```php
$stack = \GuzzleHttp\HandlerStack::create();
$stack->push(Nightwatch::guzzleMiddleware());
$client = new \GuzzleHttp\Client(['handler' => $stack]);
$client->get($url);
```

### Normalized telemetry dimensions (Situational)

*Score 3:* [@laravel](https://laravel.com/docs/13.x/pulse) (Tier 2, 3)

- **Smell:** Every record ID creates a separate metric row, hiding aggregate cache behavior.
- **Technique:** Use Pulse recorder groups to replace numeric IDs with a wildcard.
- **When to use:** Many cache keys differ only by entity ID.
- **When not to:** Each individual key is operationally significant and grouping would hide failures.
- **Packages:** `laravel/pulse`

Before:
```php
Recorders\CacheInteractions::class => ['groups' => []],
```
After:
```php
Recorders\CacheInteractions::class => [
    'groups' => ['/:\d+/' => ':*'],
],
```

### Presigned multipart uploads (Situational)

*Score 3:* [Pascal Baljet](https://x.com/pascalbaljet/status/1481251161487425539) (Tier 1, 3)

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

*Score 3:* [Josh Cirre](https://www.youtube.com/watch?v=RY6wrHaLeCo&t=1200s) (Laracon speaker (employee), 3)

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

### Self-healing slug URLs (Situational)

*Score 3:* [Aaron Francis](https://aaronfrancis.com/2023/self-healing-urls-in-laravel-e8886c3e) (Tier 1, 3)

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

*Score 2:* [Spatie](https://freek.dev/3092-streaming-ai-replies-from-laravel-to-react) (Tier 2, 2)

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

### Declarative code scaffolding (Situational)

*Score 2:* [Jason McCreary](https://www.youtube.com/watch?v=Jna0DOgnxds&t=420s) (Laracon speaker, 2)

- **Smell:** Models, migrations, routes and controllers repeatedly describe the same feature fields by hand.
- **Technique:** Describe models and controller operations in a Blueprint YAML draft and generate coordinated Laravel components with blueprint:build. Review and customize the generated code using a Laravel 13-compatible package revision.
- **When to use:** A conventional feature needs repetitive coordinated scaffolding.
- **When not to:** This mention is supported by chapter topics and package workflow rather than a verified transcript; generation cannot infer all business requirements.
- **Packages:** `laravel-shift/blueprint`

Before:
```php
// Manually repeat Post fields in model, migration and factory.
```
After:
```php
models:
  Post:
    title: string
    content: longtext
# php artisan blueprint:build
```

### Multi-stage asset builds (Situational)

*Score 2:* [Francesco Ciulla](https://www.youtube.com/watch?v=CMqhOhRnOY0&t=690s) (Laracon speaker, 2)

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
