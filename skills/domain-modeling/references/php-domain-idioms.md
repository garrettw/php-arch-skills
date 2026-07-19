# PHP Domain Idioms (Optional)

These are PHP-language idioms that *serve* the domain-modeling goals in this skill — encapsulation, explicit invariants, and a clean error story. They are **optional**: modern PHP offers several ways to express the same idea, and the choice is a team convention, not an architectural requirement. Reach for them when they make the domain clearer; do not treat them as mandatory.

## Enums as value objects

A `backed enum` is a natural fit for a small, closed set of domain states or categories. It is a named, type-safe value object: callers can't pass an arbitrary string, and the domain meaning lives with the value.

```php
enum OrderStatus: string
{
    case Pending = 'pending';
    case Paid = 'paid';
    case Shipped = 'shipped';
    case Cancelled = 'cancelled';

    public function isTerminal(): bool
    {
        return match ($this) {
            self::Shipped, self::Cancelled => true,
            default => false,
        };
    }
}

final class Order
{
    public function __construct(
        public readonly OrderId $id,
        private OrderStatus $status,
    ) {}

    public function ship(): void
    {
        if ($this->status->isTerminal()) {
            throw new DomainException("Cannot ship an order that is {$this->status->value}");
        }
        $this->status = OrderStatus::Shipped;
    }
}
```

- Use a backed enum when the set is closed and the values cross a boundary (API, DB, queue) — the backing value is the wire format.
- Attach behavior (`isTerminal()`, `permissions()`) to keep the rule with the value, not scattered as `match ($status->value)` at every call site (DRY + encapsulation).
- Prefer this over an `int`/`string` constant or a status column read as a raw string. When the set is *open* (user-definable types), a proper value object with validation is the better fit — see [clean-code-foundations.md](clean-code-foundations.md).
- Use `readonly` properties and constructor promotion (PHP 8.0+) for immutable value objects and entities so state can only change through a method that protects the invariant.

## The Result pattern (typed alternative to exceptions)

Some operations have an expected, non-exceptional "couldn't do it" outcome (validation failed, item out of stock, duplicate). Rather than throwing for a routine failure, return a `Result` that carries success or a typed error. This makes the failure path explicit in the signature instead of hiding it in a thrown type.

```php
/**
 * @template T
 */
final readonly class Result
{
    private function __construct(
        public mixed $value,
        public ?string $error,
        public bool $success,
    ) {}

    /** @return self<T> */
    public static function success(mixed $value): self
    {
        return new self($value, null, true);
    }

    public static function failure(string $error): self
    {
        return new self(null, $error, false);
    }

    public function map(callable $fn): self
    {
        return $this->success ? self::success($fn($this->value)) : $this;
    }
}

function placeOrder(Order $order): Result
{
    if ($order->isEmpty()) {
        return Result::failure('Order has no items');
    }
    // ...
    return Result::success($order->id);
}
```

- Use `Result` for **expected** failures that are part of the use case's normal flow — the caller must decide what to do, and the type forces them to check.
- Use **exceptions** (or a Special Case) for *unexpected* or genuinely erroneous situations: broken invariants, infrastructure failure, a lookup that should never miss. Don't mask a real error behind a `Result::failure`.
- Don't return `Result` from every method — that is the same over-engineering as throwing everywhere. Apply it at the boundary of an operation whose failure is a legitimate outcome (a handler returning to the controller, a domain service reporting a business rule rejection).

## Traits for cross-cutting concerns

PHP traits are a tool for **sharing behavior across unrelated types** — most safely when that behavior is a *cross-cutting concern* that several objects need but that is not part of their core identity (timestamps, soft-delete markers, audit stamps, UUID generation).

```php
trait Timestampable
{
    private ?DateTimeImmutable $createdAt = null;
    private ?DateTimeImmutable $updatedAt = null;

    public function touch(): void
    {
        $now = new DateTimeImmutable();
        $this->createdAt ??= $now;
        $this->updatedAt = $now;
    }
}

final class Document
{
    use Timestampable;
}
```

- Best used for **cross-cutting, orthogonal** behavior (logging hooks, timestamps, serialization helpers) — not to fake multiple inheritance for core domain logic.
- Avoid traits that bundle business rules spanning several aggregates; that tangles concerns. Put shared *domain* rules on a value object, policy, or service instead (see [clean-code-foundations.md](clean-code-foundations.md) on Separation of Concerns).
- Be wary of trait conflicts (`A::hello insteadof B`); if you need conflict resolution often, the shared behavior probably belongs in a collaborator you inject, not a trait.
