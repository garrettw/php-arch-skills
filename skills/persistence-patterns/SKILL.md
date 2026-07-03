---
name: persistence-patterns
description: Use this skill when implementing database persistence, managing repository interfaces, writing mappers, or splitting large repositories. Assumes the decision between a rich domain model and a lean ORM model has already been made by the domain-modeling skill.
---

# Persistence, Mappers & Repositories in PHP

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
4. **If Yes to any:** Introduce a `[Name]RepositoryInterface` in the Domain layer.
5. **If No to all:** Create only a concrete `[Name]Repository` class. An interface is not needed if there is only one implementation in the same context.

### 4. Splitting Large Repositories
If a repository is growing too large:
1. **Check Heuristics.** Does it implement 4+ interfaces, exceed 400 lines, or have 25+ methods?
2. **Analyze Return Types.** Do methods return raw DTOs instead of Domain models?
3. **Refactor.** If yes, split the repository or extract dedicated Query services.

## Boundaries

### Always Do
- Always use the hybrid approach: Full DDD for writes (to protect invariants) and lean ORM for reads (for performance and simplicity) when appropriate.
- Always use specific repository interfaces per bounded context rather than a single massive repository shared across contexts.

### Ask First
- Ask before introducing a repository interface if there is only one concrete implementation.

### Never Do
- Never create a massive global repository (e.g., `UserRepository`) that serves multiple bounded contexts.
