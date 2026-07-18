# PHP Architecture Skills for AI Agents

A collection of redistributable AI agent skills designed to help you and your AI assistant build clean, maintainable PHP applications.

## How to Use

**Install all the skills.** They are designed to inter-relate and cross-link heavily — a migration decision in one skill points to a boundary pattern in another, and an application-layer use case relies on concepts from the domain-modeling and persistence skills. Installing the full set gives your agent the complete mental model rather than a fragment of it:

`npx skills add garrettw/php-arch-skills`

If you must install selectively, you can add a single skill directly:

`npx skills add garrettw/php-arch-skills --skill skill-name`

**Install globally if you prefer.** Add a `-g` flag to install the skills machine-wide instead of into the current repo — useful when you want the same architectural guidance across every project you work on:

`npx skills add garrettw/php-arch-skills -g`

### Pairing with the Matt Pocock skills

These architecture skills pair well with Matt Pocock's agent-skills collection, especially the `grill-with-docs` skill (stress-tests a plan against your project's domain language and documented decisions, updating CONTEXT.md and ADRs inline as choices crystallize). Install his collection with:

`npx skills add mattpocock/skills`

If you want to use `grill-with-docs` (or any skill from that repo), you should also install and run the `setup-matt-pocock-skills` skill from that same repo — it wires up the issue tracker, triage labels, and domain-doc layout those skills expect.

### How these skills are invoked

These PHP architecture skills are designed to be **proactive and auto-triggered**, rather than run on command. Each skill's `description` is written with explicit trigger phrases (e.g., "Triggers on questions about thin controllers, handler responsibilities…"), so your agent's skill loader fires the right one automatically as you work — you describe the situation in plain language and the agent consults the relevant skill. You generally would not call these skills explicitly; installing the full set is what makes that automatic selection work.

This is different from a skill like `grill-with-docs`, which is an **explicit, on-demand** workflow you deliberately launch to stress-test a plan.

Because they operate at different moments, they complement each other: let the architecture skills keep the code clean as you go, and pull in `grill-with-docs` when you hit a design decision worth challenging against your project's documented language.

## When to Use These Skills

This collection helps in two distinct situations:

1. **Architecting a greenfield project.** When starting something new, the skills steer the agent to make deliberate boundary, layering, and persistence decisions up front — so the codebase begins clean rather than drifting into a tangle of massive controllers, ORM-coupled logic, and vendor SDKs wired straight into the flow.
2. **Analyzing and improving an established codebase.** Point the agent at an existing project to map its architecture, surface architectural problems and code smells, and plan improvements. The skills direct this toward **high-impact, low-effort** changes and — critically — treat refactoring as something to do *only when a trigger is encountered* (an active feature, a complex bug, an exposed endpoint), never as aesthetic cleanup of untouched code. The `architecture-migration` skill is the playbook for this incremental, touched-area approach.

In both cases the value is the same: the agent applies Domain-Driven Design (DDD), Command/Query Responsibility Segregation (CQRS), and proper inversion of control, and proactively raises questions about boundaries, invariants, and testability *before* writing code.

## The Skills

The collection includes 13 modular skills, each focused on a specific architectural layer or concept:
- **`domain-modeling`**: Guides the agent to identify real business objects, default to Transaction Script (or Table Module for row-set logic) for simple procedures, determine when to upgrade to rich domain models versus lean ORM entities, and protect business invariants using pure PHP.
- **`bounded-contexts`**: Teaches the agent to define system boundaries using business capabilities rather than code folders, establishing clear ownership and preventing tightly-coupled monoliths.
- **`application-layer`**: Defines the Application (or Service) Layer as the boundary above the domain-logic spectrum, instructing the agent to orchestrate use cases via the Command/Query/Handler pattern, keep framework controllers extremely thin, and avoid the anemic-domain overuse anti-pattern.
- **`distribution-patterns`**: Guides the agent to design remote interfaces as coarse-grained Remote Facades (REST/GraphQL/gRPC/CQRS commands) shaped around consumer needs rather than the internal object model, and to carry data across boundaries with DTOs — keeping ORM entities and domain objects off the wire.
- **`persistence-patterns`**: Guides the agent on how to persist data across architectural boundaries, choosing a data source pattern along the spectrum of business-logic/persistence separation, understanding the ORM-internal Object-Relational patterns (Unit of Work, Identity Map, Lazy Loading, Identity/Associated/Dependent Mapping, Embedded Value) as features rather than implementations, the architecture surrounding the ORM (Metadata Mapping, Query Object, Repository, Value Object), when to use mappers and repository interfaces, and how to apply CQRS at the database edge.
- **`infrastructure-boundaries`**: Teaches the agent to isolate third-party APIs, vendor SDKs, and external systems behind clean adapters, preventing vendor lock-in.
- **`domain-events`**: Shows the agent how to decouple side-effects and cross-context communication using the record-then-publish event pattern.
- **`event-sourcing`**: Guides the agent through evaluating whether Event Sourcing is appropriate, modeling event-sourced aggregates, building projections and snapshots, and choosing PHP libraries (Ecotone, Prooph, EventSauce, Patchlevel).
- **`testing-strategy`**: Forces the agent to write tests that verify behavior rather than internal implementations, spanning pure domain unit tests to database integration tests.
- **`architecture-migration`**: Provides the agent with a playbook for incrementally refactoring legacy code into a clean architecture using the Strangler Fig approach — vertical slices, temporary shims, and clearly-named transitional architecture — and only when a trigger point (active feature, complex bug, new endpoint) is encountered, never as aesthetic cleanup.
- **`dependency-injection`**: Ensures the agent relies on constructor injection over legacy service locators and correctly registers interfaces at the framework edge.
- **`framework-integration`**: Maps clean-architecture patterns to specific PHP frameworks, providing port-to-framework mappings, anti-patterns to avoid, and composition-root wiring guidance.
- **`action-domain-responder`**: Corrects the "Web MVC" misnomer (it's really Sun's Model 2, not Smalltalk MVC), reframes the framework controller as a thin Action-Domain-Responder, and keeps HTTP/framework concerns out of the domain.

