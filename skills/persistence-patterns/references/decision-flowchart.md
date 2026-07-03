# Persistence Decision Flowchart

Use this guide to determine the appropriate persistence structure for a new feature or aggregate.

## Full DDD Persistence Structure

**Use when**: The entity has rich behavior, complex state transitions, or cross-context policies that must be tested without the framework.

**Structure**:
```text
src/Domain/[BoundedContext]/
  [EntityName].php (Pure PHP domain object)
  [EntityName]RepositoryInterface.php (Optional, if required by criteria)
src/Infrastructure/Persistence/
  [EntityName]Repository.php (Implements interface, uses mapper)
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
src/Application/[BoundedContext]/
  [CommandOrQuery]Handler.php (Uses ORM directly)
src/Infrastructure/Persistence/
  OrmEntity.php (Framework-specific active record or entity)
  OrmTable.php (Framework-specific table gateway/repository)
```

**Testing**:
- Handler tests: Integration tests covering the entire use case, including the database.

## Hybrid Approach: CQRS Read vs Write

It is common and encouraged to use **Full DDD for Writes** (to protect invariants) and **Lean ORM for Reads** (for performance and simplicity).

- **Commands** (Writes): Load aggregate through Repository and Mapper, execute behavior on aggregate, save through Repository and Mapper.
- **Queries** (Reads): Use ORM, Query Builder, or raw SQL directly to fetch data into lean DTOs, bypassing mappers and domain models entirely.
