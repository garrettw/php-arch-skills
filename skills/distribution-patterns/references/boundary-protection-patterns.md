# Boundary-Protection Patterns

The recurring theme across the distribution and supporting patterns is simple: **architecture is about protecting boundaries.** Each high-value pattern exists to cross a seam safely without leaking the internals on either side. Look at what they share:

| Pattern           | Boundary it protects                                                   | Lives in                                                                           |
|-------------------|------------------------------------------------------------------------|------------------------------------------------------------------------------------|
| **Remote Facade** | Process / network boundary (the *operation* contract)                  | [distribution-patterns](../references/remote-facade.md)                            |
| **DTO**           | Representation boundary — the *data shape* on the wire                 | [distribution-patterns](../references/dto.md)                                      |
| **Gateway**       | Infrastructure boundary — talking to an external system                | [infrastructure-boundaries](../../infrastructure-boundaries/references/gateway.md) |
| **Mapper**        | Model boundary — translating between representations                   | [infrastructure-boundaries](../../infrastructure-boundaries/references/mapper.md)  |
| **Plugin**        | Framework / extension boundary — supplying variable behavior           | [dependency-injection](../../dependency-injection/references/plugin.md)            |
| **Special Case**  | Behavioral boundary — the "nothing here" / exceptional-but-valid state | [domain-modeling](../../domain-modeling/references/special-case.md)                |

These patterns have stayed relevant because they address *enduring sources of complexity* (coupling across seams, leaking internals, conditional sprawl) rather than transient implementation details. They are not a menu to apply all at once — each one answers a specific question: *which seam am I crossing, and what must not leak through it?*

## Why they still matter in a Ports & Adapters world

These patterns align remarkably well with modern **Ports & Adapters (Hexagonal)** architecture. In most cases Hexagonal doesn't *replace* them — it **reframes and generalizes** them:

- A **Gateway** *is* an outbound Port with an Adapter behind it.
- A **Remote Facade** defines an *inbound* boundary (a driver Port's entry point).
- **DTOs** become the *messages* crossing those boundaries — in both directions.
- A **Plugin** architecture is just a structured way of *supplying* the Adapters (wired at the composition root).
- A **Mapper** is the translation the Adapter performs at the seam.
- A **Special Case** keeps the *domain* boundary clean so callers never reach across it with `null` checks.

So these classic patterns are part of the **intellectual foundation** contemporary styles (Hexagonal, Clean, Onion) were built on. When you reach for a port or adapter, you are usually instantiating one of the patterns above — know which one, and the boundary it exists to protect.

## How to use this map

1. Identify the seam you are about to cross (process, network, model, infrastructure, framework, behavior).
2. Pick the pattern that protects *that* seam — don't borrow a process-boundary pattern for an in-process model seam.
3. Keep the pattern dumb about *business decisions*: facades orchestrate, gateways/mappers translate, special cases represent valid state. None of them make the rules.

If you are crossing a remote seam, start at [distribution-patterns](../SKILL.md). If you are consuming an external system, start at [infrastructure-boundaries](../../infrastructure-boundaries/SKILL.md). If you are supplying swappable behavior, start at [dependency-injection](../../dependency-injection/SKILL.md).
