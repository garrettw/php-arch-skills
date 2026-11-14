---
name: infrastructure-boundaries
description: Use this skill when integrating third-party APIs, SDKs, or infrastructure services (like search engines or email). Provides rules for designing provider adapters/gateways, handling webhooks, and preventing vendor lock-in.
---

# Infrastructure & Integration Boundaries in PHP

## When to Use Hexagonal Architecture (and When NOT to)
Hexagonal architecture isolates the core via ports and adapters, but it is not universal. Use it for long-lived applications that need to swap infrastructure or require high test coverage. Skip it for libraries, simple CRUD APIs, or throwaway utilities.

| Use Hexagonal For                                   | Use Simpler Patterns For                                  |
|-----------------------------------------------------|-----------------------------------------------------------|
| Long-lived application with evolving infrastructure | Libraries, packages, SDKs (PSR standards usually suffice) |
| Need to swap DB, broker, or external API            | Fixed infrastructure, unlikely to change                  |
| Multiple entry points (API, CLI, events)            | Single delivery channel                                   |
| High test coverage required                         | Quick scripts, internal tools                             |

For libraries, package your code with PSR-4 autoloading and clear namespacing. Let the consumer inject dependencies. Do not add a hexagonal adapter layer for a library.

## System Overview
The Infrastructure layer contains everything that speaks to the outside world: databases, search engines, APIs, message queues, and the filesystem. It implements adapters (driven and driver) that fulfill ports defined by inner layers. This skill provides the rules for keeping infrastructure concerns separated from domain logic through adapters, ports, and proper placement.

A [**Gateway**](references/gateway.md) is the classic name for exactly this seam: an object that encapsulates communication with one external system so the application knows *that* the call happened, not *how*. In modern terms a Gateway is a Port + Adapter. Name it after the business capability (`PaymentGateway`), never the transport or vendor (`StripeHttpWrapper`).

A [**Mapper**](references/mapper.md) is the companion translation object at the same seam: it converts *between* two representations of the same fact (domain object ↔ DTO, domain ↔ external API model) while preserving meaning. It is not a Transformer — never put business rules in a mapper. The persistence-specific instance is the Data Mapper in [persistence-patterns](../persistence-patterns/references/data-mapper.md).

These two are boundary-protection patterns: a **Gateway** protects the *infrastructure* boundary and a **Mapper** protects the *model* boundary. They sit alongside Remote Facade / DTO (process & representation), Plugin (framework/extension), and Special Case (behavioral) in a shared family — see [boundary-protection-patterns.md](../distribution-patterns/references/boundary-protection-patterns.md) for how they map onto Hexagonal architecture.

## Principles Behind the Boundary

- **Dependency Inversion (DI).** The inner layers define the port; the infrastructure layer implements it. Domain code never imports the vendor SDK — that inversion is what keeps the core swappable and testable.
- **Interface Segregation.** A port should expose exactly the capability the application uses, named in business terms (`PaymentGateway`), not a vendor grab-bag. Narrow ports keep adapters thin and keep vendor surface area out of the core.
- **Single Responsibility (SRP) + Separation of Concerns.** A Gateway owns "talk to this one external system"; a Mapper owns "translate between two representations." Keep them separate and free of business rules.
- **No speculative abstraction, but honor the dependency rule.** You never need an interface “in case we switch vendors”. You do need one when application or domain code calls the adapter, because the port must belong to the inner layer (see `dependency-injection`, workflow 1). In a lean area without ports and adapters, inject the concrete adapter at the composition root.

See [solid-principles.md](../domain-modeling/references/solid-principles.md) and [clean-code-foundations.md](../domain-modeling/references/clean-code-foundations.md).

## Numbered Workflows

### 1. Placing Code in the Right Layer
If adding a new integration or third-party service:
1. **Define the Port.** Create an interface in the Domain or Application layer that describes what the application needs in business terms.
2. **Implement the Adapter.** Create the implementation class in the Infrastructure layer, using the specific vendor SDK.
3. **Verify Imports.** Ensure the Domain/Application code only imports the interface, never the vendor SDK or HTTP client.

