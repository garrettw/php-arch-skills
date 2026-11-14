# CodeIgniter 4

CodeIgniter remains popular for shared-hosting and performance-sensitive deployments where simplicity matters more than magic.

- **Thin framework, you must enforce structure.** CodeIgniter gives you Controllers, Models, and Views but no architectural guidance. Create a context-first structure yourself (`src/[BoundedContext]/{Domain,Application,Infrastructure,Interface}/`; see [directory-structure-templates.md](../../application-layer/references/directory-structure-templates.md)).
- **Models are not ActiveRecord by default.** CI4 Models are thin database wrappers. They can act as adapters, but do not let them become fat domain objects.
- **Avoid Controllers as Orchestrators.** Keep controllers thin. Delegate to Application Handlers.
- **Wire ports in `app/Config/Services.php`.** CI4 has no autowiring container; its Services class is a set of hand-written factory methods. Add one method per port that builds its adapter, and inject the result through constructors. Avoid calling `service()` or `model()` from inside handlers or domain code; that turns Services into a service locator.
- **Validation.** Use the Validation library at the boundary (controller), not inside domain logic.
