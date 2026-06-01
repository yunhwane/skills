# Style Guide — Java + Spring Boot

Opinionated coding standards for Java + Spring Boot projects. Consumed by the [`review`](./skills/engineering/review/SKILL.md) skill's Standards axis. Drop a copy into any backend repo and adapt it; the rules below are the defaults.

This guide is a complement to:

- [`CONTEXT.md`](./CONTEXT.md) — domain language (what to call things)
- [`docs/adr/`](./docs/adr/) — architecture decisions (why we chose X over Y)
- [`tdd`](./skills/engineering/tdd/SKILL.md) — test-driven development discipline
- Machine-enforced tooling (Spotless, Checkstyle, PMD, EditorConfig) — formatting; this guide does not duplicate

> **Rule of thumb**: if a linter or formatter can enforce it, this file does not document it. Standards live here only when human judgement is required.

## 1. Java baseline

- **Language version**: Java 21 LTS. Use records, sealed types, pattern matching, switch expressions, and text blocks freely.
- **Source encoding**: UTF-8. No platform-specific line endings (`.editorconfig` handles this).
- **Module system**: not used. Standard Maven/Gradle source layout.
- **Build**: Gradle (Kotlin DSL) or Maven. The choice is ADR-worthy.

## 2. Project layout

Hexagonal (ports & adapters) by default. Package by **bounded context** first, then by layer:

```
com.example.shop
├── ordering                  # bounded context
│   ├── domain                # entities, value objects, domain services, ports
│   ├── application           # use cases / application services
│   ├── infrastructure        # JPA adapters, HTTP clients, message adapters
│   └── interfaces            # REST controllers, request/response DTOs
└── billing
    └── ...
```

Rules:

- Domain code depends on **nothing Spring-specific**. No `@Component`, no `@Autowired`, no Jakarta annotations inside `domain/`.
- Infrastructure adapters implement ports defined in `domain/`.
- Cross-context calls go through application services or domain events, never reach across into another context's `domain/`.

## 3. Naming

- Use vocabulary from [`CONTEXT.md`](./CONTEXT.md). If a term is not in the glossary and is about to appear in a class, method, or variable name, add it to `CONTEXT.md` first.
- Classes: nouns from the domain (`Cart`, `Invoice`, `ShipmentDispatched`). Avoid suffixes that just describe technology (`CartManagerImpl`, `CartHelper`).
- Methods on aggregates: verbs from the ubiquitous language (`cart.add(sku, qty)`, `order.cancel(reason)`). Not `cart.setItems(...)`.
- Booleans: predicates (`isReadyForShipment`, `hasOutstandingPayment`). Never `flag` / `bool`.
- Test classes: `<ClassUnderTest>Test`. Test methods: full sentences in camelCase (`rejectsOrderWhenInventoryIsInsufficient`).

## 4. Records, value objects, immutability

- Prefer **records** for value objects and DTOs (`record Money(BigDecimal amount, Currency currency)`).
- Records may carry behaviour. Put domain operations on the record itself (`money.plus(other)`), not on a utility class.
- All collections passed around are immutable (`List.of`, `List.copyOf`). Aggregates expose `List<LineItem>` as `List.copyOf(internalList)`, never the internal reference.
- `final` on fields and local variables by default. Mutability is a deliberate choice.

## 5. Dependency injection

- **Constructor injection only.** No field `@Autowired`, no setter injection, no `@Inject` on fields.
- Constructor parameters are `final`. No Lombok required, but if Lombok is approved by ADR, use `@RequiredArgsConstructor` and nothing else.
- One bean per concrete class. Multiple implementations of the same port get distinct concrete names; pick via `@Qualifier` only when there's no better factoring.
- Application services and domain services are plain classes; mark adapters with the narrowest Spring stereotype (`@Repository`, `@RestController`, `@Component`).
- `@Configuration` classes wire third-party beans only. They do not contain business logic.

## 6. Domain model

- **Rich models, not anaemic.** Behaviour lives on the object that owns the data. See [`tdd/interface-design.md`](./skills/engineering/tdd/interface-design.md).
- Aggregate roots are the **only** public entry point for their aggregate. Child entities (`LineItem`) are package-private or accessible only via the root.
- Value objects are immutable, equal-by-value, and validated in the constructor (`throw new IllegalArgumentException` for invariant violations).
- Domain services exist only when an operation does not naturally belong to a single aggregate. Name them as verbs (`AssignDriverToShipment`).
- Use sealed interfaces for closed type hierarchies (`sealed interface PaymentMethod permits CardPayment, BankTransfer, GiftCard`).

## 7. Persistence (JPA + Spring Data)

