# Coordination Patterns (Application Layer)

Two patterns that shape how use cases and flows are coordinated: **Template Method** and **Mediator**. Both are about structuring collaboration between objects without tangling them.

## Template Method

Defines the skeleton of an algorithm in a method, deferring one or more steps to subclasses (or injected collaborators) so the invariant parts stay fixed and the variable steps vary.

- **When:** several use cases share the same overall flow but differ in one or two steps — e.g., "load → validate → execute → persist → publish" where only `execute` differs. The application handler already embodies this shape ([application-layer](../application-layer/SKILL.md)).
- **In this skill set:** a base handler/service defines the sequence; subclasses or strategy collaborators fill the variable step. This is a lighter alternative to full CQRS when the flows are similar. Keep the invariant flow in the base; never put business *decisions* in the template — only the *ordering*.
- **Watch:** avoid deep inheritance hierarchies; if only one step varies, prefer injecting a [Strategy](../domain-modeling/references/behavioral-structural-patterns.md) for that step rather than subclassing the whole handler.

## Mediator

Centralizes complex communication between a set of objects so they no longer refer to each other directly, but only through the mediator.

- **When:** many application-layer or domain objects need to coordinate and would otherwise become tightly coupled by calling each other directly (the "chat room" problem).
- **In this skill set:** the dominant realization is **domain events** ([domain-events](../domain-events/SKILL.md)) plus an event bus — objects publish intent and interested parties react, with no direct references between them. A coordinator/handler can also act as a mediator for a specific workflow. This avoids the [Service Layer](../application-layer/references/service-layer.md) anti-pattern of chaining services (`Service A → B → C`).
- **Watch:** a mediator reduces object-to-object coupling but concentrates logic in one place — keep it thin (orchestration only), and don't let it become a god-object holding business rules.

## Relationship to other skills
- **Template Method** overlaps the Command/Handler workflow in [application-layer](../application-layer/SKILL.md) — use the template when flows share a skeleton, separate handlers when flows differ.
- **Mediator** is the conceptual basis for [domain-events](../domain-events/SKILL.md): publish/subscribe is a Mediator realized as an event bus.
