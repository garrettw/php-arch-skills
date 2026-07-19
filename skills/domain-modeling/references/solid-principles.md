# SOLID Principles

SOLID is a set of five design principles that keep modules testable, substitutable, and free of unnecessary coupling. They are most useful as *review lenses* while writing or analyzing PHP backend code — not as rules to apply ritualistically. Each principle below states what to aim for, with the failure mode it prevents.

## Single Responsibility Principle (SRP)

A class should have one reason to change. Co-locate one responsibility per class; move the others out.

- Keep a domain entity to *domain state and behavior* only: identity, attributes, and the rules that keep its invariants. Do not let it open database connections, send mail, render JSON, or validate raw request arrays.
- Put persistence in a **Repository**, notifications in a notifier, serialization in a presenter/resource, and orchestration in an **Application Service / Handler**.
- A class that needs 5+ unrelated collaborators, or whose name contains `And` / `Manager` / `Helper`, usually carries more than one responsibility.

```php
// Entity owns only domain state + rules
final class User
{
    public function __construct(
        private UserId $id,
        private Email $email,
        private UserStatus $status = UserStatus::Active,
    ) {}

    public function activate(): void { $this->status = UserStatus::Active; }
}

// Persistence is a separate responsibility
interface UserRepository
{
    public function save(User $user): void;
    public function find(UserId $id): ?User;
}
```

SRP is the domain-side expression of **Separation of Concerns** (see [clean-code-foundations.md](clean-code-foundations.md)) and underlies the anemic-domain-vs-rich-domain split in this skill.

## Open/Closed Principle

Modules should be open for extension but closed for modification: add new behavior by adding new types, not by editing existing ones.

- When a method grows a chain of `if ($type === ...) { ... }` or a `match` that you edit every time a variant appears, extract a **Strategy** (or Plugin) behind an interface and register implementations.
- This is exactly the Plugin / Strategy pattern used at the infrastructure boundary (see [plugin.md](../../dependency-injection/references/plugin.md) and [behavioral-structural-patterns.md](behavioral-structural-patterns.md)). The application layer stays closed; new payment methods, discount rules, or exporters arrive as new classes.

```php
interface DiscountStrategy
{
    public function calculate(Money $amount): Money;
}

final class PercentageDiscount implements DiscountStrategy { /* ... */ }
final class FixedAmountDiscount implements DiscountStrategy { /* ... */ }

// Closed for modification; new discounts are new classes
final class DiscountCalculator
{
    public function apply(Money $amount, DiscountStrategy $strategy): Money
    {
        return $amount->subtract($strategy->calculate($amount));
    }
}
```

Watch the YAGNI line: do not build a strategy hierarchy for two variants that will never grow. Apply OCP once a real second/third variant exists (see YAGNI in [clean-code-foundations.md](clean-code-foundations.md)).

## Liskov Substitution Principle (LSP)

Subtypes must be substitutable for their base type without breaking correctness. A subclass should strengthen, never weaken, the base contract.

- Honor preconditions and postconditions of the parent. Do not throw `LogicException` from methods a base type promises to support, and do not silently change semantics (the classic `Square extends Rectangle` that overrides `setWidth` to also set height breaks any caller relying on independent dimensions).
- Prefer **composition over inheritance** when the "is-a" relationship is really "behaves-like-a in some ways." Inheritance that violates LSP is usually a misuse of subtyping (see Composition over Inheritance in [clean-code-foundations.md](clean-code-foundations.md)).
- In the domain, substitution matters for polymorphism used by the application layer: a `DiscountStrategy` must produce a *valid* discount in every implementation, or callers cannot treat them uniformly.

## Interface Segregation Principle

Clients should not depend on methods they do not use. Keep interfaces small and role-specific.

- Split fat interfaces into focused ones. A `WorkerInterface` with `work()` / `eat()` / `sleep()` forces a `Robot` to implement methods it cannot honor. Prefer `Workable`, `Feedable`, `Sleepable`.
- At the boundary this means narrow ports: an outbound `PaymentGateway` interface should expose only what the application needs, not a grab-bag of every gateway's capabilities. This also keeps adapters thin (see [adapter-design-examples.md](../infrastructure-boundaries/references/adapter-design-examples.md)).
- Interface Segregation is why a Repository interface should match the aggregate's real needs (find/save/remove), and not become a generic CRUD kitchen-sink.

## Dependency Inversion Principle (DI)

High-level policy should not depend on low-level details; both depend on abstractions. Dependencies point inward toward the domain.

- Domain and application code should depend on **interfaces** (ports), not on concrete databases, mailers, gateways, or loggers. Concrete adapters are supplied from the composition root / service container at the edge (see [plugin.md](../../dependency-injection/references/plugin.md) and the `dependency-injection` skill).
- Never `new` a concrete infrastructure object inside a domain object or handler. Receive it via constructor injection so the same code can run against an in-memory double in tests.

```php
// Depends on the abstraction, not on MySQL/Stripe/Smtp
final class OrderService
{
    public function __construct(
        private OrderRepository $orders,   // port
        private PaymentGateway $payments,  // port
        private Mailer $mailer,            // port
    ) {}
}
```

DI is the backbone of Hexagonal / Ports-and-Adapters architecture and the reason framework types must not leak across boundaries.

## How to apply SOLID without overdoing it

SOLID is a means, not a score. Over-applying it produces the same pain as ignoring it:

- SRP taken to the extreme yields call-stack-deep procedural code; keep cohesive behavior together.
- Open/Closed and DI for code with no second variant is premature abstraction (see YAGNI).
- Interface Segregation matters most at *boundaries*; internal, single-team code can use a slightly wider interface without harm.

When in doubt, favor the smallest design that satisfies the area's current complexity, and let SOLID guide the *next* refactor rather than the first draft.
