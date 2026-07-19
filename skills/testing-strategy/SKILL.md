---
name: testing-strategy
description: Use this skill when writing tests for a PHP application. Provides layer-specific guidance to choose the right testing style (unit, integration, feature) for domain objects, mappers, repositories, handlers, and controllers.
---

# Backend Testing Strategy by Architecture Layer

## System Overview
Different architectural layers have different responsibilities, and tests should be structured to verify those specific responsibilities without over-specifying internal implementation details. This skill provides rules for matching the right testing style (unit, integration, feature) to the correct layer.

## Principles Behind Good Tests

- **Arrange, Act, Assert (AAA).** One logical flow per test: set up the scenario, perform the one action under test, assert the outcome. Keep the "Given" visibly in the test body.
- **Isolation.** Each test owns its data and runs independently of others, in any order. No shared mutable state, no order dependence. Deterministic tests are the only ones worth having.
- **Mock only at boundaries.** Replace slow or non-deterministic collaborators (databases, HTTP, queues) with doubles. Do not mock the database inside a repository test (test the repository against a real test DB) and do not mock the very object under test. Mocking everything while asserting on call order verifies the implementation, not the behavior — a brittle test that breaks on every refactor.
- **Assert behavior, not internals.** Assert on the resulting state, returned value, or recorded events — not on private methods or the exact internal call sequence. A test that survives a legitimate refactor is a test worth keeping.
- **Coverage is a guide, not a target.** Aim for meaningful coverage of business logic and unhappy paths; 100% is not the goal and can incentivize worthless assertions.

See [clean-code-foundations.md](../domain-modeling/references/clean-code-foundations.md) for the encapsulation/DRY lenses that apply here.

## Numbered Workflows

### 1. Testing the Domain Layer
If testing pure domain logic (entities, value objects, policies):
1. **Use pure PHP unit tests.** Do not boot the framework or database.
2. **Setup state.** Instantiate the object directly using `new` (the "Given").
3. **Execute behavior.** Call the domain method (the "When").
4. **Assert outcomes.** Verify the new state, the returned result, or the events recorded (the "Then").

### 2. Testing Mappers and Repositories
If testing persistence boundaries:
1. **For Mappers:** Use unit tests or fast integration tests. Focus on data conversion edge cases (null values, default fallbacks, enum string conversions).
2. **For Repositories:** Use integration tests hitting a real (test) database. Do not mock the ORM inside a repository test. Write a fixture to the real database, call the repository, and assert the returned domain objects are correct.

### 3. Testing Application Handlers
If testing use case orchestration:
1. **Choose the testing style.** Use integration tests hitting a real test database, OR unit tests with fakes/mocks for infrastructure. (Integration tests often provide higher confidence with less brittle setup).
2. **Assert coordination.** Verify the handler successfully loads data, calls the domain, saves state, and publishes events.

### 4. Testing Controllers and Adapters
If testing framework edges and infrastructure adapters:
1. **For Controllers:** Write Feature/Integration tests. Assert on HTTP status codes, JSON response shapes, and validation error formats.
2. **For Infrastructure Adapters:** Write integration tests hitting a sandbox API (if available) or unit tests mocking the vendor's HTTP client. Verify that vendor exceptions are caught and translated into domain exceptions.
3. **For Architecture Boundaries:** Add automated architecture tests to enforce the dependency rule (e.g., using Pest, Deptrac, PHP Arkitect, or a custom script). Verify that the domain core has zero imports from infrastructure, HTTP, or database libraries.

## Using Tests to Analyze an Existing Codebase

Tests are also the fastest way to *characterize and probe* a legacy codebase before planning changes:

- **Run architecture tests first.** A Deptrac/PHP Arkitect rule that the domain core imports nothing from infrastructure is a one-command smell detector — it maps exactly which layers are leaking before you read a line of business logic.
- **Write characterization tests before refactoring.** Lock current behavior with a high-level integration test (per the `architecture-migration` playbook) so you can prove a Strangler-Fig slice hasn't changed anything. This is the safe on-ramp to any change.
- **Prefer real test databases over mock chains.** For handlers and repositories, an integration test against a real (test) DB gives higher confidence with less brittle setup — and doubles as documentation of what the code actually does today.
- **Look for test debt.** When auditing, watch for: critical paths with no tests; flaky tests that pass or fail non-deterministically; skipped/disabled tests with no linked issue or removal date; and suites so slow they discourage running them. These are *test debt* — they erode confidence and slow every engineer.

## Boundaries

### Always Do
- Always use the [Test Patterns by Layer](references/test-patterns-by-layer.md) to structure your tests correctly.
- Always create small, specific fixtures for the test scenario rather than relying on massive global state.
- Always make the test setup (the "Given") explicitly visible in the test method.
- Always keep tests independent and deterministic — each test sets up its own data and cleans up after itself.

### Ask First
- Ask before introducing complex mock chains in a handler test if a real test database would be simpler and more robust.

### Never Do
- Never over-specify tests. Do not assert on private methods or the exact sequence of internal calls if the external contract is met.
- Never mock the database inside a repository test.
- Never write tests that only "don't throw" without asserting meaningful behavior, or that re-encode the implementation's logic instead of verifying the outcome — those tests pass even when the behavior is broken.
