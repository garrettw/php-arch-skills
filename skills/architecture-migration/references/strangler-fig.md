# Strangler Fig Migration

Martin Fowler's **Strangler Fig** is the umbrella pattern for the incremental, touched-area migrations this skill already prescribes. The name comes from a vine that grows around a host tree, gradually drawing on it until the host can be removed — leaving the fig in its shape. Applied to software: build new behavior *alongside* the legacy system, move behavior across piece by piece, and remove the legacy only once it carries nothing.

This is the antidote to the **big-bang rewrite**, which fails because replacements take too long, users can't wait for features, and much of the old behavior isn't actually wanted (so rebuilding it is waste).

## The Four Activities (Cartwright, Horn & Lewis)
Fowler's colleagues distilled incremental modernization into four high-level activities. Ordering does **not** imply a sequence — they interleave throughout.

1. **Understand the outcomes.** Be crystal clear and aligned on what the modernization should achieve. Revisit regularly; watch for shifting priorities and muddled objectives where groups want different things.
2. **Break the problem into parts.** Legacy systems rarely have clean seams. Identify **seams** — points where you can insert code to split the system (Fowler's [Legacy Seam](https://martinfowler.com/bliki/LegacySeam.html)). A good seam often follows a business capability: extract one need into a new component.
3. **Deliver the parts.** Replace a small, isolated component behind a seam. Low risk, early value, and you learn how the legacy really behaves.
4. **Change the organization.** The legacy rotted because of the design thinking and process that built it. Without a culture/leadership shift (and awareness of [Conway's Law](https://martinfowler.com/bliki/ConwaysLaw.html)), the new system will reaccumulate the same mess.

## How It Maps to This Skill
- **Vertical slice migration** (workflow 2) *is* strangler-fig in practice: build the new Handler next to the old Controller, route to it, then remove the legacy caller.
- **Compatibility shims** (workflow 3) are Fowler's **transitional architecture** — code that exists only to let new and legacy coexist, and that you *delete* when migration completes. Don't mourn its cost; it buys reduced risk and earlier returns.
- **Seams** are where you insert those shims and split the monolith. Well-designed systems already have them; legacy systems usually don't, so finding/creating them is real work.
- **Migration triggers** (workflow 1) operationalize "understand the outcomes": only migrate in areas you're already touching.

## Practical Guidance
- Start with **new features** built in the new style on top of, but separate from, the legacy base — then migrate existing behavior into them over time.
- Keep each replaced component **small** so introducing it is low-risk and the business sees value early.
- Treat transitional architecture as first-class, named, and scheduled for removal (see the shim-management workflow).
- Remember the broader context: incremental delivery is necessary but not sufficient — without organizational change, you replicate the legacy's brittleness.

## Related
- [legacy-displacement-patterns.md](legacy-displacement-patterns.md): the delivery mechanics that operationalize Strangler Fig — Event Interception, Legacy Mimic, Divert the Flow, Revert to Source, and the Feature Parity anti-pattern.

## Further Reading
- Martin Fowler, [Strangler Fig Application](https://martinfowler.com/bliki/StranglerFigApplication.html) (canonical post).
- Patterns of Legacy Displacement: [transitional architecture](https://martinfowler.com/articles/patterns-legacy-displacement/transitional-architecture.html), [uncovering mainframe seams](https://martinfowler.com/articles/uncovering-mainframe-seams.html).
- Fowler's [Legacy Seam](https://martinfowler.com/bliki/LegacySeam.html) and [Conway's Law](https://martinfowler.com/bliki/ConwaysLaw.html).
