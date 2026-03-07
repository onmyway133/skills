# Database Setup

Configure, migrate, and seed your SQLite database.

## Basic Setup

### Step 1: Create Database Function

```swift
import OSLog
import SQLiteData

func appDatabase() throws -> any DatabaseWriter {
    @Dependency(\.context) var context

    var configuration = Configuration()

    // Optional: Enable query logging in debug
    #if DEBUG
    configuration.prepareDatabase { db in
        db.trace(options: .profile) {
            if context == .preview {
                print("\($0.expandedDescription)")
            } else {
                logger.debug("\($0.expandedDescription)")
            }
        }
    }
    #endif

    // Create context-aware database
    let database = try defaultDatabase(configuration: configuration)
    logger.info("Database opened at '\(database.path)'")

    // Run migrations
    var migrator = DatabaseMigrator()
    #if DEBUG
    migrator.eraseDatabaseOnSchemaChange = true
    #endif

    migrator.registerMigration("Create tables") { db in
        try createTables(db)
    }

    try migrator.migrate(database)
    return database
}

private let logger = Logger(subsystem: "MyApp", category: "Database")
```

### Step 2: App Entry Point

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

## DatabaseQueue vs DatabasePool

- **DatabaseQueue**: Single connection, simpler, good for most apps
- **DatabasePool**: Multiple readers, better for heavy read workloads

```swift
// Queue (default from defaultDatabase())
let queue = try DatabaseQueue(path: dbPath)

// Pool for concurrent reads
let pool = try DatabasePool(path: dbPath)
```

## Migrations

### Register Migrations

```swift
var migrator = DatabaseMigrator()

// Initial tables
migrator.registerMigration("Create initial tables") { db in
    try #sql("""
        CREATE TABLE "remindersLists" (
            "id" TEXT PRIMARY KEY NOT NULL DEFAULT (uuid()),
            "title" TEXT NOT NULL DEFAULT '',
            "color" TEXT NOT NULL DEFAULT '#007AFF'
        ) STRICT
        """).execute(db)

    try #sql("""
        CREATE TABLE "reminders" (
            "id" TEXT PRIMARY KEY NOT NULL DEFAULT (uuid()),
            "title" TEXT NOT NULL DEFAULT '',
            "notes" TEXT NOT NULL DEFAULT '',
            "isCompleted" INTEGER NOT NULL DEFAULT 0,
            "dueDate" TEXT,
            "remindersListID" TEXT NOT NULL
                REFERENCES "remindersLists"("id") ON DELETE CASCADE
        ) STRICT
        """).execute(db)
}

// Add column migration
migrator.registerMigration("Add priority column") { db in
    try #sql("""
        ALTER TABLE "reminders"
        ADD COLUMN "priority" INTEGER
        """).execute(db)
}

// Create index
migrator.registerMigration("Add indexes") { db in
    try #sql("""
        CREATE INDEX "reminders_listID"
        ON "reminders" ("remindersListID")
        """).execute(db)
}

try migrator.migrate(database)
```

### Migration Best Practices

1. **Never modify existing migrations** - Only add new ones
2. **Use STRICT tables** - Better type checking
3. **Add default values** - For backwards compatibility
4. **Add indexes for foreign keys** - Better join performance

## The #sql Macro

Type-safe SQL string interpolation:

```swift
// Safe column/table references
try #sql("""
    SELECT \(Reminder.columns)
    FROM \(Reminder.self)
    WHERE \(Reminder.isCompleted) = 1
    ORDER BY \(Reminder.title)
    """).execute(db)

// Bind values safely
let searchTerm = "groceries"
try #sql(
    "SELECT * FROM reminders WHERE title LIKE \(bind: "%\(searchTerm)%")"
).fetchAll(db)
```

## Database Seeding

### Debug Sample Data

