---
name: application-layer
description: Use this skill when writing controllers, structuring use cases, or implementing Service Layer / Application Layer and Command/Query/Handler (CQRS) patterns in PHP applications. Triggers on questions about thin controllers, handler responsibilities, service definition, or DTO usage.
---

# Application Layer & CQRS Patterns for PHP

## Service Layer = Application Layer
In DDD terminology, the **Service Layer** is interchangeable with the **Application Layer**. A service (also called a handler, operation, action, or interactor) represents one business use case: it orchestrates and coordinates work, controls the transaction, and manages side effects. It does **not** make business decisions — it is a *workflow*, not a *business rule*. The actual decisions live in the domain (rich objects, policies, or a table module). See [service-layer.md](references/service-layer.md) for the full definition, benefits, the overuse anti-pattern, and how it sits above the domain-modeling spectrum. A Command/Handler here is the in-process expression of a use case; the [`distribution-patterns`](../distribution-patterns/SKILL.md) skill covers exposing that same use case remotely as a coarse-grained Remote Facade.

### When to Use a Service Layer (and the Anti-Pattern to Avoid)
Use a Service Layer whenever more than one delivery channel (HTTP, CLI, queue) needs the same orchestration, or when you want a clear, testable entry point per use case. **But do not overuse it:** the common failure is `Controller → Service → Repository → Entity` where the entity is anemic and the service holds all the logic. That is a Transaction Script wearing extra indirection — the fault is an anemic domain, not the pattern. Push business rules down into the domain; keep the service as pure orchestration.

## When to Use CQRS (and When NOT to)
CQRS separates read and write models, but adds complexity. Use CQRS only when a bounded context has genuinely divergent read/write workloads. Skip CQRS for simple CRUD, interfaces without complex reads, or early-stage projects where premature separation slows delivery.

| Use CQRS For                                             | Use Simple Controller/Repository For         |
|----------------------------------------------------------|----------------------------------------------|
| Bounded contexts with divergent read/write workloads     | Simple CRUD, single operation per endpoint   |
| A read model with 10+ projections or dedicated reporting | Single or few read paths                     |
| Scaling reads and writes independently                   | Uniform access patterns                      |

**Most systems do not need CQRS.** Start with simple controllers and repositories. Introduce CQRS only when a specific read path has become a clear performance or organizational bottleneck.

## System Overview
The application layer orchestrates use cases while keeping framework adapters thin and domain rules explicit. It serves as the bridge between delivery mechanisms (HTTP, CLI, queues) and the core domain/infrastructure logic. This skill provides structural rules for controllers, handlers, and CQRS patterns.

## Numbered Workflows

### 0. Defining a Service / Use Case
If you are creating or reviewing a use-case coordinator (service, handler, operation, action, interactor):
1. **Name it after the use case**, not the CRUD verb. E.g., `CompleteEnrollment`, `InvoiceCustomer`, not `CustomerService.update`.
2. **Make it the single entry point.** One operation per use case; the controller/CLI/queue adapter only adapts input and delegates here.
3. **Orchestrate, do not decide.** The service loads state, calls domain objects/policies/table modules to make the business decision, persists the result, and publishes side effects. It must not contain the business rules itself.
4. **Bound the transaction at the service.** The transaction begins when the operation starts and ends when it returns — that is the unit of work.

### 1. Refactoring Controllers
If a controller contains business logic, database queries, or large DTO building:
1. **Extract the Core Logic.** Move the business rules into an application handler or domain object.
2. **Adapt the Edge.** Refactor the controller to only read inputs (route params, request body, etc.), delegate to the handler, and convert the result into an HTTP response.

### 2. Implementing CQRS Patterns
If creating a new use case that needs to be accessed via multiple delivery channels (e.g., Web and CLI):
1. **Determine the Operation Type.** Is it a write/state-transition (Command) or a read/projection (Query)?
2. **Create the Command/Query Object.** Define an intent-revealing object if the use case has multiple inputs or needs validation at the application boundary.
3. **Create the Handler.** Write a handler that executes the specific Command or Query.
4. **If the operation is extremely simple** (e.g., a single repository fetch), skip the Command/Handler pair and let the controller call the repository directly.

### 3. Creating DTOs and Command Objects
If deciding whether to introduce a new DTO or Command object:
1. **Evaluate Intent.** Does the object carry meaningful business intent across a boundary? If yes, create it.
2. **Evaluate Necessity.** Does the object merely mirror the request body or exist just to satisfy a generic pattern rule? If yes, skip it and use scalar arguments or arrays.

## Boundaries

### Always Do
- Always keep controllers thin; they should act purely as HTTP adapters.
- Always give each use case a single, well-named entry point in the Service/Application Layer (a service or handler).
- Always use the [Directory Structure Templates](references/directory-structure-templates.md) when implementing CQRS.
- Always let the handler load data, call domain objects/policies, save state, and publish events.
- Always treat the handler as an Application Service: it orchestrates the domain and manages side effects, but does **not** contain business rules itself (see [service-layer.md](references/service-layer.md)).
- Always push business decisions down into the domain; if the entity is anemic and the service holds everything, you have recreated a Transaction Script with extra indirection.

### Ask First
- Ask before introducing intermediate Command classes or Request DTOs if they do not add meaningful domain context.

### Never Do
- Never chain services together (e.g., Service A calls Service B calls Service C). A handler should own one complete business result.
- Never place business rules (decisions, invariants, calculations) inside a service; delegate them to the domain.
- Never place framework-specific globals (like request or session singletons) inside Application Handlers or Domain objects.
- Never let controllers contain persistence orchestration or cross-context policies.
