# Creation Pattern: Factory

The **Factory** (Factory Method and Abstract Factory) encapsulates object creation behind a method or interface, so callers get a fully-formed object without knowing the concrete class or the construction details.

- **When:** creating an object involves more than a simple `new` — selecting a variant by input, assembling dependencies, or keeping the concrete class out of the caller's knowledge. Examples: a `PaymentProviderFactory` that returns the right gateway for a config; a `ReportFactory` that builds the correct report type from a format string.
- **In this skill set:** the factory is a creation boundary. Prefer it over a raw `new` inside the domain or application layer when the concrete type depends on runtime input or infrastructure. The factory itself is typically wired at the [composition root](../SKILL.md) (or lives in infrastructure for vendor-specific construction), and returns an interface the domain depends on.
- **Factory vs Builder:** use a **Factory** when the caller wants "give me the right object for X" (variant selection); use a **Builder** ([domain-modeling](../../domain-modeling/references/behavioral-structural-patterns.md)) when the caller wants to assemble one complex object step by step.
- **Watch — don't reach for a Singleton:** a Factory is often misimplemented as a global Singleton/service locator. This skill set treats the **Singleton as an anti-pattern** — see the [Never Do](../SKILL.md) rules: use constructor injection and register the factory in the container at the edge, not a static global instance. A factory is a normal injectable object, not a global.

## Related
- [Plugin](references/plugin.md): a factory is the natural way to supply the Adapter an extension point needs.
- [dependency-injection](../SKILL.md): factories belong at the composition root, returning Ports the domain consumes.