## A Unifying Theme: Boundary-Protection Patterns

The distribution and supporting patterns share one job — **protecting a boundary** so internals don't leak across a seam. Each answers a specific question (*which seam am I crossing, and what must not leak through it?*):

| Pattern       | Boundary it protects                                      | Skill                       |
|---------------|-----------------------------------------------------------|-----------------------------|
| Remote Facade | Process / network (the *operation* contract)              | `distribution-patterns`     |
| DTO           | Representation (the *data shape* on the wire)             | `distribution-patterns`     |
| Gateway       | Infrastructure (talking to an external system)            | `infrastructure-boundaries` |
| Mapper        | Model (translating between representations)               | `infrastructure-boundaries` |
| Plugin        | Framework / extension (supplying variable behavior)       | `dependency-injection`      |
| Special Case  | Behavioral (the valid "nothing here" / exceptional state) | `domain-modeling`           |

These patterns have stayed relevant because they address enduring sources of complexity, not transient implementation details. They also form the **intellectual foundation of Ports & Adapters (Hexagonal) architecture**: a Gateway *is* an outbound Port+Adapter, a Remote Facade defines an inbound boundary, DTOs are the messages crossing those boundaries, and a Plugin is just a structured way to supply the Adapters. Hexagonal doesn't replace these ideas — it reframes and generalizes them. See [`distribution-patterns/references/boundary-protection-patterns.md`](skills/distribution-patterns/references/boundary-protection-patterns.md) for the full mapping.

## Sources

These skills distill established, framework-agnostic software-design literature — they are a curated synthesis for AI agents, not original research. The primary sources are:

- **Martin Fowler, *Patterns of Enterprise Application Architecture* (PoEAA / the "EAA Catalog")** — the foundation for the boundary-protection patterns (Remote Facade, DTO, Gateway, Mapper, Special Case, Plugin), the data-source spectrum (Table/Row Data Gateway, Active Record, Data Mapper), and the Object-Relational patterns (Unit of Work, Identity Map, etc.).
- **Martin Fowler's bliki & articles** — used for the modernization patterns: [Strangler Fig Application](https://martinfowler.com/bliki/StranglerFigApplication.html), [Patterns of Legacy Displacement](https://martinfowler.com/articles/patterns-legacy-displacement/) (Transitional Architecture, Event Interception, Legacy Mimic, Divert the Flow, Revert to Source, Feature Parity, Legacy Seam, Conway's Law), and [uncovering mainframe seams](https://martinfowler.com/articles/uncovering-mainframe-seams.html).
- **Gamma, Helm, Johnson & Vlissides, *Design Patterns: Elements of Reusable Object-Oriented Software* (the "Gang of Four")** — the basis for the GoF pattern references included in the domain, application-layer, persistence, dependency-injection, and event-sourcing skills (Strategy, Factory, Builder, Template Method, Composite, State, Memento, Visitor, Mediator, Observer).
- **Paul M. Jones, Action-Domain-Responder** — the basis for the `action-domain-responder` skill, including the "Web MVC is really Model 2" correction. Distilled from [MVC-MODEL-2 (Action-Domain-Responder)](https://github.com/pmjones/adr/blob/master/MVC-MODEL-2.md) and the ADR write-ups.

Where a skill references a specific page, that link appears in the relevant `references/` file.
