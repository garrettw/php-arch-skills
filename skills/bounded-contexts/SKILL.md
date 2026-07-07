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
| Team of multiple developers owning different areas | Solo developer or small team          |
| Long-lived system requiring clear ownership        | Prototypes, throwaway code            |
| Application with multiple integration surfaces     | Libraries, packages, SDKs             |

For libraries and packages, use standard PSR-4 namespacing to organize code. Bounded-context separation only applies when the project has multiple domain capabilities that different teams or subsystems must own independently.

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
1. **Evaluate Necessity.** Is this concept intentionally shared to reduce coupling?
2. **If it is an identity reference** (`UserId`), a core value object (`Money`), or a domain event contract, place it in the Shared Kernel.
3. **If it is generic utility code** (e.g., array helpers), do not place it in the Shared Kernel.

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
