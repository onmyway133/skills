# Testing

Test patterns and utilities for SQLiteData applications.

## Basic Test Setup

Use in-memory databases for isolated tests:

```swift
import Testing
import SQLiteData
@testable import MyApp

@Suite(.dependency(\.defaultDatabase, try! testDatabase()))
struct ReminderTests {
    @Dependency(\.defaultDatabase) var database

    @Test
    func createReminder() async throws {
        let reminder = Reminder(id: UUID(), title: "Test Reminder")

        try await database.write { db in
            try Reminder.insert(reminder).execute(db)
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
    migrator.registerMigration("Create tables") { db in
        try createTables(db)
    }
    try migrator.migrate(queue)
    return queue
}
```

## Testing with Seeded Data

```swift
@Suite
struct ReminderListTests {
    @Dependency(\.defaultDatabase) var database

    init() throws {
        let db = try testDatabase()
        try db.seedTestData()
        prepareDependencies {
            $0.defaultDatabase = db
        }
    }

    @Test
    func fetchReminders() async throws {
        let reminders = try await database.read { db in
            try Reminder.fetchAll(db)
        }
        #expect(reminders.count == 3)
    }
}

extension DatabaseWriter {
    func seedTestData() throws {
        try write { db in
            try db.seed {
                RemindersList(id: UUID(1), title: "Personal")
                Reminder.Draft(title: "Task 1", remindersListID: UUID(1))
                Reminder.Draft(title: "Task 2", remindersListID: UUID(1))
                Reminder.Draft(title: "Task 3", remindersListID: UUID(1))
            }
        }
    }
}

// Consistent UUIDs for testing
extension UUID {
    init(_ int: Int) {
        self.init(uuidString: "00000000-0000-0000-0000-\(String(format: "%012d", int))")!
    }
}
```

## Testing FetchKeyRequest

```swift
@Suite(.dependency(\.defaultDatabase, try! testDatabase()))
struct SearchRequestTests {
    @Dependency(\.defaultDatabase) var database

    @Test
    func searchFindsMatches() async throws {
        // Seed data
        try await database.write { db in
            try db.seed {
                Reminder.Draft(title: "Buy groceries")
                Reminder.Draft(title: "Buy milk")
                Reminder.Draft(title: "Call mom")
            }
        }

        // Test the request directly
        let request = SearchRequest(query: "buy")
        let result = try await database.read { db in
            try request.fetch(db)
        }

        #expect(result.items.count == 2)
        #expect(result.items.allSatisfy { $0.title.lowercased().contains("buy") })
    }
}
```

## Testing CRUD Operations

```swift
@Suite(.dependency(\.defaultDatabase, try! testDatabase()))
struct CRUDTests {
    @Dependency(\.defaultDatabase) var database

    @Test
    func insertAndFetch() async throws {
        let id = UUID()
        let reminder = Reminder(id: id, title: "Test")

        try await database.write { db in
            try Reminder.insert(reminder).execute(db)
        }

        let fetched = try await database.read { db in
            try Reminder.find(db, key: id)
        }

        #expect(fetched?.title == "Test")
    }

    @Test
    func update() async throws {
        let id = UUID()
        var reminder = Reminder(id: id, title: "Original")

        try await database.write { db in
            try Reminder.insert(reminder).execute(db)
        }

        reminder.title = "Updated"
        try await database.write { db in
            try Reminder.update(reminder).execute(db)
        }

        let fetched = try await database.read { db in
            try Reminder.find(db, key: id)
        }

        #expect(fetched?.title == "Updated")
    }

    @Test
    func delete() async throws {
        let id = UUID()
        let reminder = Reminder(id: id, title: "To Delete")

        try await database.write { db in
            try Reminder.insert(reminder).execute(db)
            try Reminder.delete(reminder).execute(db)
        }

        let count = try await database.read { db in
            try Reminder.fetchCount(db)
        }

        #expect(count == 0)
    }
}
```

## Testing Queries

