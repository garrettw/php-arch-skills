# Yii 1.1

Yii 1.1 has an enormous legacy install base and is still maintained. It is the most framework-coupled of the major PHP frameworks.

- **`CActiveRecord` is everything.** It is the model, the repository, and the ORM all at once. Introduce a repository interface and move logic into Application Handlers or domain services.
- **`CComponent` is the base of everything.** It provides `getX()/setX()` and event behaviors. It is a service locator and a global property bag. Avoid extending it for domain logic.
- **`Yii::app()` is a global service locator.** Refactor away from it. Start by wrapping `Yii::app()->db` in a repository adapter.
- **`CActiveForm`, `CGridView`, and widgets.** They are driver adapters (UI layer). Do not put business logic in widget classes.
- **Behaviors and events.** They silently modify object behavior. Prefer explicit composition over magic method attachment.
