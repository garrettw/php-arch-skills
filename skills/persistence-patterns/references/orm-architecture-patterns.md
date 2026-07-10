# Object-Relational Architecture Patterns

Here we shift from how the ORM behaves *internally* (the [behavioral & structural patterns](orm-patterns.md)) to the **architecture that surrounds the ORM** — the seams where your application meets persistence. These are patterns you engage with *architecturally* (unlike the ORM-internal ones), but still mostly by configuring the ORM and modeling well, not by building infrastructure by hand.

A broader lesson runs through this group: **the abstractions that endure are the ones that model the business rather than the technology.** Repositories and Value Objects remain highly relevant because they express business concepts. Metadata formats and registries, by contrast, have shifted over time because they primarily addressed implementation concerns.

## Metadata Mapping

Separate mapping information from mapping logic, so the ORM does not need special code for every entity. Instead of teaching the ORM each entity individually, you teach it a *mapping language* it can interpret.

- **Why it matters:** metadata makes ORMs extensible, configurable, and reusable.
- **The danger:** overengineering. Some ORMs accumulated dozens of mapping options until *understanding the metadata* became harder than understanding the application itself. That is a tooling issue, not a flaw in the pattern — the lesson is to keep the mapping language expressive but learnable.
- **PHP reality:** Doctrine's mapping (attributes/annotations/XML/YAML) and Eloquent's casts/attributes are both Metadata Mapping in practice.

## Query Object

Encapsulates a query into a reusable, composable object. This is the **query builder** portion of most ORMs. Note that a query builder can be used *without* a full ORM attached to it.

- **The criticism:** readability. Very complex fluent builders sometimes become harder to understand than the SQL they generate.
- **The sweet spot:** use Query Objects to express intent, but still allow raw SQL where it is clearer. Combine Query Objects with the **Specification Pattern**, **Read Models**, and **CQRS query handlers** to produce extremely expressive read layers.
- **PHP reality:** Doctrine's `QueryBuilder`/`DQL` and Laravel's query builder (with raw `DB::select` escape hatches) are both Query Objects.

## Repository

Probably the **most misunderstood** pattern. A Repository is **not** a generic CRUD wrapper.

- **Its purpose:** to provide the illusion that domain objects live in an in-memory collection, isolating the domain model from persistence.
- **The biggest misconception:** a `UserRepository` with `find()`, `findAll()`, `save()`, `delete()` is usually just a **DAO with a nicer name** — not a true Repository. A true Repository exposes operations *meaningful to the domain* that describe business concepts, not persistence mechanics.
- **Not every entity deserves its own Repository.** If an application is essentially CRUD, spraying repositories everywhere creates indirection without adding clarity. **The abstraction should earn its place.**
- **DDD refinement:** Repository becomes a foundational DDD pattern when tied closely to *aggregate roots*. That refinement makes the pattern stronger.
- **Related:** the [Gateway-vs-Repository distinction](../data-source-patterns.md#the-crucial-distinction-gateway-vs-repository) and repository-interface guidance in [SKILL.md](../SKILL.md).

## Value Object

Some concepts carry meaning but **no identity**. Examples: Money, Address, Date Range, Coordinate. Two identical values are interchangeable.

- This is one of the finest modeling ideas in software architecture.
- Developers frequently model *everything* as an entity — which is a mistake. **Most business concepts are actually values.**
- Treating them as values yields: immutability, simpler reasoning, fewer lifecycle concerns, and richer domain models. Value Objects are an indispensable modeling tool that encourages correctness, clarity, and immutability.
- **Related:** Value Objects belong to the domain, not the ORM — see the [`domain-modeling`](../domain-modeling/SKILL.md) skill for where they sit in the model.

### Money (a specialized Value Object)

Money is a specialized Value Object. Currencies, rounding, precision, exchange rates, and arithmetic all complicate the model.

- Money remains one of the most frequently modeled concepts, and one of the easiest to model incorrectly.
- It demonstrates why primitive types are inadequate: a `decimal` is not money, and a `float` certainly is not.
- A proper `Money` type protects invariants and makes illegal states hard to represent. It is the canonical example of why Value Objects matter.

## The patterns that endure

The five most important patterns from the entire object-relational section for today's architects would be:

- **Unit of Work** — [see ORM patterns](orm-patterns.md)
- **Identity Map** — [see ORM patterns](orm-patterns.md)
- **Repository** — above
- **Value Object** — above
- **Data Mapper** — [see data source spectrum](../data-source-patterns.md)

Those five continue to shape the design of modern persistence layers across languages and frameworks, even when developers rarely interact with them directly.
