# Laravel

Laravel emphasizes developer productivity with hidden magic. Clean architecture requires explicitly opting out of its shortcuts.

- **Avoid Facades.** Facades are global service locators. Refactor to constructor-injected dependencies.
- **Avoid Fat Eloquent Models.** Model classes are persistence-bound entities, not aggregates. Place business logic in Application Handlers or domain services, never in Eloquent models.
- **Avoid `$request->*` in domain.** Accept primitive inputs or DTOs in handlers. Convert the request in the controller.
- **Use the Service Container at the edge.** Register ports and concrete adapters in service providers. Never resolve the container from within a domain object.
- **Jobs and Listeners.** Treat them as Application-layer orchestrators or event subscribers, not as domain logic containers. Dispatch events for decoupling, not Jobs for synchronous business steps.
- **Routes.** Use explicit Route definitions with controller callbacks that delegate to Handlers. Avoid closures for use cases.

## General patterns that carry over from Laravel's conventions

These come from Laravel's own best-practice guidance but are framework-agnostic; apply them in any PHP backend.

- **Single-purpose action classes.** Give each discrete operation its own class (an `InviteMember` action, not a `TeamService::invite()` that shares a class with ten other methods). This is exactly the Application Service / Handler pattern — one class per use case, named for the use case. See the `application-layer` skill.
- **DTOs between layers.** Move data across the application boundary in typed DTOs (or simple value objects), not untyped arrays. A DTO carries the validated intent from the controller into the handler and gives the next layer type safety without leaking the HTTP request. See the `action-domain-responder` and `application-layer` skills.
- **Value objects for domain concepts.** Represent domain ideas like `Email`, `Money`, or `Quantity` as small objects that enforce their own invariants, instead of passing raw strings and re-validating the rule at every call site (primitive obsession). This is encapsulation and DRY at the type level — see [clean-code-foundations.md](../domain-modeling/references/clean-code-foundations.md).
- **Eager-load relationships.** When a list view walks a relationship per row, load the relationship once (a single joined/batched fetch) rather than issuing one query per row. Per-row lazy loading is the classic N+1 performance debt — see the persistence "Recognizing Problems" section.
- **Pass IDs, not models, across the serialization boundary.** When handing work to a queue job or another process, pass the aggregate's identifier (and the minimal data needed), not a hydrated model or ORM entity. The consumer re-loads from its own repository. This keeps the serialization boundary clean and avoids stale or detached objects — it is the same "reference by ID only" rule that governs aggregates.
- **Make asynchronous handlers idempotent.** A queued operation may be delivered more than once. Key the work on a stable idempotency key or dedupe on the aggregate id so a retry is safe. Pair this with the transaction/flush boundaries owned by the application layer.
- **Keep transactions short.** Do the in-process work and commit; perform slow external calls (email, webhooks, third-party APIs) after the transaction, or as event-driven side effects, so the database lock is held for the minimum time. See [transaction-boundaries.md](../persistence-patterns/references/transaction-boundaries.md).
