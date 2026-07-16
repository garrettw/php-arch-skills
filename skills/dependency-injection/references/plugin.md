# Plugin

Applications shouldn't require modification to support every variation. Instead, they **define extension points**; new behavior is added by plugging in new implementations. The goal is to **separate the stable framework from the variable behavior**.

A Plugin *is* the Hexagonal idea made operational: the stable framework is the domain/core, the extension points are its **Ports** (interfaces), and "plugging in" is wiring an **Adapter** at the **composition root** (see the [dependency-injection](../SKILL.md) and [infrastructure-boundaries](../infrastructure-boundaries/SKILL.md) skills). The pattern is the *reason* ports-and-adapters exists.

## Shape of a Plugin

- The core defines an interface for the behavior it needs (a Port): `PaymentProvider`, `TaxCalculator`, `NotificationChannel`.
- A variation is a new class implementing that interface (an Adapter / plugin).
- Wiring happens at the edge — a composition root, service provider, or config — never by editing the core.

This is what lets the same application run with Stripe in production and a fake in tests, or add a new payment gateway without touching checkout code.

## The discipline: extend, don't bypass

> Plugin systems should extend behavior, not become backdoors around the architecture.

A good plugin **reinforces the existing abstractions** rather than bypassing them:

| Good plugin (extends via the abstraction)    | Bad plugin (backdoor around architecture)        |
|----------------------------------------------|--------------------------------------------------|
| Implements the defined Port interface        | Reaches into the core / calls internals directly |
| Discovered and wired at the composition root | Registered by monkey-patching or global mutation |
| Behaves within the contract's invariants     | Special-cases its way around domain rules        |

The danger is that an "extension point" becomes an unsupervised escape hatch: a plugin that imports the domain, writes to the database directly, or skips validation in the name of convenience. That silently erodes every boundary the architecture exists to protect. Treat a plugin as a first-class citizen of the architecture — it must satisfy the same ports, same dependency rule, and same invariants as the core's own code.

## Related
- The [dependency-injection](../SKILL.md) skill is where plugins are wired — keep container wiring at the edge (composition root) and the core free of framework/plugin internals.
- A Plugin is an [Adapter](gateway.md) at an [infrastructure-boundaries](../infrastructure-boundaries/SKILL.md) seam; the Port it implements is defined by the inner layer.
- Swapping a storage implementation is the persistence instance of this idea (see [persistence-patterns](../persistence-patterns/SKILL.md)).
