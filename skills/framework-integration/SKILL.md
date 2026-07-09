---
name: framework-integration
description: Use this skill when applying clean-architecture, DDD, or hexagonal patterns inside a specific PHP framework. Provides framework-specific port mapping, adapter patterns, and pitfalls to avoid for Laravel, Symfony, Slim, CakePHP, CodeIgniter, Yii, Laminas, Mezzio, Zend Framework, and legacy/maintenance-mode frameworks. Triggers when a framework is named or when clean-architecture principles need framework-specific wiring guidance.
---

# Framework Integration for Clean Architecture & DDD

This skill assumes familiarity with the framework-agnostic skills (`domain-modeling`, `application-layer`, `infrastructure-boundaries`, etc.). It provides framework-specific mappings for ports, adapters, DI wiring, and common anti-patterns.

Note that not every feature needs full DDD wiring. The [`domain-modeling`](../domain-modeling/SKILL.md) skill defines lighter defaults — a **Transaction Script** (one procedure per request) or a **Table Module** (one class per table operating on a recordset) — for simple or data-centric features. This skill still applies to those patterns: keep the framework controller thin and delegate to the script/module, and keep the framework ORM in an adapter rather than treating it as the domain. Reach for the full DDD port/adapter wiring below only when the feature's rules justify it.

## When to Apply This Guidance
Use this skill when the project already decided which framework to use and needs to apply clean architecture patterns *within* that framework. Do not use it to choose a framework.

## Available References

| Category | Framework          | Reference File                   |
|----------|--------------------|----------------------------------|
| Modern   | Laravel            | `references/laravel.md`          |
| Modern   | Symfony            | `references/symfony.md`          |
| Modern   | Slim               | `references/slim.md`             |
| Modern   | CakePHP 5.x        | `references/cakephp-5.md`        |
| Modern   | CodeIgniter 4      | `references/codeigniter-4.md`    |
| Modern   | Yii 3              | `references/yii-3.md`            |
| Modern   | Laminas            | `references/laminas.md`          |
| Modern   | Mezzio             | `references/mezzio.md`           |
| Legacy   | Zend Framework 1   | `references/zend-framework-1.md` |
| Legacy   | Zend Framework 2   | `references/zend-framework-2.md` |
| Legacy   | CakePHP 2.x        | `references/cakephp-2.md`        |
| Legacy   | CakePHP 3.x/4.x    | `references/cakephp-3-4.md`      |
| Legacy   | Yii 1.1            | `references/yii-1.md`            |
| Legacy   | Yii 2              | `references/yii-2.md`            |
| Legacy   | CodeIgniter 3      | `references/codeigniter-3.md`    |

Read the relevant reference file for framework-specific anti-patterns and guidance. The mapping table below applies across all frameworks.

## Port-to-Framework Mapping

| Concept | Laravel | Symfony | Slim | CakePHP 3.x/4.x | CakePHP 5.x | CodeIgniter 4 | Yii 2 | Yii 3 | Laminas | Mezzio |
|---------|---------|---------|------|-----------------|-------------|---------------|-------|-------|--------|--------|
| Driver Entry | Route -> Controller -> Handler | Controller -> Handler | Route -> Callable -> Handler | Controller/Table -> Handler | Controller/Command -> Handler | Controller -> Handler | Controller -> Handler | Route -> Handler | Controller -> Handler | Middleware -> Handler |
| Repository Port | Interface in `domain/` | Interface in `domain/` | Interface in `domain/` | Interface in `domain/` | Interface in `domain/` | Interface in `domain/` | Interface in `domain/` | Interface in `domain/` | Interface in `domain/` | Interface in `domain/` |
| Repository Adapter | Eloquent in `infrastructure/persistence` | Doctrine in `infrastructure/persistence` | PDO/ORM in `infrastructure/persistence` | Table in `infrastructure/persistence` | Table in `infrastructure/persistence` | Model in `infrastructure/persistence` | ActiveRecord in `infrastructure/persistence` | ActiveRecord in `infrastructure/persistence` | TableGateway in `infrastructure/persistence` | Laminas Db in `infrastructure/persistence` |
| DI Composition Root | `AppServiceProvider` / `singleton()` | `services.yaml` / `config/services.php` | Custom factory or foreach bind | `src/Application.php` or `bootstrap.php` | `src/Application.php` or `bootstrap.php` | `Config/App.php` / `Controllers` | `config/web.php` / DI container | `config/container.php` | Module config + DI | `config/autoload/dependencies.config.php` |
| Event Bus | Laravel Events (native) | Symfony EventDispatcher | Third-party (e.g., `opis/event`) | CakePHP EventManager | CakePHP EventManager | Events class (manual) | Yii Event / yii2-event | Yii 3 Event | Laminas EventManager | Laminas EventManager |
| Domain Dispatcher | `Event::dispatch` in handler | Publish via `EventDispatcher` | Manual dispatch | `EventManager->dispatch` | `EventManager->dispatch` | Manual publish | Publish via event class | Publish via event class | Publish via `EventManager` | Publish via `EventManager` |

## Boundaries

### Always Do
- Always keep framework-specific objects (Request, Eloquent Model, AR Entity, Table) in driver/driven adapters.
- Always register adapter-to-port bindings in a single composition root.
- Always write domain logic in plain PHP classes with zero framework dependencies.
- Always convert framework request/response objects to domain-neutral DTOs at the boundary.

### Ask First
- Ask before using a framework's built-in "magic" (Facades, Service Locators, Gii, Behaviors) in a new codebase. Is there a cleaner explicit alternative?
- Ask before deciding to fight the framework's conventions if the project is small and short-lived. Sometimes the convention is the right trade-off. For simple or data-centric features, prefer a [Transaction Script](../domain-modeling/references/transaction-script.md) or [Table Module](../domain-modeling/references/table-module.md) over either full DDD wiring or dumping logic in the controller.

### Never Do
- Never import framework base classes, facades, or static service locators into domain or application code.
- Never use framework-specific exceptions or response objects as return types from handlers.
- Never treat a framework's default persistence model (Eloquent AR, Doctrine, Cake Table) as your aggregate root. It is an adapter, not the domain.
- Never assume that "the framework does it" is a reason to skip clean-architecture boundaries in a complex, long-lived application.
- Never put business logic directly in a framework controller, even for lighter patterns. Delegate to a Transaction Script or Table Module and keep the controller as a thin adapter.
