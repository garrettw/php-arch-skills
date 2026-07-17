---
name: action-domain-responder
description: Use this skill when building server-side web/HTTP request handling, controllers, views, or templates in PHP frameworks, or when a framework's "MVC" terminology causes confusion about where logic belongs. Triggers on questions about controllers, the Web MVC pattern, Action-Domain-Responder, view/template separation, or keeping HTTP and framework concerns out of the domain.
---

# Web Presentation Patterns for PHP

This skill governs the **web/HTTP layer** — the thin edge that turns an HTTP request into a call on the application and turns the result into an HTTP response. Its central job is to keep that layer from leaking into the domain, and to correct a widespread misnomer.

## The most important correction: "Web MVC" is not MVC

What PHP (and Rails, Spring, etc.) frameworks call "MVC" is not the original Model-View-Controller. It is **Model 2** — Sun's server-side appropriation of the *names*, popularized by Apache Struts (Java) and Ruby on Rails, then copied by PHP frameworks. The original MVC (Smalltalk-80, 1979) was a *client-side, in-memory GUI* pattern: one MVC triad **per screen element**, connected by an observer graph that continuously notified views/controllers of state changes in memory. That has nothing to do with receiving one HTTP request and returning one HTTP response.

Worse, Model 2 prescribes putting **business logic in the Controller** — a UI component. That is the origin of the "fat controller" anti-pattern, and it directly contradicts the original MVC (where the *model* does the work). See [web-mvc-is-not-mvc.md](references/web-mvc-is-not-mvc.md) for the full distillation.

## The better mental model: Action-Domain-Responder

Reframe the framework "controller" using **Action-Domain-Responder** (Paul M. Jones):

- **Action** — receives the HTTP request, invokes the domain, and builds (not renders) the response. HTTP glue only.
- **Domain** — "whatever does the domain work": the application/Service layer (orchestration, use cases) *and* the DDD domain layer beneath it (entities, policies, invariants). In ADR the Domain is the entry point into all of that, not just one layer.
- **Responder** — builds the *entire* HTTP response: status, headers, cookies, content-type, and body. A view/template is only one mechanism the Responder may use for the body; the Responder owns all response construction, not just rendering.

A framework "controller" is really an **Action + Responder** pair; the "model" is really the **Domain** plus the persistence behind it. Not every framework that claims MVC actually separates these — many controllers still mix request handling, domain calls, and response building. Use ADR as the *placement* rule, not a guarantee of the scaffold. See the caveat in [web-mvc-is-not-mvc.md](references/web-mvc-is-not-mvc.md). For per-framework guidance on getting as close to proper ADR as each framework allows, see [adr-by-framework.md](references/adr-by-framework.md). The classic patterns these build on — Page Controller, Front Controller, Template/Transform/Two Step View, Application Controller — are catalogued in [classic-web-patterns.md](references/classic-web-patterns.md) as vocabulary for recognizing what the framework already gives you.

## Workflow: placing web-layer code

If adding a route/endpoint/controller in a PHP framework:
1. **Identify the Action.** The handler receives the request, validates input, and delegates. It holds no business rules.
2. **Delegate to the Domain.** Call an application-layer handler/service (see [application-layer](../application-layer/SKILL.md)) or a domain operation. Do not put processing logic in the action.
3. **Separate the Responder.** Build the full HTTP response (status, headers, body) in a dedicated Responder — not inline in the action. A view/template is just the mechanism the Responder uses for the body; the Responder owns the whole response.
4. **Keep the Domain HTTP-free.** The domain must not import the framework request/response or HTTP client (see [infrastructure-boundaries](../infrastructure-boundaries/SKILL.md)).

## Boundaries

### Always Do
- Always treat the framework controller as a thin **Action**: receive input, delegate to the domain, hand off to a responder.
- Always put business logic in the **Domain** (application layer / domain model), never in the web/HTTP layer.
- Always separate response building (status, headers, cookies, content-type, body) from request handling — think Action + Responder, not "fat controller."
- Always keep the domain free of framework and HTTP concerns.

### Ask First
- Ask whether the framework's "controller" is already mixing Action and Responder concerns; if so, plan to split them rather than assume the scaffold did.

### Never Do
- Never put business/processing logic in the controller "because Web MVC says so" — that is the Model 2 fat-controller anti-pattern.
- Never let the domain import the HTTP request/response, framework container, or view templating engine.
- Never build the HTTP response directly inside the action (setting status, headers, or rendering the body) when a separate Responder keeps the seam clean.

## Related Skills
- [application-layer](../application-layer/SKILL.md): the Domain in ADR includes the Application/Service layer (the Action delegates to it), but it also spans the DDD Domain layer — entities, policies, and invariants.
- [infrastructure-boundaries](../infrastructure-boundaries/SKILL.md): the domain must stay free of framework and HTTP concerns; adapters sit at the same edge as the web layer.
- [framework-integration](../framework-integration/SKILL.md): framework-specific mappings for keeping controllers thin across Laravel, Symfony, Slim, CakePHP, CodeIgniter, Yii, Laminas, Mezzio.
- [distribution-patterns](../distribution-patterns/SKILL.md): a Remote Facade is the cross-process cousin of a web Action; both define an inbound boundary and must hold no domain logic.
- [adr-by-framework.md](references/adr-by-framework.md): concrete steps to reach a proper Action-Domain-Responder split in each listed framework.
- [classic-web-patterns.md](references/classic-web-patterns.md): the original patterns (Page/Front Controller, Template/Transform/Two Step View, Application Controller) as vocabulary for what the framework already provides.
