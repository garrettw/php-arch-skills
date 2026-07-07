---
name: domain-events
description: Use this skill when introducing domain events, deciding how to decouple side-effects, or implementing the record-then-publish pattern in PHP apps. Triggers on questions about event naming, payloads, or cross-context ownership.
---

# Domain Events in PHP Applications

## When to Use Domain Events (and When NOT to)
Domain Events add indirection that must justify its cost. Use them when you need eventual consistency, decoupled downstream reactions, or a history of changes. Skip them for single-action uses where direct method calls suffice.

| Use Domain Events For                               | Use Direct Calls For                                  |
|-----------------------------------------------------|-------------------------------------------------------|
| Decoupled side effects (email, notifications, sync) | Single-step operations without downstream reactions   |
| Cross-bounded-context communication                 | Same-context synchronous updates                      |
| Audit trail or event replay requirements            | Stateless operations with no history needed           |
| Multiple downstream triggers from one change        | One downstream reaction                               |

**Start without events.** Introduce them only when coupling becomes a real maintenance or reliability problem.

## System Overview
Domain Events represent facts that have already happened in the business. They decouple side effects from the primary transaction and serve as contracts between bounded contexts. This skill covers the implementation of the Record-Then-Publish pattern and the rules for defining events.

## Numbered Workflows

### 1. Deciding to Introduce an Event
If evaluating whether a new behavior should be triggered by an event:
1. **Check for downstream needs.** Does another bounded context need to know that a fact occurred? If yes, use an event.
2. **Check for non-blocking side effects.** Must a side effect (like sending an email or syncing to a third party) happen without blocking the primary business transaction? If yes, use an event.
3. **Check for multiple triggers.** Do you need to trigger multiple downstream actions from a single domain change without tightly coupling the orchestrating handler? If yes, use an event.
4. **If the goal is just logging or a synchronous internal call**, do NOT use an event.

### 2. Defining the Event
If creating a new domain event class:
1. **Name it in the past tense.** Use business terms (e.g., `OrderPlaced`, `EnrollmentCompleted`), not CRUD terms (`StudentRecordUpdated`).
2. **Keep the payload lean.** The event should carry IDs and the specific facts that changed.
3. **Ensure immutability.** Once created, the event cannot be changed. No setters.
4. **Cross-Aggregate Consistency.** Use events to maintain eventual consistency between aggregates. One transaction changes one aggregate and raises an event; the event triggers an update in another aggregate in a later transaction.

### 3. Implementing the Record-Then-Publish Pattern
If implementing an aggregate that produces events:
1. **Record.** The domain object creates the event and holds it internally in an array of pending events.
2. **Persist.** The application handler tells the repository to persist the aggregate.
3. **Release & Publish.** If the transaction commits, the application handler pulls the recorded events from the aggregate and passes them to an Event Dispatcher.
4. **Reference Examples.** See [Event Examples](references/event-examples.md) for structural patterns.

### 4. Handling Cross-Context Ownership
If passing events between bounded contexts:
1. **Define in Source Context.** The bounded context that owns the source fact defines the domain event contract. (e.g., `Billing` defines `SubscriptionPaid`).
2. **Listen in Target Context.** The consuming context listens to the event and translates it into a local command. (e.g., `Entitlements` listens to `SubscriptionPaid`).

## Boundaries

### Always Do
- Always use the record-then-publish pattern. Never publish a domain event directly from deep within a domain object.
- Always use the ubiquitous language to name events.

### Ask First
- Ask before adding an event listener that executes heavy business logic directly inline. (Prefer queuing a job or delegating to a proper handler).

### Never Do
- Never pass ORM entities or framework request objects inside an event payload.
- Never allow a listener to modify the event payload.
- Never attempt to cancel or roll back the primary transaction that emitted the event from within a listener.
