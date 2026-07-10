# Service Layer

Service Layer is a pattern interchangeable with Application Layer in DDD terminology.

## What It Is

A Service Layer defines an application's boundary with a layer of services that establishes a set of available operations and coordinates the application's response in each operation. An application usually has many interfaces (UI, batch, integration gateways, CLI) that all need the same complex interactions — coordinating transactions across resources, invoking business logic, assembling responses. Encoding that logic separately in each interface causes duplication. The Service Layer centralizes it behind one boundary.

### A Service represents a use case
A service (or handler, or operation) is one business use case. It:

- **Orchestrates and coordinates** work across the domain, persistence, and side effects.
- **Defines a clear entry point** for the use case — one well-named operation that clients call.
- **Controls transactions** and **manages side effects** (publishing events, sending messages).
- **Isolates presentation technology** — the same service is called by HTTP controllers, CLI commands, and queue workers.

### A Service is NOT the business rules
The single most useful thing to understand about a service: **it does not make business decisions.** A service is a *workflow*, not a *business rule*. The actual decisions — "can this customer be invoiced?", "what is the discount?" — belong in the domain: rich domain objects, policies, or the table module, depending on the domain-logic style you chose (see the `domain-modeling` skill's Transaction Script → Table Module → Domain Model spectrum).

### Related terms
These all name the same concept (a use-case-level coordinator) in different dialects:

- **Service Layer** / **Application Layer** — this skill.
- **Command Handler / Query Handler** — the CQRS dialect (one handler per command/query).
- **Operation**, **Action**, **Interactor**, **Use Case** — minor naming variations on the same idea.

## What a Service Layer Provides

- A **clear application API** — one operation per use case, intent-revealing by name.
- **Isolation from presentation technology** — HTTP, CLI, and queue clients all call the same service.
- **Easier testing** — the use case runs in-memory without booting a delivery framework.
- **Reusable workflows** — one orchestration shared across delivery channels.
- **Cleaner transaction boundaries** — the service marks where a transaction begins and ends.
- Most importantly: **every use case gets a well-defined entry point.**

## The Overuse Anti-Pattern

A Service Layer is easy to overuse. Many codebases devolve into:

```
Controller → Service → Repository → Entity
```

where the **Entity has no meaningful behavior** and the **Service contains everything**. This is not a failure of the pattern — it is a failure of the domain model beneath it. You have recreated a **Transaction Script with more indirection**: the coordination layer is doing the work that an anemic domain should have owned.

The pattern itself isn't at fault. The domain model is simply too anemic to justify the layering. If there is no real domain behavior, prefer an explicit Transaction Script (or Table Module) and skip the extra service indirection; if there *is* real domain behavior, push it down into the domain objects so the service only orchestrates.

## Architect Decisions the Service Layer Answers

Using a Service Layer forces — and answers — three questions every architect must decide:

1. **Where does a use case begin?** At the service operation (its single entry point).
2. **Where do transactions start and end?** At the service boundary.
3. **Who coordinates the domain?** The service, delegating decisions to the domain.

## Where It Sits (Not on the Spectrum)

The Service Layer is **not** another point on the domain-modeling spectrum. It sits **above** it. The domain-logic spectrum (Transaction Script → Table Module → Domain Model) is about *how business rules are organized*; the Service Layer is about *where the application boundary is drawn and who coordinates the use case*.

```
                    ┌───────────────────────────────┐
   Delivery         │      SERVICE / APP LAYER      │  ← one entry point per use case,
   (HTTP/CLI/queue) │  orchestrates; makes NO rules │     coordinates, controls txns
                    └──────────────┬────────────────┘
                                   │ coordinates
                                   ▼
        ┌──────────────────────────────────────────────┐
        │  DOMAIN-LOGIC STYLE (pick one per feature)   │
        │  Transaction Script → Table Module → Domain  │
        └──────────────────────────────────────────────┘
```

The Service Layer coordinates *whichever* domain-logic style you chose. A use case over a rich Domain Model delegates decisions to domain objects; a use case over a Transaction Script may simply be a thin wrapper whose procedure is the operation body. Either way, the Service Layer is the boundary above.

## In This Skill Set

- The `application-layer` skill (this one) is the Service Layer. Use it to give every use case a single entry point and to keep delivery mechanisms thin.
- The `domain-modeling` skill chooses *how* business rules are organized (Transaction Script, Table Module, or Domain Model). The Service Layer sits above that choice and coordinates it.
- The `persistence-patterns` skill decides how the service's repository/adapter talks to storage.
- The `testing-strategy` skill verifies service workflows at the application boundary and domain rules at the domain boundary — never blur the two.
