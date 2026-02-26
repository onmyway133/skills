# Swift 5.9

## Overview

Swift 5.9 brought macros to the language, if/switch expressions, noncopyable type improvements, 128-bit integers, and significant Swift Testing framework enhancements.

## Macros (SE-0382, 0389, 0397)

Compile-time code generation with validation:

### Expression Macros
Generate expressions:

```swift
// #stringify macro
let (value, code) = #stringify(2 + 3)
// value = 5, code = "2 + 3"

// #URL with compile-time validation
let url = #URL("https://apple.com")  // Validated at compile time
// #URL("not a url")  // Compile error!
```

### Attached Macros
Transform declarations:

```swift
@Observable
class UserSettings {
    var theme: Theme = .system
    var fontSize: Int = 14
}
// Expands to full observation infrastructure
```

### Freestanding Macros
Generate new declarations:

```swift
#warning("TODO: Implement this")

#Preview {
    ContentView()
}
```

### Creating Macros
Macros are Swift packages using SwiftSyntax:

```swift
@freestanding(expression)
public macro stringify<T>(_ value: T) -> (T, String) = #externalMacro(
    module: "MyMacros",
    type: "StringifyMacro"
)
```

## If/Switch Expressions (SE-0380)

Full expression support for control flow:

```swift
// Assignment
let grade = if score >= 90 { "A" }
            else if score >= 80 { "B" }
            else if score >= 70 { "C" }
            else { "F" }

// Return
func emoji(for weather: Weather) -> String {
    switch weather {
    case .sunny: "☀️"
    case .cloudy: "☁️"
    case .rainy: "🌧️"
    }
}

// Function arguments
print(if isDebug { "Debug mode" } else { "Release" })
```

## Noncopyable Types Improvements (SE-0427, 0429, 0432)

### Noncopyable Generics
Generic code can work with noncopyable types:

```swift
struct Container<T: ~Copyable>: ~Copyable {
    var value: T
}
```

### Partial Consumption
Consume parts of noncopyable values:

```swift
struct TwoResources: ~Copyable {
    var first: Resource
    var second: Resource
}

func useFirst(_ pair: consuming TwoResources) -> Resource {
    _ = pair.second  // Discard
    return pair.first
}
```

### Switch with Borrowing
```swift
func inspect(_ resource: borrowing Resource) {
    switch resource {
    case .typeA(let data):
        print(data)
    case .typeB:
        print("Type B")
    }
}
```

## 128-bit Integers (SE-0425)

Extended range integer types:

```swift
let big: Int128 = 170_141_183_460_469_231_731_687_303_715_884_105_727
let unsigned: UInt128 = 340_282_366_920_938_463_463_374_607_431_768_211_455

// Same operations as other integers
let sum = big + 1
let product = unsigned * 2
```

## BitwiseCopyable Protocol (SE-0426)

Compiler optimization for simple types:

```swift
// Automatic inference for private/internal types
struct Point {
    var x: Double
    var y: Double
}
// Compiler infers BitwiseCopyable

// Public types need @frozen
@frozen
public struct PublicPoint: BitwiseCopyable {
    public var x: Double
    public var y: Double
}

// Opt out if needed
struct NotBitwiseCopyable: ~BitwiseCopyable {
    var callback: () -> Void
}
```

## Task Group Discarding (SE-0381)

Auto-cleanup for long-running task groups:

```swift
// Old: Must consume results to prevent memory leak
await withTaskGroup(of: Void.self) { group in
    for request in requests {
        group.addTask { await handle(request) }
    }
    // Must iterate to clean up
    for await _ in group { }
}

// New: Automatic cleanup
await withDiscardingTaskGroup { group in
    for request in requests {
        group.addTask { await handle(request) }
    }
    // Completed tasks automatically cleaned up
}
```

Essential for servers processing unlimited requests.

## Swift Testing Improvements

### Return Errors from #expect (ST-0006)
```swift
let error = try #require(throws: ValidationError.self) {
    try validate(input)
}
// Now can inspect error properties
#expect(error.field == "email")
```

### Confirmation Ranges (ST-0005)
```swift
await confirmation(expectedCount: 3...5) { confirm in
    for item in items {
        if item.isValid {
            confirm()
        }
    }
}

// At least N times
await confirmation(expectedCount: 1...) { confirm in
    // Must be called at least once
}
```

## Type Placeholders (SE-0315)

Partial type inference with underscore:

```swift
let optional: _? = getValue()  // Infers wrapped type
let dict: [String: _] = decode(json)  // Infers value type
let result: Result<_, MyError> = .success(data)
```

## Practical Example: Custom Macro

```swift
// Usage
@AddInit
struct User {
    let id: UUID
    let name: String
    var email: String?
}
// Generates memberwise init automatically

// Definition (in macro package)
public struct AddInitMacro: MemberMacro {
    public static func expansion(
        of node: AttributeSyntax,
        providingMembersOf declaration: some DeclGroupSyntax,
        in context: some MacroExpansionContext
    ) throws -> [DeclSyntax] {
        // Generate init using SwiftSyntax
    }
}
```

## Migration Notes

1. **Explore macros** for reducing boilerplate (Observable, Codable extensions)
2. **Use if/switch expressions** for cleaner value assignments
3. **Consider noncopyable types** for unique resource ownership
4. **Adopt withDiscardingTaskGroup** for long-running servers
5. **Enable BitwiseCopyable** for performance-critical simple types
