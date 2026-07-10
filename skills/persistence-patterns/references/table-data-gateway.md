# Table Data Gateway

A Table Data Gateway is one class per database table. It knows how to retrieve and persist rows for that table. **Nothing more** — no business behavior, no domain concepts.

```php
final class ContractTableGateway
{
    public function __construct(private readonly PDO $pdo) {}

    /** Returns rows (a recordset), NOT domain objects. */
    public function findByCustomer(int $customerId): array
    {
        $stmt = $this->pdo->prepare(
            'SELECT id, revenue, cost FROM contracts WHERE customer_id = ?'
        );
        $stmt->execute([$customerId]);

        return $stmt->fetchAll(PDO::FETCH_ASSOC);
    }

    public function insert(array $row): void
    {
        // plain INSERT into the contracts table
    }
}
```

Notice it returns arrays (rows), not `Contract` domain objects. That is the tell of a gateway.

### Where it works well
- Reporting systems
- ETL pipelines
- Import / export tools
- Administrative utilities
- Microservices with minimal domain logic
- Read models (CQRS query side)
- Anywhere the database schema already *is* the application model

ORMs now generate much of this code automatically (e.g., CakePHP `Table`, Laminas `TableGateway`, Laravel's query builder / `DB` facade, raw Doctrine DBAL queries are all gateways in spirit).

### Where it struggles
As soon as business behavior becomes rich, gateways begin accumulating methods like:

- `findEligiblePremiumCustomers()`
- `findOrdersAwaitingShipment()`
- `findCustomersMissingTaxInformation()`
- `findInvoicesOverCreditLimit()`

Eventually they become giant procedural collections. **At that point you have started building a Repository without realizing it.** The fix is not to keep piling query methods onto the gateway — it is to move the behavior into a domain model and let a true Repository return domain objects, or to split into dedicated query services.

### See also
- The [Gateway vs Repository](../data-source-patterns.md#the-crucial-distinction-gateway-vs-repository) distinction — a Table Data Gateway is *not* a Repository.
- When you want behavior on the row, graduate to [Active Record](../active-record.md); when the domain should be fully decoupled, use a [Data Mapper](../data-mapper.md) with repositories.
