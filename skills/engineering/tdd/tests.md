# Good and Bad Tests

## Good Tests

**Behavior-style**: Drive aggregates through their public API. Verify the outcome the caller cares about, not the internal mechanics.

```java
@Test
@DisplayName("customer can check out with a valid cart")
void customerCanCheckOutWithValidCart() {
    var cart = Cart.empty(customerId);
    cart.add(productSku, Quantity.of(2));

    var result = checkoutService.checkOut(cart, paymentMethod);

    assertThat(result.status()).isEqualTo(CONFIRMED);
}
```

Characteristics:

- Reads like a domain specification — test name uses ubiquitous language (`Cart`, `Quantity`, `checkOut`)
- Exercises the aggregate through its public methods only
- Asserts on a behavior outcome (`CONFIRMED`), not on internal collaborators
- Survives internal refactors — splitting `Cart` into `Cart` + `LineItem`, swapping the persistence adapter, or extracting a domain service does not break it

For integration coverage, use Spring test slices over hand-built mocks:

```java
@DataJpaTest
@Testcontainers
class JpaCartRepositoryTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16");

    @Autowired CartRepository cartRepository;

    @Test
    void savedCartIsRetrievableByItsId() {
        var cart = Cart.empty(CustomerId.newOne());
        cart.add(SKU.of("BOOK-1"), Quantity.of(3));

        cartRepository.save(cart);

        var loaded = cartRepository.findById(cart.id()).orElseThrow();
        assertThat(loaded.lines()).hasSize(1);
        assertThat(loaded.total()).isEqualTo(cart.total());
    }
}
```

Testcontainers gives a real Postgres so the test verifies the **adapter**, not a fiction. `@DataJpaTest` keeps the slice narrow — only JPA components are loaded.

## Bad Tests

**Implementation-detail tests**: Coupled to internal collaborators or persistence shape.

```java
// BAD: asserts the choreography between checkout and an internal service
@Test
void checkoutDelegatesToPaymentGateway() {
    when(paymentGateway.charge(any())).thenReturn(success());

    checkoutService.checkOut(cart, paymentMethod);

    verify(paymentGateway).charge(eq(cart.total()));
}
```

Red flags:

- Mocking a collaborator the team owns (the `paymentGateway` adapter is yours — boundary mocks live one level further out, at the Stripe SDK)
- Asserting on `verify(...)` for internal calls — the choreography is implementation, not behavior
- The test breaks the moment `checkOut` is refactored to compute the charge differently, even though the user-visible outcome is identical
- Test name describes *how* (`delegates to`) instead of *what* (`charges the customer`)

Bypassing the aggregate to inspect persistence is the same anti-pattern:

```java
// BAD: bypasses the Cart aggregate to read rows directly
@Test
void addingProductInsertsCartLineRow() {
    cart.add(productSku, Quantity.of(1));
    cartRepository.save(cart);

    var rows = jdbcTemplate.queryForList(
        "SELECT * FROM cart_line WHERE cart_id = ?", cart.id().value());

    assertThat(rows).hasSize(1);
}

// GOOD: verifies through the aggregate's public surface
@Test
void addedProductAppearsInCartLines() {
    cart.add(productSku, Quantity.of(1));
    cartRepository.save(cart);

    var loaded = cartRepository.findById(cart.id()).orElseThrow();
    assertThat(loaded.lines())
        .singleElement()
        .satisfies(line -> {
            assertThat(line.sku()).isEqualTo(productSku);
            assertThat(line.quantity()).isEqualTo(Quantity.of(1));
        });
}
```

The bad version locks the test to the current table layout. Rename a column, switch from JPA to JdbcTemplate, or split the table — the test breaks even though `Cart` still behaves correctly. The good version asks the aggregate the same question a real caller would.

## Naming

Use names from `CONTEXT.md`. Test descriptions are part of the codebase's ubiquitous language — they should read the same way a domain expert speaks.

- GOOD: `void rejectsOrderWhenInventoryIsInsufficient()`
- BAD: `void testOrder1()` / `void shouldReturnFalse()`
