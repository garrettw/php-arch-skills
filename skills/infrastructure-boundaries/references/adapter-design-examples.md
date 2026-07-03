# Adapter Design Examples

This document demonstrates how to properly design infrastructure adapters that protect your domain from vendor lock-in and leaky abstractions.

## The Goal: Inversion of Control

The Application or Domain layer defines what it needs (the Port). The Infrastructure layer provides the implementation (the Adapter).

### 1. The Port (Domain/Application Layer)

Define an interface using your business language, not the vendor's language.

```php
namespace App\Application\Billing\Port;

use App\Domain\Billing\ValueObject\Money;

interface PaymentGatewayInterface
{
    /**
     * @throws PaymentDeclinedException
     * @throws GatewayUnavailableException
     */
    public function chargeCreditCard(string $token, Money $amount): string;
}
```

### 2. The Adapter (Infrastructure Layer)

Implement the interface using the specific vendor SDK. Catch their exceptions and throw yours.

```php
namespace App\Infrastructure\Stripe;

use App\Application\Billing\Port\PaymentGatewayInterface;
use App\Application\Billing\Port\PaymentDeclinedException;
use App\Application\Billing\Port\GatewayUnavailableException;
use App\Domain\Billing\ValueObject\Money;
use Stripe\StripeClient;
use Stripe\Exception\CardException;
use Stripe\Exception\ApiConnectionException;

class StripePaymentAdapter implements PaymentGatewayInterface
{
    public function __construct(private StripeClient $client) {}

    public function chargeCreditCard(string $token, Money $amount): string
    {
        try {
            $charge = $this->client->charges->create([
                'amount' => $amount->getInCents(),
                'currency' => $amount->getCurrency(),
                'source' => $token,
            ]);
            
            return $charge->id;
        } catch (CardException $e) {
            throw new PaymentDeclinedException($e->getMessage(), 0, $e);
        } catch (ApiConnectionException $e) {
            throw new GatewayUnavailableException('Stripe is currently unreachable.', 0, $e);
        }
    }
}
```

## Anti-Patterns (What to Avoid)

### Anti-Pattern 1: Leaking Vendor Objects

Do not return vendor-specific objects from the adapter. If the application layer receives a `Stripe\Charge` object, the application is now tightly coupled to Stripe.

### Anti-Pattern 2: Leaking Vendor Exceptions

Do not let `GuzzleHttp\Exception\ClientException` or `Stripe\Exception\ApiErrorException` bubble up to the application layer. The application handler shouldn't need to know it was an HTTP failure; it only needs to know the payment gateway is unavailable.

### Anti-Pattern 3: Passing Framework Globals

Do not pass the entire configuration object or framework container into the adapter. Inject only the specific credentials or clients the adapter needs.
