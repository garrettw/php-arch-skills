# Zend Framework 1

ZF1 is legacy but still maintained in many enterprise codebases. It is heavily coupled to service locators and lacks a modern DI container.

- **Service Locator everywhere.** `Zend_Registry`, `Zend_Application`, and `Zend_Db_Table` are global singletons. Refactor by wrapping them in adapter interfaces and injecting those adapters into handlers.
- **No real DI.** Use the `architecture-migration` skill to introduce constructor injection gradually. Start at the controller boundary.
- **Controllers are orchestrators.** They contain business logic and DB calls. Extract the logic into Application Handlers first.
- **`Zend_Db_Table` is a gateway, not a repository.** It couples table structure to business objects. Introduce a repository interface and map `Zend_Db_Table_Row` to domain objects via mappers.
- **Front Controller plugin chain.** It does everything (auth, ACL, routing, dispatch). Treat plugins as driver adapters or AOP concerns, not as domain logic containers.
