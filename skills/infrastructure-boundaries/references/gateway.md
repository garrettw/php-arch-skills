# Gateway

A Gateway encapsulates communication with an external system. The application shouldn't know *how* external communication happens — only *that* it happens.

> The application should know that the payment was made. It should not know it went over HTTPS to Stripe.

## What a Gateway wraps

Anything outside your process that the application depends on:

- payment provider (Stripe)
- SMTP server / email delivery
- cache (Redis)
- search engine (Elasticsearch)
- weather API
- object storage (AWS S3)

The common thread: the application talks to a *capability*, never to the transport.

## Modern terminology: Gateway ≈ Port + Adapter

This is one area where the vocabulary has shifted. Many developers would now call this a **Port + Adapter** rather than a Gateway. The concepts are largely equivalent — Hexagonal Architecture simply generalized the idea into a broader architectural philosophy:

- **Gateway** = the seam that hides one external system.
- Hexagonal's **Port** (interface, defined in the Domain/Application layer) + **Adapter** (implementation, in the Infrastructure layer) = the same seam, made explicit and generalized to *every* external dependency.

In this skill set, the [infrastructure-boundaries](../SKILL.md) skill is the Port/Adapter treatment of exactly this idea: define a business-language Port, implement it as an Adapter in the Infrastructure layer, and keep the SDK out of the core. A Gateway *is* that adapter; the naming below is what makes it a good one.

## Naming: represent the capability, not the transport

A Gateway should represent an **external capability**, not an HTTP client.

| Good (speaks the business) | Bad (leaks implementation) |
|----------------------------|----------------------------|
| `PaymentGateway`           | `StripeHttpWrapper`        |
| `SearchIndex`              | `ElasticsearchHttpClient`  |
| `Mailer`                   | `SmtpGuzzleClient`         |

The first speaks the language of the business. The second leaks implementation details — and locks the name (and often the interface) to a vendor or protocol you may later replace.

## Related
- This pattern is the Port/Adapter idea in [infrastructure-boundaries](../SKILL.md); see [Adapter Design Examples](adapter-design-examples.md) for the `PaymentGatewayInterface` / `StripePaymentAdapter` structure and the anti-patterns (leaking vendor objects, leaking vendor exceptions, passing framework globals).
- The data crossing into or out of a Gateway is usually a [DTO](../distribution-patterns/references/dto.md) — primitive values, domain value objects, or a generic transfer object, never the raw vendor response.
