# Code Best Practices

> Detailed code conventions for Swift/iOS. See [AGENTS.md](../AGENTS.md) for quick summaries.

---

## Naming Conventions

### Types and Members

<!-- CUSTOMIZE: Replace with your project's naming conventions -->

| Element | Convention | Example |
|---------|-----------|---------|
| Types (class, struct, enum, protocol) | PascalCase | `DriverListViewModel`, `LoadService` |
| Methods and properties | camelCase | `fetchDrivers()`, `isLoading` |
| Constants (module-level) | `k` prefix + PascalCase | `kMaxRetryCount`, `kDefaultTimeout` |
| Enum cases | camelCase | `.loading`, `.loaded(items)` |
| Protocols | Adjective or `…Protocol` suffix | `Fetchable`, `DriverServiceProtocol` |
| Boolean properties | `is`, `has`, `should`, `can` prefix | `isValid`, `hasChanges`, `shouldRefresh` |

### Files

- **One primary type per file** — filename matches the type (`DriverListViewModel.swift`)
- **Extensions:** `Type+Context.swift` (`Date+Formatting.swift`, `UIView+Layout.swift`)
- **Test files:** `TypeTests.swift` (`DriverListViewModelTests.swift`)

---

## Struct vs Class

Prefer `struct` (value types) unless you need identity or inheritance:

```swift
// ✅ Good — value type for data models
struct Driver: Identifiable, Codable {
    let id: String
    var name: String
    var phone: String?
}

// ✅ Good — class for identity and @MainActor binding
@MainActor
final class DriverListViewModel: ObservableObject {
    @Published private(set) var drivers: [Driver] = []
    private let service: DriverServiceProtocol

    init(service: DriverServiceProtocol) {
        self.service = service
    }
}

// ❌ Bad — class for plain data
class DriverResponse { var name: String = "" }
```

---

## Access Control

<!-- CUSTOMIZE: Adjust based on your project's module structure -->
- **`private`** by default for properties and helpers
- **`private(set)`** for read-only published properties
- **`internal`** (implicit) for types used within the module
- **`public`** only for SPM package APIs or framework targets
- **`final`** on all classes unless designed for subclassing

```swift
@MainActor
final class DriverDetailViewModel: ObservableObject {
    @Published private(set) var driver: Driver?    // External read, internal write
    @Published private(set) var isLoading = false

    private let service: DriverServiceProtocol     // Private dependencies
    private let coordinator: DriversCoordinator

    init(service: DriverServiceProtocol, coordinator: DriversCoordinator) {
        self.service = service
        self.coordinator = coordinator
    }

    func loadDriver(id: String) async { ... }      // Internal — called by View
    private func handleError(_ error: Error) { ... }
}
```

---

## SwiftUI Patterns

### View + ViewModel

<!-- CUSTOMIZE: Replace with your project's SwiftUI conventions -->

```swift
struct DriverListView: View {
    @StateObject private var viewModel: DriverListViewModel

    init(viewModel: DriverListViewModel) {
        _viewModel = StateObject(wrappedValue: viewModel)
    }

    var body: some View {
        List(viewModel.drivers) { driver in
            DriverRow(driver: driver)
        }
        .task { await viewModel.loadDrivers() }
    }
}
```

### Previews Required

Every SwiftUI View must include a preview with mock data:

```swift
#Preview {
    DriverListView(viewModel: DriverListViewModel(service: MockDriverService()))
}
```

### Property Wrapper Usage

| Wrapper | Use Case |
|---------|----------|
| `@StateObject` | ViewModel owned by this View |
| `@ObservedObject` | ViewModel passed from parent |
| `@EnvironmentObject` | Shared app-wide state |
| `@State` | View-local transient UI state |
| `@Binding` | Two-way binding from parent |

---

## UIKit Patterns

<!-- CUSTOMIZE: Remove this section if your project is SwiftUI-only -->

### File Structure with MARK Sections

