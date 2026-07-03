# Glossary Template

This document serves as the canonical domain glossary for the application. As context-specific glossary files are introduced, move context-owned terms out of this global file and into the specific context.

## Product and Domain

- **Product**: [App Name]
- **Shape**: [e.g., e-commerce platform, SaaS reporting tool]
- **Architecture style**: [e.g., modular monolith with bounded contexts]

## Canonical System Terms

*(For each term, provide a clear definition, identify the owning bounded context, and explicitly list terms to avoid. The "Avoid" section is critical for preventing language drift.)*

**[Term 1]**:
[Definition of the term and its lifecycle]
Owned by the **[Owning Context Name]** context.
_Avoid_: [List synonyms or vague terms that should not be used, e.g., generic "module", "bucket", "user" when "learner" is meant]

**[Term 2]**:
[Definition of the term]
Not a bounded context. [Explain why, e.g., it is a third-party integration or a delivery mechanism]. Source business concepts belong to their domain contexts.
_Avoid_: [Explain the trap to avoid, e.g., creating a Notification bounded context just because email templates are shared infrastructure]

**[Term 3]**:
[Definition of the term]
Owned by the **[Owning Context Name]** context.
_Avoid_: [Explain the trap, e.g., conflating identity (User) with participation (Student) across context boundaries]

## Glossary Guidelines

- **Be Precise**: If a term is overloaded (like "Assignment"), prefer a qualified term (like "Group Learning Path Assignment" or "License Assignment").
- **Establish Ownership**: Clearly state which bounded context owns the lifecycle and rules for the term.
- **Clarify Projections**: If a term is a reporting-oriented projection derived from source facts, state that clearly and identify the source context.
