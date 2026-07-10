# Data Source Architectural Patterns (Martin Fowler / PoEAA)

The ways business logic talks to storage can be grouped into a progression of **increasing separation between business logic and persistence**:

```
Table Data Gateway  →  Row Data Gateway  →  Active Record  →  Data Mapper
 (schema-bound,        (one gateway          (row + behavior   (objects fully
  lightest)             per row)              in one object)    isolated from DB)
```

This is a *different* axis from the domain-logic spectrum discussed in the `domain-modeling` skill. The data-source spectrum is purely about **how persistence is structured**, independent of how business rules are organized. A rich Domain Model can sit on top of a Data Mapper; a Transaction Script can sit on top of a Table Data Gateway.

## The core philosophy

This entire category reveals one architectural principle:

> **Persistence should be isolated from business logic to whatever degree the application's complexity warrants.**

The spectrum is not a ladder you are meant to climb to the top. Each pattern trades a different amount of separation for a different cost, and the "right" choice is the lightest one that fits the problem in front of you.

Active Record and Data Mapper are **not competitors with a single winner** — they optimize for different goals:

- **Active Record** optimizes for **developer throughput and simplicity**.
- **Data Mapper** optimizes for **architectural longevity and domain isolation**.

Neither is universally superior. The common mistake is choosing one based on fashion (or team dogma) instead of the actual complexity of the problem being solved. Pick the pattern by the problem, not the other way around.

## The crucial distinction: Gateway vs Repository

This is subtle but extremely important, and inexperienced developers routinely get it wrong.

|           | Table Data Gateway                       | Repository                              |
|-----------|------------------------------------------|-----------------------------------------|
| Thinks in | **tables**                               | **business concepts**                   |
| Answers   | "How do I fetch rows from this table?"   | "How do I retrieve domain objects?"     |
| Language  | persistence terms (rows, columns, joins) | ubiquitous language (Customer, Invoice) |
| Returns   | rows / recordsets                        | domain objects                          |

**A Table Data Gateway is not a Repository.** Everything in a gateway is expressed in terms of persistence, not business behavior. A Repository, by contrast, hides the table thinking behind business concepts. Many teams name a gateway `CustomerRepository` and blur the line — but the naming does not change what it is.

Modern equivalents of a Table Data Gateway go by many names: **query services, table repositories, DAOs (Data Access Objects), SQL abstraction classes**. If it thinks in tables, it is a gateway regardless of the label.

## Related: Object-Relational Patterns

The patterns above are about *structure*. Once an ORM is in play, a second family of Object-Relational patterns appears:

- **ORM-internal Behavioral & Structural patterns** — Unit of Work, Identity Map, Lazy Loading, Identity Field, Foreign Key Mapping, Association Table Mapping, Dependent Mapping, Embedded Value. You should understand these as **ORM features**, not implement them. See [Object-Relational Behavioral & Structural Patterns](orm-patterns.md).
- **Patterns surrounding the ORM** — Metadata Mapping, Query Object, Repository, Value Object. These shape the *architecture* around persistence. See [Object-Relational Architecture Patterns](orm-architecture-patterns.md).

## The patterns

- [Table Data Gateway](table-data-gateway.md) — the lightest. One class per table; fetches/persists rows and nothing more. For reporting, ETL, import/export, admin utilities, read models, and CRUD where the schema *is* the model.
- [Row Data Gateway](row-data-gateway.md) — **recommend against** in new code. One object per row, loads/updates/deletes itself but little business behavior; a faded pattern that awkwardly mixes persistence, identity, and row state.
- [Active Record](active-record.md) — a row object that also carries its business behavior and knows how to persist itself. Outstanding for CRUD-heavy apps with modest rules; a liability as the domain grows richer.
- [Data Mapper](data-mapper.md) — domain objects fully isolated from the database; a Mapper moves data between them. The preferred model for complex enterprise software (DDD, Hexagonal/Onion/Clean, CQRS writes).

## In This Skill Set

- The `domain-modeling` skill chooses *how business rules are organized*. This spectrum chooses *how persistence is structured*. They are orthogonal.
- A [Table Module](../domain-modeling/references/table-module.md) is usually paired with a Table Data Gateway that returns the recordset it operates on.
- The CQRS read side (Queries) is a natural home for Table Data Gateways; the write side leans toward Data Mapper + Repository once invariants matter.
