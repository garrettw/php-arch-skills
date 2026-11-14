# Transaction & Flush Boundaries

A Unit of Work (Doctrine's `UnitOfWork`) defers writes and flushes them together. Active Record ORMs such as Eloquent have no Unit of Work: each `save()` writes immediately, so atomicity exists only if you wrap the writes in an explicit transaction. Either way, the trap is treating the ORM's **default flush or save timing** as your transaction boundary. Transaction boundaries are a **business decision**, not an ORM default.

## The Core Rule
The application/handler — not the ORM, not the framework's request-end auto-flush — owns *when* a commit happens. Persist explicitly wherever the workflow genuinely needs a commit point.

## Default: One Flush Per Use Case
For a single business operation (load aggregate → mutate → persist), flush **once** at the end:
- Either the whole unit commits, or none of it does — no orphaned rows, no partial state.
- `flush()` still throws on failure; wrap it in a transaction and surface the exception. Errors are *deferred to commit*, not *lost*.
- This is the point of the UoW: consistency across the user and their related rows.

## Flush Is Not Commit
A flush inside an open transaction sends SQL on the current connection, but nothing outside that connection can see the rows until **commit**. Keep the two apart:
- **Flush mid-transaction** only for needs on the same connection: getting a database-generated ID, letting a later query in the same transaction see the row, or surfacing a constraint violation early.
- **Anything outside the transaction** (a queued job, a worker, another bounded context, a webhook receiver) cannot see uncommitted rows, and must not be told about them before commit. Record the intent in the outbox in the same transaction; a relay publishes it after commit (see [reliable-messaging](../../reliable-messaging/SKILL.md)).
- **External calls** (payment gateway, third-party API) should not run inside the database transaction: they hold locks while waiting on the network and cannot be rolled back. Call them before the transaction with an idempotency key, or after commit from an outbox-driven consumer.

If you write logic that *assumes a row exists mid-flow* for someone other than the current transaction, you have coupled business logic to ORM flush timing. Redesign around commit plus outbox rather than flushing earlier.

## "I wanted a meaningful error if the insert failed"
You still get it. `flush()` throws; you catch at the boundary and return the error. The only difference from piecemeal inserts is *when* the error surfaces (at commit, not after each row). That is the correct trade for atomicity. If you need per-row failure semantics (e.g., "user saved but notification failed"), those are separate operations with separate boundaries — not one giant transaction.

## "Tracking callbacks for failures"
ORMs provide lifecycle hooks (`onFlush`, `postFlush`, Doctrine lifecycle callbacks; Eloquent events). More importantly, **failure handling is an application concern**: wrap the commit in try/catch at the handler and react there. Don't bury recovery logic in ORM callbacks.

## The Real Anti-Pattern
Hiding the DB behind ORM magic so that business logic is unknowingly coupled to end-of-request flush ordering. Fix by:
- Injecting repositories / persisting through explicit calls at known points.
- Keeping the domain **persistence-ignorant** — it never knows about flush timing (see [dependency-injection](../../dependency-injection/SKILL.md); persistence is infrastructure).
- Putting transaction boundaries in the application layer, not the ORM config.

## When the Row-Exists Requirement Is a Smell
If you constantly need "the row must exist right now so something else can read it," that is a signal to use an **outbox / domain-event** pattern instead of piecemeal flushing or committing: write your domain events to the outbox in the same transaction, and let a separate process publish them after commit. See [domain-events](../../domain-events/SKILL.md). This keeps the atomic guarantee while decoupling the downstream consumer.

## Related
- [orm-patterns.md](orm-patterns.md): Unit of Work framed as an ORM feature — selection and configuration, not reimplementation.
- [dependency-injection](../../dependency-injection/SKILL.md): persistence is infrastructure; the domain stays ignorant of flush timing.
- [domain-events](../../domain-events/SKILL.md): outbox pattern for cross-boundary consistency without mid-request flushes.
