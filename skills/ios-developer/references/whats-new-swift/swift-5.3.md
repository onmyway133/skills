# Swift 5.3

## Overview

Swift 5.3 brought multiple trailing closures, synthesized Comparable for enums, the `@main` attribute, and major Swift Package Manager improvements including resource bundling.

## Multiple Trailing Closures (SE-0279)

Functions with multiple closure parameters can use trailing closure syntax for all of them, dramatically improving SwiftUI readability.

```swift
// Before
Button(action: { save() }, label: { Text("Save") })

// After
Button {
    save()
} label: {
    Text("Save")
}
```

The first closure uses traditional trailing syntax, subsequent closures use labeled syntax:

```swift
animate(withDuration: 0.3) {
    view.alpha = 1.0
} completion: { finished in
    print("Done")
}
```

## Multi-Pattern Catch Clauses (SE-0276)

Catch multiple error cases in a single block:

```swift
do {
    try performOperation()
} catch NetworkError.timeout, NetworkError.noConnection {
    showOfflineMessage()
} catch NetworkError.unauthorized {
    promptLogin()
}
```

## Synthesized Comparable for Enums (SE-0266)

Enums can automatically conform to `Comparable` based on case declaration order:

```swift
enum Priority: Comparable {
    case low, medium, high, critical
}

let tasks = [task1, task2, task3]
let sorted = tasks.sorted(by: \.priority)  // low < medium < high < critical
```

Works with associated values if all associated types are themselves Comparable.

## Implicit Self in Closures (SE-0269)

`self` is no longer required in many closure contexts where retain cycles are impossible:

```swift
struct Counter {
    var count = 0

    mutating func increment() {
        DispatchQueue.main.async {
            count += 1  // No self. required in struct
        }
    }
}
```

Particularly beneficial in SwiftUI where views are structs.

## @main Attribute (SE-0281)

Declare program entry points with the `@main` attribute instead of `main.swift`:

```swift
@main
struct MyApp {
    static func main() {
        print("App started")
    }
}
```

SwiftUI uses this pattern:

```swift
@main
struct MyApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
```

## Where Clauses on Functions (SE-0267)

Functions inside generic types can have their own `where` clauses:

```swift
extension Array {
    func sorted() -> [Element] where Element: Comparable {
        sorted(by: <)
    }

    func sum() -> Element where Element: Numeric {
        reduce(0, +)
    }
}
```

## Enum Cases as Protocol Witnesses (SE-0280)

Enum cases can satisfy static protocol requirements:

```swift
protocol Defaultable {
    static var `default`: Self { get }
}

enum Direction: Defaultable {
    case north, south, east, west
    case `default`  // Satisfies protocol requirement
}
```

## Float16 Type (SE-0277)

Half-precision floating-point type for graphics and ML:

```swift
let value: Float16 = 3.14
```

Joins `Float` (Float32), `Double` (Float64), and `Float80`.

## Refined didSet Semantics (SE-0268)

Property observers are now more efficient. When `oldValue` isn't used and there's no `willSet`, the compiler avoids unnecessary value copying:

```swift
var items: [Item] {
    didSet {
        // oldValue not accessed - compiler optimizes
        updateUI()
    }
}
```

## Swift Package Manager Improvements

### Resources (SE-0271)
Bundles images, JSON, audio files with packages:

```swift
// Package.swift
.target(
    name: "MyLibrary",
    resources: [.process("Resources")]
)

// Access via Bundle.module
let image = UIImage(named: "icon", in: .module, with: nil)
```

### Binary Dependencies (SE-0272)
Reference pre-built XCFrameworks:

```swift
.binaryTarget(
    name: "SomeSDK",
    url: "https://example.com/SomeSDK.xcframework.zip",
    checksum: "..."
)
```

### Conditional Dependencies (SE-0273)
Platform-specific dependencies:

```swift
.target(
    name: "MyTarget",
    dependencies: [
        .target(name: "LinuxSupport", condition: .when(platforms: [.linux]))
    ]
)
```

### Localized Resources (SE-0278)
Include localized strings and assets in packages.
