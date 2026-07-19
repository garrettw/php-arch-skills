# Clean-Code Foundations

These are the cross-cutting principles that keep backend code readable, testable, and cheap to change. They underpin the SOLID principles ([solid-principles.md](solid-principles.md)) and the boundary-protection patterns. Stated as aims, each implies the failure mode it prevents.

## Separation of Concerns

Each module, class, and function should address a single concern. HTTP parsing, authentication, validation, business rules, persistence, and presentation are different concerns and belong in different places.

- In a handler, convert the request to primitives/DTOs first, run domain logic, then persist — do not interleave `$_POST`/`$request` access with business rules or SQL.
- This is the structural form of SRP and the reason an application layer exists above the domain (see the `application-layer` skill).

## Law of Demeter (tell, don't ask)

A method should talk only to its immediate collaborators, not to strangers reached through chains. Do not write `$order->getCustomer()->getAddress()->getCountry()->getTaxRules()->calculateTax($amount)`.

- Ask the object that owns the data to do the work: `$order->taxAmount()` (the `Order` asks its `Customer`, which asks its `Address`).
- Prefer returning a value object over exposing a deep graph. This keeps the dependency surface small and prevents one object from reaching across another's internals (related to encapsulation and to keeping aggregates referenced by ID).

## Encapsulation

Hide internal state and expose behavior. Protect invariants through the object's own methods; never leave public fields that let external code break a business rule.

```php
final class BankAccount
{
    public function __construct(private Money $balance) {}

    public function withdraw(Money $amount): void
    {
        if ($amount->greaterThan($this->balance)) {
            throw new InsufficientFunds($amount);
        }
        $this->balance = $this->balance->subtract($amount);
    }
}
```

- Encapsulation is why domain rules live *inside* the aggregate, not in services that mutate public properties (see the anemic-domain smell in this skill).
- Validation that protects an invariant belongs in the method that changes state, not in a separate layer that can be skipped.

## Fail Fast

Detect and report errors as early as possible: validate at the boundary, check preconditions at the top of a function, and throw immediately when something is wrong.

- Validate raw input in the controller/request layer; throw domain exceptions the moment an invariant would be broken.
- Do not "continue and hope" with `?->` chains over possibly-null values that should never be null — fail fast with a clear exception so the bug surfaces at its source, not three calls later.
- Fail-fast dovetails with LSP preconditions and with the Special Case pattern: a *valid* absence is modeled as an object; a *broken* precondition is an exception.

## YAGNI (You Aren't Gonna Need It)

Build only what the current requirement demands. Do not add speculative fields, flags, or branches "in case we need them later."

- Do not create an abstraction (interface, base class, strategy hierarchy) until a second concrete need exists. A single implementation behind an interface "for flexibility" is premature — see OCP in [solid-principles.md](solid-principles.md).
- Resist speculative configuration, plug-in points, and generic "framework" scaffolding for a one-off feature. Add them when the second use case arrives.

## Composition Over Inheritance

Prefer building objects from focused collaborators over deep inheritance hierarchies. Composition avoids the fragile base-class problem and keeps each piece replaceable.
Be very skeptical about usage of inheritance, and if composition isn't the appropriate solution, consider an interface + corresponding trait.

- Use inheritance only for a true "is-a" subtype that honors LSP. Otherwise inject a collaborator. This is why the application layer composes a Repository + domain object + notifier rather than subclassing a `BaseService` that knows everything.
- GoF patterns such as Strategy, State, and Composite (catalogued in [behavioral-structural-patterns.md](behavioral-structural-patterns.md)) are composition in action.

## KISS (Keep It Simple)

Favor the simplest design that satisfies the requirement. Readability beats cleverness.

- Choose the lightest domain-logic style for the area's complexity (Transaction Script → Table Module → Domain Model — see this skill's decision flowchart). Do not reach for a rich Domain Model where a Transaction Script is enough.

## DRY (Don't Repeat Yourself)

Every piece of knowledge should have a single, authoritative representation. Duplicate logic drifts out of sync.

- When the same validation, calculation, or policy appears in more than one place, move it to one shared, named location (a domain method, a small service, or a value object).
- Be precise about "knowledge": two pieces of code that *look* similar but encode different rules are not duplication. DRY the rule, not the shape.
- The Repository pattern ([persistence-patterns](../persistence-patterns/SKILL.md)) is DRY applied to data access: one place knows how to load/save an aggregate.

### When a little repetition is fine

DRY is easy to over-apply: collapsing two pieces of merely *similar* code into one shared unit couples them and forces conditional logic that neither needed on its own. Tolerate some duplication rather than create over-abstracted, highly coupled code to save a few keystrokes.

1. **Wait for the Rule of Three.** Use the Write Everything Twice (WET) approach: do not abstract the moment you copy a snippet. Tolerating the duplication until a third use case appears lets the two existing ones diverge naturally, and a clearer, more fitting abstraction usually emerges the third time. Abstracting at two is often guessing.

2. **Duplicate knowledge over syntax.** DRY is about the duplication of *intent*, not lines. If a shipping module and a billing module both hard-code the same tax-rate constant, that is duplicated knowledge worth centralizing. If two unrelated features happen to share an identical `for` loop, that is incidental similarity and should not be unified.

3. **Ask "do these parts always change together?"** The guiding test: if it changes together, it stays together; otherwise keep it separate. A single shared "super-function" for two slightly different workflows forces `if` branches and boolean flags the moment one diverges, making the code rigid and error-prone.

4. **Prefer simple utilities over complex abstractions.** To cut repetition without sacrificing readability, extract small, independent utility functions instead of rigid base classes, inheritance, or heavily parameterized generic functions. Keep dependencies injected and the control flow flat.
