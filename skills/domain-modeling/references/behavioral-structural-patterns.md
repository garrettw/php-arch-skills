# Behavioral & Structural Patterns (Domain Toolkit)

A focused set of patterns that earn their place in domain modeling: **Strategy, Builder, Composite, State, Visitor, Observer**. Each is described as a *domain* tool — when to reach for it, and how it interacts with the rest of the architecture. (PHP has `clone`, so Prototype is omitted; Flyweight/Interpreter are rarely needed in business PHP.)

## Strategy

Defines a family of interchangeable algorithms behind a common interface, so the caller picks (or injects) one without knowing the variant.

- **When:** a business rule has several valid implementations that vary independently of the object holding them — pricing rules, tax calculators, discount strategies, shipping methods.
- **In this skill set:** the strategy interface is a Port; each variant is an Adapter wired at the composition root (see [dependency-injection](../dependency-injection/SKILL.md) and [Plugin](../dependency-injection/references/plugin.md)). The domain depends on the interface, never the concrete algorithm.
- **Watch:** don't encode the variant as a string switch inside the domain — inject the strategy instead.

## Builder

Encapsulates the stepwise construction of a complex object, keeping the construction logic out of the object's public constructor.

- **When:** an entity or value object needs many optional parts, or must be assembled in a specific order (e.g., an `Order` built from line items, a `Money`/`Decimal` constructed from parts, a complex query/report object).
- **In this skill set:** keep the Builder in the domain layer as plain PHP; it is a creation concern, not infrastructure. Avoid letting a framework's query builder leak into the domain as if it were this pattern.
- **Watch:** prefer a Builder over a constructor with a dozen positional arguments; never put persistence logic in the builder.

## Composite

Treats individual objects and compositions of objects uniformly through a common interface, so callers don't care whether they address one item or a tree of them.

- **When:** a domain has part-whole hierarchies addressed the same way — org charts, menu/category trees, bill-of-materials, a single `Discount` vs a `DiscountGroup`.
- **In this skill set:** the composite and its leaves share the domain interface (like a [Special Case](special-case.md) implementing the same type). Traversal/aggregation logic lives in the composite, not in callers.
- **Watch:** only use it when clients genuinely treat the single and the composite identically; a forced tree where callers always special-case leaves is a smell.

## State

Allows an object to alter its behavior when its internal state changes, by delegating to a current state object that encapsulates both the behavior and the transitions.

- **When:** an entity's valid actions and transitions depend on a well-defined lifecycle — order status (pending → paid → shipped → refunded), subscription state, job state.
- **Related:** distinct from **Application Controller** ([web-presentation-patterns](../../web-presentation-patterns/references/classic-web-patterns.md)), which decides *which screen/step comes next* (presentation flow). State is about *how a domain object behaves* in a given condition. A wizard flow may use State internally and an Application Controller for navigation — different concerns.
- **Watch:** prefer modeling explicit states (often as a small set of classes or a typed enum with transition methods) over a tangle of `if ($status === ...)` checks scattered through the entity. Keep transitions in one place.

## Visitor

Lets you define new operations over a heterogeneous object structure without changing the classes of the elements, by putting the operation in a separate visitor object the structure accepts.

- **When:** you must perform many different, unrelated operations on a stable set of domain types — e.g., compute totals, export, validate, or apply discounts across varied `OrderItem` subtypes. The object structure is stable; the operations vary.
- **In this skill set:** the accept/visit split keeps domain types free of cross-cutting operation code. Be cautious in PHP: a visitor usually needs a method per concrete type, which couples it to the type hierarchy.
- **Watch:** if the object structure changes often, Visitor becomes a maintenance burden — prefer adding the method to the type instead.

## Observer

Defines a one-to-many dependency so that when one object changes state, its dependents are notified automatically.

- **In this skill set:** this is the *original* MVC mechanism (the model broadcasts changes to views/controllers) — see [web-presentation-patterns](../../web-presentation-patterns/references/web-mvc-is-not-mvc.md). In server-side PHP we rarely use live in-memory observers for UI; instead we use **domain events** ([domain-events](../domain-events/SKILL.md)) to achieve the same decoupling across a request or asynchronously. Treat Observer as the conceptual ancestor of the domain-event pattern.
- **Watch:** an in-process Observer list inside a domain entity is fine for pure domain reactions, but cross-boundary side effects belong in the domain-event publisher, not a registered listener inside the entity.

## How these relate to the surrounding skills
- **Strategy / State / Observer variants are Ports** wired at the composition root ([dependency-injection](../dependency-injection/SKILL.md)).
- **Composite / State** often pair with [Special Case](special-case.md) (a degenerate node implements the same interface).
- **Observer** is realized as **domain events** ([domain-events](../domain-events/SKILL.md)); the live-observer form is the original MVC idea ([web-presentation-patterns](../../web-presentation-patterns/references/web-mvc-is-not-mvc.md)).
- **Builder / Composite** are creation/structure concerns kept in the domain as plain PHP.
