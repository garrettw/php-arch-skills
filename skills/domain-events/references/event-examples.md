# Domain Event Examples

This document demonstrates the Record-Then-Publish pattern and how to structure a domain event payload.

## 1. The Domain Event Shape

Keep it simple, immutable, and strictly typed.

```php
namespace App\Domain\Billing\Event;

use App\Domain\Shared\EventInterface;
use DateTimeImmutable;

final class SubscriptionPaid implements EventInterface
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
namespace App\Domain\Billing;

use App\Domain\Billing\Event\SubscriptionPaid;
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
     * @return EventInterface[]
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

The application handler orchestrates loading, acting, saving, and then publishing.

```php
namespace App\Application\Billing\Handler;

use App\Domain\Billing\SubscriptionRepositoryInterface;
use Psr\EventDispatcher\EventDispatcherInterface;

class ProcessPaymentHandler
{
    public function __construct(
        private SubscriptionRepositoryInterface $repository,
        private EventDispatcherInterface $dispatcher
    ) {}

    public function handle(string $subscriptionId): void
    {
        $subscription = $this->repository->get($subscriptionId);
        
        // 1. Act
        $subscription->markAsPaid(new \DateTimeImmutable());
        
        // 2. Persist
        $this->repository->save($subscription);
        
        // 3. Publish (only happens if save() did not throw)
        foreach ($subscription->releaseEvents() as $event) {
            $this->dispatcher->dispatch($event);
        }
    }
}
```

## 4. The Listener (Cross-Context Integration)

Another context listens to the event. In a modular monolith, this often means adapting the event into a Command for the local context.

```php
namespace App\Application\Entitlements\Listener;

use App\Domain\Billing\Event\SubscriptionPaid;
use App\Application\Entitlements\Command\GrantAccessCommand;
use App\Application\Entitlements\Handler\GrantAccessHandler;

class GrantAccessOnSubscriptionPaidListener
{
    public function __construct(
        private GrantAccessHandler $handler
    ) {}

    public function __invoke(SubscriptionPaid $event): void
    {
        // Translate the remote fact into a local command
        $command = new GrantAccessCommand(
            accountId: $event->accountId,
            tierName: $event->tierName
        );
        
        $this->handler->handle($command);
    }
}
```
