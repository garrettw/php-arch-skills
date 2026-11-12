---
name: security-architecture
description: Use this skill when designing authentication, authorization, tenant isolation, sensitive-data handling, or trust boundaries in a PHP application. Focuses on security responsibilities across domain, application, delivery, and infrastructure layers.
---

# Security Architecture for PHP Applications

## Security Starts at Trust Boundaries

Treat every value from a browser, API client, queue, webhook, file, or third-party system as untrusted until validated for its specific use. Identify the assets, actors, trust boundaries, and high-impact abuse cases before choosing controls. Security checks belong at boundaries and in the layer that owns the policy; framework middleware alone cannot protect business invariants.

This skill addresses architectural placement and decisions, not a complete secure-coding checklist or substitute for threat modeling, dependency scanning, penetration testing, or specialist review.

## Numbered Workflows

### 1. Separating Identity from Permission
When a use case needs access control:
1. **Authenticate at the delivery boundary.** Establish the caller's identity using the project's chosen mechanism. Public operations can intentionally allow anonymous callers.
2. **Authorize the operation.** Check whether that identity may perform this action on this specific resource, not merely whether the caller is signed in.
3. **Enforce business invariants in the core.** If a rule must hold regardless of entry point (HTTP, CLI, queue), enforce it in the application/domain use case, not only in a controller or route middleware.
4. **Fail closed.** Missing, malformed, expired, or unverifiable identity/permission data must not grant access. Return a safe denial without disclosing whether protected resources exist when that distinction is sensitive.

Prefer explicit policies or narrow authorization ports over passing a framework user/session object deep into the application. Keep authentication protocol details in the delivery adapter; pass only the identity and relevant authorization context needed by the use case.

### 2. Enforcing Tenant and Resource Ownership
For multi-tenant or user-owned data:
1. Derive the tenant and subject from verified server-side identity/context, never from an untrusted request field alone.
2. Scope reads and writes by tenant/owner at the data-access boundary, and authorize the requested action at the use-case boundary.
3. Apply the same rule to background jobs, exports, caches, search indexes, and administrative paths; these often bypass HTTP middleware.
4. Test that a caller from tenant A cannot read, mutate, infer, or receive tenant B's data, including by guessing identifiers or replaying a queued message.

Do not treat an unguessable ID as authorization. Do not rely on UI hiding, route naming, or a single controller check to provide tenant isolation.

### 3. Handling Secrets and Sensitive Data
- Minimize collection, message payloads, logs, traces, and API responses to the data required for the task.
- Keep secrets out of source control, exception messages, event payloads, and diagnostic output. Inject only the required secret/configuration into the infrastructure adapter that uses it.
- Define retention and deletion behavior for personal and security-sensitive data, including backups and asynchronous queues.
- Separate business audit records from diagnostic logs. Record who did what and when where required, but avoid copying credentials or unnecessary personal data into the audit trail.
- Use explicit response DTOs and message contracts; never serialize ORM entities or authenticated session/token objects by default.

### 4. Securing External Inputs and Callbacks
- Validate shape, size, type, and allowed values at the edge before application work; use parameterized database operations and context-appropriate output encoding in presentation adapters.
- For webhooks, verify the provider signature over the raw request body, enforce timestamp/replay protections where supported, and durably deduplicate provider event IDs before applying effects.
- For outbound requests, constrain destinations and protocols to prevent server-side request forgery; do not let untrusted input choose arbitrary URLs or internal network targets.
- Rate-limit expensive or abuse-prone operations and apply request/body size limits before expensive parsing.
- Return generic production errors with a correlation ID; keep sensitive diagnostics in access-controlled logs.

### 5. Verifying Security Behavior
Test both allowed and denied outcomes at the actual boundary: unauthenticated access, insufficient permissions, cross-tenant access, stale/replayed callbacks, and malformed or oversized input. Add architecture checks for framework/vendor dependencies in core layers, but do not mistake dependency rules for proof of security.

## Boundaries

### Always Do
- Make the security owner and enforcement point explicit for each use case and entry point.
- Check object-level authorization for the specific resource and action.
- Keep identity-provider and framework types at adapters; pass a minimal, stable identity/context inward.
- Apply least privilege to service identities, database credentials, and infrastructure permissions.
- Review new data flows for secrets and personal data, including logs, queues, caches, and exports.

### Ask First
- Ask whether an endpoint is intentionally public and what abuse controls it needs.
- Ask how account recovery, identity linking, privilege changes, and tenant membership changes are authorized and audited.
- Ask about data classification, retention, residency, and deletion requirements before expanding stored or transmitted data.
- Ask for security review when changing authentication protocols, cryptography, privilege boundaries, or handling regulated data.

### Never Do
- Never equate authentication with authorization.
- Never trust a client-supplied user or tenant ID as proof of ownership.
- Never put secrets in source, URLs, logs, telemetry, or durable messages.
- Never rely on obscurity, UI behavior, or unguessable identifiers as access control.
- Never expose raw exception details or sensitive existence information to untrusted callers.

## Related Skills
- [distribution-patterns](../distribution-patterns/SKILL.md): API contracts and transport-level security for exposed interfaces.
- [infrastructure-boundaries](../infrastructure-boundaries/SKILL.md): webhook verification and secret isolation at provider adapters.
- [application-layer](../application-layer/SKILL.md): use cases enforce authorization-independent business invariants.
- [testing-strategy](../testing-strategy/SKILL.md): tests should cover denied access and tenant isolation behavior.
