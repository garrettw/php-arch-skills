# Persistence Decision Flowchart

Use this guide to determine the appropriate persistence structure for a new feature or aggregate.

## Data Source Spectrum (pick the lightest that fits)

Choose a data source pattern before choosing the structure below: Table Data Gateway → Row Data Gateway (avoid in new code) → Active Record → Data Mapper, in increasing separation of business logic from storage. [data-source-patterns.md](data-source-patterns.md) is the full guide, including the Gateway-vs-Repository distinction; [orm-patterns.md](orm-patterns.md) and [orm-architecture-patterns.md](orm-architecture-patterns.md) cover the patterns inside and around the ORM.

## Full DDD Persistence Structure

**Use when**: The entity has rich behavior, complex state transitions, or cross-context policies that must be tested without the framework.

**Structure**:
```text
src/[BoundedContext]/
  Domain/
    [EntityName].php (Pure PHP domain object)
    [EntityName]Repository.php (Interface; optional, if required by criteria)
  Infrastructure/Persistence/
    [Orm][EntityName]Repository.php (Implements the interface, uses mapper; e.g., DoctrineSubscriptionRepository)
    [EntityName]Mapper.php (Converts ORM <-> Domain)
```

**Testing**:
- Domain tests: Pure PHP unit tests without database.
- Mapper tests: Verify data conversion (including nulls, defaults, enums).
- Repository tests: Integration tests verifying query behavior against a real test database.

## Lean ORM Structure

**Use when**: The entity is a data bag, report projection, configuration record, or a simple CRUD entity.

**Example Structure**:
```text
src/[BoundedContext]/
  Application/
    [CommandOrQuery]Handler.php (Uses ORM directly)
  Infrastructure/Persistence/
    OrmEntity.php (Framework-specific active record or entity)
    OrmTable.php (Framework-specific table gateway/repository)
```

**Testing**:
- Handler tests: Integration tests covering the entire use case, including the database.

## Hybrid Approach: Lean Reads, Rich Writes

It is common and encouraged to use **Full DDD for Writes** (to protect invariants) and **Lean ORM for Reads** (for performance and simplicity).

- **Commands** (Writes): Load aggregate through Repository and Mapper, execute behavior on aggregate, save through Repository and Mapper.
- **Queries** (Reads): Use ORM, Query Builder, or raw SQL directly to fetch data into lean DTOs, bypassing mappers and domain models entirely.

This is not CQRS; both paths use the same model and store. See `application-layer` for when separate read and write models are worth it.
