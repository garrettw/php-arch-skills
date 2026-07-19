# Writing Tasteful Code (the antithesis of AI-generated "slop")

The following are phrased as what to do, so that the unwanted patterns fall out automatically. They are about readability and maintenance cost, not metrics — code that passes every lint check can still be hard to maintain.

## Naming
- Reach for names that say what the value *is* or *means* in the domain, not what type it is. Prefer `invoice` over `data`, `customer` over `result`, `pendingOrders` over `list`.
- Keep names concise. A name like `theUserWhoIsCurrentlyLoggedIn` is harder to scan than `currentUser`; a register of short, consistent names reads as human-maintained.
- Name classes after the responsibility or business capability (`PaymentGateway`, `InvoiceCalculator`), not with catch-all suffixes (`*Helper`, `*Manager`, `*Util`, `*Wrapper`) unless the word earns its place.

## Comments
- Write comments only when they add information the code does not already clearly state — the *why* behind a non-obvious decision, a constraint, a workaround, or a particularly obtuse algorithm or block of code.
- Let the function and variable names carry the *what*; do not narrate the next line (e.g. `// create user` above `User::create(...)`).
- Keep docblocks meaningful. A `/** Get the user */` above a typed `getUser()` restates the signature and adds noise. Mark genuine, time-boxed workarounds with `// HACK:` / `// XXX:` so they are findable, not invisible.
- Only use docblock-style comments (`/** ... */`) where they should actually be understood by tooling (e.g. on declarations of classes, methods, properties, and variables).
- Only include PHPDoc-style directives in docblocks (e.g. `@param`, `@return`) when there is information to convey that is not already present in the signature (e.g. what an empty array means).

## Structure & abstraction
- Make a class the right size for its one job; if it would wrap a single function called from one place, write the function instead.
- Introduce an interface only when a second implementation, a boundary to invert, or a decorator is real — not "for flexibility." One interface with one implementation is dead weight.
- Reuse an existing dependency before adding a new one that does the same job.
- Split a change so each class/file has a clear, single reason to change; if editing five files for one behavior, that responsibility is scattered.

## Defensive code
- Guard the places that can actually fail — external calls get timeouts and the real caller handles the error. Do not wrap code that cannot throw in broad `catch` blocks that only log.
- Check for `null` only where a value can genuinely be null; after a non-null assertion or a type guarantee, the check is noise.

## Tests
- Assert on the outcome (state, return value, recorded events), not on private methods or the exact internal behavior.
- Mock only at boundaries (DB, HTTP, queue); a test that mocks everything and re-encodes the implementation verifies the implementation, not the behavior, and breaks on every refactor.

## Style
- Remove debug artifacts (`var_dump`, `dd()`, `dump()`) before code is considered done; they do not belong in delivered code.
