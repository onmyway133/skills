# Swift 5.10

## Overview

Swift 5.10 completed data race safety checking, preparing the path to Swift 6. This release closed remaining concurrency safety holes and introduced nested protocols.

## Complete Data Race Safety

Swift 5.10 achieved full compile-time data race checking, closing safety gaps from earlier versions:

### Region-Based Isolation
The compiler now understands data flow across isolation boundaries:

```swift
actor Counter {
    var value = 0
}

// Compiler tracks that 'counter' is isolated
func process(counter: Counter) async {
    await counter.increment()
    // Compiler knows value is actor-isolated
}
```

### False-Positive Reduction
Many spurious warnings from Swift 5.9 eliminated through improved analysis:

```swift
// Previously warned incorrectly
let task = Task {
    let data = localValue  // Now correctly analyzed as safe
    return process(data)
}
```

### Sendable Inference Improvements
Better automatic Sendable conformance detection:

```swift
struct Config {
    let timeout: Int
    let retries: Int
}
// Automatically inferred as Sendable (all stored properties are Sendable)
```

## Nested Protocols (SE-0404)

Protocols can now be nested inside other types:

```swift
struct Animation {
    protocol Curve {
        func value(at progress: Double) -> Double
    }

    struct Linear: Curve {
        func value(at progress: Double) -> Double { progress }
    }

    struct EaseInOut: Curve {
        func value(at progress: Double) -> Double {
            // Smooth curve calculation
        }
    }
}

// Usage
func animate(with curve: some Animation.Curve) { }
```

### Benefits
- **Namespace organization**: Related protocols grouped with their types
- **Naming conflicts resolved**: `Bank.Transaction` vs `Animation.Transaction`
- **Discoverability**: Protocols appear in type's namespace

### Nesting Locations
Protocols can nest in:
- Structs
- Classes
- Enums
- Actors
- Functions (local protocols)

```swift
class ViewController {
    protocol Delegate {
        func viewControllerDidUpdate(_ vc: ViewController)
    }

    weak var delegate: (any Delegate)?
}
```

## Deprecations (SE-0383)

### @UIApplicationMain and @NSApplicationMain
These attributes are deprecated in favor of `@main`:

```swift
// Deprecated
@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate { }

// Preferred
@main
class AppDelegate: UIResponder, UIApplicationDelegate { }

// Or SwiftUI style
@main
struct MyApp: App {
    var body: some Scene {
        WindowGroup { ContentView() }
    }
}
```

## Concurrency Checking Modes

Swift 5.10 supports different concurrency strictness levels:

```swift
// Package.swift
.target(
    name: "MyTarget",
    swiftSettings: [
        .enableExperimentalFeature("StrictConcurrency")
        // Options: "minimal", "targeted", "complete"
    ]
)
```

### Minimal
Basic checking, most permissive.

### Targeted
Check Sendable in specific contexts.

### Complete
Full Swift 6 checking (what 5.10 defaults to with the flag).

## Practical Example: Safe Concurrent Code

```swift
// Nested protocol for clean API
struct DataStore {
    protocol Observer: AnyObject, Sendable {
        func storeDidUpdate(_ store: DataStore)
    }

    private let actor = StorageActor()
    private var observers: [any Observer] = []

    func addObserver(_ observer: any Observer) {
        observers.append(observer)
    }

    func save(_ data: Data) async {
        await actor.save(data)
        for observer in observers {
            observer.storeDidUpdate(self)
        }
    }
}

actor StorageActor {
    private var storage: [String: Data] = [:]

    func save(_ data: Data) {
        storage["latest"] = data
    }
}
```

## Migration to Swift 6

Swift 5.10 is the final preparation step before Swift 6:

### Checklist
1. **Enable strict concurrency**: Test with `StrictConcurrency=complete`
2. **Audit Sendable**: Ensure all cross-isolation types conform
3. **Review actors**: Verify isolation boundaries are correct
4. **Fix warnings**: All concurrency warnings become errors in Swift 6
5. **Use nested protocols**: Organize related protocols

### Common Migration Patterns

```swift
// Global state → Actor
// Before
var globalCache: [String: Data] = [:]

// After
actor Cache {
    private var storage: [String: Data] = [:]
    func get(_ key: String) -> Data? { storage[key] }
    func set(_ key: String, _ data: Data) { storage[key] = data }
}

// Delegate → Sendable protocol
// Before
protocol Delegate: AnyObject { }

// After
protocol Delegate: AnyObject, Sendable { }
```

## Migration Notes

1. **Test with strict concurrency** to preview Swift 6 behavior
2. **Adopt nested protocols** for better code organization
3. **Replace deprecated entry points** with `@main`
4. **Verify Sendable conformances** across your codebase
5. **Address all concurrency warnings** before Swift 6 upgrade
