# Production Schema Rollout

Database migrations in a live system are deployment protocols: old and new application versions may run at the same time, and schema changes can outlive a rollback window. Plan for compatibility and data volume, not only whether the migration succeeds on an empty development database.

## Expand, Migrate, Contract

For a change that cannot be deployed atomically with every application instance:
1. **Expand.** Add backward-compatible schema (nullable/additive columns, tables, or indexes) while old code still works.
2. **Deploy compatible code.** Read old and new forms as needed; write in a way that keeps both versions safe during rollout. Avoid uncontrolled dual writes to separate sources of truth.
3. **Backfill.** Move existing data in bounded, restartable batches. Track progress, make the job idempotent, and monitor lock/load impact.
4. **Switch reads and writes.** Confirm all instances and consumers use the new representation and verify data consistency.
5. **Contract later.** Remove the old column/shape only after old code, workers, and rollback paths no longer depend on it.

For small, backward-compatible changes in a coordinated single-version deployment, the full sequence may be unnecessary. Choose based on deployment overlap, table size, lock behavior, and rollback constraints.

## Operational Checks

- Test migrations against realistic schema/data volume and the production database engine; migration behavior and locking vary by engine.
- Identify table locks, table rewrites, long-running index creation, and transaction limits before deployment.
- Separate schema changes from large data backfills when they have different runtime and recovery needs.
- Make backfills restartable and observable; pause or throttle them if they affect application latency.
- Define rollback as either a tested schema rollback or a forward-fix path. Dropping data is not safely reversible merely because a down migration exists.
- Keep migration history as the source of schema changes and verify a clean database can be built from it.

## Related
- [architecture-migration](../SKILL.md): migrate one vertical slice at a time and preserve behavior.
- [transaction-boundaries.md](../../persistence-patterns/references/transaction-boundaries.md): application transaction ownership.
