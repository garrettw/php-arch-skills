# Transaction Script

Transaction Script is a pattern that organizes business logic. It is the natural simpler default when a rich Domain Model would be overkill.

## What It Is

A Transaction Script organizes business logic as a single procedure per request/transaction from the presentation layer. Each procedure handles one interaction (a "transaction" in the business sense, not necessarily a DB transaction) and makes calls directly to the database, usually through a thin database wrapper or the framework's query/ORM layer.

```
Client request
      │
      ▼
Transaction Script (one procedure per use case)
      │  direct calls
      ▼
Database / thin query wrapper
```

Fowler lists the main alternatives in PoEAA as:

- **Transaction Script** — procedure per request, logic calls the DB directly.
- **Domain Model** — behavior lives on rich objects that protect their own invariants.
- **Table Module** — one class per database table, with one instance handling all rows of that table.

Transaction Script is the lightest of these and the one to reach for by default.

## When to Use Transaction Script

Use a Transaction Script when the feature is a straightforward procedure: pull some data, do a little validation or calculation, and write a result.

| Use Transaction Script For                                  | Reach for a Domain Model Instead                          |
|-------------------------------------------------------------|-----------------------------------------------------------|
| Simple CRUD and request/response procedures                 | Complex business domain with many invariants              |
| Few business rules, mostly read-then-write steps            | Complex state transitions or cross-entity policies        |
| Prototype, MVP, throwaway code                              | Long-lived system maintained by a team for years          |
| Single, linear path per request (validate → save → return)  | Behavior that changes based on the object's own state     |
| Report-style or one-off operations                          | Shared rules that must stay consistent across many flows  |

**Default to Transaction Script. Reach for a Domain Model only when the rules start repeating across scripts and drifting out of sync.** The trigger to upgrade is duplicated logic: when the same validation or calculation appears in several scripts and risks drifting, that behavior belongs on a domain object (see the main skill's "Deciding to Use a Rich Domain Model" step).

## Structure in PHP

A Transaction Script is typically a single public method on a small class (often an application service/handler, or a plain procedure in a controller for throwaway code), not a chain of services:

```php
final class RegisterUserScript
{
    public function __construct(
        private readonly PdoUserGateway $users,
        private readonly PasswordHasher $hasher,
    ) {}

    public function execute(RegisterUserRequest $request): UserId
    {
        if ($this->users->existsWithEmail($request->email)) {
            throw new EmailAlreadyTaken($request->email);
        }

        $id = UserId::generate();
        $this->users->insert(new UserRow(
            $id,
            $request->email,
            $this->hasher->hash($request->email, $request->password),
        ));

        return $id;
    }
}
```

Notes:
- Each request gets its own script; common subtasks can be extracted into private helper methods or a shared gateway.
- The script talks to the database through a thin gateway/query object, not raw SQL scattered through the procedure.
- Business rules that are simple and local to this one script can live inside the script. Rules that recur across scripts should be lifted into a domain object or policy.

## Relationship to the Rest of This Skill Set

- Transaction Script is the "simpler pattern" column of the main skill's DDD comparison table.
- It is compatible with thin controllers (see `application-layer`): the controller adapts the HTTP edge, the script owns the procedure.
- When a script's logic grows and starts duplicating across scripts, refactor the shared rule into an active domain object and keep the script as the orchestrator (the "Deciding to Use a Rich Domain Model" step).
- Transaction Script does not require DTOs, mappers, or repositories. Add those only when the persistence structure diverges from the domain language (see `persistence-patterns`).
