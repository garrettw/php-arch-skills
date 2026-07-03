# Test Patterns by Architecture Layer

Examples of appropriate testing styles for different architectural layers.

## 1. Domain Test (Pure PHP)

Notice the lack of mocks, databases, or frameworks. Just objects and behavior.

```php
namespace App\Test\TestCase\Domain\Billing;

use App\Domain\Billing\Subscription;
use App\Domain\Billing\ValueObject\Money;
use PHPUnit\Framework\TestCase;

class SubscriptionTest extends TestCase
{
    public function testMarkAsPaidTransitionsStateAndRecordsEvent(): void
    {
        // Given
        $subscription = new Subscription(id: 'sub-123', isPaid: false);
        $now = new \DateTimeImmutable('2024-01-01 12:00:00');

        // When
        $subscription->markAsPaid($now);

        // Then
        $this->assertTrue($subscription->isPaid());
        
        $events = $subscription->releaseEvents();
        $this->assertCount(1, $events);
        $this->assertEquals('sub-123', $events[0]->subscriptionId);
    }
}
```

## 2. Mapper Test

Verify the translation between the framework's data structure and the pure domain object.

```php
namespace App\Test\TestCase\Infrastructure\Persistence\Billing;

use App\Infrastructure\Persistence\Billing\SubscriptionMapper;
use App\Domain\Billing\Subscription;
use PHPUnit\Framework\TestCase;

class SubscriptionMapperTest extends TestCase
{
    public function testMapsOrmEntityToDomainObject(): void
    {
        $mapper = new SubscriptionMapper();
        
        // Given an ORM entity (mocked or real, depending on framework)
        $entity = new \stdClass(); // Simplified for example
        $entity->id = 'sub-123';
        $entity->status = 'active';
        $entity->paid_at = '2024-01-01 12:00:00';
        
        // When
        $domainObject = $mapper->toDomain($entity);
        
        // Then
        $this->assertInstanceOf(Subscription::class, $domainObject);
        $this->assertEquals('sub-123', $domainObject->getId());
        $this->assertTrue($domainObject->isPaid());
    }
}
```

## 3. Handler Test (Integration Style)

For handlers, using a real database and real domain objects provides much higher confidence than mocking every repository call.

```php
namespace App\Test\TestCase\Application\Billing\Handler;

use App\Application\Billing\Handler\ProcessPaymentHandler;
use App\Application\Billing\Command\ProcessPaymentCommand;
use App\Test\Fixture\SubscriptionFixture; // Framework-specific fixture builder
use PHPUnit\Framework\TestCase;

class ProcessPaymentHandlerTest extends TestCase // or Framework integration base class
{
    public function testSuccessfullyProcessesPaymentAndSaves(): void
    {
        // Given
        $subscriptionId = SubscriptionFixture::createUnpaidInDatabase();
        $handler = $this->getContainer()->get(ProcessPaymentHandler::class);
        $command = new ProcessPaymentCommand($subscriptionId);

        // When
        $handler->handle($command);

        // Then
        $updatedRecord = $this->fetchFromDatabase('subscriptions', ['id' => $subscriptionId]);
        $this->assertEquals('paid', $updatedRecord['status']);
        
        // Verify event was dispatched (using a test event dispatcher spy)
        $this->assertEventDispatched(SubscriptionPaid::class);
    }
}
```

## Domain Event Testing (The Three Levels)

When testing domain events, test them where they matter:

1. **In the Domain Test**: Assert that the aggregate recorded the correct event in memory.
2. **In the Handler Test**: Assert that the framework's Event Dispatcher actually received the event after persistence.
3. **In the Listener Test**: Pass a manually constructed Event to the Listener and assert it performs the expected side effect (like dispatching a new Command or queueing a job).
