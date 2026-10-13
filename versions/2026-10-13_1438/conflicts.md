# Conflicts needing a decision

1. Codebase — Persistence coupling: independent domain entities or Eloquent-backed domains.
2. Codebase — Test isolation: collaborator-focused tests or request-boundary tests with real persistence.
3. Codebase — Event payload semantics: event-time values or participating model references.
4. Codebase — Validation location: model-centered rules or operation-specific validation.
5. Codebase — Universal TDD obligation: every kind of work or risk-based testing discipline.
6. Codebase — Form Requests: avoid them or use them for substantial validation.
7. Codebase — Client validation timing: early feedback or submission-only backend errors.
8. Codebase — Interactive frontend preference: JavaScript page components or Blade with Livewire and Alpine.
9. Codebase — Compromised-password policy: permit warnings or require rejection.
10. Codebase — Reusable-action validation: validate inside actions or require validated inputs.
11. Codebase — Workflow status: stored current status or event-history-derived state.
12. Infra — Staging before release: staging QA or direct production delivery.

## Codebase

### 1. Persistence coupling

- Side A: Keep entities independent of the framework and persistence behind replaceable adapters — sources: 037-QaIGN_cTcc8 <https://www.youtube.com/watch?v=QaIGN_cTcc8&t=660s>; 152-NeXQEJNWO5w <https://www.youtube.com/watch?v=NeXQEJNWO5w&t=1980s> <https://www.youtube.com/watch?v=NeXQEJNWO5w&t=3330s>.
- Side B: Use Eloquent-backed domains and introduce abstractions only when their value is demonstrated — sources: 029-6SBjKOwVq0o <https://www.youtube.com/watch?v=6SBjKOwVq0o&t=2010s>; 049-5DVDewOReoY <https://www.youtube.com/watch?v=5DVDewOReoY&t=60s>.
- Recommendation: Prefer Eloquent for straightforward workflows; isolate a business core when domain complexity or change costs justify it (framework independence adds mapping and maintenance costs).

### 2. Test isolation

- Side A: Design and test object interactions using substituted collaborators — sources: 038-X6y-OyMPqfw <https://www.youtube.com/watch?v=X6y-OyMPqfw> <http://www.slideshare.net/everzet/design-how-your-objects-talk-through-mocking>; 000-ajoFwWwSHTI <https://www.youtube.com/watch?v=ajoFwWwSHTI> <https://code.tutsplus.com/testing-laravel-controllers--net-31456t> (controller-isolation attribution is reconstructed and untimed).
- Side B: Prioritize request-boundary behavior tests with real persistence — sources: 132-jQ1vYzoCubA <https://www.youtube.com/watch?v=jQ1vYzoCubA> <https://codingjames.ca/my-notes-from-laracon-online-2017/> <https://tighten.com/insights/q-and-a-with-the-speakers-from-laracon-online-2017/> (untimed).
- Recommendation: Prefer feature tests for application confidence and interaction tests for required collaborator contracts (mocked messages alone cannot establish query or wiring correctness; the decision concerns the default testing boundary, not whether both forms can coexist).

### 3. Event payload semantics

- Side A: Carry immutable IDs and event-time values, avoiding mutable entities — sources: 037-QaIGN_cTcc8 <https://www.youtube.com/watch?v=QaIGN_cTcc8&t=1260s>.
- Side B: Carry explicit participating Eloquent models in events — sources: 070-FOSpYr7jDog <https://www.youtube.com/watch?v=FOSpYr7jDog&t=3450s> <https://www.youtube.com/watch?v=FOSpYr7jDog&t=4140s>.
- Recommendation: Use snapshot values for historical business facts and model references when current-state rehydration is intended (queued Eloquent serialization re-fetches database state rather than preserving event-time values).

### 4. Validation location

- Side A: Center all validation and business rules on the data model — sources: 193-R_0Q5rXGHXg <https://www.youtube.com/watch?v=R_0Q5rXGHXg&t=600s>.
- Side B: Separate operation-specific validation from persistence — sources: 036-XfXcnjgM6Aw <https://image.slidesharecdn.com/laraconeutalk2014-140829083735-phpapp02/85/Comprehensive-Validation-with-Laravel-4-80-320.jpg> <https://image.slidesharecdn.com/laraconeutalk2014-140829083735-phpapp02/85/Comprehensive-Validation-with-Laravel-4-43-320.jpg>; 018-EcBIzAHIYAw <https://www.youtube.com/watch?v=EcBIzAHIYAw&t=1530s>.
- Recommendation: Keep business invariants in domain operations and contextual input validation and authorization in Form Requests or shared validators (input rules differ by use case, and non-Eloquent writes bypass model hooks).

### 5. Universal TDD obligation

- Side A: Advocate TDD for everyone — sources: 152-NeXQEJNWO5w <https://www.youtube.com/watch?v=NeXQEJNWO5w&t=2730s>.
- Side B: Judge testing automation by behavioral risk and permit exemptions for low-value work — sources: 071-LbPx8gdgLFc <https://www.youtube.com/watch?v=LbPx8gdgLFc&t=870s>.
- Recommendation: Use small TDD steps where helpful and prioritize consequential behavior rather than imposing the same discipline on every task (the disagreement concerns universal applicability, not support for testing).

### 6. Form Requests

