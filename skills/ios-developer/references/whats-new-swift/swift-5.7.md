# Swift 5.7

## Overview

Swift 5.7 was a major release introducing regular expressions, the Clock/Duration APIs, opaque parameter types, primary associated types, and if/switch expressions. This release significantly improved generic programming ergonomics.

## Regular Expressions (SE-0350, 0351, 0354, 0357)

### Regex Literals
Compile-time validated regular expressions:

```swift
let pattern = /\d{3}-\d{4}/
let phone = "555-1234"

if phone.contains(pattern) {
    print("Valid phone format")
}
```

### Named Capture Groups
```swift
let datePattern = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/

if let match = "2024-03-15".firstMatch(of: datePattern) {
    print(match.year)   // "2024"
    print(match.month)  // "03"
    print(match.day)    // "15"
}
```

### RegexBuilder DSL
Type-safe regex construction:

```swift
import RegexBuilder

let emailPattern = Regex {
    OneOrMore(.word)
    "@"
    OneOrMore(.word)
    "."
    Repeat(2...6) { .word }
}
```

### String Methods
```swift
let text = "Price: $42.99"
let numbers = text.ranges(of: /\d+\.?\d*/)
let cleaned = text.replacing(/\$/, with: "")
let trimmed = text.trimmingPrefix(/Price:\s*/)
```

## Clock, Instant, Duration (SE-0329)

Standardized time measurement replacing raw nanoseconds:

```swift
// Sleep with duration
try await Task.sleep(for: .seconds(1))
try await Task.sleep(for: .milliseconds(500))

// With tolerance for power efficiency
try await Task.sleep(for: .seconds(1), tolerance: .milliseconds(100))

// Measure execution time
let clock = ContinuousClock()
let elapsed = await clock.measure {
    await performWork()
}
print("Took \(elapsed)")  // e.g., "1.234 seconds"
```

## If/Switch Expressions (SE-0380)

Control flow as expressions returning values:

```swift
let status = if score > 90 { "Excellent" }
             else if score > 70 { "Good" }
             else { "Needs work" }

let icon = switch direction {
    case .up: "arrow.up"
    case .down: "arrow.down"
    case .left: "arrow.left"
    case .right: "arrow.right"
}
```

All branches must return the same type.

## Opaque Parameter Types (SE-0341)

Use `some` in parameter position for cleaner generic constraints:

```swift
// Before
func process<T: Collection>(_ items: T) where T.Element: Equatable { }

// After
func process(_ items: some Collection<some Equatable>) { }
```

## Primary Associated Types (SE-0346)

Protocols can declare primary associated types for easier specialization:

```swift
// Standard library now includes:
// protocol Collection<Element>
// protocol Sequence<Element>

func process(_ items: some Collection<String>) {
    for item in items {
        print(item)  // Known to be String
    }
}
```

## Constrained Existential Types (SE-0353)

Combine `any` with primary associated types:

```swift
let collections: [any Collection<Int>] = [
    [1, 2, 3],
    Set([4, 5, 6])
]

for collection in collections {
    print(collection.count)
}
```

## Structural Opaque Types (SE-0328)

Multiple opaque types in compound positions:

```swift
func makePair() -> (some Equatable, some Equatable) {
    (42, "hello")
}

func makeSequences() -> [some Sequence<Int>] {
    [[1, 2], [3, 4, 5]]
}
```

## Unlock Existentials for All Protocols (SE-0309)

Protocols with `Self` requirements can now be used as existential types:

```swift
let values: [any Equatable] = [1, "hello", 3.14]

// Can store different types
// But comparison requires same concrete type at runtime
```

## Distributed Actors (SE-0336, 0344)

Actors that work across process/network boundaries:

```swift
distributed actor GamePlayer {
    distributed func makeMove(_ position: Position) throws -> GameState {
        // Can be called remotely
    }
}

// Remote call looks identical to local
try await remotePlayer.makeMove(position)
```

## Implicitly Opened Existentials (SE-0352)

Generic functions accept existential arguments directly:

```swift
func compare<T: Equatable>(_ a: T, _ b: T) -> Bool {
    a == b
}

let x: any Equatable = 42
let y: any Equatable = 42

// Now works - Swift opens the existential
compare(x, x)  // true
```

## AsyncStream Improvements (SE-0388)

Simplified stream creation:

```swift
let (stream, continuation) = AsyncStream.makeStream(of: Int.self)

Task {
    for i in 1...10 {
        continuation.yield(i)
    }
    continuation.finish()
}

for await value in stream {
    print(value)
}
```

## buildPartialBlock for Result Builders (SE-0348)

Simplified result builder implementation:

```swift
@resultBuilder
struct ListBuilder {
    static func buildPartialBlock(first: String) -> [String] {
        [first]
    }

    static func buildPartialBlock(accumulated: [String], next: String) -> [String] {
        accumulated + [next]
    }
}
```

## Type Inference from Defaults (SE-0347)

Generic parameters can have default concrete types:

```swift
func createCollection<T = [Int]>() -> T where T: RangeReplaceableCollection {
    T()
}

let array = createCollection()  // Infers [Int]
```

## @available(*, noasync) (SE-0340)

Mark APIs unsuitable for async contexts:

```swift
@available(*, noasync, message: "Uses thread-local storage")
func legacyOperation() {
    // Thread-local dependent code
}
```

## Migration Notes

Swift 5.7's improvements to generics and existentials enable cleaner APIs:

1. Use `some` in parameters instead of explicit generics where possible
2. Leverage primary associated types for clearer constraints
3. Consider `any` vs `some` tradeoffs (flexibility vs performance)
4. Adopt Clock/Duration for time-related code
5. Use regex literals for pattern matching
