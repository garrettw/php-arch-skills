# DTO (Data Transfer Object)

Crossing a process boundary is fundamentally different from calling a method in memory. That distinction is arguably even more important in today's API-driven and service-oriented architectures. A [Remote Facade](remote-facade.md) defines the *operation* you expose remotely; a DTO defines the *data shape* that travels across the seam.

## Why objects don't belong on the wire

Objects inside an application routinely carry things that are meaningless, dangerous, or simply none-of-the-consumer's-business once serialized and sent elsewhere:

- **behavior** — methods that only make sense in-process
- **lazy-loading proxies** — references that trigger queries on the server, not the client
- **internal references** — graph edges into the rest of your object model
- **invariants** — rules enforced by the domain, not the contract
- **persistence state** — ORM bookkeeping, dirty flags, IDs of no interest remotely

None of those belong on the wire. DTOs provide simple structures specifically for moving data between boundaries. The modern software ecosystem runs on message exchange, and DTOs *define those messages*.

## What DTOs buy you

DTOs create excellent boundaries: they make your API independent from your persistence model. That freedom becomes invaluable as applications evolve — you can change the domain or storage without breaking every consumer, and vice versa.

### "DTOs are just duplicated classes"

Some developers complain that DTOs duplicate the domain model. Sometimes that's true — but the duplication is **intentional**. The API contract and the domain model change for *different reasons*; separating them protects both. Coupling the two guarantees that one side's change breaks the other.

## Modern DTOs are richer than a transport container

DTOs have evolved. They commonly now include:

- **validation** — contract-level rules enforced at the boundary
- **serialization** — explicit control over wire format
- **versioning** — safe evolution of the public contract
- **documentation metadata** — the DTO *is* part of the public API spec

They're no longer merely dumb data holders. They are part of the **public contract**.

## CQRS amplifies the pattern

CQRS pushes this idea all the way: Commands, Queries, Responses, and Events all become explicit DTOs, and the application communicates through messages rather than exposing internal objects. See the [application-layer](../application-layer/SKILL.md) skill for Command/Query/Handler structure — each of those messages is a DTO by construction.

## When to use a DTO *within* an application (and when not to)

The "don't put objects on the wire" rule is absolute at a **process/network boundary**. But inside one running application, the answer is a judgment call. The decision hinges on one question:

> **Do the two sides of the seam change for different reasons, or speak different languages?**

### Reach for a DTO when the seam is real
- **Crossing a process or network boundary** (remote API, queue, gRPC). Always — this is the canonical case.
- **Crossing a bounded-context boundary that is separately deployable or externally owned.** When the other context ships on its own cadence, a DTO (or explicit contract) is the translation currency that protects both sides.
- **The presentation needs a view-specific shape** that composes or flattens several domain objects the domain shouldn't know about. A CQRS read model / query result is a DTO by construction.
- **The contract is public and versioned**, evolving for reasons unrelated to the domain.

### Prefer reusing the existing layer object when it already fits
If the consumer is **in-process and inside the same bounded context**, the object that already exists in a standard layer is usually the right thing to pass — adding a DTO there is just the "duplicated classes" anti-pattern the stub warns about:

- **Domain / Repository.** A Repository's contract *is* to return domain objects in the ubiquitous language (see [persistence-patterns](../persistence-patterns/SKILL.md)). Don't DTO-wrap them for in-process callers that already speak the domain.
- **Application / Service layer.** A handler can hand a domain object or read model straight to a presenter when both live in the same context; only reshape into a DTO when the caller is external or the shape is view-specific.
- **Infrastructure.** Adapters translate at their own seam (rows / SDK ↔ domain); that translation is the adapter's job, not a DTO concern for internal consumers (see [infrastructure-boundaries](../infrastructure-boundaries/SKILL.md)).
- **Presentation.** A view can bind directly to a domain object or a simple query result when no cross-context or cross-language translation is needed.

### Rule of thumb
**Same process + same context + same language → reuse the existing object** (domain object, read model, value object). **Different process, different context, or different change cadence → DTO.**

In a monolith, in-process cross-context sharing is usually handled by a **Shared Kernel** — shared value objects like `UserId` and `Money`, and domain-event contracts (see [bounded-contexts](../bounded-contexts/SKILL.md)) — not by DTOs. Reach for a DTO only when the boundary is remote or the contract must be explicitly versioned and owned separately.

## Related
- A [Remote Facade](remote-facade.md) is the operation; a DTO is the data that crosses it. They are the two halves of this skill.
- The [application-layer](../application-layer/SKILL.md) skill treats Commands/Queries/Responses/Events as explicit DTOs (the CQRS angle).
- The [bounded-contexts](../bounded-contexts/SKILL.md) skill handles in-process cross-context sharing via a Shared Kernel (shared value objects and domain events) — an alternative to DTOs when the contexts share a process.
