# Interface Design for Testability

Good interfaces fall out of domain-driven, object-oriented design — not the other way around. The same shape that makes a class testable also makes it cohesive, encapsulated, and aligned with the ubiquitous language.

## 1. Accept dependencies, don't create them

Pass collaborators through the constructor. Field `@Autowired` and `new` calls inside business methods both make tests painful and hide the real coupling.

```java
// Testable, intention-revealing
public final class ProcessOrder {
    private final PaymentGateway gateway;
    private final OrderRepository orders;

    public ProcessOrder(PaymentGateway gateway, OrderRepository orders) {
        this.gateway = gateway;
        this.orders = orders;
    }
}

// Hard to test, hidden dependencies
public final class ProcessOrder {
    @Autowired PaymentGateway gateway;

    public void process(OrderId id) {
        var orders = new JpaOrderRepository(/* ... */);
        // ...
    }
}
```

Constructor injection also makes the dependency graph explicit — reading the constructor tells you everything the class needs.

## 2. Return values, don't mutate

Pure, value-returning methods are trivial to test: call them, assert on the result.

```java
// Testable: Discount is a value object the caller can inspect
public Discount calculateDiscount(Cart cart) { /* ... */ }

// Hard to test: mutates the cart's total, then the test must inspect the cart's state
public void applyDiscount(Cart cart) {
    cart.subtractFromTotal(discountFor(cart));
}
```

Value Objects in DDD are immutable by definition (`Money`, `Quantity`, `Email`). When in doubt, return a new value rather than mutating an existing one.

## 3. Tell, don't ask — keep behavior with the data

Anaemic domain models force callers to pull fields out, compute something, and push the result back in. That logic belongs **on the object**.

```java
// Anaemic — caller reaches inside Money to do the math
var total = order.getTotal();
var discounted = new Money(total.amount().subtract(d.amount()), total.currency());
order.setTotal(discounted);

// Rich — Money knows how to subtract Money; Order knows how to apply a discount
order.applyDiscount(discount);

// inside Order:
//   this.total = this.total.minus(discount.amount());
// inside Money:
//   public Money minus(Money other) { /* ... */ }
```

Tests for the rich version are short because the behavior lives in one place. Tests for the anaemic version repeat the same arithmetic across every caller and break when the rules change.

## 4. Small public surface, deep behavior

A class should expose the smallest set of methods that satisfy its callers. Everything else is package-private or hidden inside the aggregate.

- **Aggregate roots** are the only public entry point for their aggregate. `LineItem` is not directly mutable from outside `Cart`; callers go through `cart.add(sku, qty)` and `Cart` decides what to do.
- **Repositories** expose `findById`, `save`, and a small number of domain-specific finders (`findActiveByCustomer`, `findPendingShipments`) — never `findAll` plus a query builder.
- **Domain services** have one or two methods that read like a verb from the ubiquitous language (`AssignDriverToShipment`, `ReleaseInventoryHold`).

Small surface = fewer test setup paths, fewer parameters per test, less brittle to internal change. This is the same principle as **deep modules** in [deep-modules.md](deep-modules.md).

## 5. Define ports in the domain, adapters in infrastructure

Hexagonal architecture is also testability architecture. The port is an interface that says **what** the domain needs; the adapter says **how** it's done.

```java
// domain layer
public interface OrderRepository {
    Optional<Order> findById(OrderId id);
    void save(Order order);
}

// infrastructure layer
@Repository
class JpaOrderRepository implements OrderRepository { /* ... */ }
```

Domain code depends only on the port. Unit tests drive the port with an in-memory implementation; integration tests drive the JPA adapter against a Testcontainers Postgres. Either way, the domain is unchanged.

## Checklist

- [ ] Every collaborator arrives through the constructor
- [ ] Methods return values; side effects are limited and explicit
- [ ] Behavior lives on the object that owns the data (no anaemic models)
- [ ] Public surface is the smallest set the caller needs
- [ ] Domain depends on ports; adapters live in infrastructure
- [ ] Class and method names use vocabulary from `CONTEXT.md`
