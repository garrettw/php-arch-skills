# CakePHP 3.x / 4.x

Both versions share the same Table/Entity architecture. CakePHP 3.x is EOL (Dec 2022). CakePHP 4.x support ends September 2026. Both are legacy maintenance targets.

- **Understand the pattern.** CakePHP Table/Entity pairs are equivalent to Repository/ActiveRecord. They work for simple CRUD but become anti-patterns when rich domain logic accumulates.
- **Avoid Table fatness.** `Table` classes should behave as driven adapters, not as aggregate roots. Use them as thin persistence implementations behind a repository interface.
- **Avoid Components as Service Locators.** Components are convenient but hide dependencies. Extract logic into injectable services when complexity grows.
- **RequestAction and dispatching.** Treat them as driver adapter entry points. Do not use them to call business logic directly from views.
- **ClassRegistry (3.x) / `TableLocator` (4.x).** Both are global locators for persistence adapters. Refactor by injecting repository interfaces instead of reaching into the locator from controllers or handlers.
- **`beforeSave`, `afterFind` callbacks.** These are ORM lifecycle hooks. Move invariants into domain objects or handlers; keep callbacks thin.
