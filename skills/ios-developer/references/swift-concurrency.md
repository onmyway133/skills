# Swift Concurrency

## Overview

Expert guidance on async/await patterns, actors, tasks, Sendable conformance, and Swift 6 migration.

## Core Principles

1. **Analyze project settings first** - Check Swift version, strict concurrency level
2. **Identify isolation boundaries** - Map `@MainActor`, custom actors, nonisolated contexts
3. **Avoid blanket `@MainActor`** - Justify main-actor isolation with reasoning
4. **Prefer structured concurrency** - Use task groups over `Task.detached`
5. **Document unsafe patterns** - Any `@unchecked Sendable` requires safety documentation

## Project Settings Check

```swift
// Check Package.swift for concurrency settings
// .enableExperimentalFeature("StrictConcurrency")

// Check .pbxproj for:
// SWIFT_STRICT_CONCURRENCY = complete
// SWIFT_DEFAULT_ACTOR_ISOLATION = MainActor
```

## Patterns

### async/await Basics

```swift
// Simple async function
func fetchUser(id: String) async throws -> User {
    let url = URL(string: "https://api.example.com/users/\(id)")!
    let (data, _) = try await URLSession.shared.data(from: url)
    return try JSONDecoder().decode(User.self, from: data)
}

// Calling async code
Task {
    let user = try await fetchUser(id: "123")
    print(user.name)
}
```

### Parallel Execution

```swift
// async let for fixed parallel operations
async let user = fetchUser(id: "123")
async let posts = fetchPosts(userId: "123")
let (userData, userPosts) = try await (user, posts)

// Task group for dynamic parallel work
func fetchAllUsers(ids: [String]) async throws -> [User] {
    try await withThrowingTaskGroup(of: User.self) { group in
        for id in ids {
            group.addTask { try await fetchUser(id: id) }
        }
        return try await group.reduce(into: []) { $0.append($1) }
    }
}
```

### Actors

```swift
// Protect shared mutable state
actor ImageCache {
    private var cache: [URL: Data] = [:]

    func image(for url: URL) -> Data? {
        cache[url]
    }

    func store(_ data: Data, for url: URL) {
        cache[url] = data
    }
}

// Usage
let cache = ImageCache()
await cache.store(imageData, for: imageURL)
let cached = await cache.image(for: imageURL)
```

### MainActor

```swift
// UI updates must be on MainActor
@MainActor
final class ViewModel: ObservableObject {
    @Published var items: [Item] = []

    func load() async {
        let fetched = await fetchItems() // Can call off MainActor
        items = fetched // Back on MainActor for UI update
    }
}
```

### Sendable

```swift
// Value types are implicitly Sendable
struct User: Sendable {
    let id: String
    let name: String
}

// Classes need explicit conformance
final class Settings: Sendable {
    let apiKey: String // Only immutable properties
    init(apiKey: String) { self.apiKey = apiKey }
}

// Actor-isolated classes
@MainActor
final class ViewModel: Sendable {
    var state: State = .idle // Mutable but actor-isolated
}
```

## Common Errors

| Error | Solution |
|-------|----------|
| Non-Sendable type crossing actor | Make type Sendable or copy values |
| Main actor isolation conflict | Check if `@MainActor` is appropriate |
| Data race warning | Use actor or proper synchronization |
| Task canceled unexpectedly | Check task lifecycle and cancellation |

## Swift 6 Migration

1. Enable strict concurrency checking incrementally
2. Fix Sendable warnings first
3. Add actor isolation where needed
4. Document any `@unchecked Sendable` usage
5. Test thoroughly for data races

## External References

For comprehensive Swift Concurrency guidance, see:

- **[Swift-Concurrency-Agent-Skill](https://github.com/AvdLee/Swift-Concurrency-Agent-Skill)** - Complete concurrency patterns
- **[Dimillian/Skills](https://github.com/Dimillian/Skills)** - `swift-concurrency-expert` skill for Swift 6.2+ codebases
