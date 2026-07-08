---
name: event-sourcing
description: Use this skill when evaluating or implementing Event Sourcing in a PHP application. Triggers on questions about event stores, aggregates, projections, snapshots, event replay, temporal queries, CQRS with event sourcing, or PHP event-sourcing libraries (Ecotone, Prooph, EventSauce, Patchlevel). Distinguishes Event Sourcing from domain events and guides when the pattern is appropriate versus overkill.
---

# Event Sourcing for PHP Applications

Event Sourcing is a state-management pattern where every change to application state is captured as an immutable sequence of events. The event log is the single source of truth; current state is derived by replaying events.

**Event Sourcing is not:** event-driven development, microservices, CQRS, or eventual consistency. Those are related but separate concerns that may coexist with Event Sourcing.

Event Sourcing naturally enables CQRS because the write model is the event stream and projections are the read models. But CQRS is optional.

## When to Use Event Sourcing (and When NOT to)

Event Sourcing adds significant complexity. Use it only when you need its specific capabilities.

| Use Event Sourcing For                                        | Use CRUD / Domain Events For                 |
|---------------------------------------------------------------|----------------------------------------------|
| Audit requirements (complete history of every change)         | Simple audit logs or domain events           |
| Temporal queries (state at any point in time)                 | Current-state-only queries                   |
| Event replay for debugging or parallel testing                | Standard integration tests                   |
| Retrocactive corrections (fix past events and replay)         | Direct data corrections                      |
| Parallel models / multiple read projections from one stream   | Single read model                            |
| High scalability with event-driven architecture               | Standard request/response                    |

**Start without Event Sourcing.** Introduce it only when you have a concrete need for audit trails, temporal queries, or replay. Most systems do not need it.

## Core Concepts

<dl>
  <dt>Event</dt>
  <dd>A fact about something that happened in the past and caused a state change. Immutable. Past-tense naming (e.g., <code>OrderPaid</code>).</dd>

  <dt>Event Stream</dt>
  <dd>Ordered sequence of events for a single aggregate. The stream is the source of truth.</dd>

  <dt>Aggregate</dt>
  <dd>A cluster of domain objects treated as a unit for data changes. In Event Sourcing, the aggregate rebuilds its state by replaying its event stream.</dd>

  <dt>Event Store</dt>
  <dd>Persistence mechanism for event streams. Supports append, retrieve by stream, and subscribe to new events.</dd>

  <dt>Projection</dt>
  <dd>A read model built by consuming events. Like a materialized view. Answers questions or feeds UI.</dd>

  <dt>Snapshot</dt>
  <dd>A cached representation of aggregate state at a point in time, used to avoid replaying thousands of events on every load.</dd>
</dl>

## Numbered Workflows

### 1. Deciding to Event-Source an Aggregate
If considering whether an aggregate should be event-sourced:
1. **Check for audit needs.** Do you need a complete, immutable history of every state change? If yes, Event Sourcing provides this by design.
2. **Check for temporal queries.** Do you need to know the state at a past point in time? If yes, Event Sourcing enables this without custom history tables.
3. **Check for replay value.** Would you benefit from replaying events into a test environment, or reprocessing after a bug fix? If yes, Event Sourcing makes this safe and repeatable.
4. **Check aggregate size.** Does the aggregate have a high event frequency (thousands of events)? If yes, plan for snapshots from the start.
5. **If No to all:** Use a regular repository with CRUD operations. Event Sourcing is not needed.

### 2. Modeling an Event-Sourced Aggregate
If implementing an event-sourced aggregate:
1. **Define events first.** Every state change must correspond to an event. Events are immutable and carry the data needed to reconstruct state.
2. **Make the aggregate replayable.** The aggregate must be able to rebuild its current state by applying events in order. Use an `apply()` method or event handlers (`whenXxx()`) to mutate state.
3. **Record, don't publish.** The aggregate records events internally (e.g., `$this->recordThat(new OrderPaid(...))`). It never dispatches them directly.
4. **Expose a stream identity.** Every aggregate has a unique identifier that becomes the event stream key.
5. **Protect invariants in the aggregate.** Business rules that must always hold belong in the aggregate's command/action methods, not in the event store or projections.

### 2.5. Designing Event Granularity
If designing the events for an event-sourced aggregate:
1. **Cut along decisions, not rows or columns.** Ask "what did the user or business actually do?" and let the answer name the event. If you cannot describe an event without saying "and", it might be two events.
2. **Avoid database-row-update events.** A single fat event that dumps the entire record on every change is a CRUD pattern in event clothing. It throws away intent and forces subscribers to diff state to discover what changed.
3. **Avoid property-update events.** One event per field (`CustomerStreetChanged`, `CustomerZipChanged`) shreds a single decision into fragments. Subscribers must stitch them back together and guess whether they belong to the same action.
4. **Use coarse events for whole-record decisions.** When one action genuinely produces the entire record — e.g., `CustomerDuplicated`, `CustomerSynced` from an external CRM — the coarse event is the honest choice. Do not invent fine-grained intent you did not witness.
5. **Use fine-grained events for partial decisions.** When a decision changes only a small piece of the record and you control the system — e.g., `CustomerEmailChanged`, `CustomerMoved` — record it as a specific event. This lets subscribers react directly without diffing.
6. **Don't slice finer than the business behaves.** If the UI presents a single "edit profile" action that changes name and email together, and the domain treats them as one decision, one `CustomerProfileEdited` event is correct. Splitting it only pays off if those changes carry distinct meaning.
7. **Watch for the property bag smell.** If a single event carries every field and is emitted on every change regardless of what changed, you have rebuilt the database-row-update extreme.
8. **Listen to your subscribers.** If you keep pointing several events at one `apply()` method, and subscribers handle them together in one method, the split is not real. Those events probably want to be one.
9. **Name in past tense using ubiquitous language.** `CustomerMoved`, not `UpdateAddress`. Prefix with aggregate name where helpful (e.g., `customer.moved`). A generic `customer.updated` is the loudest smell: it names no decision and discards intent.