### 2. Classifying the Adapter
If determining where a new adapter belongs:
1. **Driver Adapters.** Does the adapter receive external input and trigger application logic (e.g., HTTP controllers, CLI commands, message queue consumers)? Place it in the context's `Interface/` folder (e.g., `Interface/Http`, `Interface/Console`, `Interface/Messaging`), or in `Infrastructure/Http` if the project keeps all adapters under Infrastructure. The full layout is in [directory-structure-templates.md](../application-layer/references/directory-structure-templates.md).
2. **Driven Adapters.** Does the adapter implement an external concern called by the application (e.g., database repository, email sender, search engine client)? Place it in the context's `Infrastructure/` folder (e.g., `Infrastructure/Persistence`, `Infrastructure/Messaging`).
3. **Composition Root.** Wire all adapter implementations to their ports in a single location (e.g., the DI configuration or the framework bootstrap) to keep the dependency rule intact.

### 3. Designing a Provider Adapter
If building a new infrastructure adapter:
1. **Define Inputs.** The adapter must take primitive values or domain value objects, not framework request objects.
2. **Translate Exceptions.** Catch vendor-specific exceptions (e.g., `GuzzleHttp\Exception\ClientException`) and wrap them in domain-meaningful exceptions defined by the port (e.g., `PaymentDeclinedException`, `GatewayUnavailableException`).
3. **Format Outputs.** Return primitive values, generic DTOs, or domain objects—never the raw vendor response object (e.g., `Stripe\Charge`).
4. **Reference Examples.** Check the [Adapter Design Examples](references/adapter-design-examples.md) for structural patterns.

### 4. Handling Webhooks and Callbacks
If building an endpoint to receive a vendor webhook:
1. **Authenticate.** Verify the provider signature over the raw request body (before any parsing or framework normalization) in an HTTP middleware or the Action, and enforce the provider's timestamp/replay window where it has one.
2. **Deduplicate.** Providers redeliver. Record the provider's event ID durably, in the same transaction as the resulting state change, so a repeat delivery is a no-op (see `reliable-messaging`, workflow 2).
3. **Parse.** Extract the relevant data from the vendor-specific payload into a generic DTO or array.
4. **Hand Off.** Pass the domain-neutral facts to an application handler to execute the business use case. Acknowledge quickly; if the work is slow, record it and process it asynchronously rather than holding the provider's request open.

See `security-architecture`, workflow 4, for the security side of callbacks.

### 5. Operational Signals
Emit logs, metrics, and traces at delivery and infrastructure boundaries, where request, dependency, and failure context is available. Propagate correlation through queued work, keep telemetry out of domain dependencies, and redact secrets and unnecessary personal data. See [operational-signals.md](references/operational-signals.md).

## Boundaries

### Always Do
- Always treat third-party integrations (like a CRM) and infrastructure systems (like a search engine) as infrastructure adapters (Gateways), not bounded contexts.
- Always name the port (the interface the application depends on) after the business capability it provides (`PaymentGateway`, `SearchIndex`, `Mailer`), never the vendor or transport. The adapter that implements it may name the vendor alongside the capability (`StripePaymentAdapter`), because it is the one vendor-specific class.
- Always limit a Mapper to representation conversion only — preserve meaning, never embed business rules or derivation (that belongs in the domain or an application-layer operation).
- Always inject configuration values (like feature flags) into domain objects as simple scalar values (e.g., `bool $isNewFeatureEnabled`) via constructors.

### Ask First
- Ask before passing a `ConfigResolver` or `FeatureFlagClient` directly into a domain object or application handler.
- Ask before introducing a vendor-type wrapper class that only delegates to the SDK once. Name the adapter after the business capability and inject the SDK directly.

### Never Do
- Never let vendor-specific exceptions bubble up to the application layer.
- Never pass the entire configuration object or framework container into an adapter. Inject only the specific credentials or clients the adapter needs.
- Never mix Domain code with infrastructure SDKs.
- Never violate the dependency rule: Domain must not import database, HTTP, or messaging libraries. If it does, the boundary is broken (this is the classic *leaky abstraction* design-debt smell).
- Never let application or domain code depend on a vendor- or transport-shaped abstraction (`StripeHttpWrapper`, `ElasticsearchHttpClient`). It depends on the capability-named port; only the adapter knows the vendor.
