# Test Conventions

> Detailed testing rules. See [AGENTS.md](../AGENTS.md) for quick summaries and the full agent workflow.

---

## Test Naming Convention

<!-- CUSTOMIZE: Replace with your project's naming pattern if different -->
**Format:** `{method}_{condition(s)}__{expected_result}` — all lowercase with underscores, double underscore (`__`) separates conditions from expected result.

```java
@Test
void is_valid_with_valid_date_range__returns_true() { }

@Test
void calculate_order_total_with_multiple_items__returns_correct_sum() { }

@Test
void validate_order_with_null_customer__throws_validation_exception() { }
```

**Rules:**
- All lowercase with underscores
- Method names should be self-documenting
- Apply consistently across all test methods

---

## Given-When-Then Structure

Every test method follows the Given-When-Then pattern:

```java
@Test
void method_name_with_condition__expected_result() {
    // Given
    SomeEntity entity = createValidEntity();
    when(mockService.method()).thenReturn(expectedValue);

    // When
    Result result = serviceUnderTest.performOperation(entity);

    // Then
    assertThat(result.isSuccess()).isTrue();
    assertThat(result.getValue()).isEqualTo(expectedValue);
}
```

---

## Integration Tests

Integration tests should test real data persistence and retrieval without mocking repository layers.

```java
<!-- CUSTOMIZE: Replace import path with your project's TestUtils location -->
import static com.example.project.core.TestUtils.FAKER;

class ServiceIntegrationTest extends AbstractServiceTest {

    @Autowired
    private ServiceUnderTest serviceUnderTest;

    @Autowired
    private EntityRepository entityRepository; // Real repository, not mocked

    @Test
    void create_entity_with_valid_data__persists_to_database() {
        // Given — prepare test data with random values
        String entityName = FAKER.company().name();
        CreateEntityRequest request = CreateEntityRequest.builder()
            .name(entityName)
            .build();

        // When — execute service method
        Long entityId = serviceUnderTest.createEntity(request);

        // Then — verify data was persisted
        Optional<Entity> savedEntity = entityRepository.findById(entityId);
        assertThat(savedEntity).isPresent();
        assertThat(savedEntity.get().getName()).isEqualTo(entityName);
    }
}
```

**Rules:**
- `@Autowired` real repositories — never mock them in integration tests
- **ALWAYS use randomly generated data (`FAKER`)** — hardcoded values cause conflicts
- Store generated values in variables when you need to assert against them later

---

## Parameterized Tests

Use `@ParameterizedTest` + `@MethodSource` for multiple similar test cases:

```java
@ParameterizedTest
@MethodSource("testCases")
void method_with_input__returns_expected(InputType input, OutputType expected) {
    // Given
    Entity entity = createEntity(input);

    // When
    OutputType result = service.process(entity);

    // Then
    assertThat(result).isEqualTo(expected);
}

private static Stream<Arguments> testCases() {
    return Stream.of(
        Arguments.of(InputType.A, OutputType.X),
        Arguments.of(InputType.B, OutputType.Y)
    );
}
```

---

## Controller Tests

<!-- CUSTOMIZE: Replace with your project's controller test pattern -->
Always assert both handler type and method:

```java
.andExpect(handler().handlerType(YourController.class))
.andExpect(handler().method(YourController.class.getMethod(
    "methodName",
    ParameterType.class
)))
```

---

## Test Base Classes

<!-- CUSTOMIZE: Replace with your project's test base classes and their locations -->
| Base Class | Use When | Context |
|---|---|---|
| `AbstractServiceTest` | Testing service logic with real DB | Full Spring Boot context; external services `@MockitoBean`, real repositories |
| `AbstractWebMvcTest` | Testing controllers in isolation | MockMvc, all services `@MockitoBean` |
| `AbstractWebMvcFullContextTest` | Testing controllers with full app context | Full Spring Boot context + MockMvc |

---

## Test Helper Services

<!-- CUSTOMIZE: Replace with your project's test helpers -->
Builder-pattern services for creating test entities:

| Service | Creates |
|---|---|
| `TestUserService` | User entities with sample values |
| `TestOrderService` | Order entities with transport data |

`@Autowired` these in your test classes extending `AbstractServiceTest`.

---

## Test Data

<!-- CUSTOMIZE: Replace with your project's TestUtils import path -->
**Import:** `import static com.example.project.core.TestUtils.FAKER;`

Key utilities:
- `FAKER` — DataFaker instance for generating realistic test data
- `randomBigDecimal(min, max)` — random BigDecimal values

---

## Anti-Patterns (DO NOT)

1. **DO NOT use `@SpringTransactional` on test methods** — it rolls back changes automatically and hides transaction-related bugs
2. **DO NOT mock repositories in integration tests** — use real repositories to test actual DB interactions
3. **DO NOT hardcode test values** — use `FAKER` for random data to avoid conflicts
4. **DO NOT duplicate tests** — use `@ParameterizedTest` + `@MethodSource` for similar test cases
