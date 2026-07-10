# Data Mapper

This is the preferred solution for complex enterprise software.

- **Business objects know nothing about databases.**
- **Persistence objects know nothing about business rules.**
- **The mapper sits between them**, moving data between the two.

Each layer has exactly one responsibility. It eliminates the coupling between business logic and relational storage: the domain stays focused entirely on the business, and persistence becomes replaceable. The richer your business rules become, the more valuable this separation is.

```php
// Domain: knows nothing about the database
final class Customer
{
    public function __construct(
        public readonly int $id,
        private Money $lifetimeValue,
        private bool $churned,
    ) {}

    public function isEligibleForPremium(): bool
    {
        return $this->lifetimeValue->gte(Money::from(10000)) && !$this->churned;
    }
}

// Mapper: the only place that bridges the two
final class CustomerMapper
{
    public function toDomain(array $row): Customer { /* ... */ }
    public function toRow(Customer $c): array { /* ... */ }
}

// Repository: returns domain objects, hides the table thinking
final class CustomerRepository
{
    public function __construct(
        private readonly PDO $pdo,
        private readonly CustomerMapper $mapper,
    ) {}

    public function findEligibleForPremium(): array { /* uses mapper */ }
}
```

### What it enables

- **Persistence ignorance** — domain objects are oblivious to how they are stored
- **Rich domain models** — behavior lives on the object without ORM constraints
- **Easier testing** — domain logic is testable without a database
- **Multiple storage mechanisms** — the same domain can be served by SQL, a document store, or an API
- **Clearer separation of concerns** — each layer has one job

It lets developers ask *"What should this business object do?"* rather than *"How should this object save itself?"* — a fundamentally better question to be asking.

### The biggest criticism: complexity

Compared to [Active Record](active-record.md), there are more moving pieces. For small applications that overhead is often unnecessary — which is exactly why Data Mapper and Active Record are different points on the same spectrum.

### The myth that refuses to die

"Data Mapper lets you swap databases easily." Technically true; practically, that is rarely the primary benefit. **The real value is not swapping databases — it is preventing database concerns from leaking into the domain model.** That dividend is paid every day, whereas the database swap almost never happens.

### Where it dominates

Data Mapper is the dominant persistence strategy for:

- Domain-Driven Design
- Hexagonal (Ports & Adapters) Architecture
- Onion Architecture
- Clean Architecture
- many CQRS implementations (the write side)

It has become a foundational idea of enterprise software. For software with substantial business complexity it remains the preferred persistence model.

The additional indirection is justified **only when it buys you something** — namely, a domain model that would otherwise be compromised by persistence concerns. For CRUD-heavy or small apps, [Active Record](active-record.md) is the better tradeoff.

### See also
- The [Data Source spectrum & core philosophy](../data-source-patterns.md) — pick by the problem's complexity, not fashion.
- The write side of a CQRS split leans toward Data Mapper + Repository once invariants matter (see the [Decision Flowchart](decision-flowchart.md)).
