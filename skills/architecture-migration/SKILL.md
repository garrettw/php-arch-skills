---
name: architecture-migration
description: Use this skill when refactoring legacy PHP code toward a cleaner architecture. Provides guidance on incremental migrations (Strangler Fig), vertical slicing, writing characterization tests, and managing temporary compatibility shims.
---

# Incremental Architecture Migration for PHP Apps

## When to Migrate (and When NOT to)
Big-bang rewrites are risky and halt product momentum. Migrate only when actively building a feature or fixing a complex bug in an area. Do not migrate code that has not been touched in years solely for aesthetic reasons.

| Good Migration Trigger                     | Poor Migration Trigger                     |
|--------------------------------------------|--------------------------------------------|
| Actively building a new feature            | "Cleaning up" untouched code               |
| Fixing a complex bug in legacy code        | Compliant refactor of inactive code        |
| Exposing a web feature as an API endpoint  | Preference for architecture purity         |

Evolutionary, touched-area migrations using vertical slices and temporary compatibility shims. This is the **Strangler Fig** approach: grow new behavior alongside the legacy system, move behavior across behind seams, and remove the legacy only once it carries nothing (see [strangler-fig.md](references/strangler-fig.md)). The delivery mechanics — transitional architecture, event interception, legacy mimics, diverting the flow — are catalogued in [legacy-displacement-patterns.md](references/legacy-displacement-patterns.md). It is the antidote to the big-bang rewrite, which fails because replacements take too long, users can't wait for features, and much of the old behavior isn't actually wanted.

## System Overview
This skill guides the process of moving PHP applications toward a cleaner architecture through incremental, touched-area migrations using vertical slices and temporary compatibility shims. When adopting layered DDD, follow this order: discover the domain, model aggregates and value objects, define ports, implement use cases, and add adapters last.

## Numbered Workflows

### 1. Evaluating a Migration Trigger
If considering refactoring legacy code into a new architecture:
1. **Check the Motivation.** Are you actively building a new feature in this area, fixing a complex bug, or exposing an existing web feature as an API endpoint? If yes, it is a good trigger. Proceed.
2. **Evaluate Complexity Fit.** Is the domain complex enough to warrant full DDD? If the rules are mostly CRUD with few invariants, favor lean ORM entities over full DDD aggregates. Start simple and evolve only when needed.
3. **Reject Poor Triggers.** Are you just doing it to "clean things up" or to fix a standard violation in code that hasn't been touched in years? If yes, halt the migration.

### 2. Executing a Vertical Slice Migration
If migrating a specific use case (e.g., one API endpoint or queue job):
1. **Write Characterization Tests.** Lock in the current behavior of the existing legacy code (e.g., the massive Controller) with a high-level integration test. Ensure it passes.
2. **Extract to Handler.** Move the core logic out of the legacy location into a new Application Handler. This is the "grow the new fig alongside the host" step — the legacy still serves other use cases until they too are migrated (see [strangler-fig.md](references/strangler-fig.md)).
3. **Adapt the Edge.** Update the original Controller to instantiate the new Handler and pass it the required inputs.
4. **Verify.** Run the characterization tests to prove behavior hasn't changed.
5. **Push Down to Domain.** Extract pure business rules from the Handler into Domain objects or policies.
6. **Verify Again.** Run the tests one final time.
7. **Cleanup.** Remove legacy compatibility shims if all callers are migrated.

### 3. Managing Compatibility Shims
If introducing a temporary class to bridge old and new code (e.g., a legacy facade implementing a new interface):
1. **Mark Clearly.** Name it clearly as temporary (e.g., `LegacyUserAdapter`).
2. **Define Removal Conditions.** Add comments specifying what must happen before it can be removed (e.g., "@todo Remove once all Controllers are migrated to Handlers"). These shims are **transitional architecture** — they exist only so new and legacy coexist, and you delete them when migration completes (see [strangler-fig.md](references/strangler-fig.md) and the delivery patterns in [legacy-displacement-patterns.md](references/legacy-displacement-patterns.md)).

## Boundaries

### Always Do
- Always write characterization tests against the legacy code *before* extracting logic.
- Always migrate one complete vertical slice at a time, rather than horizontal layers (e.g., don't just "move all controllers" at once).

### Ask First
- Ask before introducing a compatibility shim if there is an easy way to fully migrate all legacy callers instead.

### Never Do
- Never mix massive feature changes with architectural migrations. Move the code first, prove it works, then change the feature.
- Never leave half-migrated workflows. If you start migrating a use case, finish it.
- Never attempt a big-bang rewrite of the whole system. Use the Strangler Fig approach (see [strangler-fig.md](references/strangler-fig.md)): grow the new system alongside the legacy and remove the legacy only once it carries nothing.
- Never pursue **Feature Parity** — replicating the old system's full "as-is" behavior on a new stack. It drives a big-bang cutover and rebuilds behavior that isn't wanted. Migrate by business capability and needed outcome instead (see [legacy-displacement-patterns.md](references/legacy-displacement-patterns.md)).
