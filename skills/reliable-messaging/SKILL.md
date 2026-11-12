---
name: reliable-messaging
description: Use this skill when designing asynchronous communication, publishing domain events, consuming messages, or coordinating work across transactions and bounded contexts in PHP applications. Covers delivery guarantees, outbox/inbox patterns, idempotency, retries, ordering, and sagas.
---

# Reliable Messaging in PHP Applications

## When to Use Asynchronous Messaging (and When NOT to)
Messaging introduces eventual consistency, operational failure modes, and more difficult debugging. Use it when work can complete independently, a consumer must scale or fail independently, or a bounded-context boundary should be asynchronous. Keep work synchronous when the caller needs an immediate result and one transaction can safely own the operation.

| Use Asynchronous Messaging For                                    | Prefer Synchronous Work For                                |
|-------------------------------------------------------------------|------------------------------------------------------------|
| Independent side effects such as notifications or search indexing | A decision that must be returned to the caller immediately |
| Cross-context reactions that can converge eventually              | A small same-context workflow with one transaction         |
| Work that benefits from independent scaling or failure isolation  | A workflow whose intermediate state must remain invisible  |

Do not introduce a broker simply to make code appear decoupled. First define what may be delayed, what happens on failure, and how the user or operator can observe the outcome.

## Delivery Guarantees

Assume a message can be delivered more than once. Most practical brokers provide at-least-once delivery; a consumer may finish its work and crash before acknowledging the message. "Exactly once" across a database and broker is not a general guarantee. Design for duplicate delivery with idempotent consumers and durable deduplication.

## Numbered Workflows

### 1. Publishing a Message from a Database Transaction
If a state change and a message must agree:
1. **Write atomically.** Persist the aggregate changes and an outbox record in the same database transaction.
2. **Publish separately.** A worker claims unpublished outbox records, publishes them, and marks them delivered only after broker confirmation.
3. **Expect duplicates.** A crash after publish but before marking delivered causes a repeat; consumers must be idempotent.
4. **Retain and observe.** Keep enough delivery state for retries and diagnosis, and define how old outbox rows are archived or removed.

Do not publish to the broker before the database commit, or commit the database and then publish without an outbox. Either ordering can lose a message or expose an event for state that never committed. See [transaction-boundaries.md](../persistence-patterns/references/transaction-boundaries.md).

### 2. Consuming Messages Idempotently
If processing a message changes durable state:
1. **Choose a stable message identity.** Include a unique event/message ID and a schema version.
2. **Deduplicate atomically.** Record the message ID in an inbox/deduplication table in the same transaction as the consumer's state change. A duplicate then becomes a harmless no-op.
3. **Acknowledge after commit.** Do not acknowledge before durable work completes. If acknowledgment fails after commit, a retry is safe because it is deduplicated.
4. **Make external effects explicit.** An email, payment, or other remote side effect cannot join the local database transaction. Use that provider's idempotency facility where available, or record a separate delivery intent and reconcile ambiguous outcomes.

An in-memory flag or cache entry is not sufficient deduplication for business effects; it can disappear or expire while a broker still retries the message.

### 3. Handling Retries and Poison Messages
For transient failures, retry with bounded exponential backoff and jitter. Set a maximum attempt count and maximum message age. Classify permanent failures (invalid payload, unsupported schema, invariant violation) so they do not retry forever. After exhaustion, route the message to a dead-letter/quarantine path with the original payload, failure details, and correlation identifiers; provide a safe replay procedure after correction.

Do not swallow exceptions and acknowledge a message as successful when its required work did not complete. Conversely, do not retry permanent failures without a policy: that creates noisy, stuck queues and can repeat harmful side effects.

### 4. Ordering and Schema Evolution
- Require ordering only when the business invariant needs it; global ordering reduces throughput and is rarely available across partitions.
- When per-aggregate order matters, partition or serialize by aggregate ID and include a sequence/version so consumers can detect gaps or stale events.
- Treat messages as durable contracts. Add fields compatibly, tolerate unknown optional fields, and use explicit versions for incompatible changes.
- Publish facts in the source context's language. Consumers translate those facts into their own commands and models; do not share ORM entities or internal classes.

### 5. Coordinating Multi-Step Work
Use a saga/process manager when a business workflow spans independently committed steps and needs timeouts, durable progress, and compensation. Record each step and its outcome; make forward and compensating actions idempotent. Compensation is a new business action, not a rollback of an already committed remote transaction.

If the workflow is short and belongs to one database, prefer one local transaction. Do not use a saga as a substitute for a transaction the system can actually provide.

## Boundaries

### Always Do
- Define the delivery semantics, duplicate behavior, retry policy, and terminal failure path before shipping a consumer.
- Give messages stable IDs, explicit versions, and enough context to correlate producer and consumer work.
- Keep broker and provider types inside infrastructure adapters; expose application-level message contracts at the boundary.
- Make queue handlers thin adapters that validate/translate the message and invoke an application use case.
- Test duplicate delivery, retryable and permanent failures, and the database/broker failure window.

### Ask First
- Ask whether eventual consistency is acceptable to the business and how long it may take.
- Ask who owns replay, dead-letter review, and message-schema compatibility across independently deployed consumers.
- Ask before introducing a saga, event store, or new broker when a local transaction or simple job would suffice.

### Never Do
- Never assume a message is delivered exactly once.
- Never rely on a database transaction to atomically commit to an unrelated broker unless the infrastructure explicitly provides that guarantee.
- Never acknowledge before durable processing succeeds.
- Never put credentials, unnecessary personal data, ORM entities, or unserializable objects in message payloads.
- Never perform a non-idempotent external effect on every retry without provider-side idempotency or reconciliation.

## Related Skills
- [domain-events](../domain-events/SKILL.md): defines business facts and their ownership; this skill handles reliable transport and consumption.
- [application-layer](../application-layer/SKILL.md): owns use-case orchestration and transaction boundaries.
- [persistence-patterns](../persistence-patterns/SKILL.md): defines repository and transaction choices; see its [transaction-boundary guidance](../persistence-patterns/references/transaction-boundaries.md).
- [infrastructure-boundaries](../infrastructure-boundaries/SKILL.md): isolates brokers and external providers behind adapters.
- [testing-strategy](../testing-strategy/SKILL.md): use integration tests for persistence and message-boundary behavior.