```swift
final class DriverDetailViewController: UIViewController {

    // MARK: - Properties

    private let viewModel: DriverDetailViewModel
    private var cancellables = Set<AnyCancellable>()

    // MARK: - UI Components

    private lazy var nameLabel: UILabel = { ... }()
    private lazy var phoneLabel: UILabel = { ... }()

    // MARK: - Init

    init(viewModel: DriverDetailViewModel) {
        self.viewModel = viewModel
        super.init(nibName: nil, bundle: nil)
    }

    @available(*, unavailable)
    required init?(coder: NSCoder) { fatalError() }

    // MARK: - Lifecycle

    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
        bindViewModel()
    }

    // MARK: - Private

    private func setupUI() { ... }
    private func bindViewModel() { ... }
}
```

### Delegation

Use protocol delegation over closures for multi-event communication:

```swift
protocol DriverDetailViewControllerDelegate: AnyObject {
    func driverDetail(_ controller: DriverDetailViewController, didUpdate driver: Driver)
    func driverDetailDidRequestDismiss(_ controller: DriverDetailViewController)
}
```

---

## Protocol-Oriented Design

Define protocols for all services to enable testability:

```swift
protocol DriverServiceProtocol {
    func fetchDrivers() async throws -> [Driver]
    func fetchDriver(id: String) async throws -> Driver
    func updateDriver(_ driver: Driver) async throws
}

final class DriverService: DriverServiceProtocol {
    private let apiClient: APIClientProtocol
    init(apiClient: APIClientProtocol) { self.apiClient = apiClient }

    func fetchDrivers() async throws -> [Driver] {
        try await apiClient.request(DriverEndpoint.list)
    }
}
```

---

## Async/Await Patterns

### @MainActor for UI Updates

```swift
@MainActor
final class DriverListViewModel: ObservableObject {
    @Published private(set) var drivers: [Driver] = []
    @Published private(set) var error: Error?

    func loadDrivers() async {
        do {
            drivers = try await service.fetchDrivers()
        } catch {
            self.error = error
        }
    }
}
```

### Task Management

```swift
// ✅ Good — cancel previous task on new request
private var loadTask: Task<Void, Never>?

func search(query: String) {
    loadTask?.cancel()
    loadTask = Task {
        try? await Task.sleep(for: .milliseconds(300))
        guard !Task.isCancelled else { return }
        await performSearch(query)
    }
}
```
```

---

## Error Handling

### Typed Errors

```swift
enum DriverError: LocalizedError {
    case notFound(id: String)
    case validationFailed(field: String, reason: String)
    case networkUnavailable

    var errorDescription: String? {
        switch self {
        case .notFound(let id): "Driver \(id) not found"
        case .validationFailed(_, let reason): reason
        case .networkUnavailable: "Network connection unavailable"
        }
    }
}
```

### Error Propagation

```swift
// ✅ Good — propagate with context
func fetchDriver(id: String) async throws -> Driver {
    guard !id.isEmpty else {
        throw DriverError.validationFailed(field: "id", reason: "ID must not be empty")
    }
    return try await apiClient.request(DriverEndpoint.detail(id: id))
}
```

---

## Anti-Patterns

| Anti-Pattern | Correct Approach |
|-------------|-----------------|
| Force unwraps (`!`) in production code | Use `guard let`, `if let`, or nil coalescing |
| Massive ViewController (500+ lines) | Extract to ViewModel + smaller views |
| Singleton services (`shared`) | Initializer injection with protocols |
| Raw `DispatchQueue.main.async` | `@MainActor` and `async/await` |
| Stringly-typed identifiers | Use `enum` or typed constants |
| Implicit dependencies via globals | Pass dependencies through initializers |
| `class` for plain data models | Use `struct` for value types |
| Completion handlers in new code | Use `async/await` |
| Deep inheritance hierarchies | Composition via protocols and extensions |
| `Any` or `AnyObject` for typed data | Use generics or associated types |
