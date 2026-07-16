# PHP Architecture Skills for AI Agents

A collection of redistributable AI agent skills designed to help you and your AI assistant build clean, maintainable PHP applications.

## How to Use

Install the skills using the skills.sh installer, which allows you to select the ones you want:

`npx skills add garrettw/php-arch-skills`

Alternatively, install any single skill directly:

`npx skills add garrettw/php-arch-skills --skill skill-name`

## The Problem

Out of the box, AI coding assistants tend to default to the simplest, most tightly-coupled framework patterns. If you ask an agent to build a new feature in PHP, it will usually write a massive controller, couple your core business logic to the database ORM, and tightly bind third-party APIs directly into your application flow.

## The Solution

This repository contains a suite of **framework-agnostic architecture skills** extracted from real-world, large-scale PHP applications. 

By installing these skills into your AI agent's configuration, you teach the agent how to apply Domain-Driven Design (DDD), Command/Query Responsibility Segregation (CQRS), and proper inversion of control. The agent will proactively ask you about boundaries, invariants, and testability *before* it writes the code.

## The Skills

The collection includes 11 modular skills, each focused on a specific architectural layer or concept:
- **`domain-modeling`**: Guides the agent to identify real business objects, default to Transaction Script (or Table Module for row-set logic) for simple procedures, determine when to upgrade to rich domain models versus lean ORM entities, and protect business invariants using pure PHP.
- **`bounded-contexts`**: Teaches the agent to define system boundaries using business capabilities rather than code folders, establishing clear ownership and preventing tightly-coupled monoliths.
- **`application-layer`**: Defines the Application (or Service) Layer as the boundary above the domain-logic spectrum, instructing the agent to orchestrate use cases via the Command/Query/Handler pattern, keep framework controllers extremely thin, and avoid the anemic-domain overuse anti-pattern.
- **`distribution-patterns`**: Guides the agent to design remote interfaces as coarse-grained Remote Facades (REST/GraphQL/gRPC/CQRS commands) shaped around consumer needs rather than the internal object model, and to carry data across boundaries with DTOs — keeping ORM entities and domain objects off the wire.
- **`persistence-patterns`**: Guides the agent on how to persist data across architectural boundaries, choosing a data source pattern along the spectrum of business-logic/persistence separation, understanding the ORM-internal Object-Relational patterns (Unit of Work, Identity Map, Lazy Loading, Identity/Associated/Dependent Mapping, Embedded Value) as features rather than implementations, the architecture surrounding the ORM (Metadata Mapping, Query Object, Repository, Value Object), when to use mappers and repository interfaces, and how to apply CQRS at the database edge.
- **`infrastructure-boundaries`**: Teaches the agent to isolate third-party APIs, vendor SDKs, and external systems behind clean adapters, preventing vendor lock-in.
- **`domain-events`**: Shows the agent how to decouple side-effects and cross-context communication using the record-then-publish event pattern.
- **`event-sourcing`**: Guides the agent through evaluating whether Event Sourcing is appropriate, modeling event-sourced aggregates, building projections and snapshots, and choosing PHP libraries (Ecotone, Prooph, EventSauce, Patchlevel).
- **`testing-strategy`**: Forces the agent to write tests that verify behavior rather than internal implementations, spanning pure domain unit tests to database integration tests.
- **`architecture-migration`**: Provides the agent with a playbook for incrementally refactoring legacy code into a clean architecture using vertical slices and temporary shims, avoiding big-bang rewrites.
- **`dependency-injection`**: Ensures the agent relies on constructor injection over legacy service locators and correctly registers interfaces at the framework edge.
- **`framework-integration`**: Maps clean-architecture patterns to specific PHP frameworks, providing port-to-framework mappings, anti-patterns to avoid, and composition-root wiring guidance.

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
