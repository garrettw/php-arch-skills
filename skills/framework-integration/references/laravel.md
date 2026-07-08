# Laravel

Laravel emphasizes developer productivity with hidden magic. Clean architecture requires explicitly opting out of its shortcuts.

- **Avoid Facades.** Facades are global service locators. Refactor to constructor-injected dependencies.
- **Avoid Fat Eloquent Models.** Model classes are persistence-bound entities, not aggregates. Place business logic in Application Handlers or domain services, never in Eloquent models.
- **Avoid `$request->*` in domain.** Accept primitive inputs or DTOs in handlers. Convert the request in the controller.
- **Use the Service Container at the edge.** Register ports and concrete adapters in service providers. Never resolve the container from within a domain object.
- **Jobs and Listeners.** Treat them as Application-layer orchestrators or event subscribers, not as domain logic containers. Dispatch events for decoupling, not Jobs for synchronous business steps.
- **Routes.** Use explicit Route definitions with controller callbacks that delegate to Handlers. Avoid closures for use cases.
