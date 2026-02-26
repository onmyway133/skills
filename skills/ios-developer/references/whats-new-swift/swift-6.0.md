# Swift 6.0

## Overview

Swift 6.0 enables complete concurrency checking by default, introduces typed throws, pack iteration, and collection operations on noncontiguous elements. This is a landmark release for memory safety.

## Complete Concurrency by Default (SE-0414, 0430, 0423, 0420)

Data race safety is now enforced at compile time:

### Strict Sendable Checking
All values crossing isolation boundaries must be Sendable:

```swift
// Error in Swift 6: Type not Sendable
class UnsafeData {
    var value: Int = 0
}

Task {
    let data = UnsafeData()
    await process(data)  // Error: UnsafeData is not Sendable
}
```

### Region-Based Isolation
The compiler tracks data regions for safe concurrent access:

```swift
func process() async {
    var localData = [1, 2, 3]

    // Safe: data stays in local region
    localData.append(4)

    // Safe: transferred to task
    await withTaskGroup(of: Void.self) { group in
        group.addTask {
            print(localData)
        }
    }
}
```

### Sending Keyword
Explicit transfer between isolation regions:

```swift
func transfer(_ data: sending [Int]) async {
    // 'data' ownership transferred to this function
    await actor.process(data)
}
```

## Global Variable Safety (SE-0412)

Global and static variables must be concurrency-safe:

```swift
// Error: Mutable global state
var globalCounter = 0

// Option 1: Make it Sendable and immutable
let globalConfig = Config()

// Option 2: Actor isolation
@MainActor
var mainThreadCounter = 0

// Option 3: Explicit unsafe opt-out
nonisolated(unsafe) var unsafeGlobal = 0
```

## Typed Throws (SE-0413)

Functions can specify exact error types:

```swift
enum ValidationError: Error {
    case invalidEmail
    case passwordTooShort
}

func validate(email: String) throws(ValidationError) {
    guard email.contains("@") else {
        throw .invalidEmail
    }
}

// Caller knows exact error type
do {
    try validate(email: input)
} catch .invalidEmail {
    showEmailError()
} catch .passwordTooShort {
    showPasswordError()
}
// No catch-all needed!
```

### Benefits
- Exhaustive error handling without catch-all
- Better documentation of failure modes
- Type-safe error propagation

```swift
// Error types propagate
func process() throws(ProcessError) {
    try validate()  // ProcessError must handle ValidationError
}
```

## Pack Iteration (SE-0408)

Loop over parameter packs:

```swift
func compare<each T: Equatable>(_ lhs: (repeat each T), _ rhs: (repeat each T)) -> Bool {
    for (left, right) in repeat (each lhs, each rhs) {
        guard left == right else { return false }
    }
    return true
}

// Works with any tuple size
compare((1, "a", true), (1, "a", true))  // true
compare((1, 2, 3, 4, 5), (1, 2, 3, 4, 6))  // false
```

Removes Swift 5.x's arbitrary arity limits (previously max 6 elements).

## Count Where (SE-0220)

Single-pass conditional counting:

```swift
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

// Before: Creates intermediate array
let evenCount = numbers.filter { $0.isMultiple(of: 2) }.count

// After: Single pass, no allocation
let evenCount = numbers.count(where: { $0.isMultiple(of: 2) })
```

## Noncontiguous Collection Operations (SE-0270)

Work with discontiguous index sets:

```swift
var array = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

// Create RangeSet of indices
let indices = RangeSet([1..<3, 5..<7])

// Get discontiguous slice
let slice = array[indices]  // DiscontiguousSlice: [1, 2, 5, 6]

// Remove multiple ranges
array.removeSubranges(indices)  // [0, 3, 4, 7, 8, 9]

// Set algebra operations
let otherIndices = RangeSet([2..<4, 6..<8])
let union = indices.union(otherIndices)
let intersection = indices.intersection(otherIndices)
```

## Access-Level Import Modifiers (SE-0409)

Control dependency visibility:

```swift
// Only visible within this file
private import HelperLibrary

// Only visible within this module (default in Swift 6)
internal import InternalDependency

// Exposed to clients (explicit in Swift 6)
public import PublicAPI
```

### Benefits
- Prevent accidental API exposure
- Clearer module boundaries
- Better encapsulation of implementation details

## Isolated Default Values (SE-0411)

Function default values inherit isolation:

```swift
@MainActor
func defaultHandler() -> Handler { MainHandler() }

@MainActor
func setup(handler: Handler = defaultHandler()) {
    // handler default evaluated on MainActor
}
```

## Noncopyable Type Upgrades (SE-0427, 0429, 0432)

Full generic support for noncopyable types:

```swift
struct UniqueResource: ~Copyable {
    let id: UUID

    consuming func release() {
        // Resource freed
    }
}

// Generic containers work with noncopyable types
struct Box<T: ~Copyable>: ~Copyable {
    var value: T

    consuming func take() -> T {
        value
    }
}
```

## 128-bit Integers (SE-0425)

Full support for extended integers:

```swift
let bigNumber: Int128 = 170_141_183_460_469_231_731_687_303_715_884_105_727
let unsigned: UInt128 = bigNumber.magnitude * 2

// Standard arithmetic
let sum = bigNumber + 1
let product = unsigned * 2
```

## BitwiseCopyable (SE-0426)

Optimization hint for simple types:

```swift
// Automatic for eligible types
struct Point: BitwiseCopyable {
    var x: Double
    var y: Double
}

// Opt out for types with special semantics
struct Handle: ~BitwiseCopyable {
    var rawValue: Int
}
```

## Migration Notes

### Breaking Changes
1. Sendable checking is now enforced by default
2. Global variables require explicit handling
3. Import visibility defaults to internal

### Migration Strategy
1. **Enable Swift 6 mode** in Xcode or Package.swift
2. **Fix Sendable errors** systematically
3. **Adopt typed throws** for new error-handling code
4. **Use count(where:)** instead of filter().count
5. **Review imports** for appropriate visibility
