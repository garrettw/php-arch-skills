# Domain Modeling Decision Flowchart

When deciding whether to use a full Domain-Driven Design (DDD) structure (domain model, mapper, repository interface) or a lean ORM structure, use this decision flowchart.

Apply this flowchart **per area of the system, not once for the whole codebase**. Different bounded contexts can legitimately use different styles: a simple CRUD context may be a Transaction Script, a reporting context a Table Module, and the rule-heavy core a Domain Model. Pick the lightest pattern that fits each area's rules and let each area evolve on its own (see the `bounded-contexts` skill for carving those boundaries).

## The Decision Rule

Before adding a new mapped domain entity or full DDD structure, ask these questions:

1. **Is this feature a single linear procedure per request?** (read some data, validate, calculate, write a result — with few business rules)
2. **Does this entity have meaningful domain behavior?** (e.g., business rules, invariants, complex state transitions)
3. **Would the core rule be easier to test in memory?** (without booting the PHP framework or database)
4. **Is the persistence structure fundamentally different from the domain language?**
5. **Would using the ORM entity directly leak persistence concerns into important business rules?**

### If YES to #1 and NO to #2-#5:
Use a **Transaction Script**. This is the lightest option and the right default for simple CRUD and linear request/response flows (see [transaction-script.md](transaction-script.md)).
- Write one procedure per request that orchestrates the steps and talks to the database through a thin gateway.
- Keep validation/calculation local to the script unless it recurs across scripts.
- Avoid mappers, repositories, domain interfaces, and DTOs unless they earn their cost.

### If the logic is row-set-oriented (one table, many rows) but #2-#5 are partial:
Use a **Table Module** — the middle option between Transaction Script and Domain Model (see [table-module.md](table-module.md)).
- Create one class per table whose single instance operates over a recordset (all rows).
- Pair it with a thin Table Data Gateway that returns the recordset; keep SQL out of the module.
- Upgrade to a Domain Model only when rows need their own identity, relationships, or polymorphic behavior.

### If YES to #2-#5 (behavior outweighs procedure):
Use the **Full DDD Structure**. The entity has rich behavior that justifies the cost of separation.
- Create an active domain object that protects its invariants.
- Keep it pure PHP, free from framework ORM dependencies.

### If NO to all (no procedure complexity, no behavior):
Use the **Lean ORM Structure**. The object is likely a data bag, a report projection, a configuration record, or a simple CRUD entity.
- Use the framework's ORM entities directly.
- Avoid the overhead of mappers and domain interfaces.

## When to Upgrade from Transaction Script or Table Module to a Domain Model

A Transaction Script is fine until the same rule appears in more than one script and starts drifting. When a validation, calculation, or policy recurs across procedures, lift it into a shared active domain object or policy and keep the script as the orchestrator. This is the main trigger to move from the Transaction Script path toward the Full DDD path.

A Table Module is fine while its rules stay table-wide. It stops scaling once rules become **individualized per row** — pricing policies, tax exemptions, regional regulations, contract terms, and subscription behavior are properties of each record, not of the table as a whole. When those per-record rules pile up, give each row its own object with identity and move the rules onto it. That is the trigger to move from the Table Module path toward the Full DDD path.

## Cost of Full DDD

Full DDD should earn its cost. Adding a field to a mapped entity often requires changes in:
- Domain model
- ORM entity
- Mapper
- Repository
- Factory / Test Fixtures
- Mapper tests
- Domain tests

If that change amplification does not buy protection around real behavior, use the lean path.

## Testability Signals

Use these signals to determine if your domain modeling is on the right track.

### Good Signs
- The test constructs a domain object or policy directly (using `new`).
- The test names the business scenario clearly.
- The expected result is a business outcome, state transition, or domain event.
- Infrastructure appears only as simple input data (e.g., plain arrays or DTOs).

### Bad Signs
- The domain test requires a database, an HTTP request, a provider SDK, or booting the PHP application framework.
- The test mocks a large chain of infrastructure calls.
- The test asserts internal method calls instead of the final business outcome.
- A rule is easier to test through a controller than through the domain object that should logically own it.
