# Directory Structure Templates for CQRS

Use this default shape for complex backend use cases in PHP applications to implement the Command/Query/Handler pattern.

## Target Structure

```text
src/[BoundedContext]/
  Domain/            entities, value objects, domain events, repository ports
  Application/       Command/ and Query/ objects with their handlers; ports for outbound adapters
    Command/
      [ActionName]Command.php
      [ActionName]Handler.php
    Query/
      [ReadOperation]Query.php
      [ReadOperation]Handler.php
  Infrastructure/    driven adapters: persistence, gateways
  Interface/         driver adapters: HTTP actions/responders, console commands, message consumers
src/SharedKernel/    identity references, published versioned event contracts, a few stable shared value types — never utilities
src/Platform/        cross-context technical infrastructure with no domain model (outbox, transaction manager, message bus wiring)
```

Group by bounded context first, then by layer, so one capability's code stays together and the boundary is visible in the tree. A layer-first layout (`src/Domain/[Context]/`, `src/Application/[Context]/`) is acceptable for small applications with one or two contexts — choose one and use it consistently.

- **`src/SharedKernel/`** holds identity references, published versioned event contracts, and a small set of stable value types with no context-specific rules (e.g., `Money`, `DateRange`) that more than one context needs. Never utilities. See [bounded-contexts](../../bounded-contexts/SKILL.md), workflow 3, for what qualifies.
- **`src/Platform/`** holds cross-context technical infrastructure that has no domain model: the outbox, the transaction manager, message bus wiring.

### Example

```text
src/Bootcamps/
  Application/
    Command/
      CompleteEnrollmentCommand.php
      CompleteEnrollmentHandler.php
    Query/
      GetLearnerScheduleQuery.php
      GetLearnerScheduleHandler.php
```

## Reuse Across Delivery Channels

The same application operation should be reusable when the same business use case appears in multiple delivery channels. By structuring the application layer with Handlers, you decouple the business orchestration from the delivery mechanism.

- **CLI / Console Commands**: A CLI command should instantiate and call an application handler rather than duplicate logic that lives in a web controller.
- **API Endpoints**: An API endpoint adapts the JSON request into a Command, calls the Handler, and adapts the result back to a JSON response, but it reuses the exact same behavior as the web application.
- **Queue Jobs / Workers**: A queue job consumes an event payload, transforms it into a Command, and calls the exact same Handler used in synchronous flows.

## Framework Boundaries

Keep framework-specific details (like routing, request objects, response formats, and console output buffering) completely outside of the Application layer:

- Controllers own HTTP request/response.
- Commands/Shells own CLI input/output.
- ORM repositories own table queries and persistence.

Domain and Application code must receive explicit inputs (like scalar types or DTOs) and dependencies, rather than reading framework globals or container singletons.
