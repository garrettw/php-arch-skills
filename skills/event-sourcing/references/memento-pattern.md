# Memento Pattern

The **Memento** pattern captures and externalizes an object's internal state so it can be restored later, without violating encapsulation.

- **When:** you need to save and rebuild an object's state — undo/redo, checkpoints, or reconstructing an aggregate at a point in time. In event-sourced systems this is realized by replaying events; in CRUD systems a snapshot/memento serves the same purpose.
- **In this skill set:** Memento is the conceptual cousin of **Event Sourcing** ([event-sourcing](../event-sourcing/SKILL.md)). Event sourcing stores a sequence of state *changes* (events) and rebuilds current state by replaying them; a Memento stores a *snapshot* of state directly. Both let you reconstruct past state without mutating the domain's encapsulation — the memento/snapshot is produced by and applied to the aggregate, never assembled from outside by reaching into private fields.
- **Snapshotting:** event-sourced aggregates can grow long replay chains; a periodic Memento/snapshot (see [testing-event-sourced-aggregates.md](testing-event-sourced-aggregates.md)) short-circuits replay. The snapshot is the Memento; the event log is the history.
- **Watch:** keep the memento opaque to the rest of the system — only the originating aggregate should know how to create or restore it. Don't expose internal structure through the memento.

## Related
- [event-sourcing](../event-sourcing/SKILL.md): events vs snapshots as two ways to recover state.
- [domain-modeling](../../domain-modeling/references/behavioral-structural-patterns.md): Memento pairs with **State** when you need to save/restore a lifecycle state.
