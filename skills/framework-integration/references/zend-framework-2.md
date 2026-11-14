# Zend Framework 2

ZF2 introduced a service manager and module system but still leans on service locators and ActiveRecord-like patterns.

- **Service Manager over DI.** The `ServiceManager` is a service locator, not a DI container. Register ports and concrete adapters in module configs. Prefer `factories` over `invokables` for anything with dependencies.
- **Module structure.** Use it to enforce bounded context boundaries at the module level. Each module can own its domain, application, and infrastructure subdirectories.
- **TableGateway is better than ZF1.** `Zend\Db\TableGateway\TableGateway` is closer to a driven adapter. Wrap it in a repository interface.
- **EventManager (`Zend\EventManager`).** It is in-process: use it only for best-effort reactions inside one context. Events that must not be lost, or that cross contexts, go to the outbox in the handler's transaction (see `domain-events` and `reliable-messaging`). Do not let event listeners grow into God classes.
