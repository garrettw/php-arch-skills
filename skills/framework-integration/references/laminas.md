# Laminas

Laminas is the community-driven continuation of Zend Framework. It is used in enterprise and legacy-migration contexts.

- **Components over framework.** Laminas is a component library first. Use individual components (DI, DB, EventManager) rather than the full MVC stack when building clean architecture.
- **Better DI than ZF2.** `laminas/laminas-di` and `laminas/laminas-servicemanager` support constructor injection and configuration-driven wiring. Register ports and bind adapters in module or global config.
- **TableGateway is still the persistence pattern.** `Laminas\Db\TableGateway\TableGateway` is a driven adapter. Wrap it in a repository interface.
- **EventManager.** Use it for domain or infrastructure events. Do not let listeners grow into God classes.
- **Migration from ZF2.** If migrating from Zend Framework 2, start by replacing `Zend\*` namespaces with `Laminas\*` and then introduce proper DI.
