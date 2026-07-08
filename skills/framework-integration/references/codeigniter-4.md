# CodeIgniter 4

CodeIgniter remains popular for shared-hosting and performance-sensitive deployments where simplicity matters more than magic.

- **Thin framework, you must enforce structure.** CodeIgniter gives you Controllers, Models, and Views but no architectural guidance. Define `Domain/`, `Application/`, and `Infrastructure/` directories yourself.
- **Models are not ActiveRecord by default.** CI4 Models are thin database wrappers. They can act as adapters, but do not let them become fat domain objects.
- **Avoid Controllers as Orchestrators.** Keep controllers thin. Delegate to Application Handlers.
- **Use the DI Container.** CI4 has built-in autowiring and a simple service container. Register ports at the `Config/App.php` level or in service providers.
- **Validation.** Use the Validation library at the boundary (controller), not inside domain logic.
