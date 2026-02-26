# Swift 5.4

## Overview

Swift 5.4 formalized Result Builders (powering SwiftUI), enabled multiple variadic parameters, and improved implicit member syntax for chained expressions.

## Result Builders (SE-0289)

Previously `@_functionBuilder`, now officially `@resultBuilder`. This is the foundation of SwiftUI's declarative syntax.

```swift
@resultBuilder
struct ArrayBuilder<T> {
    static func buildBlock(_ components: T...) -> [T] {
        components
    }

    static func buildOptional(_ component: [T]?) -> [T] {
        component ?? []
    }

    static func buildEither(first: [T]) -> [T] { first }
    static func buildEither(second: [T]) -> [T] { second }
}

@ArrayBuilder<String>
func makeStrings() -> [String] {
    "Hello"
    "World"
    if Bool.random() {
        "Maybe"
    }
}
```

### Builder Methods
- `buildBlock()` - Combines multiple expressions
- `buildOptional()` - Handles `if` without `else`
- `buildEither(first:)` / `buildEither(second:)` - Handles `if/else`
- `buildArray()` - Handles `for` loops
- `buildExpression()` - Transforms individual expressions
- `buildFinalResult()` - Post-processes the final result

## Improved Implicit Member Syntax (SE-0287)

Implicit member expressions now support chained calls:

```swift
// Before
let color: Color = Color.red.opacity(0.5)

// After
let color: Color = .red.opacity(0.5)

// Multiple chains work
let style: Font = .body.bold().italic()
```

Particularly useful in SwiftUI:

```swift
Text("Hello")
    .foregroundStyle(.blue.opacity(0.8))
    .font(.title.weight(.semibold))
```

## Multiple Variadic Parameters (SE-0284)

Functions can now have multiple variadic parameters. All parameters after the first variadic must be labeled:

```swift
func log(levels: LogLevel..., messages: String...) {
    for (level, message) in zip(levels, messages) {
        print("[\(level)] \(message)")
    }
}

log(levels: .info, .warning, messages: "Started", "Check this")
```

## Local Function Improvements

### Overloading
Nested functions can now be overloaded:

```swift
func outer() {
    func process(_ value: Int) { }
    func process(_ value: String) { }

    process(42)
    process("hello")
}
```

### Forward References
Local functions can be called before their declaration:

```swift
func example() {
    helper()  // Works now

    func helper() { }
}
```

### Same-Named Variables
Variables can be assigned from same-named functions:

```swift
func color(forRow row: Int) -> Color { ... }

for i in 0..<10 {
    let color = color(forRow: i)  // Now allowed
}
```

## Property Wrappers for Local Variables (SE-0293)

Property wrappers work on local variables within functions:

```swift
func example() {
    @Clamped(0...100) var percentage = 50

    percentage = 150  // Clamped to 100
}
```

Note: Local wrapped variables maintain the same immutability restrictions as regular local variables.

## Executable Targets in SPM (SE-0294)

Packages can explicitly declare executable targets:

```swift
// Package.swift
.executableTarget(
    name: "MyTool",
    dependencies: []
)
```

This enables using `@main` without a `main.swift` file. Requires `swift-tools-version:5.4` or later.

## Practical Example: Custom View Builder

```swift
@resultBuilder
struct ConditionalViewBuilder {
    static func buildBlock<V: View>(_ content: V) -> V {
        content
    }

    static func buildIf<V: View>(_ content: V?) -> V? {
        content
    }

    static func buildEither<First: View, Second: View>(
        first: First
    ) -> _ConditionalContent<First, Second> {
        .init(first: first)
    }

    static func buildEither<First: View, Second: View>(
        second: Second
    ) -> _ConditionalContent<First, Second> {
        .init(second: second)
    }
}
```

## Migration Notes

If you were using `@_functionBuilder`, migrate to `@resultBuilder`. The semantics are identical, but the underscore prefix indicated experimental status that's now resolved.
