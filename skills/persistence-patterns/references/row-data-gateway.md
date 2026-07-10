# Row Data Gateway (recommend against)

Instead of representing a whole table, each object represents **exactly one database row**. It knows how to load itself, update itself, and delete itself — but it does **not** necessarily contain business behavior.

It attempted to bring object-oriented structure to relational rows without introducing a full Domain Model. In the early 2000s, many developers wanted object APIs without fully embracing domain modeling, and Row Data Gateway was the practical compromise: objects represented *data*, not necessarily *business concepts*.

**This pattern has largely faded** — not because it is incorrect, but because newer patterns provide better separation. The core problem is that the object simultaneously represents persistence, identity, and row state, but not really business behavior. That makes it an awkward middle ground, and developers naturally drift toward either **Active Record** (if behavior on the row is acceptable) or **Data Mapper** (if the domain should be decoupled from the database).

**Recommendation: do not reach for Row Data Gateway in new code.** It is included here for historical literacy as an evolutionary step in ORM design, not as a pattern to apply. If you just need raw row access, a [Table Data Gateway](table-data-gateway.md) is simpler (no per-row object overhead). If you want behavior on the row, use [Active Record](active-record.md). If you want the domain decoupled from the database, use a [Data Mapper](data-mapper.md) with repositories.
