# Swift 5.6

## Overview

Swift 5.6 introduced the `any` keyword for existential types, type placeholders for partial inference, and began the roadmap toward strict concurrency checking in Swift 6.

## Existential Any (SE-0335)

The `any` keyword explicitly marks existential types, distinguishing them from opaque types (`some`):

```swift
// Existential - can hold any conforming type
let vehicle: any Vehicle = Car()

// Opaque - specific but hidden type
func makeVehicle() -> some Vehicle {
    Car()
}
```

### Why This Matters
Existentials have runtime overhead (type erasure, dynamic dispatch). Making them explicit helps developers understand performance implications:

```swift
// any = existential box, dynamic dispatch
func process(vehicles: [any Vehicle]) { }

// some = specific type, static dispatch
func process<V: Vehicle>(vehicles: [V]) { }
```

### Migration Path
- Swift 5.6: `any` optional but encouraged
- Swift 5.10: Warnings without `any`
- Swift 6: Required

## Type Placeholders (SE-0315)

Use underscore `_` for partial type inference when full inference isn't possible:

```swift
// Let compiler infer the optional wrapper
let value: _? = getValue()  // Infers Optional<SomeType>

// Specify container, infer element
let dict: [String: _] = json  // Infers value type

// In generic contexts
let result: Result<_, Error> = .success(data)
```

Useful during prototyping or when you want explicit types but not all of them.

## CodingKeyRepresentable (SE-0320)

Dictionaries with enum keys now encode as keyed containers instead of arrays:

```swift
enum Setting: String, CodingKeyRepresentable {
    case theme, language, notifications
}

let settings: [Setting: String] = [
    .theme: "dark",
    .language: "en"
]

// Encodes as: {"theme": "dark", "language": "en"}
// Previously: [["theme", "dark"], ["language", "en"]]
```

## Unavailability Condition (SE-0290)

Inverse of `#available` for cleaner fallback code:

```swift
// Before
if #available(iOS 15, *) {
    // Empty
} else {
    useLegacyAPI()
}

// After
if #unavailable(iOS 15) {
    useLegacyAPI()
}
```

Note: `#unavailable` doesn't support the platform wildcard `*`.

## Concurrency Improvements

### @preconcurrency
Suppress concurrency warnings when adopting async code incrementally:

```swift
@preconcurrency import OldLibrary

// Warnings about Sendable suppressed for types from OldLibrary
```

### Strict Concurrency Roadmap (SE-0337)
Swift 5.6 began warning about potential data races:

```swift
// Warning: Capture of non-Sendable type in @Sendable closure
Task {
    self.nonSendableProperty.mutate()
}
```

## Swift Package Manager Plugins

### Build Tool Plugins
Extend the build process with custom tools:

```swift
// Package.swift
.plugin(
    name: "GenerateResources",
    capability: .buildTool()
)
```

### Command Plugins
Add custom commands to `swift package`:

```swift
// Package.swift
.plugin(
    name: "Format",
    capability: .command(
        intent: .sourceCodeFormatting()
    )
)

// Usage: swift package format
```

### Use Cases
- Code generation (SwiftGen, Sourcery)
- Documentation generation (DocC)
- Formatting (swift-format)
- Linting

## Practical Example: Gradual Concurrency Adoption

```swift
// Step 1: Mark legacy types with @preconcurrency
@preconcurrency
class LegacyDataManager {
    var data: [String] = []
}

// Step 2: Add Sendable where safe
final class ThreadSafeCache: @unchecked Sendable {
    private let queue = DispatchQueue(label: "cache")
    private var storage: [String: Data] = [:]

    func get(_ key: String) -> Data? {
        queue.sync { storage[key] }
    }
}

// Step 3: Use actors for new code
actor ModernDataStore {
    private var items: [Item] = []

    func add(_ item: Item) {
        items.append(item)
    }
}
```

## Migration Notes

Start preparing for Swift 6's strict concurrency:

1. Enable warnings: `-warn-concurrency`
2. Audit Sendable conformances
3. Consider actor isolation for shared state
4. Use `any` for existential types
5. Replace `@escaping` closures with `@Sendable` where appropriate
