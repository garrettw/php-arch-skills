# Slim

Slim's minimalism is a feature. The framework will not force you into bad habits unless you let it.

- **Explicit over implicit.** Define route callbacks that delegate to Handlers. Avoid anonymous closures for use cases.
- **PSR-7 request/response.** They are clean value objects. Accept validated DTOs in handlers, not raw server request objects.
- **No enforced structure.** This is the biggest risk. Create `src/Domain`, `src/Application`, `src/Infrastructure` directories yourself. Do not let `routes.php` grow into a business-logic dump.
