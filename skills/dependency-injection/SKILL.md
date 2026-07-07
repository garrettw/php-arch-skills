---
name: dependency-injection
description: Use this skill when wiring dependencies, configuring a DI container, or migrating away from legacy service locators in PHP. Triggers on questions about constructor injection, interface registration, or testing without booting the container.
---

# Dependency Injection Best Practices for PHP

## When to Use a DI Container (and When NOT to)
A DI container is an infrastructure concern. Use it for applications that need flexible wiring, interface swapping, or framework integration. Libraries should use constructor injection directly and let the consumer resolve dependencies.

| Use a DI Container For                            | Do Not Use a Container For               |
|---------------------------------------------------|------------------------------------------|
| Applications with many interchangeable services   | Libraries and packages                   |
| Need to swap implementations (test vs production) | Simple scripts with one or two classes   |
| Framework integration (Symfony, Laravel)          | Throwaway code or prototypes             |

The container should wire the application together at the edge, while the core remains ignorant of the container. This skill provides the rules for registering dependencies, migrating legacy locators, and injecting objects.

## Numbered Workflows

### 1. Registering Dependencies
If deciding whether to create an interface and register it in the DI container:
1. **Check for Multiple Implementations.** Are there multiple implementations (e.g., `StripeAdapter` and `PayPalAdapter`)?
2. **Check for Boundary Inversion.** Is the application defining a port that the infrastructure implements?
3. **Check for Decorators.** Do you need to wrap the class with caching or logging?
4. **If Yes to any:** Create an interface and register the binding in the container configuration.
5. **If No to all:** Depend on the concrete class directly and let the container autowire it. Do not blindly create interfaces for every class.

### 2. Testing with Constructor Injection
If writing tests for classes that use constructor injection:
1. **For Domain/Handler Tests:** Construct the object using `new`. Pass in test doubles (mocks or fakes) for any infrastructure dependencies. Do not boot the DI container.
2. **For Integration/Feature Tests:** Ask the test framework to resolve the controller or handler from the DI container.

### 3. Migrating Legacy Code
If modifying a legacy class that uses a service locator (`Container::getInstance()->get()`):
1. **Follow the Migration Sequence.** Read the [Migration Sequence for Legacy Containers](references/migration-sequence.md).
2. **Extract to Constructor.** Find every internal `getInstance()` call and add those dependencies as typed constructor properties.
3. **Fix Callers.** Update all instantiations of the legacy class to pass the dependencies.
4. **Use Shims if Necessary.** If the class is instantiated in 50 places you cannot change, use the temporary shim pattern to bridge the gap.

## Boundaries

### Always Do
- Always use constructor injection for services, handlers, repositories, adapters, and listeners.
- Always keep the container wiring logic at the framework edge (composition roots, service providers).
- Always keep the domain core free of framework dependencies. It should resolve from tests or a composition root with no framework boot.

### Ask First
- Ask before introducing a legacy shim if it is feasible to simply update all callers instead.

### Never Do
- Never use service locators inside domain objects, application handlers, or policies. They hide dependencies and couple core code to the framework.
