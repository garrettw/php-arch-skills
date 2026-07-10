---
name: domain-modeling
description: Use this skill when deciding whether to use rich domain objects vs lean ORM entities, choosing Transaction Script vs Table Module vs a Domain Model, identifying real business objects, or evaluating the cost-benefit of full DDD vs lean structure in a PHP application.
---

# Domain Modeling for PHP Apps

## When to Use (and When NOT to)
DDD adds ceremony that must justify its cost. Use DDD when the project has a complex business domain with many rules and will be maintained for years by a team. Skip DDD for prototyping, MVPs, solo-developer projects, or simple CRUD with few business rules.

| Use DDD For                                  | Use Simpler Patterns For                     |
|----------------------------------------------|----------------------------------------------|
| Complex business domain with many invariants | Simple CRUD, few business rules              |
| Long-lived system (years of maintenance)     | Prototype, MVP, throwaway code               |
| Team of multiple developers                  | Solo developer or tiny team (1-2)            |
| Multiple entry points (API, CLI, events)     | Single entry point, simple API               |
| Need to swap infrastructure (DB, broker)     | Fixed infrastructure, unlikely to change     |
| High test coverage required                  | Quick scripts, internal tools                |

**Start simple. Evolve complexity only when needed.** Most systems do not need full DDD. For libraries, packages, or SDKs that have no user-facing behavior, plain PSR-4 classes with clear names are usually sufficient. For simple CRUD, lean ORM entities are preferable to rich domain objects.

Note: the choices below (Transaction Script, Table Module, Domain Model) are about *how business rules are organized*. The **Application (or Service) Layer** sits **above** this spectrum — it defines the application's boundary and coordinates whichever domain-logic style you pick. See the `application-layer` skill for Service Layer definition and the overuse anti-pattern (`Controller → Service → Repository → Entity` with an anemic domain).

### You do not pick one style for the whole codebase
An application does **not** have to commit to a single domain-modeling style everywhere. Different areas of a system have different complexity, and the right pattern varies by area:

- A reporting/reporting-adjacent area may be best as a **Table Module** (set-oriented, data-centric).
- A simple admin or CRUD feature may be best as a **Transaction Script**.
- The core, rule-heavy part of the business (pricing, billing, entitlements) may warrant a rich **Domain Model**.

Choose the lightest pattern that fits *that area's* rules, and let each area evolve independently toward a heavier pattern as its rules demand. This is exactly the job of **bounded contexts** (see the `bounded-contexts` skill): a context is a natural boundary inside which one domain-logic style can be the default without forcing it on the rest of the system. Do not let a heavy pattern leak from a complex context into a simple one, and do not force a simple pattern onto a context that clearly needs rich behavior.

### Transaction Script: the simpler default
Before reaching for a rich Domain Model, consider a [Transaction Script](references/transaction-script.md). A Transaction Script is a single procedure per request that orchestrates the steps (validate → fetch → calculate → save) and talks to the database through a thin gateway. It is the lightest way to organize business logic and the right default for simple CRUD, linear request/response flows, and prototypes. Upgrade to a Domain Model only when the same rules start repeating across scripts and drifting out of sync.

### Table Module: the middle ground
When logic is shaped around *sets of rows in a table* rather than individual objects, a [Table Module](references/table-module.md) is the compromise between Transaction Script and a Domain Model. It is one class per table whose single instance operates over a recordset of all rows, bundling behavior with the data without paying the O/R-mapping cost of a Domain Model. Use it for table-centric logic (totals, summaries, batch updates, data-grid/report UIs). Move to a Domain Model only when rows need their own identity, relationships, or polymorphic behavior.

## System Overview
Use Domain-Driven Design (DDD) to make business rules explicit, not to add ceremony. Every backend abstraction should justify itself by reducing complexity, protecting an invariant, improving locality, or making tests clearer. This skill guides the decision process for introducing rich domain objects and structuring business operations.

## Numbered Workflows

### 1. Identifying the Real Business Object
If the user asks to model a new noun or entity:
1. **Challenge the initial noun.** Ask if the core concept is actually an operation, lifecycle, policy, or transaction that keeps several facts consistent.
2. **Reframe around events.** Instead of asking "Can this user do X?", ask "Can this operation happen?"
3. **If the object is just a data shell**, do not give it domain rules. See step 2.

### 2. Choosing Transaction Script vs a Rich Domain Model
If you are deciding how to organize a feature's business logic:
1. **Prefer Transaction Script by default.** If the feature is a linear, single-path procedure (read some data, validate, write a result) with few or no business rules, implement it as a Transaction Script (one procedure per request, thin DB gateway). Do not introduce a rich domain object.
2. **Watch for duplication.** If the same validation, calculation, or policy already appears in more than one script (or is about to), that behavior belongs on a shared domain object, not copied across scripts. Upgrade then.
3. **Consider a Table Module for row-set logic.** If the feature computes over many rows of one table (totals, summaries, batch updates) or backs a data-grid/report UI, use a Table Module: one class per table operating on a recordset, paired with a thin gateway. Move to a Domain Model once the rules become individualized per row (per-customer pricing, tax exemptions, regional rules, contract terms, subscriptions).
4. **Otherwise evaluate Domain Behavior.** Does the feature have meaningful invariants, business rules, state transitions, or cross-context policy that need object identity and relationships?
2. **Evaluate Testability.** Would the core rule be easier to test in-memory without booting the framework or database?
3. **If Yes to either:** Propose building an active domain object. Note that this will require separating persistence into mappers and repositories (see the `persistence-patterns` skill).
4. **If No to both:** Propose using lean ORM entities directly. Do not build a separate pure PHP domain object.

### 3. Structuring Services and Handlers
If you are writing an application service or handler:
1. **Load data:** Retrieve required state from repositories or adapters.
2. **Decide:** Pass the data to an active domain object to make the business decision.
3. **Persist:** Save the result via the repository.

### 4. Defining Aggregate Boundaries
If deciding whether related entities belong in the same aggregate:
1. **Transaction Consistency.** Must they be consistent together in a single transaction? If yes, place them in the same aggregate.
2. **Reference by ID Only.** If they would be referenced directly (not by ID), they belong in the same aggregate.
3. **Split Large Aggregates.** If an aggregate contains more than 10 entities, split it into smaller aggregates and use domain events for eventual consistency.
4. **Rule:** One aggregate per transaction. Cross-aggregate consistency is handled via domain events, not distributed transactions.

## Boundaries

### Always Do
- Always evaluate whether the abstraction justifies its cost before introducing a mapper or domain interface.
- Always ensure active domain objects can be tested with plain PHP, without booting the framework.
- Always read the [Decision Flowchart](references/decision-flowchart.md) before deciding on a structure.
- Always consider a [Transaction Script](references/transaction-script.md) before reaching for a rich Domain Model; it is the lighter default for simple, linear procedures.
- Always consider a [Table Module](references/table-module.md) when logic operates over sets of rows in a single table, before paying for a full Domain Model's O/R mapping.
- Always use "unique identity that persists" → Entity, "defined only by attributes" → Value Object when deciding between the two.

### Ask First
- Ask before extracting intermediate Command classes, Request DTOs, or "domain services" unless they carry meaningful domain data.
- Ask before splitting a lean CRUD entity into a full DDD aggregate.

### Never Do
- Never chain services together (e.g., Service A calls Service B calls Service C). Each handler should own one complete business result.
- Never place core business rules in framework controllers, ORM callbacks, or provider adapters.
- Never build a repository per entity. Use one repository per aggregate root to preserve consistency boundaries.
