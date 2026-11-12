# Operational Signals at Application Boundaries

Observability is most useful when a request or message can be followed across the adapters and use cases that handle it. Keep telemetry at delivery and infrastructure boundaries; do not make the domain depend on a logging, metrics, or tracing vendor.

## Correlation

- Create or accept a validated request/correlation ID at the entry point and include it in the response where appropriate.
- Propagate correlation and trace context through queue messages and outbound calls using the platform's standard context format.
- Record stable operation names, message IDs, and bounded business identifiers where safe; avoid using raw payloads as diagnostic context.
- Ensure asynchronous work can be traced back to its initiating request without relying on process-local state.

## Logs, Metrics, and Traces

- Use structured logs with consistent fields for operation, outcome, duration, and correlation. Log failures once at a boundary that has enough context; avoid logging and rethrowing the same exception at every layer.
- Measure user-relevant outcomes and saturation: request/job duration, error rates, queue age/depth, retry and dead-letter counts, and dependency latency.
- Use traces to diagnose cross-service latency and fan-out, not as a substitute for useful logs or metrics.
- Keep metric labels low-cardinality. Never label metrics with unbounded IDs, emails, URLs, or arbitrary user input.

## Privacy and Failure Behavior

- Redact credentials, tokens, payment data, and unnecessary personal data before telemetry is emitted. Apply access controls and retention limits to logs and traces.
- Telemetry failure should not normally fail a business operation. Make delivery durability an explicit requirement only for records that are themselves part of the business transaction, such as an audit trail.
- Alert on actionable symptoms tied to service objectives; queue depth alone may be normal, while queue age or dead-letter growth may reveal stalled work.

## Related
- [reliable-messaging](../../reliable-messaging/SKILL.md): correlation, retries, and dead-letter handling for asynchronous work.
- [http-api-contract.md](../../distribution-patterns/references/http-api-contract.md): request IDs and safe error responses.
- [infrastructure-boundaries](../SKILL.md): telemetry remains an adapter concern.
