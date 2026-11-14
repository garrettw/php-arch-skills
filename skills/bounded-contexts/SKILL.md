---
name: bounded-contexts
description: Use this skill when designing system boundaries, discovering or defining bounded contexts, deciding where code belongs, or defining shared kernel and context map relationships in a PHP monolith.
---

# Bounded Context Design for PHP Monoliths

## When to Use Bounded Contexts (and When NOT to)
Bounded contexts are appropriate for complex applications with multiple business capabilities. They are not needed for every project.

| Use Bounded Contexts For                           | Do Not Use For                        |
|----------------------------------------------------|---------------------------------------|
| Complex business domain with multiple capabilities | Simple CRUD apps with one clear model |
| Distinct languages or lifecycles in one system     | One model with one vocabulary         |
| Long-lived system requiring clear ownership        | Prototypes, throwaway code            |
| Application with multiple integration surfaces     | Libraries, packages, SDKs             |

For libraries and packages, use standard PSR-4 namespacing to organize code. Bounded-context separation applies when the project has multiple domain capabilities with their own language and lifecycle, whether one developer or several teams own them. Team structure is useful evidence of where a boundary lies (Conway’s law), but it does not create or remove one. A small team mainly changes how formally the boundaries are documented and enforced, not whether they exist.

## System Overview
The backend north star for modular PHP applications is **business-capability-first bounded contexts**. Current folders, namespaces, controllers, database prefixes, and API route groups are supporting evidence, not the source of truth for ownership. This skill helps map out business boundaries and clarify cross-context integration patterns.

## Numbered Workflows

### 1. Discovering Bounded Contexts
If evaluating whether a grouping of code should be a bounded context:
1. **Identify the Capability.** Does this grouping represent a distinct business capability, process, or lifecycle? If yes, it is a candidate.
2. **Apply the "Not a Bounded Context" Classifier.** If the grouping is primarily an infrastructure service (e.g., Search Engine), third-party integration (e.g., Salesforce), API surface, delivery channel (e.g., Email), or config mechanism, then it is **not** a bounded context.
3. **Assign Ownership.** Map the underlying business concepts to their true domain contexts (e.g., a payment webhook belongs to Billing, not a "Webhook" context).

### 2. Defining Cross-Context Ownership
If passing data between contexts:
1. **Identify the Source Fact.** The context that originates the fact (e.g., `Billing` for payments) remains the owner.
2. **Project the Fact.** If another context needs the fact (e.g., `Entitlements` needs to know if a user paid), it consumes the fact without taking ownership.

### 3. Designing the Shared Kernel
If extracting code to share across multiple contexts:
1. **Evaluate Necessity.** Share a concept only once a second context needs it. Until then it stays in its owning context's `Domain/`.
2. **If it is an identity reference** (`UserId`), a published, versioned event contract, or a stable value type with no context-specific rules (`Money`, `DateRange`), place it in the Shared Kernel. Every context that uses it owns it jointly, so changes need their agreement; keep the set small, and prefer a well-known library (e.g., `brick/money`) when one fits.
3. **If its meaning or rules differ between contexts** (`Address`, `Price`, `Customer`), do not share it. Each context models its own version and translates at the boundary; tax, discounting, and context-specific rounding stay in the context that has them.
4. **Keep event contracts plain.** Published contracts carry primitive values (`amountMinor: int`, `currency: string`), not Shared Kernel value objects, so consumers and the message format do not depend on PHP classes.
5. **If it is generic utility code** (e.g., array helpers), do not place it in the Shared Kernel.

## Recognizing Boundary Problems in an Existing Codebase

When analyzing an established codebase, these are signals of misplaced or missing boundaries (judge against real business capabilities; do not reshuffle folders for tidiness):

- **"Infrastructure" contexts.** A context named after a transport or vendor — `Webhooks`, `Email`, `Salesforce` — instead of a business capability. Webhooks/integrations are infrastructure, not contexts; re-home them to the capability that owns the fact (e.g., a payment webhook belongs to Billing).
- **Shared Kernel as a util dump.** The "common" namespace holds array helpers, base classes, and generic traits rather than identity references, stable shared value types, or published event contracts.
- **Structure-by-layer, not by-capability.** Code is sliced by technical layer (`Controllers/`, `Models/`, `Repositories/`) across the whole app rather than grouped by business capability, so a single feature's logic is scattered.
- **Hidden ownership.** No context map or glossary; the same term means different things in different areas, or facts are owned by the consumer rather than their source.
- **Circular dependencies between contexts.** Two contexts import each other to get a job done. Break the cycle with a port, an event, or by moving the shared concept into a Shared Kernel or down into the domain. Cyclic context coupling is a core *design debt* smell.
- **Shotgun surgery.** One business change forces edits across many contexts because a single responsibility is scattered. Consolidate that responsibility into the context that owns the capability (this is Separation of Concerns at the context scale).
- **Premature context extraction.** A context carved out "in case we need to scale it later" before the capability actually exists. Don't split until there is a real, distinct business capability and a real owner (YAGNI at the architecture scale).

## Boundaries

### Always Do
- Always use the [Context Map Template](references/context-map-template.md) to document how contexts interact.
- Always establish clear ownership for terms using the [Glossary Template](references/glossary-template.md).
- Always treat the context map as a living document; revisit boundaries when new lifecycle behaviors emerge.

### Ask First
- Ask before introducing a new bounded context; ensure it aligns with a distinct business capability.

### Never Do
- Never mix infrastructure boundaries (like third-party APIs) with domain boundaries.
- Never turn the Shared Kernel into a generic "utilities dump" or "common bucket".
- Never skim DDD as just a directory structure. Bounded contexts are defined by a ubiquitous language and business capability, not by folders or namespaces.
