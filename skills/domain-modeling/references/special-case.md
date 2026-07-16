# Special Case

Instead of returning `null` (or some sentinel that forces defensive checks), return an object that represents the exceptional situation and implements the **same interface** as the "real" object. Examples: `GuestUser`, `AnonymousCustomer`, `EmptyCart`, `NoDiscount`.

It is the domain's polymorphic answer to "what do we do when there's nothing here?" — and it removes enormous amounts of defensive code.

## The problem it solves

Without it:

```php
if ($customer !== null) {
    $rate = $customer->discountRate();
}
```

With it:

```php
$rate = $customer->discountRate(); // GuestCustomer returns 0.0
```

No conditional, no null-check, no scattered `if`. The exceptional case is just another object that knows how to behave.

## Special Case is broader than Null Object

Many developers know this as the **Null Object pattern**. Special Case is the wider idea: it represents *any meaningful exceptional state*, not just absence.

- **Null Object** = absence (`GuestUser`, `EmptyCart`): "there is nothing here."
- **Special Case** = a meaningful alternative state (`NoDiscount`, `AnonymousCustomer`): "there is something here, and it has defined behavior."

Both implement the same interface; the difference is intent. A `NoDiscount` is not "no discount object exists" — it is a real, named business state ("this customer gets no discount"). Model that as a first-class object.

## Strengths
- **Fewer conditionals** — the `if ($x !== null)` ceremony disappears.
- **Polymorphic behavior** — callers treat every case uniformly.
- **Clearer intent** — `GuestUser` / `NoDiscount` name the situation explicitly instead of hiding it behind `null`.
- **Easier testing** — the exceptional path is a concrete object with predictable behavior, not a missing value.

## Weaknesses / when NOT to use it

**It must not hide genuine errors.** The exceptional state has to be a *valid* outcome of the domain, not a cover for a failure.

- ✅ Return `AnonymousUser` when authentication is *optional* — that's a real, expected state.
- ❌ Return `AnonymousUser` because the *database lookup failed* — that's an error, and swallowing it hides a bug.

If the situation is exceptional in the sense of *broken* (lookup failed, invariant violated, upstream timeout), surface it as an exception or an explicit error result. Reserve Special Case for states the domain genuinely recognizes.

## Related
- This is a domain-modeling tactic — define the Special Case as a class implementing the same interface/abstract type as the real object (see the [domain-modeling](../SKILL.md) skill, "Defining Aggregate Boundaries"). Shared-kernel value objects such as `Money` and `UserId` are natural homes for `NoDiscount`/`EmptyX` cases (see [bounded-contexts](../bounded-contexts/SKILL.md)).
- It pairs with a [Repository](../persistence-patterns/references/data-source-patterns.md) that may return a Special Case instead of `null` for "not found" when that is a valid domain state.
