# CodeIgniter 3

CodeIgniter 3 is still widely used on shared hosting. It is extremely thin, which means you must enforce all architecture boundaries manually.

- **No DI container.** CI3 has no built-in dependency injection. Use the `Loader` class sparingly; it is a service locator. Introduce a simple factory or manual composition root.
- **Models are thin wrappers.** CI3 Models are not ActiveRecord by default. They are database wrappers. Use them as adapters, not as domain objects.
- **Controllers are the only orchestration point.** By default, controllers contain all logic. Extract use cases into Application Handlers immediately.
- **`$this->load` everywhere.** The Loader class loads models, libraries, and helpers globally. Refactor to constructor or method injection.
- **HMVC (Modular Extensions).** If the project uses HMVC, each module can become a bounded context. Enforce that modules communicate via interfaces, not direct calls.
- **No built-in events.** Use a simple event dispatcher library or implement the record-then-publish pattern manually in handlers.
