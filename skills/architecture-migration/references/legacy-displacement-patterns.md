# Legacy Displacement Patterns

These patterns (from Fowler et al., *Patterns of Legacy Displacement*) are the concrete delivery toolkit that makes a [Strangler Fig](strangler-fig.md) migration actually work. They answer "how do we insert a seam, coexist with the legacy, and cut over without a big bang?" Several overlap with this skill's shim and vertical-slice workflows; here they are named and framed as a catalog.

## Transitional Architecture
Software elements installed **only** to ease the displacement of a legacy system, which you intend to remove when displacement completes. This is the same idea as the compatibility shims in workflow 3 — a `LegacyUserAdapter`, an Event Router, a parallel table.

- **Treat it as a deliberate investment, not waste.** Teams reflexively cut transitional architecture to "save cost," but it buys reduced risk and earlier value. Be explicit about the trade-off: *value of risk mitigation* vs *cost of transitional architecture*. Track the investment against the risk it mitigates.
- **Always name it and schedule its removal** (as with shims: `@todo Remove once all Controllers are migrated to Handlers`).

## Event Interception
Intercept updates to system state and route **some** of them to a new component. The canonical move: insert an Event Router on an existing message queue or stream as a pure refactor (no observable behavior change), then configure it to gradually divert traffic.

- This is how you **insert a seam** (Fowler's [Legacy Seam](https://martinfowler.com/bliki/LegacySeam.html)) without changing legacy behavior. You grow the new fig by intercepting the flow, not by rewriting the host.
- Works at PHP edges too: wrap a queue consumer, HTTP call, or DB trigger in a router you control.

## Legacy Mimic
The new system interacts with the legacy in a way that the **old system is not aware of any changes** — typically an adapter that keeps legacy side-effects (reports, warehouses, downstream consumers) working while you migrate.

- In this skill set's terms it is a **Gateway/Adapter** at a boundary (see [infrastructure-boundaries](../infrastructure-boundaries/SKILL.md)): isolate the new implementation from legacy quirks so future legacy changes don't ripple in.
- Often required in surprising places (the article found KPI reports fed by a replicated legacy DB needed a Wire Tap mimic). Expect "archeology": hidden consumers surface only after go-live.

## Divert the Flow
First divert cross-organization activities away from the legacy before replacing it. Establish the new path for traffic, prove it, then decommission the old one.

- Pairs with segmentation (cut over by a business slice — product type, tenant, region) so rollout is risk-managed and reversible.

## Revert to Source
Identify the **originating source of data** and integrate to that, rather than to an intermediate legacy layer that merely passes data through.

- Cuts a needless dependency: if legacy is just a passthrough, integrate the new component to the real source instead of mimicing the middleman.

## Feature Parity (Anti-Pattern)
Replicate the existing functionality of a legacy system on a new stack, promising the business "exactly what you have today, just improved behind the covers."

- **Why it's a trap:** defining current "as-is" behavior is huge effort, it leads to a large **big-bang cutover**, and much of that behavior isn't actually wanted — so you rebuild waste. This skill's "Never Do" already rejects big-bang rewrites; Feature Parity is the planning failure that produces them.
- **Instead:** migrate by outcome and business capability (the four activities in [strangler-fig.md](strangler-fig.md)). Build the new slice to meet the *needed* behavior, not a 1:1 copy.

## How They Fit Together (the article's example)
Replace integration middleware by: (1) inserting **Event Interception** on the queue as a no-behavior-change refactor; (2) building the new component with **Legacy Mimics** to keep legacy reports/DB working — all part of the **Transitional Architecture**; (3) **Diverting the Flow** by product segment to roll out gradually; (4) decommissioning the legacy once 100% of traffic is on the new path.

## Related
- [strangler-fig.md](strangler-fig.md): the umbrella pattern; these are its delivery mechanics.
- [infrastructure-boundaries](../infrastructure-boundaries/SKILL.md): Legacy Mimic is a Gateway/Adapter at a boundary.
- Source: [Patterns of Legacy Displacement](https://martinfowler.com/articles/patterns-legacy-displacement/) (Transitional Architecture, Event Interception, Legacy Mimic, Divert the Flow, Revert to Source, Feature Parity).
