# Swift 5.2

## Overview

Swift 5.2 focused on developer ergonomics with key path expressions as functions and callable types. This release also brought significant improvements to error diagnostics.

## Key Path Expressions as Functions (SE-0249)

Key paths can be used wherever `(Root) -> Value` functions are expected, dramatically simplifying collection operations.

```swift
struct User {
    let name: String
    let age: Int
}

let users = [User(name: "Alice", age: 30), User(name: "Bob", age: 25)]

// Before
let names = users.map { $0.name }

// After
let names = users.map(\.name)
let ages = users.map(\.age)
```

Works with any function expecting a closure that reads a property:

```swift
users.filter(\.isActive)
users.sorted(by: \.age)
users.compactMap(\.email)
```

## Callable Types (SE-0253)

Types implementing `callAsFunction()` can be invoked like functions, enabling function-like objects.

```swift
struct Multiplier {
    let factor: Int

    func callAsFunction(_ value: Int) -> Int {
        value * factor
    }
}

let double = Multiplier(factor: 2)
let result = double(5)  // 10
```

Supports parameters, return types, throws, and async:

```swift
struct NetworkFetcher {
    func callAsFunction(url: URL) async throws -> Data {
        try await URLSession.shared.data(from: url).0
    }
}

let fetcher = NetworkFetcher()
let data = try await fetcher(url: someURL)
```

## Subscripts with Default Arguments

Custom subscripts can now declare default argument values:

```swift
struct Grid<T> {
    subscript(x: Int, y: Int, default default: T) -> T {
        // Return value or default
    }
}

let value = grid[5, 10, default: .empty]
```

## Diagnostic Improvements

### Better Error Messages
Completely rewritten diagnostic architecture provides clearer error messages, especially for:

- Type inference failures
- Protocol conformance issues
- SwiftUI view building errors

### Lazy Filter Order Fix
Lazy filter chains now execute in the correct order. Previously, multiple lazy filters could execute in reversed order compared to eager evaluation.

```swift
// Now behaves identically to non-lazy version
let result = array.lazy.filter(condition1).filter(condition2)
```

## Practical Applications

### Cleaner SwiftUI Bindings
Key path syntax particularly shines in SwiftUI:

```swift
List(items, id: \.id) { item in
    Text(item.name)
}

ForEach(users.sorted(by: \.lastName)) { user in
    UserRow(user: user)
}
```

### Callable View Models
Callable types can simplify action handling:

```swift
struct SubmitAction {
    let viewModel: FormViewModel

    func callAsFunction() async throws {
        try await viewModel.submit()
    }
}
```
