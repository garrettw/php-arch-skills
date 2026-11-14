# CakePHP 5.x

CakePHP 5.x is the current supported major version. This file builds on `cakephp-3-4.md`; read that first for the shared Table/Entity baseline. The breaking changes from 4.x remove legacy magic and make clean-architecture boundaries easier to enforce.

- **Auth is now plugin-based.** The monolithic `AuthComponent` is gone. Use `cakephp/authentication` and `cakephp/authorization`. Authentication runs as middleware at the delivery boundary; pass only a minimal identity (e.g., a user ID and roles) inward, never the plugin's identity object. When a use case needs a permission check, put it behind a narrow authorization port the handler depends on (see `security-architecture`).
- **Commands replace Shells.** CLI entry points are now `Command` classes implementing `CommandInterface`. Keep them thin — they are driver adapters that delegate to Application Handlers.
- **No more dynamic properties.** `AllowDynamicProperties` was removed from framework classes. This enforces explicit composition over the old magic property patterns, which aligns with clean architecture.
- **Stricter types help boundaries.** Function parameters, return types, and properties now carry strict types. This makes DTOs and port interfaces more self-documenting and catches boundary violations earlier.
- **Collection is stricter.** `combine()` now throws on null values. This makes data transformations in adapters more predictable and reduces silent failures.
