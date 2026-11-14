# Domain Event Examples

This document demonstrates the Record-Then-Publish pattern and how to structure a domain event payload.

## 1. The Domain Event Shape

Keep it simple, immutable, and strictly typed.

```php
namespace App\Billing\Domain\Event;

use DateTimeImmutable;

final class SubscriptionPaid
{
    public function __construct(
        public readonly string $accountId,
        public readonly string $subscriptionId,
        public readonly string $tierName,
        public readonly DateTimeImmutable $paidAt,
    ) {}
}
```

## 2. Recording the Event (Domain Layer)

The domain object (aggregate root) makes the decision and records the fact, but does not publish it to the framework.

```php
namespace App\Billing\Domain;

use App\Billing\Domain\Event\SubscriptionPaid;
use DateTimeImmutable;

class Subscription
{
    private array $recordedEvents = [];
    private bool $isPaid = false;

    // ... constructor, etc.

    public function markAsPaid(DateTimeImmutable $now): void
    {
        if ($this->isPaid) {
            return;
        }

        $this->isPaid = true;
        
        // Record the event internally
        $this->recordedEvents[] = new SubscriptionPaid(
            $this->accountId,
            $this->id,
            $this->tierName,
            $now
        );
    }

    /**
     * @return list<object>
     */
    public function releaseEvents(): array
    {
        $events = $this->recordedEvents;
        $this->recordedEvents = [];
        return $events;
    }
}
```

## 3. Releasing and Publishing (Application Layer)

The handler orchestrates: load, act, then save and record the events in the outbox in one transaction. A relay publishes them after commit.

```php
namespace App\Billing\Application\Command;

use App\Billing\Domain\SubscriptionRepository;
use App\Platform\Outbox\Outbox;
use App\Platform\Transaction\TransactionManager;
use Psr\Clock\ClockInterface;

final class ProcessPaymentHandler
{
    public function __construct(
        private SubscriptionRepository $repository,
        private Outbox $outbox,
        private TransactionManager $transactions,
        private ClockInterface $clock,
    ) {}

    public function handle(ProcessPaymentCommand $command): void
    {
        $this->transactions->run(function () use ($command): void {
            $subscription = $this->repository->get($command->subscriptionId);

            // 1. Act: the domain decides and records the event
            $subscription->markAsPaid($this->clock->now());

            // 2. Persist state and events atomically
            $this->repository->save($subscription);
            $this->outbox->record(...$subscription->releaseEvents());
        });
        // 3. The outbox relay publishes after commit; nothing is dispatched here
    }
}
```

## 4. Crossing to Another Context

Billing translates its internal `SubscriptionPaid` into a published contract (for example, in the outbox relay or a small translator). Entitlements consumes only that contract, asynchronously and idempotently.

```php
namespace App\SharedKernel\Contract\Billing;

/** Published contract, version 1. Change only by adding a new version. */
final class SubscriptionPaidV1
{
    public function __construct(
        public readonly string $messageId,
        public readonly string $accountId,
        public readonly string $tierName,
        public readonly string $paidAt, // ISO 8601
    ) {}
}
```

```php
namespace App\Entitlements\Interface\Messaging;

use App\Entitlements\Application\Command\GrantAccessCommand;
use App\Entitlements\Application\Command\GrantAccessHandler;
use App\SharedKernel\Contract\Billing\SubscriptionPaidV1;

final class GrantAccessOnSubscriptionPaid
{
    public function __construct(private GrantAccessHandler $handler) {}

    public function __invoke(SubscriptionPaidV1 $message): void
    {
        // Translate the published fact into a local command. The handler runs its own
        // transaction and records $message->messageId in the inbox, so redelivery is a no-op.
        $this->handler->handle(new GrantAccessCommand(
            accountId: $message->accountId,
            tierName: $message->tierName,
            causationId: $message->messageId,
        ));
    }
}
```
