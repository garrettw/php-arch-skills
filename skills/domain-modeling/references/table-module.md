# Table Module

Table Module is a pattern positioned as the middle ground between Transaction Script and Domain Model.

## What It Is

A Table Module is a single class per database table or view. Unlike a Domain Model (one object per row, each with its own identity), a Table Module has **one instance that handles the business logic for all rows** of the table. Its methods operate over a recordset (a set of rows), rather than over a single entity.

```
Client request
      │
      ▼
Table Module (one class per table; methods act on a recordset)
      │  operates over
      ▼
Recordset / DataSet (rows from the table)
```

The three business-logic patterns can be framed as a spectrum:

- **Transaction Script** — one procedure per request; logic calls the DB directly. Lightest.
- **Table Module** — one class per table; a single instance handles all rows via a recordset. Middle ground.
- **Domain Model** — one object per row, each with identity and its own behavior. Heaviest, richest.

The key distinction: with a Domain Model, `Employee` has one object *per employee*; with a Table Module, one `Employee` object handles *all employees*.

## Why It Exists

The Domain Model's hard part is the relational-database interface — the gymnastics of transforming between rows and objects (the O/R mapping problem). A Table Module sidesteps much of that by keeping data in its table/recordset shape and attaching behavior at the table level. It is especially natural in environments that already hand you a recordset (e.g., a grid-bound UI, report builders, or data-centric business apps), and it is usually paired with a Table Data Gateway that returns the recordset.

## When to Use Table Module

Reach for a Table Module when your logic is shaped around *sets of rows in a table* rather than around individual objects with identity.

| Use Table Module For                                                             | Prefer Domain Model / Transaction Script Instead            |
|----------------------------------------------------------------------------------|-------------------------------------------------------------|
| Logic that computes over many rows of a table (totals, summaries, batch updates) | Behavior centered on a single object's identity/lifecycle.  |
| Data-grid / report-centric UIs already backed by recordsets                      | Rich object relationships, polymorphism, state machines     |
| You want behavior bundled with data but want to avoid O/R mapping cost           | The domain has complex invariants best expressed per-object |
| Table structure maps cleanly onto the business logic                             | Behavior crosses many tables in graph-like ways             |

**It is the compromise.** Transaction Script is too flat when you have real per-table logic that should be co-located with the data; a full Domain Model is too heavy when you have no need for object identity and want to avoid mapping rows to objects. Table Module lands in between.

### When Table Module works best
Table Module shines when the application is fundamentally **data-centric**:

- Records don't carry much individual behavior of their own.
- Operations naturally span many rows (totals, summaries, batch updates, bulk validation).
- Business rules are oriented to the table as a whole rather than to single rows.

### Its weakness: rules that become individualized
Table Module scales poorly once business rules stop being table-wide and start differing **per row**. Pricing policies, tax exemptions, regional regulations, contract terms, and subscription behavior are not properties of "the customer table" — they are properties of *each* customer. When rules like these pile up, they no longer belong to the table module; they belong to each individual object. **That is the moment a Domain Model becomes preferable** — give each row its own object with identity, and let the per-customer rules live on it.

## Structure in PHP

PHP has no built-in ADO.NET-style `DataTable`, so a "recordset" is usually an array of associative arrays, a typed row collection, or a query result. The Table Module takes that recordset and exposes behavior that operates across it:

```php
final class ContractTableModule
{
    public function __construct(
        private readonly ContractGateway $gateway,
    ) {}

    /** Operates over all rows of the contract table for a customer. */
    public function revenueByCustomer(int $customerId): int
    {
        $rows = $this->gateway->findByCustomer($customerId);

        return array_sum(array_map(
            static fn (array $row) => $row['revenue'] - $row['cost'],
            $rows,
        ));
    }

    /** Returns the rows with a derived field already computed. */
    public function withCalculatedMargin(array $rows): array
    {
        return array_map(
            static fn (array $row) => $row + [
                'margin' => $row['revenue'] - $row['cost'],
            ],
            $rows,
        );
    }
}
```

Notes:
- One class per table; instantiate it once and call methods that take/return recordsets.
- Pair it with a [Table Data Gateway](../persistence-patterns/references/data-source-patterns.md) that returns the recordset, keeping SQL out of the module.
- Behavior is row-set-oriented. If you find yourself needing per-row identity, relationships, or polymorphic behavior, that is the signal to move to a Domain Model.

## Relationship to the Rest of This Skill Set

- Table Module is the middle column of the Transaction Script → Table Module → Domain Model spectrum, described in [transaction-script.md](transaction-script.md).
- It still relies on a thin data gateway (see `persistence-patterns`); it just keeps data in table shape instead of mapping to rich objects.
- When a table module's per-row behavior grows to need identity, invariants, and relationships, refactor those rows into active domain objects and let the Table Module become a Domain Model.
- Table Module does not require repositories or mappers; add those only when the persistence structure diverges enough to warrant a Domain Model (see `persistence-patterns`).
