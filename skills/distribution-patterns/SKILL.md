---
name: distribution-patterns
description: Use this skill when designing remote interfaces or APIs, deciding how coarse-grained an endpoint or operation should be, or shaping data that crosses a process or network boundary. Covers Remote Facade and DTO (Data Transfer Object). Triggers on questions about REST/GraphQL/gRPC endpoint design, exposing use cases remotely, avoiding chatty APIs, preventing ORM entities from leaking through the API, or carrying data safely across bounded contexts or layers.
---

# Distribution Patterns

These are the patterns about communicating *across a boundary* (process, network, or layer). They address one root problem: **crossing a seam safely and cheaply without leaking the internals on either side.** Two patterns make up this skill:

- **Remote Facade** — the *operation contract* you expose to callers (coarse-grained, consumer-shaped).
- **DTO (Data Transfer Object)** — the *data shape* that travels across the seam.

Remote Facade is most often the **external**-facing contract (a REST endpoint, GraphQL mutation, gRPC method, CQRS command, message handler). A DTO is the **data shape** used both at that external boundary *and* at internal seams (between the application layer and the presentation layer, or across bounded contexts). The distinction is operation vs data, not strictly external vs internal — but the motivation in both cases is the same: don't drag the internal object model across the wire.

## Remote Facade

A Remote Facade is a coarse-grained facade over a web of fine-grained objects, applied specifically to make remote calls cheap. See [remote-facade.md](references/remote-facade.md) for the full definition, rules, tradeoff, and examples. Key constraints: the facade holds **no domain logic** and is the *only* remote surface (the objects beneath it are in-process with no remote interface), and the contract is shaped around **consumer needs**, not the object model.

## DTO (Data Transfer Object)

A DTO is a simple structure for moving data across a boundary — deliberately *not* a domain object. It keeps the API contract independent from the persistence/domain model so each can evolve for its own reasons. See [dto.md](references/dto.md) for the full treatment, why domain objects don't belong on the wire, and the CQRS angle (Commands/Queries/Responses/Events are explicit DTOs).

## Boundaries

### Always Do
- Design the remote contract around **consumer needs**, not the internal object model.
- Make remote operations **coarse-grained** — one call carries everything the consumer needs.
- The Remote Facade holds **no domain logic** and is the *only* remote surface; the objects beneath it are in-process and have no remote interface.
- Carry data across the boundary as a **DTO**, not a domain object or ORM entity — the API contract and the domain model change for different reasons and should be separated.
- Treat DTOs as part of the **public contract**: give them validation, explicit serialization, and versioning as needed.

### Ask First
- Ask whether the call is genuinely **remote / cross-process**. In-process calls do not need a Remote Facade — adding one there is unnecessary indirection.
- Before introducing a DTO for an **in-process** seam, ask whether the two sides change for different reasons or speak different languages. If it's the same bounded context and ubiquitous language, reuse the existing layer object (domain object from a Repository, read model, value object) instead of duplicating it into a DTO — see [dto.md](references/dto.md).

### Never Do
- Never expose your entire object model remotely (`getCustomer()`, `getOrders()`, `getAddresses()`...). That is exactly what a Remote Facade exists to prevent.
- Never leak ORM entities, domain objects, lazy-loading proxies, or persistence state across the boundary — send a DTO instead.
- Never make many fine-grained remote calls where a single coarse-grained operation would do.

## Related Skills
- [application-layer](../application-layer/SKILL.md): a CQRS command/handler is the in-process expression of a use case; a Remote Facade is its remote-facing expression.
- [infrastructure-boundaries](../infrastructure-boundaries/SKILL.md): that skill is about *consuming* external systems behind adapters; this one is about *exposing* your own remote interface.
- [bounded-contexts](../bounded-contexts/SKILL.md): DTOs are the common currency when translating between contexts.
