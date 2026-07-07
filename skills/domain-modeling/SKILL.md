---
name: domain-modeling
description: Use this skill when deciding whether to use rich domain objects vs lean ORM entities, identifying real business objects, or evaluating the cost-benefit of full DDD vs lean structure in a PHP application.
---

# Domain Modeling for PHP Apps

## System Overview
Use Domain-Driven Design (DDD) to make business rules explicit, not to add ceremony. Every backend abstraction should justify itself by reducing complexity, protecting an invariant, improving locality, or making tests clearer. This skill guides the decision process for introducing rich domain objects and structuring business operations.

## Numbered Workflows

### 1. Identifying the Real Business Object
If the user asks to model a new noun or entity:
1. **Challenge the initial noun.** Ask if the core concept is actually an operation, lifecycle, policy, or transaction that keeps several facts consistent.
2. **Reframe around events.** Instead of asking "Can this user do X?", ask "Can this operation happen?"
3. **If the object is just a data shell**, do not give it domain rules. See step 2.

### 2. Deciding to Use a Rich Domain Model
If you are deciding whether a feature warrants a rich domain object:
1. **Evaluate Domain Behavior.** Does the feature have meaningful invariants, business rules, state transitions, or cross-context policy?
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
- Always use "unique identity that persists" → Entity, "defined only by attributes" → Value Object when deciding between the two.

### Ask First
- Ask before extracting intermediate Command classes, Request DTOs, or "domain services" unless they carry meaningful domain data.
- Ask before splitting a lean CRUD entity into a full DDD aggregate.

### Never Do
- Never chain services together (e.g., Service A calls Service B calls Service C). Each handler should own one complete business result.
- Never place core business rules in framework controllers, ORM callbacks, or provider adapters.
- Never build a repository per entity. Use one repository per aggregate root to preserve consistency boundaries.
