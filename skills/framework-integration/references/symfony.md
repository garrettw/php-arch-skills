# Symfony

Symfony's structure already aligns well with clean architecture if you avoid common misuses.

- **Prefer Entity Domain Objects.** Doctrine entities should be persistence-aware. Use setters as normal; keep invariants in Application Services or domain services, not in entity lifecycle callbacks.
- **Use the DI Container correctly.** Register ports -> bind concrete adapters in `services.yaml`. Autowiring reduces ceremony without breaking boundaries.
- **HttpFoundation.** The `Request` object is a clean transfer object. Pass only validated data to handlers as DTOs or command objects.
- **EventDispatcher.** Use it for domain events or infrastructure events. Do not use application events when a direct method call inside the handler is clearer.
- **HttpKernel / Controller.** Keep controllers thin. The controller's only job is to convert request -> handler input -> response.
