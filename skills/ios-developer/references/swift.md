# Swift

## Overview

General Swift development patterns and best practices for iOS/macOS development.

## Core Principles

1. **Value types by default** - Use structs unless you need reference semantics
2. **Protocol-oriented design** - Prefer protocols over inheritance
3. **Type safety** - Leverage the type system, avoid `Any`
4. **Immutability** - Use `let` by default, `var` only when needed
5. **Error handling** - Use `throws` and typed errors

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

## Error Handling

```swift
// Define typed errors
enum NetworkError: Error {
    case invalidURL
    case noConnection
    case serverError(statusCode: Int)
    case decodingFailed(Error)
}

// Throwing functions
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

## External References

For iOS-specific Swift development, see:

- **[Dimillian/Skills](https://github.com/Dimillian/Skills)** - iOS development skills:
  - `ios-debugger-agent` - Build and debug workflows
  - `gh-issue-fix-flow` - GitHub issue resolution
  - `app-store-changelog` - Release notes generation
  - `macos-swiftpm-app-packaging` - SwiftPM app packaging
