# CakePHP 2.x

CakePHP 2.x is still maintained in many legacy apps despite being EOL. Its patterns are ActiveRecord-ish but with heavy framework coupling.

- **ClassRegistry is a service locator.** It loads models and makes them globally available. Refactor by injecting repository interfaces instead of calling `ClassRegistry::init()`.
- **Models as ActiveRecord.** `AppModel` and its subclasses know about the database. Treat them as infrastructure adapters behind a repository interface.
- **App::uses / App::import.** Autoloading is implicit and global. Move to PSR-4 if possible; if not, keep domain classes completely independent of the autoloader.
- **Components and Helpers as service locators.** They pull dependencies from the controller or request. Extract logic into plain services and inject them.
- **`beforeSave`, `afterFind` callbacks.** These are ORM lifecycle hooks. Move invariants into domain objects or handlers; keep callbacks thin.
