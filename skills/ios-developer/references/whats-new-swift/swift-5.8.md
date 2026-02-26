# Swift 5.8

## Overview

Swift 5.8 was a refinement release focusing on result builder improvements, function back-deployment, implicit self after weak capture unwrapping, and preparation for Swift 6.

## Result Builder Improvements (SE-0373)

Result builders now support more variable types:

### Lazy Variables
```swift
var body: some View {
    lazy var expensiveData = computeExpensiveData()

    VStack {
        if showDetails {
            DetailView(data: expensiveData)
        }
        Text("Summary")
    }
}
```

### Computed Properties
```swift
var body: some View {
    var greeting: String {
        isLoggedIn ? "Welcome back!" : "Hello, guest"
    }

    Text(greeting)
}
```

### Property Wrappers
```swift
var body: some View {
    @AppStorage("username") var username = "Guest"

    Text("Hello, \(username)")
}
```

## Function Back Deployment (SE-0376)

Make new functions available on older OS versions by embedding the implementation:

```swift
extension Text {
    @backDeployed(before: iOS 17)
    func customStyle() -> some View {
        self.font(.headline)
            .foregroundStyle(.primary)
    }
}

// Available on iOS 16 even though defined for iOS 17+
// On iOS 17+, uses system implementation
// On iOS 16, uses embedded copy
```

### Limitations
- Functions only, not types or properties
- Implementation copied into app binary
- Runtime checks use native implementation when available

## Implicit Self After Weak Capture (SE-0365)

After unwrapping `weak self`, implicit self is allowed:

```swift
class ViewController {
    var name = "Test"

    func setupCallback() {
        fetchData { [weak self] result in
            guard let self else { return }

            // No more self. required!
            print(name)
            updateUI()
            handleResult(result)
        }
    }
}
```

Works with any unwrapping pattern:

```swift
// guard let self
guard let self else { return }
print(name)

// if let self
if let self {
    print(name)
}

// Optional binding
self?.doSomething() ?? defaultAction()
```

## Concise Magic File Names (SE-0274)

`#file` now returns a shorter format:

```swift
// Old behavior
print(#filePath)  // /Users/dev/MyApp/Sources/File.swift

// New behavior
print(#file)      // MyApp/File.swift
```

Enable with `-enable-upcoming-feature ConciseMagicFile`.

Benefits:
- Smaller binary size
- Reduced log noise
- Faster string comparisons

## Opening Existentials to Optional Parameters (SE-0375)

Generic functions with optional parameters now accept existentials:

```swift
func process<T: Equatable>(_ value: T, default: T? = nil) {
    // ...
}

let item: any Equatable = "hello"
process(item)  // Now works (was error in 5.7)
```

## Collection Downcasts in Patterns

Direct downcasting of collections in switch statements:

```swift
func process(_ items: [Any]) {
    switch items {
    case let strings as [String]:
        print("Strings: \(strings)")
    case let numbers as [Int]:
        print("Numbers: \(numbers)")
    default:
        print("Mixed types")
    }
}
```

Previously required temporary variables or explicit casting.

## Upcoming Feature Flags

Swift 5.8 introduced flags to opt into Swift 6 behaviors early:

```swift
// In Package.swift
.target(
    name: "MyTarget",
    swiftSettings: [
        .enableUpcomingFeature("ConciseMagicFile"),
        .enableUpcomingFeature("ForwardTrailingClosures"),
        .enableUpcomingFeature("BareSlashRegexLiterals"),
    ]
)
```

Or via compiler flags:
```bash
swiftc -enable-upcoming-feature ConciseMagicFile
```

## Practical Example: Modern Callback Pattern

```swift
class DataManager {
    private var cache: [String: Data] = [:]

    func fetch(key: String, completion: @escaping (Result<Data, Error>) -> Void) {
        networkClient.fetch(key) { [weak self] result in
            guard let self else {
                completion(.failure(CancelledError()))
                return
            }

            switch result {
            case .success(let data):
                // Implicit self - clean code!
                cache[key] = data
                notifyObservers(key: key)
                completion(.success(data))

            case .failure(let error):
                logError(error)
                completion(.failure(error))
            }
        }
    }
}
```

## Migration Notes

1. **Adopt implicit self** after weak capture unwrapping for cleaner closure code
2. **Use @backDeployed** to support older OS versions without #available checks
3. **Enable upcoming features** to prepare for Swift 6
4. **Leverage result builder improvements** for complex SwiftUI views
5. **Consider #file vs #filePath** based on your logging needs
