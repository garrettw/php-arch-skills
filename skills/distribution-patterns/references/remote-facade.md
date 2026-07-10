# Remote Facade

Calling code across a network is fundamentally different from calling code in memory. A Remote Facade provides a **coarse-grained facade over fine-grained objects to improve efficiency over a network**.

> Network calls are expensive.

In-process, fine-grained interaction works well: small objects, small methods, intention-revealing names, easy substitution of behavior. But any inter-process call is **orders of magnitude** more expensive than an in-process call — marshaling, security checks, routing, even the speed of light — *even when both processes run on the same machine*. So anything you expose remotely needs a coarse-grained interface that minimizes the number of calls.

Therefore: don't expose your entire object model remotely. **Expose larger operations instead.**

### Bad (chatty, leaks the object model)
- `getCustomer()`
- `getOrders()`
- `getAddresses()`
- `getInvoices()`
- `getPayments()`

### Good (one call carries everything the consumer needs)
- `GetCustomerDashboard()`
- `Checkout()`
- `RenewSubscription()`
- `SubmitOrder()`

This pattern appears almost everywhere today: REST endpoints, GraphQL mutations, CQRS commands, gRPC methods, RPC services, even message handlers. **Different transport, same architectural idea.**

## Rules

- **The Remote Facade contains no domain logic.** It is a thin translator: it only translates coarse-grained method calls onto the underlying fine-grained objects. Business decisions stay in the domain layer; orchestration/coordination belongs to the [application-layer](../application-layer/SKILL.md). A Remote Facade is not a service with behavior.
- **Only the facade is remote.** None of the underlying fine-grained objects have a remote interface. The facade sits on top of a web of in-process objects.
- **Design the contract around consumer needs**, not the internal object model.

### Example
Rather than `getOrder()` then `getOrderLines()` then `getAddresses()` in separate calls, expose `GetOrderDashboard(id)` that returns the order, its lines, and the addresses in a single round trip.

## The tradeoff (be honest about it)

Coarse-graining forces you to give up the intention-revealing naming and fine-grained control that small objects give you. Programming becomes harder and productivity slows. Remote Facade is a deliberate trade of design clarity for network efficiency — apply it *only* where the seam is genuinely remote. If the call is in-process, a Remote Facade is unnecessary indirection.

## Common mistake: exposing ORM entities

Many developers accidentally expose ORM entities through APIs. That's exactly what Fowler warned against. A Remote Facade returns purpose-built responses — usually [DTOs](dto.md) — not hydrated domain or ORM objects.

## Related
- A Remote Facade is the remote-facing expression of a use case — see the [application-layer](../application-layer/SKILL.md) skill (CQRS commands *are* remote facades).
- The data crossing the wire is usually a [DTO](dto.md), covered in this skill.
- Distinct from [infrastructure-boundaries](../infrastructure-boundaries/SKILL.md), which is about *consuming* external systems behind adapters, not *exposing* your own remote interface.