```swift
#if DEBUG
extension DatabaseWriter {
    func seedSampleData() throws {
        try write { db in
            try db.seed {
                // Create lists
                RemindersList(
                    id: UUID(1),
                    title: "Personal"
                )
                RemindersList(
                    id: UUID(2),
                    title: "Work"
                )

                // Create reminders
                Reminder.Draft(
                    title: "Buy groceries",
                    remindersListID: UUID(1)
                )
                Reminder.Draft(
                    title: "Call mom",
                    remindersListID: UUID(1)
                )
                Reminder.Draft(
                    title: "Finish report",
                    remindersListID: UUID(2)
                )
            }
        }
    }
}

// Helper for consistent UUIDs
extension UUID {
    init(_ int: Int) {
        self.init(uuidString: "00000000-0000-0000-0000-\(String(format: "%012d", int))")!
    }
}
#endif
```

### Use in Previews

```swift
#Preview {
    let _ = prepareDependencies {
        let db = try! appDatabase()
        try! db.seedSampleData()
        $0.defaultDatabase = db
    }

    ReminderListView()
}
```

## Preview Database

Each preview gets its own in-memory database:

```swift
extension DatabaseWriter where Self == DatabaseQueue {
    static var previewDatabase: Self {
        let queue = try! DatabaseQueue()

        var migrator = DatabaseMigrator()
        migrator.registerMigration("Create tables") { db in
            try createTables(db)
        }
        try! migrator.migrate(queue)

        // Optionally seed
        try! queue.seedSampleData()

        return queue
    }
}

#Preview {
    let _ = prepareDependencies {
        $0.defaultDatabase = .previewDatabase
    }
    ContentView()
}
```

## Test Database

```swift
import Testing
@testable import MyApp

@Suite(.dependency(\.defaultDatabase, try! testDatabase()))
struct ReminderTests {
    @Dependency(\.defaultDatabase) var database

    @Test
    func createReminder() async throws {
        try await database.write { db in
            try Reminder.insert(
                Reminder(id: UUID(), title: "Test")
            ).execute(db)
        }

        let count = try await database.read { db in
            try Reminder.fetchCount(db)
        }
        #expect(count == 1)
    }
}

func testDatabase() throws -> any DatabaseWriter {
    let queue = try DatabaseQueue()
    var migrator = DatabaseMigrator()
    // Register migrations...
    try migrator.migrate(queue)
    return queue
}
```

## Configuration Options

```swift
var configuration = Configuration()

// Tracing
configuration.prepareDatabase { db in
    db.trace { print($0) }  // All SQL
    db.trace(options: .profile) { print($0) }  // With timing
}

// Foreign keys (enabled by default)
configuration.foreignKeysEnabled = true

// Busy timeout
configuration.busyMode = .timeout(5.0)

// Read-only mode
configuration.readonly = true
```

## Custom Database Path

```swift
func appDatabase() throws -> any DatabaseWriter {
    let fileManager = FileManager.default
    let appSupport = try fileManager.url(
        for: .applicationSupportDirectory,
        in: .userDomainMask,
        appropriateFor: nil,
        create: true
    )
    let dbPath = appSupport
        .appendingPathComponent("MyApp")
        .appendingPathComponent("database.sqlite")
        .path

    // Create directory if needed
    try fileManager.createDirectory(
        atPath: dbPath.deletingLastPathComponent,
        withIntermediateDirectories: true
    )

    return try DatabaseQueue(path: dbPath)
}
```

## Full-Text Search (FTS5)

```swift
@Table
struct ReminderText: FTS5 {
    let rowid: Int
    let title: String
    let notes: String
}

migrator.registerMigration("Create FTS") { db in
    try #sql("""
        CREATE VIRTUAL TABLE "reminderTexts"
        USING fts5(title, notes, content='reminders', content_rowid='rowid')
        """).execute(db)

    // Triggers to keep FTS in sync
    try #sql("""
        CREATE TRIGGER "reminders_ai" AFTER INSERT ON "reminders" BEGIN
            INSERT INTO "reminderTexts"(rowid, title, notes)
            VALUES (new.rowid, new.title, new.notes);
        END
        """).execute(db)
}

// Search
let results = try ReminderText
    .where { $0.match("groceries") }
    .fetchAll(db)
```
