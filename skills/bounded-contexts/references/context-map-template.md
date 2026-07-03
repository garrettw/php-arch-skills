# Context Map Template

This template helps you define the bounded contexts for your application using business capabilities as the primary split.

## Contexts

*(List each bounded context and briefly describe the business facts, lifecycles, and policies it owns.)*

- **[Context Name 1]** — [List of core concepts, e.g., billing address, payment methods, checkout options]
- **[Context Name 2]** — [List of core concepts, e.g., product catalog, asset visibility, search recommendations]
- **[Context Name 3]** — [List of core concepts, e.g., user identity, authentication, session lifecycle, roles]

## Target PHP Path Segments

Use these paths for new or meaningfully refactored Domain/Application code. The canonical context name remains the business-language name above; the context path is only the filesystem/namespace segment.

| Bounded Context   | Domain Path                     | Application Path                     |
|-------------------|---------------------------------|--------------------------------------|
| [Context Name 1]  | `src/Domain/[PathSegment1]/`    | `src/Application/[PathSegment1]/`    |
| [Context Name 2]  | `src/Domain/[PathSegment2]/`    | `src/Application/[PathSegment2]/`    |

## Relationships

*(Document how contexts depend on each other or consume facts from one another.)*

- **[Context 1] → [Context 2]**: [Explain the relationship, e.g., Context 1 paid status contributes to Context 2 access decisions]
- **[Context A] → [Context B]**: [Explain the relationship]

## Shared Kernel Candidates (Cross-Context Concepts)

*(List the minimal, intentional shared concepts)*

- `[IdentityReferenceId]` identity references
- `[ValueObject]` value concept (e.g., Money, DateRange)
- Domain event contracts used for asynchronous integration between contexts

## Boundary Clarifications

*(List specific, nuanced decisions about what belongs where, especially for concepts that could easily be misplaced)*

- [Clarification 1, e.g., SSO login flows belong to Identity, but Enterprise SSO settings belong to Account Management]
- [Clarification 2, e.g., Integration X is a third-party boundary, not a bounded context]

## Revisit Triggers

*(List specific future scenarios that would warrant reconsidering these boundaries)*

- Reconsider **[Potential New Context]** if the app starts owning [specific new capability or lifecycle].
- Reconsider splitting **[Current Context]** if [specific sub-areas] develop distinct policies, workflows, and language.
