# Domain Modeling Decision Flowchart

When deciding whether to use a full Domain-Driven Design (DDD) structure (domain model, mapper, repository interface) or a lean ORM structure, use this decision flowchart.

## The Decision Rule

Before adding a new mapped domain entity or full DDD structure, ask these questions:

1. **Does this entity have meaningful domain behavior?** (e.g., business rules, invariants, complex state transitions)
2. **Would the core rule be easier to test in memory?** (without booting the PHP framework or database)
3. **Is the persistence structure fundamentally different from the domain language?**
4. **Would using the ORM entity directly leak persistence concerns into important business rules?**

### If YES to any of the above:
Use the **Full DDD Structure**. The entity has rich behavior that justifies the cost of separation.
- Create an active domain object that protects its invariants.
- Keep it pure PHP, free from framework ORM dependencies.

### If NO to all of the above:
Use the **Lean ORM Structure**. The object is likely a data bag, a report projection, a configuration record, or a simple CRUD entity.
- Use the framework's ORM entities directly.
- Avoid the overhead of mappers and domain interfaces.

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
