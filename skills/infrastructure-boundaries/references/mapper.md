# Mapper

Different layers often represent the same information differently. A Mapper translates between them. **Each representation serves a different purpose.**

The underlying idea is extremely valuable, and it shows up everywhere a boundary separates two representations: domain object ↔ database row, domain object ↔ DTO, view model ↔ domain, external API response ↔ internal model.

## What has changed: explicit mapping is now a choice, not a given

Our *attitude* toward mapping has shifted, but the architectural responsibility has not gone away:

- Some teams prefer **libraries or code generation** (automapper-style tooling, ORM metadata mapping).
- Others embrace **hand-written mappers** for explicitness and control.

Either way, *something* still has to translate between the representations. The decision is how the mapping is expressed, not whether it exists.

## Mapper is not a Transformer

Many teams confuse **Mapping** with **Transformation**. Keep them separate:

- A **Mapper** *preserves meaning while adapting representation*. Same fact, different shape. `Customer` ↔ `customer_rows`, `Order` ↔ `OrderDto`.
- A **Transformer** *changes meaning* — it applies business rules, derives new facts, enforces policy.

> A Mapper should preserve meaning while adapting representation. It shouldn't become a place for business rules.

If business logic creeps into a mapper, the boundary has leaked: the rule now lives in a translation object where it can't be reasoned about alongside the rest of the domain, and the domain loses ownership of its own behavior. Put derivation and policy in the domain (or an application-layer operation); keep the mapper to shape-shifting only.

## Good vs bad mapper

| Good (preserves meaning, no rules)       | Bad (becomes a transformer)               |
|------------------------------------------|-------------------------------------------|
| `CustomerMapper::toEntity($row)`         | `CustomerMapper` that computes a discount |
| `OrderToDtoMapper::toDto($order)`        | Mapper that decides eligibility           |
| Plain field copy + structural adaptation | Mapper calling a pricing service          |

## Related
- A Mapper is a boundary-translation object, the same seam family as a [Gateway](gateway.md): both isolate one side from the other's representation.
- The persistence-specific form is the **Data Mapper** in [persistence-patterns](../persistence-patterns/references/data-mapper.md) — domain object ↔ database row. That is one instance of this general pattern; do not put mapping logic in the domain object or repository (see [persistence-patterns](../persistence-patterns/SKILL.md)).
- Mapping *to* the wire shape is the domain→DTO direction covered in [distribution-patterns](../distribution-patterns/references/dto.md).