- Side A: Avoid Form Requests because of authorization boilerplate — sources: 195-FNyUX2jVMeE <https://www.youtube.com/watch?v=FNyUX2jVMeE&t=1860s>.
- Side B: Use Form Requests for substantial validation and custom messages — sources: 232-rkdlJeHTeCY <https://speakerdeck.com/caneco/the-art-of-form-validation>; 246-TdPrfAmbVWY <https://www.youtube.com/watch?v=TdPrfAmbVWY&t=1440s>.
- Recommendation: Prefer Form Requests for substantial validation and preparation (they make that responsibility explicit; authorization can live elsewhere, with `authorize()` omitted or returning true). Retain this as a preference disagreement rather than treating historical boilerplate as a current framework requirement.

### 7. Client validation timing

- Side A: Give early browser validation feedback and avoid obviously invalid submissions — sources: 232-rkdlJeHTeCY <https://speakerdeck.com/caneco/the-art-of-form-validation> <https://van-ons.nl/en/blog/development/4-take-outs-van-laracon-2019-het-laravel-event-van-het-jaar/>.
- Side B: Submit and display backend errors rather than duplicating validation rules — sources: 260-O9jTwPHB-wA <https://www.youtube.com/watch?v=O9jTwPHB-wA&t=1170s> <https://www.youtube.com/watch?v=O9jTwPHB-wA&t=1200s>.
- Recommendation: Keep server rules authoritative and add early feedback where useful through simple local checks or Precognition (submission-only feedback reduces duplication but delays feedback; backend rule reuse can reduce that tradeoff).

### 8. Interactive frontend preference

- Side A: Prefer JavaScript page components — sources: 224-8to7QtuD3is <https://www.youtube.com/watch?v=8to7QtuD3is&t=60s> <https://www.youtube.com/watch?v=8to7QtuD3is&t=270s>; 279-3MABSYPxpbs <https://www.youtube.com/watch?v=3MABSYPxpbs&t=360s> <https://www.youtube.com/watch?v=3MABSYPxpbs&t=1260s>.
- Side B: Prefer Blade with selective Livewire and Alpine — sources: 317-K5nk15XOMTI <https://www.youtube.com/watch?v=K5nk15XOMTI&t=30s>; 290-bmtDuUXiiqU <https://www.youtube.com/watch?v=bmtDuUXiiqU&t=200s> <https://www.youtube.com/watch?v=bmtDuUXiiqU&t=401s>.
- Recommendation: Choose the default stack by interaction requirements and team skills (these are competing stack preferences, and both approaches can avoid a separate API).

### 9. Compromised-password policy

- Side A: Allow either a warning or rejection for a known compromised password — sources: 320-BBuUrHXh2Ug <https://www.youtube.com/watch?v=BBuUrHXh2Ug&t=480s> <https://www.youtube.com/watch?v=BBuUrHXh2Ug&t=570s>.
- Side B: Reject known compromised passwords — sources: 340-bFMBHRlEOjo <https://stephenreescarter.net/talks/hacking-laravel/#password-security> <https://stephenreescarter.net/should-you-block-compromised-passwords/> (timestamp unavailable).
- Recommendation: Reject compromised passwords when creating or changing credentials and document deliberate exceptions (warnings permit continued use of credentials already known to attackers).

### 10. Reusable-action validation

- Side A: Validate registration input inside the reusable action — sources: 359-0Rq-yHAwYjQ <https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=1890s> <https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=2520s>.
- Side B: Pass already-validated attributes into actions — sources: 435-BNhbgcNJyAk <https://www.youtube.com/watch?v=BNhbgcNJyAk&t=540s> <https://www.youtube.com/watch?v=BNhbgcNJyAk&t=600s>.
- Recommendation: Keep actions independent of HTTP requests and ensure every HTTP or CLI caller crosses the same validation policy; validate inside the action when callers could otherwise bypass it (reusability requires a clear validation contract at every entry point). This concerns the action's input contract, separately from model-centered validation in conflict 4.

### 11. Workflow status

- Side A: Persist explicit current status and resolve state objects from it — sources: 402-1A1xFtlDyzU <https://speakerdeck.com/jacobbennett/state-machines-in-laravel> <https://gist.github.com/markjaquith/dcb0c32cf670ae22d1c96a608bff1672> (timestamp unavailable).
- Side B: Reject status-column-driven designs in favor of event history — sources: 419--mb2L97Yq88 <https://www.youtube.com/watch?v=-mb2L97Yq88&t=450s> <https://www.youtube.com/watch?v=-mb2L97Yq88&t=810s> <https://www.youtube.com/watch?v=-mb2L97Yq88&t=240s>.
- Recommendation: Use status for current workflow control and durable events when historical reconstruction is required; status may remain an event-derived projection (current state and reconstructable history serve different needs).

## Infra

### 12. Staging before release

- Side A: Validate on staging before automated production deployment — sources: 375-0Rq-yHAwYjQ <https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=33660s> <https://www.youtube.com/watch?v=0Rq-yHAwYjQ&t=33840s>.
- Side B: Omit staging when tests and controlled feature flags support direct production delivery — sources: 548-hpUrWWlBKCY <https://www.youtube.com/watch?v=hpUrWWlBKCY&t=225s> <https://www.youtube.com/watch?v=hpUrWWlBKCY&t=432s>.
- Recommendation: Keep staging for risky dependencies and migrations unless automated coverage, reversible rollout, and operational confidence justify omitting it (the workflows disagree on the release gate and assume different levels of release risk).