- Repository interfaces are **ports** in the domain package. Spring Data JPA repositories are adapters in `infrastructure/persistence/`.
- The domain never imports `org.springframework.data.*` or `jakarta.persistence.*`. Map domain aggregates to JPA entities at the adapter boundary.
- Use **Flyway** for schema migrations. One change per migration file. Migrations are immutable once merged.
- No business logic in `@Entity` classes. They are persistence shapes; the domain aggregate is separate.
- No `findAll()` on repositories used by application code. Add domain-specific finders (`findActiveByCustomer`, `findPendingShipments`).
- Lazy loading is acceptable in the adapter; eager fetch via `@EntityGraph` when N+1 would matter. Annotate the test that locks the fetch plan.

## 8. Web layer (Spring MVC / WebFlux)

- Controllers contain **no business logic**. They map HTTP to application service calls and back.
- Request/response DTOs are records in `interfaces/rest/`. They are independent of domain types; map explicitly.
- Validation via Jakarta Bean Validation on the request DTO (`@NotNull`, `@Size`). Business-rule validation lives in the domain.
- Error handling via a `@RestControllerAdvice`. Map domain exceptions to HTTP status; never expose stack traces.
- API versioning in the URL (`/v1/orders`) until there is a load-bearing reason to do otherwise (ADR).

## 9. Error handling

- **Throw exceptions for exceptional cases.** Use checked exceptions only at the boundary of an external API the team owns and exposes; otherwise prefer unchecked.
- Domain exceptions are unchecked, named for the violation (`InsufficientInventoryException`, not `BusinessException`).
- Never catch `Exception` or `Throwable` to suppress and continue. If recovery is intentional, catch the specific type and log the decision.
- `try-with-resources` for every `Closeable`. No manual `finally { close(); }`.
- Do not throw `RuntimeException` directly; subclass.

## 10. Null handling

- Method **return** types may be `Optional<T>` when absence is a legitimate result. Do not use `Optional` for parameters or fields.
- Method parameters are non-null by default. Use `Objects.requireNonNull` at constructor boundaries that store the value.
- Collections never return `null`; return an empty collection instead.
- `@Nullable` annotations are not used at scale. Document non-obvious null contracts in javadoc only where it matters.

## 11. Logging

- SLF4J via Spring Boot's default (`org.slf4j.Logger LOGGER = LoggerFactory.getLogger(MyClass.class)`).
- Levels:
  - `ERROR` — a request failed in a way the user notices; pageable.
  - `WARN` — a request succeeded but degraded (fallback used, retry happened); investigatable.
  - `INFO` — significant lifecycle events (startup, scheduled job summary). Not per-request.
  - `DEBUG` — diagnostic detail; off in production by default.
- Structured logs: pass parameters, do not concatenate (`LOGGER.info("user {} placed order {}", userId, orderId)`).
- Never log secrets, tokens, or PII. Validate in code review.

## 12. Concurrency & async

- Prefer virtual threads (Java 21 `Thread.ofVirtual()` or `Executors.newVirtualThreadPerTaskExecutor()`) over thread pools for I/O-bound work.
- Spring `@Async` is acceptable but requires a named executor; never use the default `SimpleAsyncTaskExecutor`.
- Time and randomness are **injected**, never read statically. Use `Clock` (not `Instant.now()`) and a `Supplier<UUID>` or `RandomGenerator` for IDs. See [`tdd/mocking.md`](./skills/engineering/tdd/mocking.md).
- Shared mutable state requires either immutability, `synchronized`, or explicit concurrency primitives — never relied-upon JVM happens-before by accident.

## 13. Tests

Test discipline lives in the [`tdd`](./skills/engineering/tdd/SKILL.md) skill. Key non-negotiables:

- JUnit 5 + AssertJ. No Hamcrest, no Spring's `assertEquals`.
- Integration tests use real infrastructure via **Testcontainers** (Postgres, Redis, Kafka). No mocking of repositories or domain services.
- Mockito is for **boundary adapters only** — external HTTP, payment SDKs, `Clock`, mail. Never for application services, repositories, or domain types.
- `@SpringBootTest` is the last resort. Prefer test slices: `@DataJpaTest`, `@WebMvcTest`, `@JsonTest`.
- One behaviour per test. Test names read like specifications.

## 14. Build, formatting, static analysis

- Formatting: Spotless with a fixed Google Java Format version. Commit the wrapper so `./gradlew spotlessApply` is reproducible.
- Style: Checkstyle for the rules above that can be mechanically checked (final fields, no field injection, package layering).
- Bug-finding: Error Prone (Gradle) or SpotBugs at warning level in CI; PRs failing these are blocked.
- Coverage: tracked, not gated by absolute percentage. A drop in coverage on touched files is a review prompt, not a CI failure.

## 15. What this guide does not cover

- Whitespace, brace placement, import order — Spotless / `.editorconfig`.
- Method length, cyclomatic complexity — Checkstyle / PMD warnings, not hard rules.
- Branch naming, commit message format, PR templates — `CONTRIBUTING.md`.
- Specific architecture decisions for this project — `docs/adr/`.
- Domain vocabulary — `CONTEXT.md`.

If a rule belongs in one of those places, put it there and link from here.
