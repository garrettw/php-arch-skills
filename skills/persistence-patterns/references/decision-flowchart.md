# Persistence Decision Flowchart

Use this guide to determine the appropriate persistence structure for a new feature or aggregate.

## Data Source Spectrum (pick the lightest that fits)

There is a progression of separation between business logic and storage (see [data-source-patterns.md](data-source-patterns.md); for the ORM-internal behavioral/structural patterns, see [orm-patterns.md](orm-patterns.md); for the patterns surrounding the ORM — Metadata Mapping, Query Object, Repository, Value Object — see [orm-architecture-patterns.md](orm-architecture-patterns.md)):

> Persistence should be isolated from business logic to whatever degree the application's complexity warrants. Active Record and Data Mapper are not competitors with a single winner — Active Record optimizes for throughput and simplicity; Data Mapper optimizes for longevity and domain isolation. Pick by the problem's complexity, not by fashion.

- **[Table Data Gateway](table-data-gateway.md)** — one class per table; fetches/persists rows, returns recordsets. Lightest. For reporting, ETL, import/export, admin utilities, read models, CRUD where the schema *is* the model.
- **[Row Data Gateway](row-data-gateway.md)** — one object per row; loads/updates/deletes itself but little business behavior. **Recommend against** in new code (faded pattern; an awkward middle ground) — use Table Data Gateway for raw row access, Active Record for row+behavior, or Data Mapper for full separation.
- **[Active Record](active-record.md)** — a row object that also carries its behavior; database-coupled. Outstanding for CRUD-heavy apps with modest rules and small teams that value rapid iteration. Becomes a liability as the domain grows richer/interconnected (the class drifts from representing the business to representing the ORM). When it outgrows its limits, move persistence/mapping out into a Data Mapper.
- **[Data Mapper](data-mapper.md)** — domain objects fully isolated from the DB; a Mapper moves data between them. Preferred for complex enterprise software (DDD, Hexagonal, Onion, Clean, CQRS writes). The real value is keeping DB concerns out of the domain, not easy DB swaps. Heaviest — only justified when persistence would otherwise compromise a rich domain model.

**Gateway vs Repository:** a Gateway thinks in tables and returns rows; a Repository thinks in business concepts and returns domain objects. A Table Data Gateway is **not** a Repository even if named one. If a gateway starts growing business-flavored query methods, that is a Repository forming — move the behavior into the domain.

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
