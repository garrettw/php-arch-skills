# Web "MVC" Is Not MVC

*Distilled from Paul M. Jones, [MVC-MODEL-2 (Action-Domain-Responder)](https://github.com/pmjones/adr/blob/master/MVC-MODEL-2.md).*

## The core correction

What server-side developers call "MVC" is not MVC. It is a different thing that borrowed the name. To use the label correctly you have to separate three ideas that get collapsed into one:

1. **Original MVC** — a 1979 client-side, in-memory *GUI* pattern (Smalltalk-80, Xerox PARC).
2. **"Model 2"** — Sun's server-side appropriation of MVC's component names, solidified by Apache Struts (Java), later popularized by Ruby on Rails. This is what nearly everyone means by "Web MVC."
3. **The modern web framework "controller"** — Rails-style request handlers that subsequent frameworks (including PHP's) copied.

MVC was originally a *user interface* pattern, not an application architecture. It describes how a single graphical screen is wired together — not how to structure a whole server-side system.

## How original MVC actually worked

In Smalltalk-80, MVC was not one triad for the whole app — it was **one MVC triad per screen element**: each text field, button, and menu had its own model, view, and controller. The pieces were connected by a subject/observer messaging system so they could continuously notify each other of state changes in memory:

- The **controller** accepted raw input events (mouse movement, key presses, button clicks) and interpreted them.
- The **model** performed the work and changed its internal state.
- The model then **broadcast** the change to all registered views and controllers, which re-read the model and re-rendered or re-enabled themselves.

That is a live, stateful, event-driven desktop GUI. A server-side app does none of that: it receives one complete HTTP request and returns one complete HTTP response. There is no in-memory observer graph, no continuous mouse tracking, no per-element triads. **MVC was never designed for over-the-network, request/response UIs.**

## How "Web MVC" (Model 2) diverged

Sun appropriated the *names* Model/View/Controller for the server side and changed what they meant and how they collaborate. The 1999 Model 2 article is explicit about the result:

> "Properly applied, the Model 2 architecture should result in the concentration of all of the processing logic in the hands of the controller servlet, with the JSP pages responsible only for the view."

So "Web MVC" prescribes putting **business logic in the Controller** — a UI component. That directly contradicts the original MVC description, where the model does the work and the controller only interprets input. Struts codified this; Rails made it famous; PHP frameworks inherited it.

This is the root of the "fat controller" anti-pattern: the pattern itself tells you to put processing logic in the controller.

## The better redefinition: Action-Domain-Responder (ADR)

If MVC doesn't fit and Model 2 mislabels the parts, the cleaner server-side pattern is **Action-Domain-Responder** (Paul M. Jones):

- **Action** — receives the HTTP request, invokes the domain, and builds (not renders) the response. The web/HTTP-specific input/output glue.
- **Domain** — the application and business logic, independent of HTTP. (This is where the real work goes, not in the Action.)
- **Responder** — builds the *entire* HTTP Response: status, headers, cookies, content-type, and body. A view or template is only one mechanism the Responder may use for the body; the Responder itself owns all HTTP-response construction, not just rendering.

ADR maps more honestly than "MVC" onto what frameworks actually do: the framework "controller" is really an **Action + Responder** pair, and the "model" is really the **Domain** plus the persistence/ORM behind it. In effect, ADR is "Web MVC" with the names and responsibilities corrected.

### Caveat: not every "MVC" framework is cleanly ADR

Do not assume a framework claiming MVC (or ADR) actually separates the pieces:

- Many frameworks' "controller" still mixes Action and Responder concerns (rendering happens inside the controller method) — closer to Model 2 than ADR.
- Some lump Domain work into the "model" layer that is really just an Active Record / ORM entity tied to the database, not a separated domain.
- Laravel/Symfony controllers, for example, are closer to ADR *Actions* only if you keep them thin and push logic into a service/application layer and a separate Responder that owns the full HTTP response. The framework does not enforce this for you.

Use ADR as the **mental model** for how to place code, not as a guarantee of what the framework scaffold gives you. When the framework's "controller" bundles request handling, domain calls, and response building together, treat splitting those out as the refactor target — not as something already done.

## Why this matters here

This repository's other skills already forbid the fat-controller style: the [application-layer](../application-layer/SKILL.md) skill puts orchestration in a Service/Handler, and the [infrastructure-boundaries](../infrastructure-boundaries/SKILL.md) skill keeps the domain free of framework and HTTP concerns. "Web MVC" as commonly practiced fights those rules. Reframing the framework controller as an ADR **Action** (thin, HTTP-only) that delegates to the **Domain** (your Application/Service layer *and* the DDD domain layer — entities, policies, invariants) and hands off to a **Responder** — which builds the entire HTTP response, using a view/template only as its body mechanism — aligns the web layer with the rest of the architecture instead of contradicting it.
