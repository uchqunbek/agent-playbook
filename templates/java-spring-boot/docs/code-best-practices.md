# Code Best Practices

> Detailed code conventions with examples. See [AGENTS.md](../AGENTS.md) for quick summaries.

---

## Formatting Rules

<!-- CUSTOMIZE: Replace line length and formatting rules with your project's conventions -->
- **Line length limit: 120 characters** (see `.editorconfig`)
- Keep method arguments on a single line when the signature stays within 120 characters;
  if it exceeds 120 characters, break right after the opening parenthesis and place each
  argument on its own line
- Apply the same line-length rule to `record` component lists
- Prefer `getFirst()` over `get(0)` when accessing the first element of a list
- Do not leave obvious comments — use self-descriptive methods and variables instead

```java
// ✅ Signature fits within 120 characters — single line
public ResponseEntity<OrderResponseDTO> saveOrder(AuthUser user, OrderRequestDTO request) {
    return orderService.save(user, request);
}

// ✅ Signature exceeds 120 characters — break after opening parenthesis
public OrderBatchResponseDTO saveBatch(
    @CurrentAuthenticatedUser AuthenticatedUser user,
    @Valid @RequestBody OrderBatchRequestDTO request
) {
    return batchOrderService.saveBatch(user, request);
}

// ✅ Short record — single line
public record SimpleRecord(Long id, String name) {}

// ✅ Long record — one component per line
public record DetailedRecord(
    Long id,
    String name,
    @JsonProperty("external_id") String externalId
) {}
```

---

## Bug Prevention

```java
// ✅ Proper null checks
public void processOrder(Order order) {
    if (order == null) {
        throw new IllegalArgumentException("Order cannot be null");
    }
}

// ✅ Use Optional instead of null returns
public Optional<User> findUser(Long id) {
    return userRepository.findById(id);
}

// ✅ Close resources properly
try (BufferedReader reader = Files.newBufferedReader(path)) {
    return reader.lines().collect(Collectors.toList());
}

// ✅ Handle InterruptedException properly
try {
    Thread.sleep(1000);
} catch (InterruptedException e) {
    Thread.currentThread().interrupt();
    throw new ProcessingException("Thread was interrupted", e);
}
```

---

## Performance

```java
// ✅ Use String.formatted() for multiple concatenations
String message = "Order %s processed successfully".formatted(orderId);

// ✅ Use streams efficiently
List<String> activeOrderIds = orders.stream()
    .filter(Order::isActive)
    .map(Order::getId)
    .toList();

// ✅ Always use merge function with Collectors.toMap to handle duplicates
Map<String, Order> ordersByCustomerId = orders.stream()
    .collect(Collectors.toMap(
        Order::getCustomerId,
        Function.identity(),
        (existing, replacement) -> existing
    ));

// ❌ No merge function — throws IllegalStateException on duplicates
Map<String, Order> ordersByCustomerId = orders.stream()
    .collect(Collectors.toMap(Order::getCustomerId, Function.identity()));
```

---

## Code Smells

```java
// ✅ Meaningful variable names
String customerCompanyName = getCompanyName();
List<Order> activeOrders = getActiveOrders();

// ❌ Abbreviations and unclear names
String ccn = getCompanyName();
List<Order> ords = getActiveOrders();

// ✅ Use constants instead of magic numbers (not required in tests)
private static final int MAX_RETRY_ATTEMPTS = 3;
private static final int CONNECTION_TIMEOUT_MS = 5000;

// ✅ Avoid deep nesting — use early returns
public ProcessResult processOrder(Order order) {
    if (order == null) {
        return ProcessResult.error("Order is null");
    }
    if (!order.isValid()) {
        return ProcessResult.error("Order is invalid");
    }
    return ProcessResult.success(doProcessing(order));
}

// ✅ Use enums instead of constants
public enum OrderStatus {
    PENDING, CONFIRMED, IN_PROGRESS, COMPLETED, CANCELLED
}
```

---

## Maintainability

```java
// ✅ Use dependency injection
@Service
@RequiredArgsConstructor
public class OrderService {
    private final OrderRepository orderRepository;
    private final OrderValidator orderValidator;
}

// ✅ Avoid long method parameter lists — use parameter objects
public Order createOrder(CreateOrderRequest request) {
    // Use request object instead of many parameters
}

// ✅ Use Builder pattern for complex objects
Order order = Order.builder()
    .customerId(customerId)
    .items(items)
    .deliveryAddress(address)
    .build();
```

---

## Security

```java
// ✅ Use parameterized queries
@Query("SELECT u FROM User u WHERE u.email = :email")
User findByEmail(String email);

// ✅ Don't hardcode credentials
@Value("${database.password}")
private String databasePassword;
```

---

## Complete Rules Checklist

<!-- CUSTOMIZE: Adjust checklist to match your project's priorities -->
- **Naming:** Clear, meaningful names; avoid abbreviations
- **Magic Numbers:** Use named constants (not required in tests)
- **Resource Management:** Try-with-resources, proper cleanup
- **Exception Handling:** Don't ignore exceptions; log and handle properly
- **Null Safety:** Use Optional, null checks, `Objects.equals()`
- **Security:** Parameterized queries, input validation, no hardcoded secrets
- **Method Complexity:** Keep methods focused, avoid deep nesting
- **Performance:** Appropriate collections, always use merge function with `Collectors.toMap`
- **Maintainability:** Single responsibility, dependency injection, builder pattern
