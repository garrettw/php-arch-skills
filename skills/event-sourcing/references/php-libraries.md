# PHP Event Sourcing Libraries

This reference covers the main PHP libraries for implementing Event Sourcing and when to choose each.

> **Batteries-included scale (low → high):** EventSauce (small, composable) → Ecotone (bundles command/query/event bus + CQRS/ES glue via attributes) → Patchlevel ES (adds its own operational tooling: subscription engine, migrations, CLI, admin UI, PHPStan/PHPUnit helpers). "More batteries" means more to learn but less you wire yourself.

## EventSauce

**Best for:** Teams that want control and composability without a full framework.

- Docs: https://eventsauce.io/docs

EventSauce is a small, focused library. It does not require CQRS, a command bus, or a specific framework. Key ideas:
- `AggregateRoot` base class with `recordThat()` and `apply()`
- `AggregateChanged` events with `aggregateId()` and `messageName()`
- Decoupled from any bus or dispatcher
- Copy-able implementation — you can fork and own it

**Trade-off:** Less "batteries included" than Ecotone. You wire more yourself.

## Ecotone

**Best for:** DDD practitioners who want lower ceremony and explicit structure.

- Docs: https://docs.ecotone.tech

Ecotone is a framework that builds on the Prooph `event-store` and picks up where Prooph's `event-sourcing` and `service-bus` libraries left off. It provides PHP 8 attributes such as:
- `#[EventSourcingAggregate]` — marks an aggregate for event sourcing
- `#[AggregateIdentifier]` — marks the identity property
- `#[CommandHandler]` — marks methods that handle commands and record events
- `#[EventSourcingHandler]` — marks methods that rebuild state from events
- `#[Projection]` — marks a class as a projection tied to an aggregate
- `#[QueryHandler]` — exposes projection state via queries
- `#[EventHandler]` — marks methods that react to events

Prooph EventStore (https://github.com/prooph/event-store) handles persistence. Supported backends: memory, PostgreSQL, MySQL, MariaDB.

**Trade-off:** Opinionated and attribute-heavy. If you dislike annotations/attributes on domain objects, this may feel intrusive. More batteries-included than EventSauce (it bundles the command/query/event bus and CQRS/ES glue via attributes), but less so than Patchlevel ES — it leans on other Symfony components rather than shipping its own operational tooling.

## Patchlevel Event Sourcing

**Best for:** Teams that want a framework-agnostic core with first-class Symfony and Laravel integration, plus a broad batteries-included feature set.

- Docs: https://patchlevel.dev/docs
- AI-friendly source: https://patchlevel.dev/llms.txt

The core is framework-agnostic. There are official integrations for Symfony (the first and most-used integration) and Laravel, but the library does not tie you to either. The event store itself is interface-based: the built-in implementation ships on Doctrine DBAL, which gives you PostgreSQL, MySQL, MariaDB, SQLite, and others — and you can plug in your own store implementation if you need to.

Features beyond basic snapshots and projections:
- Full subscription engine with catch-up and retries
- Process managers
- Crypto shredding for GDPR
- Schema management and migrations
- CLI tooling and an admin UI
- A PHPStan extension and PHPUnit helpers
- DCB (Dynamic Consistency Boundary) support coming in 4.0

**Trade-off:** The most batteries-included of the three libraries here (more than EventSauce *and* Ecotone), which means more to learn — but it stays out of your framework choice and your domain model. Ecotone bundles bus/CQRS glue; Patchlevel additionally ships the operational tooling (subscription engine, migrations, CLI, admin UI).
