---
name: application-layer
description: Use this skill when writing controllers, structuring use cases, or implementing Command/Query/Handler (CQRS) patterns in PHP applications. Triggers on questions about thin controllers, handler responsibilities, or DTO usage.
---

# Application Layer & CQRS Patterns for PHP

## System Overview
The application layer orchestrates use cases while keeping framework adapters thin and domain rules explicit. It serves as the bridge between delivery mechanisms (HTTP, CLI, queues) and the core domain/infrastructure logic. This skill provides structural rules for controllers, handlers, and CQRS patterns.

## Numbered Workflows

### 1. Refactoring Controllers
If a controller contains business logic, database queries, or large DTO building:
1. **Extract the Core Logic.** Move the business rules into an application handler or domain object.
2. **Adapt the Edge.** Refactor the controller to only read inputs (route params, request body, etc.), delegate to the handler, and convert the result into an HTTP response.

### 2. Implementing CQRS Patterns
If creating a new use case that needs to be accessed via multiple delivery channels (e.g., Web and CLI):
1. **Determine the Operation Type.** Is it a write/state-transition (Command) or a read/projection (Query)?
2. **Create the Command/Query Object.** Define an intent-revealing object if the use case has multiple inputs or needs validation at the application boundary.
3. **Create the Handler.** Write a handler that executes the specific Command or Query.
4. **If the operation is extremely simple** (e.g., a single repository fetch), skip the Command/Handler pair and let the controller call the repository directly.

### 3. Creating DTOs and Command Objects
If deciding whether to introduce a new DTO or Command object:
1. **Evaluate Intent.** Does the object carry meaningful business intent across a boundary? If yes, create it.
2. **Evaluate Necessity.** Does the object merely mirror the request body or exist just to satisfy a generic pattern rule? If yes, skip it and use scalar arguments or arrays.

## Boundaries

### Always Do
- Always keep controllers thin; they should act purely as HTTP adapters.
- Always use the [Directory Structure Templates](references/directory-structure-templates.md) when implementing CQRS.
- Always let the handler load data, call domain objects/policies, save state, and publish events.

### Ask First
- Ask before introducing intermediate Command classes or Request DTOs if they do not add meaningful domain context.

### Never Do
- Never chain services together (e.g., Service A calls Service B calls Service C). A handler should own one complete business result.
- Never place framework-specific globals (like request or session singletons) inside Application Handlers or Domain objects.
- Never let controllers contain persistence orchestration or cross-context policies.
