# Test Conventions

> Detailed testing rules for Swift/iOS. See [AGENTS.md](../AGENTS.md) for quick summaries.

---

## Test Structure

### File Organization

<!-- CUSTOMIZE: Replace with your project's test structure -->
Unit tests mirror the source directory structure in a separate test target:

```
YourApp/
├── Features/
│   ├── Drivers/
│   │   ├── DriverListViewModel.swift
│   │   └── DriverDetailViewModel.swift
│   └── Loads/
│       └── ...
YourAppTests/
├── Features/
│   ├── Drivers/
│   │   ├── DriverListViewModelTests.swift
│   │   └── DriverDetailViewModelTests.swift
│   └── Loads/
│       └── ...
├── Core/
│   └── Networking/
│       └── APIClientTests.swift
├── Helpers/
│   ├── MockDriverService.swift
│   └── TestFixtures.swift
YourAppUITests/
└── DriverFlowUITests.swift
```

### Naming

<!-- CUSTOMIZE: Replace with your project's test naming convention -->
- **Function pattern:** `test_{method}_{condition}__{expected}` (double underscore before expected)
- **Descriptive conditions:** `emptyList`, `networkError`, `invalidInput`
- **Descriptive expectations:** `showsEmptyState`, `throwsError`, `returnsNil`

```swift
func test_loadDrivers__setsDriversArray() async { ... }
func test_loadDrivers_networkError__setsError() async { ... }
func test_loadDrivers_emptyResponse__showsEmptyState() async { ... }
func test_deleteDriver_notFound__throwsNotFoundError() async { ... }
```

---

## Arrange / Act / Assert

Every test follows the AAA pattern with explicit section comments:

```swift
func test_loadDrivers__setsDriversArray() async {
    // Arrange
    let expectedDrivers = [Driver.fixture(name: "Alice"), Driver.fixture(name: "Bob")]
    let mockService = MockDriverService(drivers: expectedDrivers)
    let viewModel = DriverListViewModel(service: mockService)

    // Act
    await viewModel.loadDrivers()

    // Assert
    XCTAssertEqual(viewModel.drivers.count, 2)
    XCTAssertEqual(viewModel.drivers.first?.name, "Alice")
    XCTAssertNil(viewModel.error)
}
```

### Error Case Testing

```swift
func test_loadDrivers_networkError__setsErrorAndClearsDrivers() async {
    // Arrange
    let mockService = MockDriverService(error: DriverError.networkUnavailable)
    let viewModel = DriverListViewModel(service: mockService)

    // Act
    await viewModel.loadDrivers()

    // Assert
    XCTAssertTrue(viewModel.drivers.isEmpty)
    XCTAssertNotNil(viewModel.error)
}
```

---

## Protocol Fakes

Use manual protocol conformances — no mocking frameworks:

```swift
final class MockDriverService: DriverServiceProtocol {
    var drivers: [Driver] = []
    var error: Error?
    private(set) var fetchDriversCalled = false
    private(set) var updateDriverCalled = false
    private(set) var lastUpdatedDriver: Driver?

    init(drivers: [Driver] = [], error: Error? = nil) {
        self.drivers = drivers
        self.error = error
    }

    func fetchDrivers() async throws -> [Driver] {
        fetchDriversCalled = true
        if let error { throw error }
        return drivers
    }

    func updateDriver(_ driver: Driver) async throws {
        updateDriverCalled = true
        lastUpdatedDriver = driver
        if let error { throw error }
    }
}
```

### Verifying Interactions

```swift
func test_saveDriver__callsServiceUpdate() async {
    // Arrange
    let mockService = MockDriverService()
    let viewModel = DriverDetailViewModel(service: mockService)
    let driver = Driver.fixture()

    // Act
    await viewModel.save(driver)

    // Assert
    XCTAssertTrue(mockService.updateDriverCalled)
    XCTAssertEqual(mockService.lastUpdatedDriver?.id, driver.id)
}
```

---

## Test Data Helpers

### Fixture Extensions

Create `.fixture()` factory methods for test data:

