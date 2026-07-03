# Directory Structure Templates for CQRS

Use this default shape for complex backend use cases in PHP applications to implement the Command/Query/Handler pattern.

## Target Structure

```text
src/Application/[BoundedContext]/
  Command/
    [ActionName]Command.php
    Handler/
      [ActionName]Handler.php
  Query/
    [ReadOperation]Query.php
    Handler/
      [ReadOperation]Handler.php
```

Alternatively, Handlers can live in the same folder as their Command/Query object:

```text
src/Application/[BoundedContext]/
  Command/
    [ActionName]Command.php
    [ActionName]Handler.php
  Query/
    [ReadOperation]Query.php
    [ReadOperation]Handler.php
```

### Example

```text
src/Application/Bootcamps/
  Command/
    CompleteEnrollmentCommand.php
    Handler/
      CompleteEnrollmentHandler.php
  Query/
    GetLearnerScheduleQuery.php
    Handler/
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