### 3. Building Projections
If building a read model from events:
1. **Subscribe to events.** The projection listens for specific event types from specific aggregates.
2. **Update state incrementally.** Each event updates the projection's state. The projection is a materialized view optimized for queries.
3. **Rebuild from scratch.** Projections must be rebuildable. Discard the projection state and replay all events from the beginning. This is essential for correctness after bugs or schema changes.
4. **Keep projections simple.** Projections should not contain business logic. They transform events into query-friendly shapes.

### 4. Handling Snapshots
If the aggregate stream grows large:
1. **Take snapshots periodically.** After N events (e.g., every 100), save the aggregate's current state as a snapshot.
2. **Load from snapshot + tail.** When rebuilding, load the latest snapshot and replay only the remaining events.
3. **Rebuild snapshots asynchronously.** Snapshots can be rebuilt in the background without blocking the write path.

### 5. Managing External Interactions
If the aggregate calls external systems (payment gateways, email services, other APIs):
1. **Use gateways.** Wrap all external calls behind gateway interfaces.
2. **Disable during replay.** The gateway must detect when events are being replayed (rebuild, temporal query, test) and suppress external calls. The domain logic should never know whether it is in replay mode.
3. **Cache external query results.** If the aggregate queries an external system during event processing, cache the response keyed by the event being processed. Replay must return the same cached value, not a fresh call.

### 6. Testing Event-Sourced Aggregates
If testing event-sourced aggregates:
1. **Test event production.** Given a command, assert the aggregate records the expected events.
2. **Test event application.** Given a stream of events, assert the aggregate reaches the expected state.
3. **Test in-memory.** No event store or database is needed. Construct the aggregate, apply events, assert state.
4. **Use the reference implementation.** See `references/testing-event-sourced-aggregates.md` for PHP test examples.

## PHP Libraries

| Library                         | Approach                                | Notes                                                                                   |
|---------------------------------|-----------------------------------------|-----------------------------------------------------------------------------------------|
| **Ecotone + Prooph**            | Annotation-driven                       | Good for DDD practitioners. Lowers ceremony. Uses Prooph EventStore.                    |
| **EventSauce**                  | Explicit, compositional interfaces      | Focuses only on event sourcing, not CQRS or buses. Pragmatic. Copy-able implementation. |
| **Prooph components**           | Standalone event sourcing + event store | More control, more wiring. Foundation for Ecotone.                                      |
| **Patchlevel Event Sourcing**   | Doctrine-based, Laravel-friendly        | Good for teams already on Doctrine/Laravel. Uses Doctrine DBAL for event storage.       |

## Boundaries

### Always Do
- Always treat the event stream as the single source of truth. The current state is derivable; never treat a cached state as authoritative without the event log.
- Always make events immutable and past-tense. No setters, no mutable state.
- Always design aggregates to be replayable from their event stream.
- Always make projections rebuildable from scratch.
- Always isolate external system calls behind gateways that can be disabled during replay.
- Always write domain logic inside the aggregate, not in the event store or projection.

### Ask First
- Ask before introducing Event Sourcing for a single aggregate in an otherwise CRUD application. The operational complexity (event store, projections, snapshots, replay infrastructure) is hard to justify for one stream.
- Ask before coupling Event Sourcing to a specific framework or event bus. Keep the event store interface minimal.
- Ask before storing large payloads in events. Store IDs and facts, not whole objects.

### Never Do
- Never treat a projection or snapshot as the source of truth. If it is lost or corrupted, rebuild it from the event stream.
- Never allow event handlers to modify past events. Events are immutable facts.
- Never let external side effects (email, HTTP calls) fire during event replay.
- Never put business logic in projections. Projections transform events into read shapes; they do not enforce invariants.
- Never conflate Event Sourcing with CQRS by default. Event Sourcing can exist without CQRS. Use CQRS only when read and write models genuinely diverge.

## Relationship to Other Skills

| Skill | Relationship |
|-------|--------------|
| `domain-events` | Domain events are individual facts. Event Sourcing is a persistence strategy that *uses* domain events as its storage format. All event-sourced systems use domain events; not all systems using domain events need Event Sourcing. |
| `domain-modeling` | Event-sourced aggregates are still aggregates. The same rules apply: one aggregate per transaction, invariants protected at the aggregate root, reference by ID only. |
| `persistence-patterns` | The event store is an infrastructure adapter. Repository interfaces for event-sourced aggregates look different (find by stream, append events), but the dependency rule still applies. |
| `application-layer` | Command handlers for event-sourced aggregates follow the same pattern: load aggregate, execute command, persist events, publish. The difference is in the repository implementation. |
