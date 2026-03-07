# SwiftData Migration Guide

Compare SQLiteData with SwiftData and learn how to migrate.

## Feature Comparison

| Feature | SwiftData | SQLiteData |
|---------|-----------|------------|
| **Model Definition** | `@Model` (classes) | `@Table` (structs) |
| **Fetching in Views** | `@Query` | `@FetchAll`, `@FetchOne`, `@Fetch` |
| **Database Access** | `ModelContext` | `@Dependency(\.defaultDatabase)` |
| **Multi-query Transaction** | Not available | `FetchKeyRequest` |
| **@Observable Support** | Limited | Full support |
| **UIKit Support** | Manual setup | Built-in `observe {}` |
| **CloudKit Sync** | Built-in | Built-in with `SyncEngine` |
| **Record Sharing** | Not available | Full CKShare support |
| **Boolean/Enum Queries** | Limited (crashes) | Full support |
| **Minimum iOS** | iOS 17 | iOS 13 |
| **Performance** | Good | Near-native SQLite |
| **Schema Control** | Implicit | Explicit SQL |

## Model Definition

### SwiftData
```swift
@Model
class Item {
    var title: String
    var isInStock: Bool
    var notes: String

    init(title: String = "", isInStock: Bool = true, notes: String = "") {
        self.title = title
        self.isInStock = isInStock
        self.notes = notes
    }
}
```

### SQLiteData
```swift
@Table
struct Item: Identifiable {
    let id: UUID
    var title = ""
    var isInStock = true
    var notes = ""
}
```

**Key Differences:**
- `@Table` uses structs, `@Model` uses classes
- SQLiteData requires explicit `id` field
- No initializer needed with `@Table` (defaults inline)

## App Setup

### SwiftData
```swift
@main
struct MyApp: App {
    let container = try! ModelContainer(for: Item.self)

    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(container)
    }
}
```

### SQLiteData
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
```

## Fetching Data

### SwiftData
```swift
struct ItemsView: View {
    @Query(sort: \Item.title)
    var items: [Item]

    var body: some View {
        ForEach(items) { item in
            Text(item.title)
        }
    }
}
```

### SQLiteData
```swift
struct ItemsView: View {
    @FetchAll(Item.order(by: \.title))
    var items

    var body: some View {
        ForEach(items) { item in
            Text(item.title)
        }
    }
}
```

## @Observable Models

### SwiftData (Complex)
```swift
@Observable
class FeatureModel {
    var modelContext: ModelContext
    var items = [Item]()
    var observer: (any NSObjectProtocol)!

    init(modelContext: ModelContext) {
        self.modelContext = modelContext
        observer = NotificationCenter.default.addObserver(
            forName: ModelContext.willSave,
            object: modelContext,
            queue: nil
        ) { [weak self] _ in
            self?.fetchItems()
        }
        fetchItems()
    }

    deinit {
        NotificationCenter.default.removeObserver(observer)
    }

    func fetchItems() {
        do {
            items = try modelContext.fetch(
                FetchDescriptor<Item>(sortBy: [SortDescriptor(\.title)])
            )
        } catch { }
    }
}
```

### SQLiteData (Simple)
```swift
@Observable
class FeatureModel {
    @ObservationIgnored
    @FetchAll(Item.order(by: \.title))
    var items

    @ObservationIgnored
    @Dependency(\.defaultDatabase) var database
}
```

## Dynamic Queries (Search)

### SwiftData (Two Views Required)
```swift
struct ItemsView: View {
    @State var searchText = ""

    var body: some View {
        SearchResultsView(searchText: searchText)
            .searchable(text: $searchText)
    }
}

struct SearchResultsView: View {
    @Query var items: [Item]

    init(searchText: String) {
        _items = Query(filter: #Predicate<Item> {
            $0.title.contains(searchText)
        })
    }

    var body: some View {
        ForEach(items) { Text($0.title) }
    }
}
```

### SQLiteData (Single View)
```swift
struct ItemsView: View {
    @State var searchText = ""
    @FetchAll var items: [Item]

    var body: some View {
        ForEach(items) { Text($0.title) }
            .searchable(text: $searchText)
            .task(id: searchText) {
                await $items.load(
                    .fetchAll(Item.where { $0.title.contains(searchText) })
                )
            }
    }
}
```

## CRUD Operations

### SwiftData
```swift
@Environment(\.modelContext) var modelContext

