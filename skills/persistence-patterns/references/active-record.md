# Active Record

Each object:

- represents **one database row**
- contains **business behavior**
- knows **how to persist itself**

The goal is simplicity. Rather than separating persistence, mapping, and business behavior into different objects, Active Record **combines** them. For many applications this dramatically reduces complexity and is incredibly intuitive (Rails, Laravel, Yii, and CakePHP all default to an Active Record style ORM).

```php
final class Customer extends Model
{
    // represents one row...

    // ...and contains business behavior
    public function isEligibleForPremium(): bool
    {
        return $this->lifetimeValue >= 10000 && !$this->churned;
    }

    // ...and knows how to persist itself (save(), find(), etc. come from the framework)
}
```

### Where it works well

- Business rules are modest
- CRUD dominates
- Developers value rapid iteration
- Applications are relatively small

Used within its natural limits, Active Record is outstanding — many frameworks embrace this philosophy because developer productivity matters, and that is a perfectly valid tradeoff. Not every application needs enterprise-grade separation.

### Where it becomes a liability

Eventually the object may begin carrying too many responsibilities. It becomes responsible for:

- validation
- querying
- persistence
- relationships
- business rules
- lifecycle events
- serialization

At that point the class stops representing the **business** and starts representing the **ORM**. The real problem is not Active Record itself (it does not prevent good architecture) — it is refusing to recognize when the application has outgrown it. Many developers believe Active Record is inherently bad; the pain comes from using it beyond its natural limits, once the domain grows richer and more interconnected.

At that point, move the persistence/mapping out into a **[Data Mapper](data-mapper.md)** and let the domain objects focus on behavior.

### See also
- [Data Source spectrum & core philosophy](../data-source-patterns.md) — Active Record optimizes for throughput and simplicity; [Data Mapper](data-mapper.md) for longevity and domain isolation. Neither is universally superior.
- For raw row access without per-row objects, a [Table Data Gateway](table-data-gateway.md) is simpler.
