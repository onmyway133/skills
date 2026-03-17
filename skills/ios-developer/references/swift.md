# Swift

## Target Requirements

- Swift 6+ with modern Swift concurrency
- SwiftUI with `@Observable` for shared data
- No third-party frameworks without explicit approval
- Avoid UIKit unless explicitly requested
- Follow Apple Human Interface Guidelines

## Core Principles

1. **Value types by default** - Use structs unless you need reference semantics
2. **Protocol-oriented design** - Prefer protocols over inheritance
3. **Type safety** - Leverage the type system, avoid `Any`
4. **Immutability** - Use `let` by default, `var` only when needed
5. **Error handling** - Use `throws` and typed errors

## Code Organization

### Extension-Based Grouping

Organize related functionality using extensions:

```swift
// MARK: - Main View
struct BooksView: View {
    @State private var books: [Book] = []

    var body: some View {
        List(books) { book in
            BookCell(book: book)
        }
    }
}

// MARK: - Subviews
extension BooksView {
    struct BookCell: View {
        let book: Book

        var body: some View {
            HStack {
                BookCover(url: book.coverURL)
                BookInfo(book: book)
            }
        }
    }

    struct BookCover: View {
        let url: URL?
        var body: some View { /* ... */ }
    }

    struct BookInfo: View {
        let book: Book
        var body: some View { /* ... */ }
    }
}

// MARK: - View Model
extension BooksView {
    @Observable
    @MainActor
    final class ViewModel {
        var books: [Book] = []
        var isLoading = false

        func loadBooks() async { /* ... */ }
    }
}
```

### File Structure

- Place each struct/class/enum in separate Swift files
- Don't break views into computed properties; use separate `View` structs
- Organize by feature, not layer

```
Features/
├── Books/
│   ├── BooksView.swift
│   ├── BookDetailView.swift
│   └── Models/
├── Settings/
│   └── SettingsView.swift
└── Shared/
    ├── Components/
    └── Extensions/
```

## Naming Conventions

```swift
// Types: UpperCamelCase
struct UserProfile { }
enum NetworkError { }
protocol DataFetching { }

// Properties and methods: lowerCamelCase
var currentUser: User
func fetchUserData() async throws -> User

// Boolean properties: use assertion form
var isEnabled: Bool
var hasUnreadMessages: Bool
var canSubmit: Bool

// Collections: use plural nouns
var users: [User]
var selectedItems: Set<Item>

// Mutating vs non-mutating
func add(_ item: Item)           // mutates
func adding(_ item: Item) -> [Item]  // returns new
```

## Modern Swift Patterns

### Result Builders

```swift
@resultBuilder
struct ArrayBuilder<Element> {
    static func buildBlock(_ components: Element...) -> [Element] {
        components
    }

    static func buildOptional(_ component: [Element]?) -> [Element] {
        component ?? []
    }

    static func buildEither(first component: [Element]) -> [Element] {
        component
    }

    static func buildEither(second component: [Element]) -> [Element] {
        component
    }
}
```

### Property Wrappers

```swift
@propertyWrapper
struct Clamped<Value: Comparable> {
    private var value: Value
    private let range: ClosedRange<Value>

    var wrappedValue: Value {
        get { value }
        set { value = min(max(newValue, range.lowerBound), range.upperBound) }
    }

    init(wrappedValue: Value, _ range: ClosedRange<Value>) {
        self.range = range
        self.value = min(max(wrappedValue, range.lowerBound), range.upperBound)
    }
}

struct Volume {
    @Clamped(0...100) var level: Int = 50
}
```

### Protocols with Associated Types

```swift
protocol Repository {
    associatedtype Entity
    associatedtype ID: Hashable

    func fetch(id: ID) async throws -> Entity
    func save(_ entity: Entity) async throws
    func delete(id: ID) async throws
}

struct UserRepository: Repository {
    typealias Entity = User
    typealias ID = String

    func fetch(id: String) async throws -> User { ... }
    func save(_ entity: User) async throws { ... }
    func delete(id: String) async throws { ... }
}
```

### Opaque Types

```swift
// Return opaque type
func makeIterator() -> some IteratorProtocol {
    [1, 2, 3].makeIterator()
}

// Primary associated types (Swift 5.7+)
func process(_ collection: some Collection<Int>) -> Int {
    collection.reduce(0, +)
}
```

## Concurrency

```swift
// Modern sleep - NOT Task.sleep(nanoseconds:)
try await Task.sleep(for: .seconds(1))

// NEVER use DispatchQueue.main.async()
DispatchQueue.main.async { }  // BAD

// DO use MainActor
await MainActor.run { }
// or mark with @MainActor

// All @Observable classes should be @MainActor
@Observable
@MainActor
final class ViewModel { }
```

## Error Handling

