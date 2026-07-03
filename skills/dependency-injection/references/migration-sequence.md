# Migration Sequence for Legacy DI Containers

If you are migrating a legacy PHP codebase that uses service locators, global singletons, or an outdated DI container (like Pimple) toward modern constructor injection, use this sequence.

## The Strategy: Touched-Area First

Do not attempt to rewrite the entire application's DI graph at once. Use the "touched-area" strategy: migrate the classes you are actively changing for product work.

## Step-by-Step Migration

1. **Identify the Target**: You are modifying a legacy class (e.g., `LegacyService`) that uses `Container::getInstance()->get('db')`.
2. **Write Characterization Tests**: Lock in the current behavior of `LegacyService` with a test.
3. **Extract Dependencies to Constructor**:
   - Find every internal `getInstance()` or locator call.
   - Add those dependencies as typed constructor properties.
   ```php
   // From:
   public function doWork() {
       $db = Container::getInstance()->get('db');
   }
   
   // To:
   public function __construct(private DatabaseConnection $db) {}
   ```
4. **Fix Callers (The Ripple Effect)**:
   - Identify all places that instantiate `LegacyService`.
   - Update them to use the container or pass the dependency.
   - If a caller is a framework controller, let the modern framework container autowire `LegacyService`.
   - If a caller is another legacy class, you may need to either migrate that caller as well, or temporarily use the service locator *only* at the instantiation site.
5. **Register in Modern Container**: Ensure `LegacyService` and its dependencies are configured in the modern DI container (e.g., CakePHP 4/5 DI, Laravel Service Container, Symfony DI).
6. **Verify**: Run the characterization tests.

## The "Shim" Pattern for Deep Legacy Code

Sometimes you cannot migrate the entire caller chain. If `LegacyService` is instantiated in 50 places that you cannot change right now, use a temporary shim:

```php
// Modernized Class
class ModernService {
    public function __construct(private DatabaseConnection $db) {}
}

// Temporary Legacy Shim
class LegacyService extends ModernService {
    public function __construct() {
        // Fallback for legacy callers that still use `new LegacyService()`
        $db = Container::getInstance()->get('db');
        parent::__construct($db);
    }
}
```
*Note: Mark the shim as `@deprecated` and log its usage if possible so you can eliminate the remaining calls over time.*
