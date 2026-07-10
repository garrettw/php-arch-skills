# Object-Relational Behavioral & Structural Patterns

These patterns are **not patterns you should have to implement** if you use an ORM. Modern ORMs already implement them internally. The value here is *literacy*: knowing what these patterns are, what tradeoffs they carry, and how your ORM exposes them — so you can select an ORM whose persistence model aligns with your application's complexity and domain model, and use its features deliberately instead of fighting them.

> Architectural decisions about these patterns should focus on **ORM selection and configuration**, not reimplementation. If you find yourself hand-rolling a Unit of Work or an Identity Map, you have almost certainly chosen the wrong tool (or the wrong layer for the job).

They split into two groups:

- **Behavioral patterns** — how objects and the database relate *at runtime* (tracking changes, identity, on-demand loading).
- **Structural patterns** — how *object structure* maps onto *relational structure* (keys, relationships, value objects).

## Behavioral Patterns

### Unit of Work
Tracks all objects affected by a business transaction and orders the writes (and the single transaction/flush) for you.

- **ORM feature:** Doctrine's `UnitOfWork` computes the change set and flushes in one transaction; Eloquent does per-model dirty tracking but relies on you wrapping writes in a DB transaction.
- **Tradeoff:** centralizes write ordering and transactional boundaries (good), but couples "what changed" to the ORM's identity tracking. The architectural decision is whether your ORM's unit-of-work semantics match your transaction boundaries — not whether to build your own.

### Identity Map
Guarantees that a row fetched twice within a session returns the *same* object instance, so in-memory changes stay consistent.

- **ORM feature:** Doctrine's `EntityManager` is an Identity Map by design (same PK → same object). Eloquent maintains a per-request identity cache keyed by class+PK.
- **Tradeoff:** prevents duplicate/conflicting objects, but the map is *scoped to the session/EntityManager*. Holding a long-lived EntityManager (or request-scoped cache) silently grows memory and can serve stale state across requests — a common source of subtle bugs. Know the scope your ORM uses.

### Lazy Loading
Defers loading a related object until it is actually accessed.

- **ORM feature:** Doctrine uses proxy objects; Eloquent loads relationships on first access (and offers explicit `with()` eager loading).
- **Tradeoff:** less work up front, but runaway lazy loading causes the classic **N+1 query problem**. The skill is choosing *what to eager-load* (`with()`, `fetch joins`) versus what to leave lazy. This is a configuration/usage decision, not a custom implementation.

## Structural Patterns

### Identity Field
An object carries a field that maps to the database primary key, so the object can be saved and re-fetched without a separate handle.

- **ORM feature:** the entity's `@Id` / primary key in any ORM.
- **Tradeoff:** trivial in an ORM, but the decision that matters is *identity strategy* (auto-increment vs UUID vs sequence) and whether domain objects should expose the surrogate key or a domain-natural identity.

### Foreign Key Mapping
An object reference to another object is stored as a foreign key in the referencing table.

- **ORM feature:** a mapped association whose column holds the related PK.
- **Tradeoff:** choose one-to-many vs many-to-one ownership carefully — it determines which table carries the FK and therefore the direction of writes. Let the ORM own the column mapping; own the *relationship direction* yourself.

### Association Table Mapping
A many-to-many relationship is stored in a separate join (association) table.

- **ORM feature:** a `@ManyToMany` / `belongsToMany` with a pivot table.
- **Tradeoff:** if the association itself carries data (timestamps, roles, ordering), it is no longer a pure join table — it becomes its own entity with a Domain Model, not a structural mapping. Recognize that transition.

### Dependent Mapping
A dependent (child) object has no meaning without its owner and is deleted when the owner is deleted.

- **ORM feature:** cascade remove / `orphanRemoval` / `onDelete: CASCADE`.
- **Tradeoff:** convenient, but cascades (especially `orphanRemoval`) can wipe data faster than you expect; decide cascade rules at the aggregate boundary, not reflexively on every relationship.

### Embedded Value
A small value object is mapped to columns of the owning object's table rather than its own table.

- **ORM feature:** Doctrine `@Embeddable`/`@Embedded`; in Eloquent this is approximated with typed attributes/casts rather than a true value-object mapping.
- **Tradeoff:** keeps a cohesive concept (Money, Address) in one row without a join. The architectural point: model it as a value object in the domain and let the ORM flatten it — don't promote every small concept into its own entity/table just because the ORM makes entities easy.

## In This Skill Set

- These sit *on top of* the [Data Source spectrum](data-source-patterns.md): Unit of Work, Identity Map, and Lazy Loading are how an Active Record or Data Mapper ORM behaves at runtime; the structural patterns describe how either style maps object structure to tables.
- A [Data Mapper](data-mapper.md) ORM (e.g., Doctrine) tends to expose these patterns explicitly and richly; an [Active Record](active-record.md) ORM (e.g., Eloquent) hides much of it behind the model. Your choice of ORM is partly a choice of *how visible these patterns are to you*.
- Prefer configuring and using these features over reimplementing them. If a pattern here is missing from your ORM, that is a selection signal — not an invitation to build it by hand.
- For the patterns that surround the ORM rather than live inside it — Metadata Mapping, Query Object, Repository, Value Object — see [Object-Relational Architecture Patterns](orm-architecture-patterns.md).