```swift
// Define typed errors
enum NetworkError: LocalizedError {
    case invalidURL
    case noConnection
    case serverError(statusCode: Int)
    case decodingFailed(Error)

    var errorDescription: String? {
        switch self {
        case .invalidURL:
            return "Invalid URL"
        case .noConnection:
            return "Network connection unavailable"
        case .serverError(let code):
            return "Server returned error \(code)"
        case .decodingFailed:
            return "Failed to process server response"
        }
    }
}

// Throwing functions (Swift 6+)
func fetch(url: URL) async throws(NetworkError) -> Data {
    guard url.scheme == "https" else {
        throw .invalidURL
    }
    // ...
}

// Error handling
do {
    let data = try await fetch(url: myURL)
} catch .invalidURL {
    print("Invalid URL")
} catch .serverError(let code) {
    print("Server error: \(code)")
} catch {
    print("Other error: \(error)")
}
```

## Collections

```swift
// Lazy operations
let results = largeArray
    .lazy
    .filter { $0.isActive }
    .map { $0.name }
    .prefix(10)

// Dictionary operations
let grouped = items.reduce(into: [:]) { dict, item in
    dict[item.category, default: []].append(item)
}

// Set operations
let common = set1.intersection(set2)
let unique = set1.symmetricDifference(set2)

// ForEach with enumerated
ForEach(Array(items.enumerated()), id: \.element.id) { index, item in
    Text("\(index): \(item.name)")
}
```

## Memory Management

```swift
// Weak references in closures
class ViewController {
    func loadData() {
        service.fetch { [weak self] result in
            guard let self else { return }
            self.handleResult(result)
        }
    }
}

// Unowned when lifetime is guaranteed
class Parent {
    var child: Child?

    class Child {
        unowned let parent: Parent
        init(parent: Parent) { self.parent = parent }
    }
}
```

## Modern Foundation

```swift
// Directory access
let documents = URL.documentsDirectory
let cache = URL.cachesDirectory

// Path manipulation
let file = documents.appending(path: "data.json")

// String operations - NOT replacingOccurrences(of:with:)
let result = text.replacing("old", with: "new")

// Text filtering - NOT contains()
items.filter { $0.name.localizedStandardContains(searchText) }

// Text formatting
Text(price, format: .currency(code: "USD"))
Text(date, format: .dateTime.month().day())
Text(count, format: .number.precision(.fractionLength(2)))
```

## SwiftData

### Basic Setup

```swift
@Model
final class Book {
    var title: String
    var author: String
    var publishedDate: Date
    var rating: Int?

    @Relationship(deleteRule: .cascade)
    var chapters: [Chapter]?

    init(title: String, author: String, publishedDate: Date) {
        self.title = title
        self.author = author
        self.publishedDate = publishedDate
    }
}
```

### CloudKit Compatibility

When using iCloud sync:
- NEVER use `@Attribute(.unique)`
- All properties need default values or be optional
- Mark ALL relationships as optional

## Testing

### Swift Testing Framework

```swift
import Testing

struct AuthenticationTests {
    @Test("User can log in with valid credentials")
    func successfulLogin() async throws {
        let auth = AuthService()
        let result = try await auth.login(email: "test@example.com", password: "valid")
        #expect(result.isAuthenticated)
    }

    @Test("Login fails with wrong password")
    func failedLogin() async {
        let auth = AuthService()
        await #expect(throws: AuthError.invalidCredentials) {
            try await auth.login(email: "test@example.com", password: "wrong")
        }
    }
}
```

### Testing Guidelines

- Write unit tests for core application logic
- UI tests only when unit tests aren't feasible
- Place view logic in view models for testability

## Debugging

```swift
// Conditional compilation
#if DEBUG
print("Debug mode")
#endif

// Assertions
assert(index >= 0, "Index must be non-negative")
precondition(!array.isEmpty, "Array must not be empty")

// Fatal errors
guard let config = loadConfig() else {
    fatalError("Missing configuration")
}
```

## iOS Version Features

### iOS 17+
- `@Observable` macro replaces `ObservableObject`
- `@Bindable` for bindings to observable objects
- SwiftData for persistence
- TipKit for onboarding hints

### iOS 18+
- Enhanced SwiftData with `#Index` and `#Unique`
- Control Center widgets
- `@Previewable` macro for simpler previews

### iOS 26+
- Liquid Glass design system
- New translucent materials
- Modern Tab API

## Things to Avoid

1. **Force unwrapping** - Use `if let`, `guard let`, or nil coalescing
2. **AnyView** - Use `@ViewBuilder` or concrete types
3. **GeometryReader** - Try `containerRelativeFrame` first
4. **UIScreen.main.bounds** - Use proper layout APIs
5. **ObservableObject** - Use `@Observable` instead (iOS 17+)
6. **DispatchQueue.main.async** - Use `@MainActor`
7. **Hard-coded sizes** - Respect Dynamic Type
8. **UIKit colors** - Use SwiftUI colors
9. **Computed property views** - Use separate View structs
10. **Single-param onChange** - Use two-parameter version

## Security

- NEVER commit secrets, API keys, or configuration data
- Use environment variables or secure storage
- Follow App Store Review Guidelines