```swift
extension Driver {
    static func fixture(
        id: String = "driver-001",
        name: String = "Test Driver",
        phone: String? = "+1234567890"
    ) -> Driver {
        Driver(id: id, name: name, phone: phone)
    }
}
```

Place fixtures in `YourAppTests/Helpers/TestFixtures.swift` or alongside the type's test file.

## SwiftUI Testing

### ViewInspector

<!-- CUSTOMIZE: Remove if not using ViewInspector -->
Use ViewInspector for unit-testing SwiftUI views:

```swift
import ViewInspector

func test_driverListView__displaysDriverNames() throws {
    // Arrange
    let viewModel = DriverListViewModel(service: MockDriverService(
        drivers: [Driver.fixture(name: "Alice")]
    ))

    // Act
    let view = DriverListView(viewModel: viewModel)
    let list = try view.inspect().list()

    // Assert
    let text = try list.forEach(0).hStack(0).text(0).string()
    XCTAssertEqual(text, "Alice")
}
```

### Snapshot Testing

<!-- CUSTOMIZE: Replace with your snapshot testing library -->
Use snapshot tests for visual regression of complex views:

```swift
import SnapshotTesting

func test_driverCard__snapshotLight() {
    let view = DriverCard(driver: Driver.fixture())
    let hostingController = UIHostingController(rootView: view)
    assertSnapshot(matching: hostingController, as: .image(on: .iPhone13))
}

func test_driverCard__snapshotDark() {
    let view = DriverCard(driver: Driver.fixture())
        .environment(\.colorScheme, .dark)
    let hostingController = UIHostingController(rootView: view)
    assertSnapshot(matching: hostingController, as: .image(on: .iPhone13))
}
```

---

## Async Test Patterns

### Testing async ViewModel Methods

```swift
func test_loadDrivers__setsDriversArray() async {
    // Arrange
    let mockService = MockDriverService(drivers: [Driver.fixture()])
    let viewModel = DriverListViewModel(service: mockService)

    // Act
    await viewModel.loadDrivers()

    // Assert
    XCTAssertFalse(viewModel.isLoading)
    XCTAssertFalse(viewModel.drivers.isEmpty)
}
```

### Testing Thrown Errors

```swift
func test_fetchDriver_invalidId__throwsValidationError() async {
    // Arrange
    let service = DriverService(apiClient: MockAPIClient())

    // Act & Assert
    do {
        _ = try await service.fetchDriver(id: "")
        XCTFail("Expected validationFailed error")
    } catch let error as DriverError {
        guard case .validationFailed(let field, _) = error else {
            return XCTFail("Expected validationFailed, got \(error)")
        }
        XCTAssertEqual(field, "id")
    } catch {
        XCTFail("Unexpected error type: \(error)")
    }
}
```

---

## Integration Tests

<!-- CUSTOMIZE: Replace with your project's integration test setup -->
Integration tests live in a separate test target with real service interactions:

```swift
final class DriverAPIIntegrationTests: XCTestCase {
    private var apiClient: APIClient!

    override func setUp() {
        super.setUp()
        apiClient = APIClient(baseURL: URL(string: "https://staging-api.example.com")!)
    }

    func test_fetchDrivers__returnsNonEmptyList() async throws {
        let drivers = try await apiClient.request(DriverEndpoint.list) as [Driver]
        XCTAssertFalse(drivers.isEmpty)
    }
}
```

Run integration tests separately:

```bash
xcodebuild test -scheme YourAppIntegrationTests -destination 'platform=iOS Simulator,name=iPhone 16'
```

---

## Anti-Patterns

| Anti-Pattern | Correct Approach |
|-------------|-----------------|
| Mocking frameworks (Cuckoo, Mockingbird) | Manual protocol fakes |
| Real network calls in unit tests | Protocol fakes returning fixtures |
| `XCTAssert(true)` / empty test body | Assert specific values and behaviors |
| Shared mutable state between tests | Independent setup per test in `setUp()` |
| `sleep()` for async synchronization | Use `async/await` and `XCTestExpectation` |
| Testing private methods directly | Test through public API surface |
| Snapshot tests without light + dark | Always test both color schemes |
| Giant test methods (50+ lines) | Extract helpers, keep tests focused |
