---
name: "swift-data"
description: "SwiftData persistence framework with models, queries, relationships, and CloudKit sync"
---

# SwiftData

You are a SwiftData expert. Apply these patterns when working with data persistence in Swift apps.

## When to Use

- Adding persistence to SwiftUI apps
- Defining data models with relationships
- Querying and filtering data
- Syncing with iCloud/CloudKit
- Migrating from Core Data

## Decision Tree

| Need | Reference |
|------|-----------|
| Define models and properties | → [model-basics.md](references/model-basics.md) |
| Create relationships between models | → [relationships.md](references/relationships.md) |
| Query, filter, and sort data | → [queries-filtering.md](references/queries-filtering.md) |
| Configure containers and contexts | → [containers-contexts.md](references/containers-contexts.md) |
| Enable iCloud sync | → [cloudkit-sync.md](references/cloudkit-sync.md) |
| Handle schema migrations | → [migration.md](references/migration.md) |
| Optimize performance | → [performance.md](references/performance.md) |
| Debug and troubleshoot | → [common-issues.md](references/common-issues.md) |

## Core Principles

1. **Use @Model for all persistent types** - Classes only, not structs
2. **Relationships must be optional or have defaults** - For CloudKit compatibility
3. **ModelContext is thread-bound** - Never pass contexts between threads
4. **Prefer @Query in SwiftUI** - Automatic updates when data changes
5. **Autosave is on by default** - Changes persist automatically

## Quick Reference

### Basic Model

```swift
import SwiftData

@Model
final class Book {
    var title: String
    var author: String
    var publishedDate: Date
    var rating: Int?

    init(title: String, author: String, publishedDate: Date) {
        self.title = title
        self.author = author
        self.publishedDate = publishedDate
    }
}
```

### App Setup

```swift
@main
struct MyApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(for: Book.self)
    }
}
```

### Query in SwiftUI

```swift
struct BookList: View {
    @Query(sort: \Book.title) var books: [Book]
    @Environment(\.modelContext) var modelContext

    var body: some View {
        List(books) { book in
            Text(book.title)
        }
    }
}
```

### CRUD Operations

```swift
// Create
let book = Book(title: "Swift Guide", author: "Apple", publishedDate: .now)
modelContext.insert(book)

// Read - use @Query or fetch
let descriptor = FetchDescriptor<Book>(
    predicate: #Predicate { $0.rating ?? 0 > 3 },
    sortBy: [SortDescriptor(\.title)]
)
let books = try modelContext.fetch(descriptor)

// Update - modify properties directly
book.title = "Updated Title"

// Delete
modelContext.delete(book)
```

## Note

For comprehensive SwiftData tutorials, check [Hacking with Swift - SwiftData](https://www.hackingwithswift.com/quick-start/swiftdata/).

## References

- **[model-basics.md](references/model-basics.md)** - @Model macro, properties, attributes
- **[relationships.md](references/relationships.md)** - One-to-one, one-to-many, many-to-many
- **[queries-filtering.md](references/queries-filtering.md)** - @Query, predicates, sorting
- **[containers-contexts.md](references/containers-contexts.md)** - ModelContainer, ModelContext, configuration
- **[cloudkit-sync.md](references/cloudkit-sync.md)** - iCloud sync setup and constraints
- **[migration.md](references/migration.md)** - Lightweight and complex migrations
- **[performance.md](references/performance.md)** - Batch operations, background contexts
- **[common-issues.md](references/common-issues.md)** - Debugging and troubleshooting
