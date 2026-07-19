# HTTP API Contract (REST)

When a Remote Facade ([remote-facade.md](remote-facade.md)) is exposed as a REST/HTTP interface, it takes on a concrete *contract* in addition to its operation shape: resource naming, method semantics, status codes, a consistent error envelope, pagination/versioning conventions, and basic transport security. This reference covers that contract layer. It is about the HTTP surface, not the domain — keep domain logic out of the controller and in the application layer (see the `application-layer` skill). Code samples are illustrative; the principles are framework-agnostic.

## Resource design

- **Address resources with nouns, not verbs.** The HTTP method already carries the action: `GET /users`, `POST /users`, `GET /users/123`, `PUT /users/123`, `DELETE /users/123`. Avoid `/getUsers`, `/createUser`, `/updateUser/123`. Verb endpoints duplicate the method semantics and break caching/retry expectations.
- **Use consistent plural collection names.** Always `/users` and `/users/123`, never a mix of `/user` and `/users`. Predictable naming lets a client guess one resource from another and keeps documentation/SDK generation clean.
- **Use HTTP methods by their defined semantics.**
  - `GET` — safe, idempotent read. Never mutates state; safe for crawlers, caches, prefetchers.
  - `POST` — create a new resource; *not* idempotent.
  - `PUT` — replace a resource; idempotent.
  - `PATCH` — partial update; not necessarily idempotent.
  - `DELETE` — remove; idempotent.
  - A `GET` (or any safe method) that changes data is dangerous and prevents caching — never do it.
- **Model sub-resources by nesting, but keep it shallow.** `GET /users/123/orders` is fine; deep nesting (`/users/123/orders/456/items/789`) usually means the child should be addressable on its own (`/orders/456/items/789`). Non-resource actions belong as sub-resources or query params (`POST /users/123/activate`), not as top-level verbs.
- **Return semantically correct status codes.** `201` on create, `200`/`204` on success, `400` on bad input, `401` unauthenticated, `403` forbidden, `404` not found, `409` conflict, `422` validation failed, `429` rate limited, `5xx` server error. Do not return `200` with an error body — clients, caches, and monitoring read the status.

## Idempotency & safe retries

- Make safe-by-method operations (`GET`, `PUT`, `DELETE`) naturally idempotent, and **issue idempotency keys** (`Idempotency-Key` header) for non-idempotent `POST`s (payments, charges) so a retried request produces one effect, not a duplicate. Store the key with the result and return the original response on replay.

## Error handling

- **One consistent error format everywhere.** Pick a single envelope — e.g. `{ "error": { "code": "user_not_found", "message": "No user with id 123", "requestId": "…" } }` — and use it on every endpoint. Inconsistent shapes (`{ "error": … }` vs `{ "message": …, "status": … }`) force clients to special-case each call.
- **Machine-readable `code` + human `message`.** The `code` is a stable string clients branch on; the `message` is for humans and may change. Never recycle HTTP status as the only signal.
- **Field-level validation details.** On `422`, return which fields failed and why (`errors: { "email": ["must be a valid address"] }`) so the client can map back to the form.
- **Include a `requestId`** in every error response (and log it) so support can correlate a client report with server logs.
- **Never leak stack traces, internal paths, or secret state** in production error bodies. Return a safe message plus the `requestId`; log the detail server-side.

## Pagination, filtering, sorting

- **Cursor pagination for large/deep datasets.** An opaque `cursor` (pointer to the last item) lets the database seek directly to the next batch — constant cost at any depth. Offset (`?offset=500000`) degrades linearly because the DB scans and discards preceding rows.
- **Offset pagination for simple, bounded collections** where clients need numbered pages and a total count. Either way, **include pagination metadata** (page/limit or cursor, total count or `hasMore`, and next/prev links) so clients don't have to guess whether more results exist.
- **One consistent parameter convention API-wide** (`page`/`perPage` or `cursor`/`limit` — pick one) so every endpoint isn't a special case.
- **Filter and sort via query parameters on `GET`.** Filtering is a read and belongs in the URL (`GET /orders?status=shipped&created_after=…`); using a `POST` body breaks cacheability and shareable links. Offer flexible sort (`?sort=-created_at,name`) backed by indexes rather than a single hardcoded order clients must re-sort.

## Response shape

- **One response envelope convention.** Decide whether resources are returned bare or wrapped (`{ "data": … }`) and apply it uniformly so clients write one parser.
- **One JSON naming convention** (camelCase or snake_case) across the whole API; don't mix.
- **Offer sparse fieldsets** (`?fields=id,name`) for heavy resources so bandwidth-constrained clients fetch only what they need. But be sure to sanitize the fieldset to prevent injection attacks.

## Versioning

- **Version from day one.** Pick a strategy and apply it consistently:
  - **URL path** (`/v1/users`) — most explicit and tooling-friendly; the version is visible in every request.
  - **Accept header** (`Accept: application/vnd.acme.v1+json`) — keeps URLs clean and resource-centric; relies on content negotiation.
- **Stay backward compatible within a version**: deliver features as *additive* changes only. Reserve breaking changes for a new major version.
- **Have a deprecation path.** Announce removals ahead of time, emit `Deprecation` and `Sunset` headers, and publish a migration guide so consumers aren't surprised into an outage.

## Security (transport & input)

- **HTTPS only.** Redirect/refuse plaintext; never transmit API traffic unencrypted.
- **Validate and constrain all input** at the boundary — reject malformed/oversized data before it reaches the domain (this is also Fail Fast; see [clean-code-foundations.md](../../domain-modeling/references/clean-code-foundations.md)).
- **Authenticate and authorize** every request (who the caller is, then what they may do); never trust the client.
- **Configure CORS with an explicit allowlist** — never `*` with credentials.
- **Rate limit** to prevent abuse and protect availability; return `429` with a `Retry-After` hint.
- **Never expose secrets or PII** in responses (passwords, tokens, internal IDs, raw personal data). Shape the response DTO to omit them.

## Documentation

- **Publish an OpenAPI (Swagger) spec** as the machine-readable contract — it drives SDK generation, interactive docs, and contract tests.
- **Ship runnable request/response examples** (e.g. a curl per endpoint) so a consumer reaches a first successful call in minutes, not hours of schema interpretation.

## Relationship to this skill

The sections above are the *concrete HTTP form* of the Remote Facade pattern. The facade still holds no domain logic and remains the single remote surface; these conventions are about how that surface speaks HTTP so clients can integrate predictably. When the same data crosses a boundary but *not* over HTTP (a message handler, an in-process seam), apply the DTO and Remote Facade intent from [remote-facade.md](remote-facade.md) and [dto.md](dto.md) without the REST specifics.
