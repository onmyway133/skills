# Swift 5.5

## Overview

Swift 5.5 introduced structured concurrency with async/await, actors, and Sendable types. This was a transformative release that modernized how Swift handles asynchronous code.

## Async/Await (SE-0296)

Asynchronous functions use the `async` keyword and are called with `await`:

```swift
func fetchUser(id: Int) async throws -> User {
    let url = URL(string: "https://api.example.com/users/\(id)")!
    let (data, _) = try await URLSession.shared.data(from: url)
    return try JSONDecoder().decode(User.self, from: data)
}

// Calling
let user = try await fetchUser(id: 42)
```

### Async Properties (SE-0310)
Properties can be async and throwing:

```swift
var contents: String {
    get async throws {
        try await loadFromDisk()
    }
}

let text = try await document.contents
```

## Structured Concurrency (SE-0304)

### Task
Creates a new unit of asynchronous work:

```swift
Task {
    let result = await fetchData()
    await MainActor.run {
        updateUI(with: result)
    }
}

Task.detached(priority: .background) {
    await processInBackground()
}
```

### Task Groups
Coordinate multiple concurrent operations:

```swift
func fetchAllUsers(ids: [Int]) async throws -> [User] {
    try await withThrowingTaskGroup(of: User.self) { group in
        for id in ids {
            group.addTask {
                try await fetchUser(id: id)
            }
        }

        var users: [User] = []
        for try await user in group {
            users.append(user)
        }
        return users
    }
}
```

### Async Let (SE-0317)
Shorthand for concurrent child tasks:

```swift
async let user = fetchUser(id: 1)
async let posts = fetchPosts(userId: 1)
async let friends = fetchFriends(userId: 1)

let profile = try await Profile(
    user: user,
    posts: posts,
    friends: friends
)
```

## Actors (SE-0306)

Reference types with built-in synchronization for safe concurrent access:

```swift
actor BankAccount {
    private var balance: Decimal = 0

    func deposit(_ amount: Decimal) {
        balance += amount
    }

    func withdraw(_ amount: Decimal) throws {
        guard balance >= amount else {
            throw InsufficientFundsError()
        }
        balance -= amount
    }

    func getBalance() -> Decimal {
        balance
    }
}

let account = BankAccount()
await account.deposit(100)
let balance = await account.getBalance()
```

### Global Actors (SE-0316)
`@MainActor` ensures code runs on the main thread:

```swift
@MainActor
class ViewModel: ObservableObject {
    @Published var items: [Item] = []

    func load() async {
        let fetched = await fetchItems()
        items = fetched  // Safe - we're on MainActor
    }
}
```

## Sendable (SE-0302)

Protocol marking types safe to share across concurrency domains:

```swift
// Automatically Sendable
struct Point: Sendable {
    let x: Int
    let y: Int
}

// Requires explicit conformance
final class SafeCache: @unchecked Sendable {
    private let lock = NSLock()
    private var storage: [String: Data] = [:]
}

// Sendable closures
let task = Task { @Sendable in
    await processData()
}
```

## Async Sequences (SE-0298)

Iterate over asynchronous sequences of values:

```swift
for await line in url.lines {
    print(line)
}

// With AsyncStream
let stream = AsyncStream<Int> { continuation in
    for i in 1...10 {
        continuation.yield(i)
    }
    continuation.finish()
}

for await value in stream {
    print(value)
}
```

## Continuations (SE-0300)

Bridge callback-based APIs to async/await:

```swift
func fetchData() async throws -> Data {
    try await withCheckedThrowingContinuation { continuation in
        legacyFetch { result in
            switch result {
            case .success(let data):
                continuation.resume(returning: data)
            case .failure(let error):
                continuation.resume(throwing: error)
            }
        }
    }
}
```

## Other Improvements

### CGFloat/Double Interchangeability (SE-0307)
Implicit conversion between CGFloat and Double:

```swift
let width: CGFloat = 100
let half: Double = width / 2  // No cast needed
```

### Codable Enums with Associated Values (SE-0295)
Automatic Codable synthesis for enums:

```swift
enum Response: Codable {
    case success(data: Data)
    case error(message: String, code: Int)
}
```

### #if with Postfix Expressions (SE-0308)
Conditional compilation in chained expressions:

```swift
Text("Hello")
    #if os(iOS)
    .font(.title)
    #else
    .font(.headline)
    #endif
```

### Lazy in Local Scope
`lazy var` works in function scope:

```swift
func process() {
    lazy var expensive = computeExpensiveValue()
    if condition {
        use(expensive)  // Only computed if needed
    }
}
```

## Migration Notes

When adopting async/await:
1. Start with leaf functions (those that don't call other async functions)
2. Use continuations to wrap existing callback APIs
3. Migrate up the call chain
4. Consider actor isolation for shared mutable state
5. Mark types as Sendable where appropriate
