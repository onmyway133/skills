---
name: ios_developer
description: Comprehensive iOS development guidance covering SwiftUI, Swift Concurrency, Core Data, and Swift Testing
---

# iOS Developer

You are an iOS development expert. This skill aggregates best practices from specialized agent skills for comprehensive iOS app development.

## When to Use

- Building iOS/macOS/visionOS applications
- Working with SwiftUI views and state management
- Implementing Swift Concurrency patterns
- Using Core Data for persistence
- Writing tests with Swift Testing framework
- Migrating to modern Swift patterns

## Decision Tree

| Need | Reference |
|------|-----------|
| SwiftUI views, state, modern APIs | → [swiftui.md](references/swiftui.md) |
| async/await, actors, Sendable | → [swift-concurrency.md](references/swift-concurrency.md) |
| Core Data persistence, threading | → [core-data.md](references/core-data.md) |
| Swift Testing, migration from XCTest | → [swift-testing.md](references/swift-testing.md) |
| General Swift patterns and practices | → [swift.md](references/swift.md) |

## Core Principles

1. **Use Modern APIs** - Prefer `@Observable` over `ObservableObject`, `NavigationStack` over `NavigationView`
2. **Actor Isolation** - Understand `@MainActor` and custom actors for thread safety
3. **Structured Concurrency** - Use task groups over `Task.detached` when possible
4. **Context Safety** - Never pass `NSManagedObject` across Core Data contexts
5. **Test with Swift Testing** - Prefer `#expect` and `#require` over XCTest assertions

## Quick Reference

### SwiftUI State Management

```swift
@Observable
@MainActor
final class ViewModel {
    var items: [Item] = []

    func load() async {
        items = await fetchItems()
    }
}

struct ContentView: View {
    @State private var viewModel = ViewModel()

    var body: some View {
        List(viewModel.items) { item in
            Text(item.name)
        }
        .task {
            await viewModel.load()
        }
    }
}
```

### Swift Concurrency

```swift
// Structured concurrency with task group
func fetchAllData() async throws -> [Data] {
    try await withThrowingTaskGroup(of: Data.self) { group in
        for url in urls {
            group.addTask { try await fetch(url) }
        }
        return try await group.reduce(into: []) { $0.append($1) }
    }
}

// Actor for shared state
actor DataStore {
    private var cache: [String: Data] = [:]

    func get(_ key: String) -> Data? { cache[key] }
    func set(_ key: String, _ value: Data) { cache[key] = value }
}
```

### Core Data Background Context

```swift
// Safe background processing
func importData(_ items: [ImportItem]) async throws {
    let context = container.newBackgroundContext()
    try await context.perform {
        for item in items {
            let entity = Entity(context: context)
            entity.configure(from: item)
        }
        try context.save()
    }
}

// Pass IDs, not objects
let objectID = managedObject.objectID
Task.detached {
    let context = container.newBackgroundContext()
    let object = try context.existingObject(with: objectID)
    // Work with object safely
}
```

### Swift Testing

```swift
import Testing

@Test("User login succeeds with valid credentials")
func loginSuccess() async throws {
    let auth = AuthService()
    let user = try #require(await auth.login(email: "test@example.com", password: "valid"))
    #expect(user.isAuthenticated)
}

@Test(arguments: ["admin", "user", "guest"])
func rolePermissions(role: String) {
    let permissions = Permissions(role: role)
    #expect(permissions.canRead)
}
```

## References

- **[swiftui.md](references/swiftui.md)** - SwiftUI best practices, state management, modern APIs
- **[swift-concurrency.md](references/swift-concurrency.md)** - async/await, actors, Sendable, Swift 6 migration
- **[core-data.md](references/core-data.md)** - Core Data patterns, threading, migrations
- **[swift-testing.md](references/swift-testing.md)** - Swift Testing framework, XCTest migration
- **[swift.md](references/swift.md)** - General Swift development patterns

## External Resources

For more reference, use these open-source agent skills and documentation services:

### AvdLee Skills
- [SwiftUI-Agent-Skill](https://github.com/AvdLee/SwiftUI-Agent-Skill) - SwiftUI best practices
- [Swift-Concurrency-Agent-Skill](https://github.com/AvdLee/Swift-Concurrency-Agent-Skill) - Swift Concurrency patterns
- [Core-Data-Agent-Skill](https://github.com/AvdLee/Core-Data-Agent-Skill) - Core Data guidance
- [Swift-Testing-Agent-Skill](https://github.com/AvdLee/Swift-Testing-Agent-Skill) - Swift Testing framework

### Dimillian Skills
- [Skills Repository](https://github.com/Dimillian/Skills) - iOS development skills collection
  - SwiftUI Liquid Glass - iOS 26+ Liquid Glass API
  - SwiftUI UI Patterns - View composition and state
  - SwiftUI View Refactor - Code consistency patterns
  - SwiftUI Performance Audit - Performance optimization
  - Swift Concurrency Expert - Swift 6.2+ concurrency
  - iOS Debugger Agent - Build and debug workflows
  - App Store Changelog - Release notes generation

### Apple Documentation

- [sosumi.ai](https://sosumi.ai/) - AI-readable Apple Developer documentation
  - Replace `developer.apple.com` with `sosumi.ai` in URLs (e.g., `https://sosumi.ai/documentation/swiftui/view`)
  - MCP server at `https://sosumi.ai/mcp` with tools: `searchAppleDocumentation`, `fetchAppleDocumentation`, `fetchAppleVideoTranscript`