```swift
@Suite(.dependency(\.defaultDatabase, try! testDatabase()))
struct QueryTests {
    @Dependency(\.defaultDatabase) var database

    @Test
    func filterByCompleted() async throws {
        try await database.write { db in
            try db.seed {
                Reminder(id: UUID(), title: "Done", isCompleted: true)
                Reminder(id: UUID(), title: "Pending", isCompleted: false)
            }
        }

        let completed = try await database.read { db in
            try Reminder.where(\.isCompleted).fetchAll(db)
        }

        #expect(completed.count == 1)
        #expect(completed.first?.title == "Done")
    }

    @Test
    func orderByTitle() async throws {
        try await database.write { db in
            try db.seed {
                Reminder(id: UUID(), title: "Zebra")
                Reminder(id: UUID(), title: "Apple")
                Reminder(id: UUID(), title: "Mango")
            }
        }

        let sorted = try await database.read { db in
            try Reminder.order(by: \.title).fetchAll(db)
        }

        #expect(sorted.map(\.title) == ["Apple", "Mango", "Zebra"])
    }
}
```

## Testing with CloudKit

When using SyncEngine in tests:

```swift
extension DependencyValues {
    mutating func bootstrapTestDatabase() throws {
        defaultDatabase = try testDatabase()
        defaultSyncEngine = try SyncEngine(
            for: defaultDatabase,
            tables: RemindersList.self, Reminder.self
        )
    }
}

@Suite(.dependencies { try! $0.bootstrapTestDatabase() })
struct SyncTests {
    @Dependency(\.defaultDatabase) var database
    @Dependency(\.defaultSyncEngine) var syncEngine

    @Test
    func syncEngineConfigured() {
        #expect(syncEngine != nil)
    }
}
```

## Snapshot Testing

Use with swift-snapshot-testing:

```swift
import SnapshotTesting
import Testing

@Suite
struct ViewSnapshotTests {
    @Test
    func reminderListSnapshot() async {
        let db = try! testDatabase()
        try! db.seedTestData()

        prepareDependencies {
            $0.defaultDatabase = db
        }

        let view = ReminderListView()
        assertSnapshot(of: view, as: .image)
    }
}
```

## Testing Migrations

```swift
@Suite
struct MigrationTests {
    @Test
    func migrationsRunSuccessfully() throws {
        let queue = try DatabaseQueue()
        var migrator = DatabaseMigrator()

        // Register all migrations
        migrator.registerMigration("v1") { db in
            try createInitialTables(db)
        }
        migrator.registerMigration("v2") { db in
            try addPriorityColumn(db)
        }

        // Should not throw
        try migrator.migrate(queue)

        // Verify schema
        try queue.read { db in
            let columns = try db.columns(in: "reminders")
            #expect(columns.contains { $0.name == "priority" })
        }
    }
}
```

## Isolated Tests

Each test gets its own database:

```swift
@Suite
struct IsolatedTests {
    @Test
    func test1() async throws {
        let db = try testDatabase()
        prepareDependencies { $0.defaultDatabase = db }

        try await db.write { db in
            try Reminder.insert(Reminder(id: UUID(), title: "Test 1")).execute(db)
        }

        let count = try await db.read { try Reminder.fetchCount($0) }
        #expect(count == 1)
    }

    @Test
    func test2() async throws {
        let db = try testDatabase()
        prepareDependencies { $0.defaultDatabase = db }

        // Starts fresh - no data from test1
        let count = try await db.read { try Reminder.fetchCount($0) }
        #expect(count == 0)
    }
}
```

## Testing Property Wrappers

```swift
@Suite(.dependency(\.defaultDatabase, try! testDatabase()))
struct PropertyWrapperTests {
    @FetchAll(Reminder.order(by: \.title))
    var reminders

    @Dependency(\.defaultDatabase) var database

    @Test
    func fetchAllUpdatesOnChange() async throws {
        #expect(reminders.isEmpty)

        try await database.write { db in
            try Reminder.insert(Reminder(id: UUID(), title: "New")).execute(db)
        }

        // Reload
        try await $reminders.load()

        #expect(reminders.count == 1)
    }
}
```
