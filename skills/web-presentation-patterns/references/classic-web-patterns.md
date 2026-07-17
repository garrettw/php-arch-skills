# Classic Web Presentation Patterns

Today, every PHP framework implements these six patterns natively, so they rarely need *new* code — but the agent should recognize them and, more importantly, know the **one decision each implies**. They are the *mechanisms* an ADR [Responder](../SKILL.md) or [Action](../SKILL.md) uses; they are not a substitute for the Action/Domain/Responder boundary.

## Front Controller and Page Controller

These two are not a binary choice; in almost every PHP app and framework they coexist as two layers:

- **Front Controller** — a *single* handler that receives *every* request first. It boots the framework, runs cross-cutting behavior (auth, i18n, logging, error handling), and then dispatches to the right per-request handler. In PHP this is the `index.php` bootstrap plus the framework kernel/router. There is exactly one of these per application.
- **Page Controller** — the *per-route* handler the Front Controller dispatches to — one input controller per logical page/action (the classic "one route → one handler" model). This is what most framework "controllers" (and ADR Actions) are. There are many of these, one per use case.

So the Front Controller is the funnel; the Page Controllers are what it funnels *to*. "Page Controller vs Front Controller" is really "the one dispatcher" plus "the many handlers it dispatches to."

**Decision:** Where do cross-cutting concerns live? They belong in the **Front Controller / middleware**, not scattered across every Page Controller. ADR Actions stay single-responsibility and run *after* the Front Controller has handled the "front" concerns.

## Template View / Transform View / Two Step View — the Responder's body mechanism

A **Responder** owns the whole HTTP response; the *body* is built by one of these:

- **Template View** — HTML page with markers filled by domain data (Blade, Twig, Smarty, plain PHP templates). The default for server-rendered HTML.
- **Transform View** — transform domain data element-by-element into HTML programmatically (XSLT, or building a DOM/string from objects). Use when output shape is driven by code, not a markup template.
- **Two Step View** — two stages: domain data → a *logical* page (no formatting), then logical page → final HTML. The second stage is the shared site layout/skin. Use when many pages need a consistent look or multiple output "skins" from one logical page.

**Decision:** Pick the body mechanism per response. Template View for ordinary HTML; Two Step View when layout/skin must change globally without touching every template; Transform View when generation is code-driven. The pattern choice lives *inside the Responder* — it does not change the ADR split.

## Application Controller — a separate object for *navigation/flow*, not for the work

An **Application Controller** is a centralized object that decides *which screen or step comes next* based on application state and prior input. It does **not** do business work — it answers "after this, where do we go?" For example, a multi-step checkout: the Application Controller tracks that the user has completed shipping, so it routes them to the payment screen rather than back to cart; or a wizard that branches to different screens depending on an earlier answer. The individual Page Controllers (Actions) still do their own work; the Application Controller owns only the *flow* between them.

This matters because that "what screen is next" logic otherwise gets duplicated: several controllers each hard-code the same "if X then go to Y" branching, and a global change means editing all of them. Pulling it into one Application Controller removes that duplication.

**Decision:** Extract an Application Controller **only** when flow logic is genuinely repeated or conditional across steps. For a typical CRUD/API app, navigation is just the router + a redirect — an Application Controller is overengineering that fights ADR's single-responsibility Action. Reach for it for complex wizard/multi-step flows, not for ordinary request handling.

**Relation to the State pattern.** They overlap because both decide "where do we go next depends on current state + input." A wizard flow is effectively a state machine over screens, and the Application Controller is often *implemented with* the State pattern (each step is a state; transitions are the "next" decisions). But they are not the same: the GoF **State** pattern changes how a *domain object behaves* in response to messages (the object delegates to a current state object that encapsulates both behavior and transitions). **Application Controller** is narrower and presentation-scoped — it only decides *which screen/step to show next*, not how any domain object behaves. Rule of thumb: if the question is "how does this object act in this state?", that's State; if it's "which page do we navigate to next?", that's Application Controller (and a state machine is just one valid way to build it).

## Summary

| Pattern                | Modern form                                        | The decision it implies                                |
|------------------------|----------------------------------------------------|--------------------------------------------------------|
| Front Controller       | Single bootstrap/kernel that funnels every request | Hosts cross-cutting concerns (middleware)              |
| Page Controller        | One route → one handler / ADR Action               | (baseline per-route handler)                           |
| Template View          | Blade / Twig / PHP templates                       | Body mechanism (default HTML)                          |
| Transform View         | Programmatic HTML/DOM/XSLT generation              | Body mechanism (code-driven)                           |
| Two Step View          | Layout/skin separation                             | Body mechanism (shared look / skins)                   |
| Application Controller | A flow service deciding "what screen/step is next" | Extract only when navigation is duplicated/conditional |

These are vocabulary for *recognizing* what the framework hands you — use them to place code correctly within the ADR boundary, not as separate patterns to implement.
