# Persistence Performance and Caching

Performance choices should follow measured behavior and business latency needs. Optimize the query or boundary that is actually costly; do not add caching, CQRS, or a new storage system just because a table is large.

## Find the Cost Before Changing the Design

- Measure representative request/query latency and database load under realistic data volume.
- Inspect query plans and query counts for slow paths. Look for N+1 access, unbounded result sets, missing or ineffective indexes, and excessive columns/joins.
- Use pagination for collections. Prefer cursor pagination for large or deeply traversed result sets; use offset pagination when numbered pages and bounded data make it appropriate.
- Batch related reads and writes when that preserves clear transaction semantics. Avoid loading entire aggregates or object graphs for read-only screens.
- Establish a latency or throughput goal for the path and remeasure after the change.

## Choose the Least Complex Read Shape

For read-heavy endpoints, a query service, SQL projection, or lean DTO can be simpler and faster than hydrating a full domain graph. Keep writes behind the aggregate and its invariants when the domain requires them. This is a local read/write choice; it does not automatically require separate databases or a CQRS platform. See the [persistence decision flow](decision-flowchart.md).

## Introduce a Cache Deliberately

Before adding a cache, state what is cached, its owner, acceptable staleness, invalidation trigger, expiry/failure behavior, and whether a miss is safe. Cache-aside is often sufficient: read cache, load the source on miss, then populate with a bounded TTL. The database or authoritative service remains the source of truth.

- Cache derived/read data, not an unvalidated authorization decision or mutable aggregate as if it were authoritative.
- Include tenant, locale, permission scope, and relevant version in keys where those dimensions affect results; otherwise a cache can leak data between callers.
- Prefer invalidation after a successful commit. If invalidation can be lost, use versioned keys, short TTLs, or a durable invalidation/outbox mechanism appropriate to the consistency requirement.
- Treat cache outages as an explicit load/failure mode. A cache should not silently turn a transient outage into a database overload.
- Do not cache secrets or personal data without a clear access, retention, and deletion policy.

## Concurrency and Capacity

Use database constraints and transactions to protect correctness under concurrent writes; a prior read followed by a write is not itself a concurrency guarantee. Use optimistic versions or row locks only where the domain needs conflict detection/serialization. For hot paths, measure lock duration and contention before changing isolation or introducing distributed locks.

## Related
- [transaction-boundaries.md](transaction-boundaries.md): transaction ownership and atomicity.
- [decision-flowchart.md](decision-flowchart.md): selecting persistence structure and read/write models.
- [testing-strategy](../../testing-strategy/SKILL.md): test repository/query behavior against a real test database.
