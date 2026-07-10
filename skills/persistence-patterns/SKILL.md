---
name: persistence-patterns
description: Use this skill when implementing database persistence, choosing a data source pattern, managing repository interfaces, writing mappers, or splitting large repositories. Assumes the decision between a rich domain model and a lean ORM model has already been made by the domain-modeling skill.
---

# Persistence, Mappers & Repositories in PHP

## When to Use Rich Persistence vs. Simple ORM
Rich persistence (Mapper + Repository) is only worth the ceremony when the application has complex invariants or needs to swap storage. Use simple ORM for straightforward CRUD.

| Use Rich Persistence (Mapper + Repository) | Use Simple ORM (Entity + Query Builder)  |
|--------------------------------------------|------------------------------------------|
| Complex business invariants to protect     | Single-table CRUD with few rules         |
| Need to swap storage implementation        | Fixed database schema                    |
| Tests must run without DB for domain logic | Schema changes hand-in-hand with feature |
| Long-lived system, evolving requirements   | Prototype, MVP, throwaway code           |

A library or package that simply exposes data objects should not include mappers or repository interfaces.

## Data Source Spectrum & Gateway vs Repository
Persistence structure is a separate axis from domain-logic style. There is a progression of increasing separation between business logic and storage: **Table Data Gateway → Row Data Gateway → Active Record → Data Mapper** (see [data-source-patterns.md](references/data-source-patterns.md) for the spectrum, philosophy, and Gateway-vs-Repository distinction). The lightest end ([Table Data Gateway](references/table-data-gateway.md)) is one class per table that fetches/persists rows and nothing more; the heavy end ([Data Mapper](references/data-mapper.md)) fully isolates domain objects from the database. **Active Record** sits in the middle: one object per row that also carries its business behavior and knows how to persist itself. It is outstanding for CRUD-heavy apps with modest rules and small teams that value rapid iteration, but becomes a liability as the domain grows richer and more interconnected — the class drifts from representing the business to representing the ORM. Recognize when it has outgrown its limits and move persistence/mapping out into a Data Mapper. At the heavy end, **Data Mapper** fully isolates domain objects from the database (only the Mapper bridges the two); it is the preferred persistence model for complex enterprise software (DDD, Hexagonal/Onion/Clean, CQRS writes) because it keeps database concerns out of the domain. Its value is not easy database swaps — it is persistence ignorance for a rich domain model — and the extra indirection is only justified when a rich domain would otherwise be compromised by persistence concerns.

The distinction between a **Gateway** and a **Repository** is subtle but critical:
- A **Gateway thinks in tables**: "How do I fetch rows from this table?" It returns rows/recordsets and uses persistence language.
- A **Repository thinks in business concepts**: "How do I retrieve domain objects?" It returns domain objects in the ubiquitous language.

A Table Data Gateway is **not** a Repository, even when teams label it `CustomerRepository`. Modern equivalents (query services, table repositories, DAOs, SQL abstraction classes) are all gateways if they think in tables. A gateway accumulates procedural query methods (`findEligiblePremiumCustomers()`) as behavior grows; that is the signal you have started building a Repository without realizing it — move the behavior into the domain and let a true Repository return domain objects.

## Object-Relational Patterns (ORM Features, not to Implement)

Beyond the data-source spectrum, ORMs implement a second family of Object-Relational patterns internally — behavioral (Unit of Work, Identity Map, Lazy Loading) and structural (Identity Field, Foreign Key Mapping, Association Table Mapping, Dependent Mapping, Embedded Value). Architects should understand these and their tradeoffs as **ORM features**, and make the architectural decision at the level of *ORM selection and configuration* rather than reimplementing them. See [orm-patterns.md](references/orm-patterns.md). The patterns that surround the ORM rather than live inside it — Metadata Mapping, Query Object, Repository, Value Object — are engaged with architecturally; see [orm-architecture-patterns.md](references/orm-architecture-patterns.md).

## System Overview
A clear persistence strategy separates the domain model from database storage mechanisms when necessary, while avoiding boilerplate when it is not. This skill helps you decide when to use a full DDD persistence structure (Domain → Mapper → Repository → ORM) versus a lean ORM structure.

## Numbered Workflows

### 1. Implementing CQRS Read vs Write Persistence
If implementing persistence for a use case:
1. **For Writes (Commands):** If the feature uses a rich domain model, use a Mapper and a Repository to load the aggregate, mutate it, and save it. This protects invariants.
2. **For Reads (Queries):** Bypass mappers entirely. Use the ORM, Query Builder, or raw SQL to fetch data directly into lean DTOs or arrays for performance and simplicity.
3. **For Lean Features:** If the `domain-modeling` skill determined the feature is just CRUD, use the ORM directly for both reads and writes.

### 2. Implementing Mappers
If you must map between a rich domain object and an ORM entity:
1. **Isolate Mapping Logic.** Create a dedicated Mapper class. Do not put mapping logic in the domain object or the repository.
2. **Accept the Cost.** Acknowledge that adding a database column now requires updating the domain, ORM entity, mapper, and tests. Only map objects that have real business logic justifying this cost.

### 3. Introducing Repository Interfaces
If creating a new repository:
1. **Check for Multiple Implementations.** Do you expect multiple real implementations (e.g., SQL and API)?
2. **Check for Boundary Inversion.** Is the implementation provided by a third-party package?
3. **Check for Decorators.** Do you need caching or logging around the calls?
4. **If Yes to any:** Introduce a `[Name]RepositoryInterface` in the Domain layer as a driven port.
5. **If No to all:** Create only a concrete `[Name]Repository` class. An interface is not needed if there is only one implementation in the same context.
6. **Per Aggregate.** Design repositories per aggregate root, not per database table. A single aggregate should be loaded and saved through one repository.

### 4. Splitting Large Repositories
If a repository is growing too large:
1. **Check Heuristics.** Does it implement 4+ interfaces, exceed 400 lines, or have 25+ methods?
2. **Analyze Return Types.** Do methods return raw DTOs instead of Domain models?
3. **Refactor.** If yes, split the repository or extract dedicated Query services.

## Boundaries

### Always Do
- Always use the hybrid approach: Full DDD for writes (to protect invariants) and lean ORM for reads (for performance and simplicity) when appropriate.
- Always use specific repository interfaces per bounded context rather than a single massive repository shared across contexts.
- Always distinguish a Gateway (thinks in tables, returns rows) from a Repository (thinks in business concepts, returns domain objects). A [Table Data Gateway](references/table-data-gateway.md) is the right tool at the lean/CRUD end of the spectrum.

### Ask First
- Ask before introducing a repository interface if there is only one concrete implementation.
- Ask whether a class named "Repository" is actually a Gateway in disguise (returns rows, uses persistence language). If so, call it what it is.

### Never Do
- Never create a massive global repository (e.g., `UserRepository`) that serves multiple bounded contexts.
- Never let a Table Data Gateway accumulate business-flavored query methods (`findEligiblePremiumCustomers()`); that is a Repository trying to form — move the behavior into the domain instead.
- Never reach for Row Data Gateway in new code; it is a faded pattern that awkwardly mixes persistence, identity, and row state without business behavior. Use Table Data Gateway for raw row access, Active Record for row+behavior, or Data Mapper for full separation.
- Never introduce CQRS before the application has divergent read/write workloads. Start with simple read/write and evolve only when needed.
