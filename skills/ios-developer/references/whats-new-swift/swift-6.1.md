# Swift 6.1

## Overview

Swift 6.1 refines the Swift 6 experience with trailing commas, metatype keypaths, improved task group inference, and enhanced testing capabilities.

## Trailing Commas in Lists (SE-0439)

Trailing commas now permitted in more contexts:

```swift
// Arrays and dictionaries (already supported)
let colors = [
    "red",
    "green",
    "blue",  // Trailing comma OK
]

// Now also in function calls
fetchData(
    url: endpoint,
    method: .post,
    headers: headers,  // Trailing comma OK
)

// Tuple literals
let point = (
    x: 10,
    y: 20,
)

// Generic parameters
struct Container<
    Key,
    Value,  // Trailing comma OK
> { }

// Closure captures
let closure = { [
    weak self,
    unowned delegate,  // Trailing comma OK
] in
    // ...
}
```

### Benefits
- Cleaner diffs when adding/removing items
- Easier reordering of parameters
- Consistent style with comment annotations

```swift
// Perfect for documented parameters
Button(
    action: handleTap,  // Primary action
    label: buttonContent,  // Custom content
    // accessibilityLabel: "Submit",  // TODO: Add later
)
```

## Metatype Keypaths (SE-0438)

Access static properties via keypaths:

```swift
struct Configuration {
    static var maxRetries = 3
    static var timeout: TimeInterval = 30
}

// Metatype keypath
let retryPath = \Configuration.Type.maxRetries
let value = Configuration.self[keyPath: retryPath]  // 3

// Useful for dependency injection
struct Settings {
    static var apiKey: String = ""
    static var environment: Environment = .production
}

func configure<T>(
    _ keyPath: WritableKeyPath<Settings.Type, T>,
    to value: T
) {
    Settings.self[keyPath: keyPath] = value
}

configure(\.apiKey, to: "secret")
```

## TaskGroup Type Inference (SE-0442)

Simplified task group creation:

```swift
// Before: Required explicit type
await withTaskGroup(of: Int.self) { group in
    group.addTask { await fetchCount() }
    // ...
}

// After: Type inferred from addTask
await withTaskGroup { group in
    group.addTask { await fetchCount() }  // Returns Int
    // Compiler infers TaskGroup<Int>
}

// Works with throwing groups too
try await withThrowingTaskGroup { group in
    group.addTask { try await fetchData() }
}
```

## Nonisolated for Types (SE-0449)

Apply `nonisolated` to entire types:

```swift
@MainActor
class ViewModel {
    // Everything here is MainActor-isolated
}

// Opt out entire type from inherited isolation
nonisolated class Utilities {
    static func format(_ date: Date) -> String {
        // No actor isolation
    }
}

// Also works with protocols, structs, enums
nonisolated protocol DataProcessor {
    func process(_ data: Data) -> Result
}
```

### Use Case
Prevent unintentional actor inheritance in utility types:

```swift
@MainActor
class ViewController {
    // Nested type would inherit @MainActor
    nonisolated struct Constants {
        static let padding: CGFloat = 16
        static let cornerRadius: CGFloat = 8
    }
}
```

## Member Import Visibility (SE-0444)

Stricter import scoping:

```swift
// File: DataManager.swift
import Foundation

extension Data {
    func compressed() -> Data { /* ... */ }
}

// File: Other.swift
// Cannot see 'compressed()' unless this file also imports DataManager's module
// Prevents accidental reliance on transitive extensions
```

Enable with feature flag:
```swift
.enableExperimentalFeature("MemberImportVisibility")
```

## Compiler Warning Control (SE-0443)

Fine-grained warning configuration:

```bash
# Upgrade specific warnings to errors
swiftc -Werror DeprecatedDeclaration

# Keep specific warnings as warnings (not errors)
swiftc -Wwarning UnusedResult

# List available diagnostic groups
swiftc -print-diagnostic-groups
```

In Package.swift:
```swift
.target(
    name: "MyTarget",
    swiftSettings: [
        .unsafeFlags(["-Werror", "DeprecatedDeclaration"])
    ]
)
```

## Swift Testing Enhancements

### Range-Based Confirmations (ST-0005)
```swift
@Test
func testNotifications() async {
    await confirmation(expectedCount: 2...5) { confirm in
        notificationCenter.addObserver { _ in
            confirm()
        }
        await triggerNotifications()
    }
}

// At least N times
await confirmation(expectedCount: 1...) { confirm in
    // Must confirm at least once
}
```

### Error Returns (ST-0006)
```swift
@Test
func testValidation() throws {
    let error = try #require(throws: ValidationError.self) {
        try validator.validate(badInput)
    }

    // Inspect the error
    #expect(error.field == "email")
    #expect(error.reason == .invalidFormat)
}
```

### Test Scoping Traits (ST-0007)
```swift
struct MockEnvironment: TestScoping {
    @TaskLocal static var apiClient: APIClient = RealClient()
}

extension Trait where Self == MockEnvironment {
    static var mockAPI: Self { MockEnvironment() }
}

@Test(.mockAPI)
func testWithMockedAPI() async {
    // MockEnvironment.apiClient is mocked here
    let result = await fetchData()
}
```

## Language Mode Clarification (SE-0441)

Terminology standardized:
- **Swift version**: Compiler version (6.1)
- **Language mode**: Source compatibility level (Swift 5, Swift 6)

```swift
// Package.swift
.target(
    name: "MyTarget",
    swiftSettings: [
        .swiftLanguageMode(.v6)  // Use Swift 6 language mode
    ]
)
```

## Migration Notes

1. **Adopt trailing commas** for cleaner multi-line code
2. **Use metatype keypaths** for static property access patterns
3. **Simplify task groups** by omitting type parameter
4. **Apply nonisolated to types** that shouldn't inherit isolation
5. **Configure warnings** appropriately for CI pipelines
6. **Update tests** to use new Swift Testing features
