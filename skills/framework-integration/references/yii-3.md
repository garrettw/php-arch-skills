# Yii 3

Yii 3 is a set of PSR-based packages built around dependency injection. It is far less coupled than Yii 2 and does not push a default ORM.

- **DI by configuration.** Bind ports to adapters in the container definitions under `config/common/di/*.php` (environment-specific overrides go in their own folders). Inject through constructors; do not resolve the container from handlers or domain code.
- **Persistence is your choice.** Yii offers a first-party ActiveRecord implementation and an official adapter for Cycle ORM (a Data Mapper). Because the container and packages are PSR-based, Doctrine or any other ORM also works, with community or hand-written integration rather than official support. Choose per area as `persistence-patterns` describes: ActiveRecord for lean CRUD areas, Cycle ORM or a hand-written mapper where a rich domain model must stay free of persistence code. In either case, domain logic does not belong in ActiveRecord models.
- **Several Action styles.** Routes can target controller methods, single-action classes, closures, or PSR-15 middleware. Prefer single-action classes or PSR-15 handlers for use cases; avoid closures for anything beyond trivial routes (see `action-domain-responder`).
- **Less mature ecosystem.** Expect gaps in official packages. Build small boundaries before they become churn points.
