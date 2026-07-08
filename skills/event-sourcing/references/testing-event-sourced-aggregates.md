# Testing Event-Sourced Aggregates

Event-sourced aggregates are exceptionally testable because they have no infrastructure dependencies. Tests run in-memory with plain PHP.

## Test Event Production

Given a command, assert the aggregate records the expected events.

```php
final class OrderTest extends TestCase
{
    public function test_paying_an_order_records_order_paid_event(): void
    {
        $order = Order::start($orderId, $accountId);

        $order->pay($paidAt);

        $events = $order->releaseEvents();

        $this->assertCount(1, $events);
        $this->assertInstanceOf(OrderPaid::class, $events[0]);
    }
}
```

## Test Event Application

Given a stream of events, assert the aggregate reaches the expected state.

```php
final class OrderTest extends TestCase
{
    public function test_replaying_events_restores_state(): void
    {
        $events = [
            new OrderStarted($orderId, $accountId),
            new OrderPaid($orderId, $paidAt),
        ];

        $order = Order::reconstituteFrom($events);

        $this->assertTrue($order->isPaid());
    }
}
```

## Test Invariant Protection

Assert that illegal state transitions are rejected.

```php
final class OrderTest extends TestCase
{
    public function test_cannot_pay_an_already_paid_order(): void
    {
        $order = Order::start($orderId, $accountId);
        $order->pay($paidAt);

        $this->expectException(DomainException::class);
        $order->pay($laterDate);
    }
}
```

## Key Rules

- Construct aggregates directly with `new` for command tests.
- For state tests, use a `reconstituteFrom(array $events)` factory method that applies each event via the aggregate's `apply()` or `whenXxx()` methods.
- Never boot a framework, database, or event store for domain tests.
- Assert on event types and payloads, not on internal method calls.
