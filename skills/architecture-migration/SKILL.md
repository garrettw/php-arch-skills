---
name: architecture-migration
description: Use this skill when refactoring legacy PHP code toward a cleaner architecture. Provides guidance on incremental migrations, vertical slicing, writing characterization tests, and managing temporary compatibility shims.
---

# Incremental Architecture Migration for PHP Apps

## System Overview
Big-bang rewrites are risky and halt product momentum. This skill guides the process of moving PHP applications toward a cleaner architecture through incremental, touched-area migrations using vertical slices and temporary compatibility shims.

## Numbered Workflows

### 1. Evaluating a Migration Trigger
If considering refactoring legacy code into a new architecture:
1. **Check the Motivation.** Are you actively building a new feature in this area, fixing a complex bug, or exposing an existing web feature as an API endpoint? If yes, it is a good trigger. Proceed.
2. **Reject Poor Triggers.** Are you just doing it to "clean things up" or to fix a standard violation in code that hasn't been touched in years? If yes, halt the migration.

### 2. Executing a Vertical Slice Migration
If migrating a specific use case (e.g., one API endpoint or queue job):
1. **Write Characterization Tests.** Lock in the current behavior of the existing legacy code (e.g., the massive Controller) with a high-level integration test. Ensure it passes.
2. **Extract to Handler.** Move the core logic out of the legacy location into a new Application Handler.
3. **Adapt the Edge.** Update the original Controller to instantiate the new Handler and pass it the required inputs.
4. **Verify.** Run the characterization tests to prove behavior hasn't changed.
5. **Push Down to Domain.** Extract pure business rules from the Handler into Domain objects or policies.
6. **Verify Again.** Run the tests one final time.
7. **Cleanup.** Remove legacy compatibility shims if all callers are migrated.

### 3. Managing Compatibility Shims
If introducing a temporary class to bridge old and new code (e.g., a legacy facade implementing a new interface):
1. **Mark Clearly.** Name it clearly as temporary (e.g., `LegacyUserAdapter`).
2. **Define Removal Conditions.** Add comments specifying what must happen before it can be removed (e.g., "@todo Remove once all Controllers are migrated to Handlers").

## Boundaries

### Always Do
- Always write characterization tests against the legacy code *before* extracting logic.
- Always migrate one complete vertical slice at a time, rather than horizontal layers (e.g., don't just "move all controllers" at once).

### Ask First
- Ask before introducing a compatibility shim if there is an easy way to fully migrate all legacy callers instead.

### Never Do
- Never mix massive feature changes with architectural migrations. Move the code first, prove it works, then change the feature.
- Never leave half-migrated workflows. If you start migrating a use case, finish it.
