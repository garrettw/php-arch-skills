# Symfony

Symfony's structure already aligns well with clean architecture if you avoid common misuses.

- **Choose the entity style per area, not per app.** Doctrine is a Data Mapper ORM, so an entity can be either a lean data holder or a true aggregate:
  - **Simple CRUD areas:** a lean entity with getters and setters is fine. The logic lives in a Transaction Script or a handler (see `domain-modeling`).
  - **Rule-heavy areas:** the entity _is_ the aggregate. It has private state, no public setters, and intention-revealing methods (`$order->ship($date)`) that enforce its invariants and record domain events. Handlers orchestrate and never decide.
  - **Mapping:** keep it out of the domain where you can, with XML or PHP mapping files under `Infrastructure/Persistence`. Attributes on the entity are an acceptable, conscious trade-off. Use Embeddables for value objects and custom DBAL types for IDs.
  - **Never put business rules in lifecycle callbacks or Doctrine listeners.** They run at flush time, outside the application layer’s control, and can’t be tested without the database.
  - **No ORM at all:** for an event-sourced or deliberately ORM-free area, use plain domain objects with a DBAL repository. That’s the Data Mapper from `persistence-patterns`, with a hand-written mapper.
- **Use the DI Container correctly.** Register ports -> bind concrete adapters in `services.yaml`. Autowiring reduces ceremony without breaking boundaries.
- **HttpFoundation.** The `Request` object is a clean transfer object. Pass only validated data to handlers as DTOs or command objects.
- **EventDispatcher and Messenger.** Use EventDispatcher only for in-process, best-effort reactions inside one context. Events that must not be lost, or that cross contexts, go to the outbox in the handler's transaction and are published through Messenger (see domain-events and reliable-messaging).
- **HttpKernel / Controller.** Keep controllers thin. The controller's only job is to convert request -> handler input -> response.
