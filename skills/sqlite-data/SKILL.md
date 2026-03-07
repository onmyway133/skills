---
name: sqlite_data
description: SQLiteData persistence library (Point-Free) - a fast SwiftData alternative using SQLite/GRDB. Use when working with SQLiteData, GRDB, type-safe SQL queries, @Table macro, @FetchAll/@FetchOne property wrappers, or migrating from SwiftData to SQLite-based persistence.
---

# SQLiteData

You are a SQLiteData expert. Apply these patterns when working with SQLite persistence using Point-Free's SQLiteData library.

## When to Use

- Building apps with SQLite persistence using SQLiteData
- Migrating from SwiftData to SQLiteData
- Using GRDB with type-safe queries
- Implementing CloudKit sync with SQLite
- Working with @Table, @FetchAll, @FetchOne, or @Fetch
- Need better performance than SwiftData
- Need to work outside SwiftUI views (@Observable, UIKit)

## Decision Tree

| Need | Reference |
|------|-----------|
| Define models with @Table macro | [model-basics.md](references/model-basics.md) |
| Fetch data with @FetchAll/@FetchOne/@Fetch | [fetching.md](references/fetching.md) |
| Insert, update, delete data | [queries-crud.md](references/queries-crud.md) |
| Dynamic queries, search, FetchKeyRequest | [dynamic-queries.md](references/dynamic-queries.md) |
| Joins and multi-table queries | [relationships-joins.md](references/relationships-joins.md) |
| Database setup, migrations, seeding | [database-setup.md](references/database-setup.md) |
| CloudKit synchronization and sharing | [cloudkit-sync.md](references/cloudkit-sync.md) |
| Testing patterns and utilities | [testing.md](references/testing.md) |
| SwiftData comparison and migration | [migration-comparison.md](references/migration-comparison.md) |
| Debug and troubleshoot | [common-issues.md](references/common-issues.md) |

## Core Principles

1. **Use @Table for persistent structs** - Unlike SwiftData's @Model (classes only), @Table works with value types
2. **Use @Dependency(\.defaultDatabase) for writes** - Property wrappers are read-only; use dependency for mutations
3. **Property wrappers auto-observe changes** - @FetchAll/@FetchOne automatically update when database changes
4. **Type-safe queries via StructuredQueries** - Use fluent API or #sql macro for safe SQL
5. **Explicit database writes** - No autosave; use `database.write { }` for all mutations

## Quick Reference

### Basic Model

```swift
import SQLiteData

@Table
struct Reminder: Identifiable {
    let id: UUID
    var title = ""
    var isCompleted = false
    var dueDate: Date?
}
```

### App Setup

```swift
@main
struct MyApp: App {
    init() {
        prepareDependencies {
            $0.defaultDatabase = try! appDatabase()
        }
    }

    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}

func appDatabase() throws -> any DatabaseWriter {
    let database = try defaultDatabase()
    var migrator = DatabaseMigrator()
    migrator.registerMigration("Create tables") { db in
        try #sql("""
            CREATE TABLE "reminders" (
                "id" TEXT PRIMARY KEY NOT NULL,
                "title" TEXT NOT NULL DEFAULT '',
                "isCompleted" INTEGER NOT NULL DEFAULT 0,
                "dueDate" TEXT
            ) STRICT
            """).execute(db)
    }
    try migrator.migrate(database)
    return database
}
```

### Fetch in SwiftUI

```swift
struct ReminderList: View {
    @FetchAll(Reminder.order(by: \.title))
    var reminders

    @FetchOne(Reminder.count())
    var count = 0

    var body: some View {
        List(reminders) { reminder in
            Text(reminder.title)
        }
    }
}
```

### CRUD Operations

```swift
@Dependency(\.defaultDatabase) var database

// Create
try await database.write { db in
    try Reminder.insert(Reminder(id: UUID(), title: "Buy milk"))
        .execute(db)
}

// Read with filtering
@FetchAll(Reminder.where(\.isCompleted).order { $0.title.desc() })
var completedReminders

// Update
reminder.title = "Updated Title"
try await database.write { db in
    try Reminder.update(reminder).execute(db)
}

// Delete
try await database.write { db in
    try Reminder.delete(reminder).execute(db)
}
```

### Dynamic Query with Search

```swift
struct SearchView: View {
    @Fetch(RemindersRequest()) var data = RemindersRequest.Value()
    @State var searchText = ""

    var body: some View {
        List(data.reminders) { reminder in
            Text(reminder.title)
        }
        .searchable(text: $searchText)
        .task(id: searchText) {
            try? await $data.load(RemindersRequest(query: searchText)).task
        }
    }
}

struct RemindersRequest: FetchKeyRequest {
    var query = ""

    struct Value {
        var reminders: [Reminder] = []
        var count = 0
    }

    func fetch(_ db: Database) throws -> Value {
        let filtered = Reminder
            .where { $0.title.contains(query) }
            .order(by: \.title)
        return try Value(
            reminders: filtered.fetchAll(db),
            count: filtered.fetchCount(db)
        )
    }
}
```

## References

- **[model-basics.md](references/model-basics.md)** - @Table macro, columns, Draft types
- **[fetching.md](references/fetching.md)** - @FetchAll, @FetchOne, @Fetch property wrappers
- **[queries-crud.md](references/queries-crud.md)** - Insert, update, delete, fetch operations
- **[dynamic-queries.md](references/dynamic-queries.md)** - FetchKeyRequest, search, aggregations
- **[relationships-joins.md](references/relationships-joins.md)** - Joins, multi-table queries
- **[database-setup.md](references/database-setup.md)** - Configuration, migrations, seeding
- **[cloudkit-sync.md](references/cloudkit-sync.md)** - SyncEngine, sharing, metadata
- **[testing.md](references/testing.md)** - Test utilities and patterns
- **[migration-comparison.md](references/migration-comparison.md)** - SwiftData comparison and migration guide
- **[common-issues.md](references/common-issues.md)** - Troubleshooting and debugging
