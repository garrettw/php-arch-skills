---
name: infrastructure-boundaries
description: Use this skill when integrating third-party APIs, SDKs, or infrastructure services (like search engines or email). Provides rules for designing provider adapters, handling webhooks, and preventing vendor lock-in.
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

## Numbered Workflows

### 1. Placing Code in the Right Layer
If adding a new integration or third-party service:
1. **Define the Port.** Create an interface in the Domain or Application layer that describes what the application needs in business terms.
2. **Implement the Adapter.** Create the implementation class in the Infrastructure layer, using the specific vendor SDK.
3. **Verify Imports.** Ensure the Domain/Application code only imports the interface, never the vendor SDK or HTTP client.

### 1.5. Classifying the Adapter
If determining where a new adapter belongs:
1. **Driver Adapters.** Does the adapter receive external input and trigger application logic (e.g., HTTP controllers, CLI commands, message queue consumers)? Place it in `Infrastructure/Http` or the equivalent driver layer.
2. **Driven Adapters.** Does the adapter implement an external concern called by the application (e.g., database repository, email sender, search engine client)? Place it in `Infrastructure/Persistence`, `Infrastructure/Messaging`, or the equivalent driven layer.
3. **Composition Root.** Wire all adapter implementations to their ports in a single location (e.g., the DI configuration or the framework bootstrap) to keep the dependency rule intact.

### 2. Designing a Provider Adapter
If building a new infrastructure adapter:
1. **Define Inputs.** The adapter must take primitive values or domain value objects, not framework request objects.
2. **Translate Exceptions.** Catch vendor-specific exceptions (e.g., `GuzzleHttp\Exception\ClientException`) and wrap them in domain-meaningful exceptions defined by the port (e.g., `PaymentFailedException`).
3. **Format Outputs.** Return primitive values, generic DTOs, or domain objects—never the raw vendor response object (e.g., `Stripe\Charge`).
4. **Reference Examples.** Check the [Adapter Design Examples](references/adapter-design-examples.md) for structural patterns.

### 3. Handling Webhooks and Callbacks
If building an endpoint to receive a vendor webhook:
1. **Authenticate.** Verify the webhook signature in an HTTP middleware or controller.
2. **Parse.** Extract the relevant data from the vendor-specific payload into a generic DTO or array.
3. **Hand Off.** Pass the domain-neutral facts to an application handler to execute the business use case.

## Boundaries

### Always Do
- Always treat third-party integrations (like a CRM) and infrastructure systems (like a search engine) as infrastructure adapters, not bounded contexts.
- Always inject configuration values (like feature flags) into domain objects as simple scalar values (e.g., `bool $isNewFeatureEnabled`) via constructors.

### Ask First
- Ask before passing a `ConfigResolver` or `FeatureFlagClient` directly into a domain object or application handler.

### Never Do
- Never let vendor-specific exceptions bubble up to the application layer.
- Never pass the entire configuration object or framework container into an adapter. Inject only the specific credentials or clients the adapter needs.
- Never mix Domain code with infrastructure SDKs.
- Never violate the dependency rule: Domain must not import database, HTTP, or messaging libraries. If it does, the boundary is broken.
