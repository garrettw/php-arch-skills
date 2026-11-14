# Mezzio

Mezzio is a PSR-15 middleware micro-framework built on Laminas components. It is the Laminas ecosystem's equivalent of Slim.

- **Middleware pipeline is the driver.** Each middleware is a driver adapter that can delegate to a Handler. Keep middleware thin; it should only extract data and pass it to the handler.
- **Request handlers are Actions, not Application Services.** A Mezzio handler implements `Psr\Http\Server\RequestHandlerInterface`, so it is an HTTP driver adapter (the Action in `action-domain-responder`). Keep it in the context's `Interface/Http` folder: extract validated input, call an Application handler, and hand the result to a Responder. The Application handler loads data, calls domain logic, saves state, and records events in the outbox.
- **Routing is explicit.** Define routes in configuration or pipeline. Do not scatter business logic across anonymous middleware closures.
- **PSR-7 request/response.** They are clean value objects. Accept validated DTOs in handlers, not raw `ServerRequest` objects.
- **Laminas components.** Use `laminas/laminas-di` for wiring, `laminas/laminas-db` for persistence as a driven adapter. Do not let framework components leak into domain code.
- **No enforced structure.** Like Slim, you must create a context-first structure yourself (`src/[BoundedContext]/{Domain,Application,Infrastructure,Interface}/`; see [directory-structure-templates.md](../../application-layer/references/directory-structure-templates.md)).