// Create
let newItem = Item(title: "New")
modelContext.insert(newItem)
try modelContext.save()

// Update
existingItem.title = "Updated"
try modelContext.save()

// Delete
modelContext.delete(existingItem)
try modelContext.save()
```

### SQLiteData
```swift
@Dependency(\.defaultDatabase) var database

// Create
try database.write { db in
    try Item.insert(Item(id: UUID(), title: "New")).execute(db)
}

// Update
item.title = "Updated"
try database.write { db in
    try Item.update(item).execute(db)
}

// Delete
try database.write { db in
    try Item.delete(item).execute(db)
}
```

## Booleans and Enums

### SwiftData (Limited)
```swift
// This crashes at runtime!
@Query(filter: #Predicate { $0.priority == .high })
var highPriority: [Reminder]

// Sorting by Bool doesn't work
@Query(sort: [SortDescriptor(\.isCompleted)])  // ❌ Error
var reminders: [Reminder]
```

### SQLiteData (Full Support)
```swift
// Enums work correctly
@FetchAll(Reminder.where { $0.priority == .high })
var highPriority

// Bool sorting works
@FetchAll(Reminder.order(by: \.isCompleted))
var reminders
```

## Migrations

### SwiftData Lightweight
```swift
@Model
class Item {
    var title = ""
    var description = ""  // New field - automatic migration
}
```

### SwiftData Manual (Complex)
Requires `VersionedSchema`, `SchemaMigrationPlan`, duplicated model types, type aliases, and migration stages.

### SQLiteData (Explicit)
```swift
var migrator = DatabaseMigrator()

migrator.registerMigration("Create items") { db in
    try #sql("""
        CREATE TABLE "items" (
            "id" TEXT PRIMARY KEY NOT NULL,
            "title" TEXT NOT NULL DEFAULT ''
        ) STRICT
        """).execute(db)
}

migrator.registerMigration("Add description") { db in
    try #sql("""
        ALTER TABLE "items"
        ADD COLUMN "description" TEXT NOT NULL DEFAULT ''
        """).execute(db)
}

try migrator.migrate(database)
```

## When to Choose SQLiteData

Choose SQLiteData when you need:

1. **iOS 13+ Support** - SwiftData requires iOS 17
2. **Better Performance** - Near-native SQLite speed
3. **@Observable Integration** - Full support outside SwiftUI views
4. **UIKit Support** - Built-in observation helpers
5. **Complex Queries** - Multiple queries in single transaction
6. **Bool/Enum Filtering** - SwiftData has runtime crashes
7. **Record Sharing** - CKShare support for multi-user collaboration
8. **Explicit Schema** - Direct SQL control over migrations
9. **Type-safe SQL** - StructuredQueries with compile-time safety

## Migration Steps

### 1. Update Package Dependencies

```swift
// Package.swift
dependencies: [
    .package(url: "https://github.com/pointfreeco/sqlite-data", from: "1.0.0")
]
```

### 2. Convert Models

```swift
// Before (SwiftData)
@Model class Task {
    var title: String
    var isCompleted: Bool
    init(title: String, isCompleted: Bool = false) {
        self.title = title
        self.isCompleted = isCompleted
    }
}

// After (SQLiteData)
@Table
struct Task: Identifiable {
    let id: UUID
    var title = ""
    var isCompleted = false
}
```

### 3. Update App Entry Point

```swift
// Before
.modelContainer(for: Task.self)

// After
init() {
    prepareDependencies {
        $0.defaultDatabase = try! appDatabase()
    }
}
```

### 4. Convert Queries

```swift
// Before
@Query(sort: \Task.title) var tasks: [Task]

// After
@FetchAll(Task.order(by: \.title)) var tasks
```

### 5. Convert Mutations

```swift
// Before
@Environment(\.modelContext) var context
context.insert(task)
try context.save()

// After
@Dependency(\.defaultDatabase) var database
try database.write { db in
    try Task.insert(task).execute(db)
}
```

### 6. Create Migration for Existing Data

If you have existing SwiftData users, you may need to migrate their data. This is app-specific and depends on your schema.
