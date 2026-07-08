# Yii 2

Yii 2 is highly coupled to ActiveRecord by default. Migrating toward clean architecture requires starting with repository abstractions.

- **ActiveRecord is the default.** Treat AR models as infrastructure-bound entities. Move invariants into Form Models or Application Services.
- **Gii generates coupled code.** Regenerate carefully. Treat Gii output as a starting scaffold, not the target structure.
- **Behaviors hide coupling.** They attach cross-cutting concerns silently. Prefer explicit constructor injection or trait composition for transparency.
- **Components are service locators.** `Yii::$app` is a global container. Refactor away from it gradually using the `architecture-migration` skill.
