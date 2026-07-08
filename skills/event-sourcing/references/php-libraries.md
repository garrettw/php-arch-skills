# PHP Event Sourcing Libraries

This reference covers the main PHP libraries for implementing Event Sourcing and when to choose each.

## Ecotone + Prooph

**Best for:** DDD practitioners who want lower ceremony and explicit structure.

Ecotone is a framework that glues Prooph components together with PHP 8 attributes. It provides:
- `#[EventSourcingAggregate]` — marks an aggregate for event sourcing
- `#[AggregateIdentifier]` — marks the identity property
- `#[CommandHandler]` — marks methods that handle commands and record events
- `#[EventSourcingHandler]` — marks methods that rebuild state from events
- `#[Projection]` — marks a class as a projection tied to an aggregate
- `#[QueryHandler]` — exposes projection state via queries
- `#[EventHandler]` — marks methods that react to events

Prooph EventStore handles persistence. Supported backends: memory, PostgreSQL, MySQL, MariaDB.

**Trade-off:** Opinionated and attribute-heavy. If you dislike annotations/attributes on domain objects, this may feel intrusive.

## EventSauce

**Best for:** Teams that want control and composability without a full framework.

EventSauce is a small, focused library. It does not require CQRS, a command bus, or a specific framework. Key ideas:
- `AggregateRoot` base class with `recordThat()` and `apply()`
- `AggregateChanged` events with `aggregateId()` and `messageName()`
- Decoupled from any bus or dispatcher
- Copy-able implementation — you can fork and own it

**Trade-off:** Less "batteries included" than Ecotone. You wire more yourself.

## Prooph Components

**Best for:** Teams that want maximum control over event store and sourcing behavior.

Standalone packages:
- `prooph/event-store` — event store interface and Doctrine-based implementation
- `prooph/event-sourcing` — aggregate root trait and event factory
- `prooph/pdo-event-store` — PDO-backed event store
- `prooph/service-bus` — optional command/query/event buses

**Trade-off:** More wiring and configuration than Ecotone. Better for teams that want to own the architecture.

## Patchlevel Event Sourcing

**Best for:** Laravel or Doctrine teams who want to stay in their comfort zone.

Built on Doctrine DBAL. Provides:
- Doctrine migrations for event tables
- Laravel-friendly integration
- Snapshot support
- Projector and process manager abstractions

**Trade-off:** Less flexible than Prooph for non-Doctrine backends. Good for Laravel shops.
