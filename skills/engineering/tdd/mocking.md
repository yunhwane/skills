# When to Mock

Mock at **system boundaries** only:

- External HTTP APIs (payment provider, mail provider, third-party fulfillment)
- External infrastructure the team does not own (Stripe SDK, AWS SDK)
- Time and randomness (`Clock`, ID generators)
- File system (sometimes — prefer a temp directory)

**Do not mock**:

- Aggregates, value objects, or any domain object defined inside this bounded context
- Domain services and application services written by the team
- Repository interfaces — these are **ports** owned by the domain. Test them against a real adapter using Testcontainers, or against an in-memory implementation, not `Mockito.mock(OrderRepository.class)`
- Anything inside the bounded context

If you find yourself writing `when(orderService...)` for a service that lives in this codebase, you are testing the wrong layer. Drop the mock and exercise the real object.

## Spring Caveats

`@MockBean` and `@SpyBean` make it trivial to mock anything in the Spring context. Resist the temptation:

- Use `@MockBean` only for **boundary adapters** (`PaymentGatewayClient`, `MailSender`). Never for application services, domain services, or repositories owned by the codebase.
- Prefer constructor injection so production wiring and test wiring share the same path — no Spring context required for unit tests.
- For integration tests, prefer **test slices** (`@DataJpaTest`, `@WebMvcTest`) over `@SpringBootTest`. They start fewer beans, which means fewer reasons to reach for `@MockBean`.

## Designing for Mockability

The way production code is written determines whether mocks are needed at all.

### 1. Inject collaborators through the constructor

Accept dependencies — never construct them inside business logic.

```java
// Easy to test: gateway is injected
public final class CheckoutService {
    private final PaymentGateway gateway;
    private final Clock clock;

    public CheckoutService(PaymentGateway gateway, Clock clock) {
        this.gateway = gateway;
        this.clock = clock;
    }

    public CheckoutResult checkOut(Cart cart, PaymentMethod method) {
        return gateway.charge(cart.total(), method, clock.instant());
    }
}

// Hard to test: gateway and time are hidden
public final class CheckoutService {
    public CheckoutResult checkOut(Cart cart, PaymentMethod method) {
        var gateway = new StripeGateway(System.getenv("STRIPE_KEY"));
        return gateway.charge(cart.total(), method, Instant.now());
    }
}
```

With constructor injection the unit test wires a fake `PaymentGateway` and a fixed `Clock` directly — no Spring context, no static mocking, no surprises.

### 2. Define the port in the domain; put the adapter in infrastructure

```java
// domain — owned by the bounded context
public interface PaymentGateway {
    ChargeResult charge(Money amount, PaymentMethod method, Instant when);
}

// infrastructure — Spring-managed adapter
@Component
class StripePaymentGateway implements PaymentGateway {
    private final WebClient webClient;
    // ...
}
```

In tests, mock the **port** (`PaymentGateway`) — not the Stripe SDK or `WebClient`. The port is a small, intent-revealing interface; the SDK is noisy and changes shape with vendor releases. Hexagonal architecture earns its keep here.

### 3. Prefer SDK-style adapters over a generic HTTP client

```java
// GOOD: each operation is independently mockable
public interface CustomerApi {
    Customer getCustomer(CustomerId id);
    List<Order> getOrders(CustomerId id);
    OrderId createOrder(NewOrder request);
}

// BAD: tests need conditional logic inside the mock
public interface HttpClient {
    <T> T request(String method, String path, Object body, Class<T> responseType);
}
```

The SDK-style approach:

- Each mock returns one specific shape — no `when(http.request(eq("GET"), eq("/orders"), ...))` puzzles
- Types per endpoint, so the compiler catches bad stubs
- It's obvious from the test which capabilities a behavior depends on

### 4. Inject `Clock`, not `Instant.now()`

Anywhere a domain rule depends on time (token expiry, deadline checks, audit timestamps), inject a `Clock`. Production wires `Clock.systemUTC()`. Tests pass `Clock.fixed(Instant.parse("2026-01-01T00:00:00Z"), ZoneOffset.UTC)`. No `Mockito.mockStatic` required.
