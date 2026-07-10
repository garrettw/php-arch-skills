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
- **`persistence-patterns`**: Guides the agent on how to persist data across architectural boundaries, choosing a data source pattern along the spectrum of business-logic/persistence separation, when to use mappers and repository interfaces, and how to apply CQRS at the database edge.
- **`infrastructure-boundaries`**: Teaches the agent to isolate third-party APIs, vendor SDKs, and external systems behind clean adapters, preventing vendor lock-in.
- **`domain-events`**: Shows the agent how to decouple side-effects and cross-context communication using the record-then-publish event pattern.
- **`event-sourcing`**: Guides the agent through evaluating whether Event Sourcing is appropriate, modeling event-sourced aggregates, building projections and snapshots, and choosing PHP libraries (Ecotone, Prooph, EventSauce, Patchlevel).
- **`testing-strategy`**: Forces the agent to write tests that verify behavior rather than internal implementations, spanning pure domain unit tests to database integration tests.
- **`architecture-migration`**: Provides the agent with a playbook for incrementally refactoring legacy code into a clean architecture using vertical slices and temporary shims, avoiding big-bang rewrites.
- **`dependency-injection`**: Ensures the agent relies on constructor injection over legacy service locators and correctly registers interfaces at the framework edge.
- **`framework-integration`**: Maps clean-architecture patterns to specific PHP frameworks, providing port-to-framework mappings, anti-patterns to avoid, and composition-root wiring guidance.
