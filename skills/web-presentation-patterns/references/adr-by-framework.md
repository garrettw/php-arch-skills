# ADR by Framework

How to get as close to a proper **Action-Domain-Responder** implementation as each PHP framework allows. This is the web-presentation companion to the [framework-integration](../framework-integration/SKILL.md) skill: that skill covers ports/adapters and thin controllers in general; this one focuses on the three ADR roles (Action, Domain, Responder) and how each framework's native pieces map onto them.

The universal goal, regardless of framework:

- **Action** = one request handler (one class/closure with one main method), HTTP-only: read input, call the Domain, hand the result to a Responder.
- **Domain** = the application/Service layer *and* the DDD domain layer; never touched by HTTP objects.
- **Responder** = owns the *entire* HTTP response (status, headers, cookies, content-type, body). A template/view is only its body mechanism.

## Laravel

- **Action.** One invokable controller class per use case (`__invoke`), or a controller method that does only request → DTO → handler → responder. Avoid multi-action controller classes when you can; if you keep them, don't add pre/post hooks that blur responsibilities.
- **Domain.** Application Handlers / domain services called from the controller. Keep business logic out of Eloquent models (see [laravel.md](../../framework-integration/references/laravel.md)).
- **Responder.** The framework nudges you to `return view(...)` or `response()->json()` *inside* the controller — that mixes Action and Responder. Extract a Responder class that receives the Domain result (and the Request) and returns the `Response`. For JSON APIs, a Responder per resource or per response-shape works well; for HTML, a Responder that calls the Blade view and sets headers/status/cookies.
- **Gotcha.** `$request->*` belongs in the Action/Responder, never in the Domain. Convert to a DTO at the boundary.

## Symfony

- **Action.** A controller method or invokable controller. Symfony's structure already fits ADR: thin controllers whose "only job is to convert request → handler input → response" (see [symfony.md](../../framework-integration/references/symfony.md)). Prefer one invokable controller per route.
- **Domain.** Application Services / handlers receive validated DTOs or command objects built from the `Request`. Doctrine entities stay persistence-aware; invariants live in the application/domain layer.
- **Responder.** `AbstractController` helpers (`$this->render()`, `$this->json()`) tempt you to build the response in the controller. Move response construction into a Responder that returns a `Response` (set status, headers, cookies, and the rendered body). For APIs, a Responder that serializes a DTO and sets the content-type keeps the Action free of presentation.

## Slim

- **Action.** Slim is already ADR-friendly: each route maps to a callable/invokable object — effectively a single Action. PSR-7 `Request`/`Response` are clean value objects (see [slim.md](../../framework-integration/references/slim.md)).
- **Domain.** Call handlers with validated DTOs; never pass the raw `ServerRequest` into the Domain.
- **Responder.** In Slim the route callable often *is* the place the Response is built. Keep that building in a separate Responder invoked by the callable: the callable collects input, calls the Domain, then calls a Responder that returns the PSR-7 `Response` (body via a template engine or JSON encode, plus status/headers).

## CakePHP 5.x

- **Action.** Cake's `Controller` + `ControllerAction` model groups many actions per controller and encourages fat controllers. To approximate ADR, keep each action method to input → handler → responder and resist putting logic in the controller. CLI `Command` classes are driver adapters that should delegate the same way (see [cakephp-5.md](../../framework-integration/references/cakephp-5.md)).
- **Domain.** Delegate to Application Handlers; do not let `Table`/Entity objects become the domain.
- **Responder.** Cake renders views from the controller (`$this->set()` + `$this->viewBuilder()`). Extract a Responder that sets view vars, status, headers, and cookies, and returns the response. For JSON, a Responder that calls `$this->viewBuilder()->setClassName('Json')` (or builds the body directly) keeps presentation out of the action.

## CodeIgniter 4

- **Action.** CI4 gives Controllers/Models/Views with no architectural guidance, so you must enforce ADR yourself: one thin controller action that delegates to an Application Handler (see [codeigniter-4.md](../../framework-integration/references/codeigniter-4.md)). Validation happens at the boundary (controller), not in the Domain.
- **Domain.** Handlers / domain services; CI4 Models are adapters, not domain objects.
- **Responder.** Returning data from the controller and letting a view render it mixes roles. Build a Responder that produces the `Response` (HTML via the view, or JSON with explicit status/headers). Keep the Action free of both rendering and response shaping.

## Yii 3

- **Action.** Yii controllers tend to be multi-action and AR-centric (see [yii-3.md](../../framework-integration/references/yii-3.md)). To get close to ADR, treat each controller action as an Action: input → handler → responder, no business logic inside.
- **Domain.** Application Services / handlers; domain logic does not belong in ActiveRecord models.
- **Responder.** Yii's `render()` and response helpers live in the controller. Move full response construction (status, headers, body, content-type) into a Responder the action invokes.

## Laminas / Mezzio

- **Action.** Laminas MVC uses multi-action controllers; **Mezzio** (middleware) is closer to ADR — each route is a single handler/middleware that is essentially an Action (see [mezzio.md](../../framework-integration/references/mezzio.md)). Prefer Mezzio-style single-responsibility handlers. For full Laminas MVC, keep each action method to input → handler → responder.
- **Domain.** Handlers receive validated DTOs; persistence is a `TableGateway` adapter in `infrastructure/persistence` (see [laminas.md](../../framework-integration/references/laminas.md)).
- **Responder.** In Mezzio the middleware often returns the `Response` directly; delegate that construction to a Responder (status, headers, body via a renderer or JSON). PSR-7 `Request`/`Response` are clean value objects — pass only validated data to the Domain.

## How close you can get

The frameworks differ in how much they fight ADR:

- **Closest out of the box:** Slim and Mezzio (single-responsibility request handlers, PSR-7). ADR is almost native.
- **Close with discipline:** Symfony (thin controllers, invokable actions) and Laravel (invokable controllers, responder extraction).
- **Requires the most enforcement:** CakePHP, CodeIgniter, Yii, full Laminas MVC — they default to multi-action controllers that mix Action + Responder. You must extract the Responder and keep the Domain out of the controller yourself.

In every case the milestone is the same: the **Action** knows nothing about presentation, the **Domain** knows nothing about HTTP, and the **Responder** owns the whole response.

## Related

- [web-mvc-is-not-mvc.md](web-mvc-is-not-mvc.md): why "Web MVC" mislabels these roles and why ADR is the honest reframing.
- [classic-web-patterns.md](classic-web-patterns.md): the original Fowler patterns (Page/Front Controller, Template/Transform/Two Step View, Application Controller) that these framework mechanisms implement.
- [framework-integration](../framework-integration/SKILL.md): ports/adapters, composition roots, and per-framework anti-patterns. Its references ([laravel.md](../../framework-integration/references/laravel.md), [symfony.md](../../framework-integration/references/symfony.md), etc.) cover the boundary/Domain side; this file covers the Action/Responder side.
- [application-layer](../../application-layer/SKILL.md): the Domain role in ADR spans both the application/Service layer and the DDD domain layer.
