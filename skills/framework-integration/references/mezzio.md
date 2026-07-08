# Mezzio

Mezzio is a PSR-15 middleware micro-framework built on Laminas components. It is the Laminas ecosystem's equivalent of Slim.

- **Middleware pipeline is the driver.** Each middleware is a driver adapter that can delegate to a Handler. Keep middleware thin; it should only extract data and pass it to the handler.
- **Handlers are Application Services.** A Mezzio handler implements `Psr\Http\Server\RequestHandlerInterface`. It is the application-layer orchestrator. Load data, call domain logic, save state, publish events.
- **Routing is explicit.** Define routes in configuration or pipeline. Do not scatter business logic across anonymous middleware closures.
- **PSR-7 request/response.** They are clean value objects. Accept validated DTOs in handlers, not raw `ServerRequest` objects.
- **Laminas components.** Use `laminas/laminas-di` for wiring, `laminas/laminas-db` for persistence as a driven adapter. Do not let framework components leak into domain code.
- **No enforced structure.** Like Slim, you must create `src/Domain`, `src/Application`, `src/Infrastructure` yourself.
