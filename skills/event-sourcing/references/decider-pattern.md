# The Decider Pattern

A functional way to model an event-sourced aggregate (after Jérémie Chassaing): state is plain data, and behavior is three pure functions.

```php
final class OrderDecider
{
    public static function initialState(): OrderState
    {
        return OrderState::empty();
    }

    /** @return list<object> new events */
    public static function decide(object $command, OrderState $state, \DateTimeImmutable $now): array
    {
        return match (true) {
            $command instanceof PayOrder => $state->isPaid
                ? throw new OrderAlreadyPaid($state->id)
                : [new OrderPaid($state->id, $now)],
            default => throw new \LogicException('Unsupported command ' . $command::class),
        };
    }

    public static function evolve(OrderState $state, object $event): OrderState
    {
        return match (true) {
            $event instanceof OrderPaid => $state->withPaid(),
            default => $state,
        };
    }
}

// Application handler: load, fold, decide, append (with the outbox)
$history = $this->eventStore->load($orderId);
$state   = array_reduce($history, OrderDecider::evolve(...), OrderDecider::initialState());
$events  = OrderDecider::decide($command, $state, $this->clock->now());
$this->eventStore->append($orderId, $events, expectedVersion: count($history));
```

## Rules
- `decide` makes every business decision and enforces invariants. It may refuse by throwing a domain exception or by returning a Result (see [php-domain-idioms.md](../../domain-modeling/references/php-domain-idioms.md)).
- `evolve` never decides and never fails on a valid stream: an event is a fact that already happened.
- Neither function performs I/O, reads a clock, or generates random IDs. Time, IDs, and external answers are passed in as arguments.
- The handler, not the decider, persists events and hands them to the outbox. This is record-then-publish with the "record" step returning values instead of mutating an object.

## Testing
Given events, when a command, then events (or a refusal): fold the given events with `evolve`, call `decide`, and assert on the returned events. There is no aggregate object and no event release step.

## When to prefer it
Prefer it for rule-heavy aggregates and procedural or rules engines, where state is naturally data and decisions are naturally functions. Prefer the object style when an aggregate has few rules and the team is more familiar with it.
