# Common Issues

Troubleshooting and debugging SQLiteData applications.

## Build Errors

### "Cannot find 'Database' in scope"

Import SQLiteData:
```swift
import SQLiteData
```

### "@Table requires struct"

The @Table macro only works with structs:
```swift
// Wrong
@Table
class Item { }

// Correct
@Table
struct Item { }
```

### "Sendable conformance"

For Swift 6, mark tables as `nonisolated`:
```swift
@Table
nonisolated struct Item: Identifiable, Hashable, Sendable {
    let id: UUID
    var title = ""
}
```

## Runtime Errors

### "no such table"

The migration hasn't run or table name is wrong:

```swift
// Check migration was registered
var migrator = DatabaseMigrator()
migrator.registerMigration("Create tables") { db in
    try #sql("""
        CREATE TABLE "items" (...)
        """).execute(db)
}
try migrator.migrate(database)  // Don't forget this!
```

### "NOT NULL constraint failed"

Inserting NULL into a NOT NULL column without a default:

```swift
// Wrong - column has no default
try #sql("ALTER TABLE items ADD COLUMN priority INTEGER NOT NULL").execute(db)

// Correct - add default
try #sql("ALTER TABLE items ADD COLUMN priority INTEGER NOT NULL DEFAULT 0").execute(db)
```

### "FOREIGN KEY constraint failed"

Deleting a parent record without proper cascade:

```sql
-- Add cascade on table creation
CREATE TABLE "reminders" (
    "remindersListID" TEXT NOT NULL
        REFERENCES "remindersLists"("id") ON DELETE CASCADE
)
```

## Query Issues

### Observation Not Working

Make sure you're using property wrappers in a SwiftUI view or with `observe {}`:

```swift
// SwiftUI - works automatically
struct MyView: View {
    @FetchAll(Item.all) var items
    var body: some View {
        List(items) { ... }
    }
}

// UIKit - use observe
class MyViewController: UIViewController {
    @FetchAll(Item.all) var items

    override func viewDidLoad() {
        super.viewDidLoad()
        observe { [weak self] in
            self?.updateUI()
        }
    }
}
```

### @ObservationIgnored Required

When using with `@Observable`:

```swift
@Observable
class Model {
    // Required to avoid conflict with @Observable's tracking
    @ObservationIgnored
    @FetchAll(Item.all) var items
}
```

### Dynamic Query Not Updating

Use `.task(id:)` to reload when parameters change:

```swift
@State var searchText = ""
@FetchAll var items: [Item]

var body: some View {
    List(items) { ... }
        .task(id: searchText) {  // Re-runs when searchText changes
            try? await $items.load(
                .fetchAll(Item.where { $0.title.contains(searchText) })
            )
        }
}
```

## Database Issues

### "database is locked"

Multiple simultaneous writes - use async/await properly:

```swift
// Wrong - concurrent writes
Task { try database.write { ... } }
Task { try database.write { ... } }

// Correct - sequential or use transactions
try await database.write { db in
    try Item.insert(item1).execute(db)
    try Item.insert(item2).execute(db)
}
```

### Database Path Not Found

Use `defaultDatabase()` which handles paths correctly:

```swift
// Uses correct path for each context (live, preview, test)
let database = try defaultDatabase()
```

### Migration Order Matters

Migrations run in registration order:

```swift
// Migration 2 depends on migration 1
migrator.registerMigration("1 - Create users") { db in ... }
migrator.registerMigration("2 - Create posts") { db in ... }  // Can reference users
```

## CloudKit Issues

### Sync Not Working

1. Check entitlements are configured
2. Verify Background Modes has "Remote notifications"
3. Check iCloud account is signed in
4. Kill and relaunch app (simulators don't get push notifications)

### "Invalid record name"

Primary keys must be ASCII, < 255 chars, not start with underscore:

```swift
// Wrong
let id = "über-item-123"
let id = "_private"

// Correct - use UUID
let id = UUID()
```

### Unique Constraint Error

Unique constraints (other than primary key) aren't allowed with sync:

```swift
// Wrong - will fail with SyncEngine
CREATE TABLE "tags" (
    "id" TEXT PRIMARY KEY,
    "name" TEXT UNIQUE  // ❌ Not allowed
)

// Correct
CREATE TABLE "tags" (
    "id" TEXT PRIMARY KEY,
    "name" TEXT NOT NULL
)
```

## Performance Issues

### Slow Queries

Add indexes for frequently filtered/sorted columns:

```swift
migrator.registerMigration("Add indexes") { db in
    try #sql("""
        CREATE INDEX "reminders_listID"
        ON "reminders" ("remindersListID")
        """).execute(db)

    try #sql("""
        CREATE INDEX "reminders_dueDate"
        ON "reminders" ("dueDate")
        """).execute(db)
}
```

### Large Data Loads

Use limit and pagination:

```swift
struct PaginatedRequest: FetchKeyRequest {
    var page = 0
    var pageSize = 20

    func fetch(_ db: Database) throws -> [Item] {
        try Item
            .order(by: \.createdAt)
            .limit(pageSize, offset: page * pageSize)
            .fetchAll(db)
    }
}
```

### Blob Performance

Store large blobs in separate tables:

```swift
// Main table - fast to query
@Table
struct Item { let id: UUID; var title = "" }

// Separate blob table - only load when needed
@Table
struct ItemImage {
    @Column(primaryKey: true)
    let itemID: Item.ID
    var image: Data
}
```

## Debugging

### Enable Query Logging

```swift
var configuration = Configuration()
#if DEBUG
configuration.prepareDatabase { db in
    db.trace(options: .profile) {
        print($0.expandedDescription)
    }
}
#endif
```

### Check Database Contents

```swift
try database.read { db in
    let items = try Item.fetchAll(db)
    print("Items: \(items)")

    let count = try Item.fetchCount(db)
    print("Count: \(count)")
}
```

### Inspect Schema

```swift
try database.read { db in
    let tables = try db.tables()
    print("Tables: \(tables)")

    let columns = try db.columns(in: "items")
    print("Columns: \(columns)")
}
```

## Preview Issues

### "No database configured"

Always prepare dependencies in previews:

```swift
#Preview {
    let _ = prepareDependencies {
        $0.defaultDatabase = try! previewDatabase()
    }
    ContentView()
}
```

### Data Not Showing

Seed data in preview database:

```swift
static var previewDatabase: DatabaseQueue {
    let queue = try! DatabaseQueue()
    // Run migrations
    try! migrator.migrate(queue)
    // Seed data
    try! queue.seedPreviewData()
    return queue
}
```

## Test Issues

### Tests Affecting Each Other

Use isolated databases per test:

```swift
@Test
func myTest() throws {
    let db = try testDatabase()  // Fresh database
    prepareDependencies { $0.defaultDatabase = db }
    // Test...
}
```

### Async Test Failures

Use `async` properly:

```swift
@Test
func asyncTest() async throws {
    try await database.write { db in
        try Item.insert(item).execute(db)
    }

    // Wait for observation to update
    try await $items.load()

    #expect(items.count == 1)
}
```
