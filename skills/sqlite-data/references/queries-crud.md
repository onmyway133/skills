# CRUD Operations

Perform Create, Read, Update, Delete operations using SQLiteData.

## Database Access

Get database access via dependency injection:

```swift
@Dependency(\.defaultDatabase) var database

// Synchronous write
try database.write { db in
    // perform operations
}

// Async write
try await database.write { db in
    // perform operations
}
```

## Create (Insert)

### Insert Full Record

```swift
let reminder = Reminder(
    id: UUID(),
    title: "Buy groceries",
    isCompleted: false
)

try database.write { db in
    try Reminder.insert(reminder).execute(db)
}
```

### Insert with Draft

For tables with auto-generated columns:

```swift
@Table
struct Fact: Identifiable {
    let id: Int  // Auto-incremented
    var body: String
}

// Draft omits the auto-generated id
try database.write { db in
    try Fact.insert {
        Fact.Draft(body: "New fact")
    }.execute(db)
}
```

### Insert Multiple Records

```swift
try database.write { db in
    try Reminder.insert {
        Reminder(id: UUID(), title: "Task 1")
        Reminder(id: UUID(), title: "Task 2")
        Reminder(id: UUID(), title: "Task 3")
    }.execute(db)
}
```

### Insert or Replace

```swift
try database.write { db in
    try Reminder.insert(reminder, onConflict: .replace).execute(db)
}
```

## Read (Fetch)

### Fetch All

```swift
try database.read { db in
    let reminders = try Reminder.fetchAll(db)
}
```

### Fetch with Query

```swift
try database.read { db in
    let completed = try Reminder
        .where(\.isCompleted)
        .order(by: \.title)
        .fetchAll(db)
}
```

### Fetch One

```swift
try database.read { db in
    // First matching record
    let reminder = try Reminder
        .where { $0.id == someID }
        .fetchOne(db)  // Returns Optional<Reminder>
}
```

### Fetch by Primary Key

```swift
try database.read { db in
    let reminder = try Reminder.find(db, key: reminderID)
}
```

### Fetch Count

```swift
try database.read { db in
    let total = try Reminder.fetchCount(db)
    let completed = try Reminder.where(\.isCompleted).fetchCount(db)
}
```

### Filtering

```swift
// Key path syntax
Reminder.where(\.isCompleted)
Reminder.where { !$0.isCompleted }

// Comparison operators
Reminder.where { $0.title == "Specific Title" }
Reminder.where { $0.priority != nil }
Reminder.where { $0.dueDate < Date() }

// Contains
Reminder.where { $0.title.contains("search") }

// In list
Reminder.where { $0.id.in([id1, id2, id3]) }

// Combining conditions
Reminder.where { $0.isCompleted && $0.priority == .high }
Reminder.where { !$0.isCompleted || $0.isFlagged }
```

### Sorting

```swift
// Ascending (default)
Reminder.order(by: \.title)

// Descending
Reminder.order { $0.title.desc() }

// Multiple sort keys
Reminder.order { ($0.isCompleted, $0.title) }
Reminder.order { ($0.dueDate.desc(), $0.title.asc()) }
```

### Limiting

```swift
Reminder.limit(10)
Reminder.limit(10, offset: 20)

// First record
Reminder.order(by: \.createdAt).limit(1).fetchOne(db)
```

## Update

### Update Single Record

```swift
var reminder = existingReminder
reminder.title = "Updated Title"
reminder.isCompleted = true

try database.write { db in
    try Reminder.update(reminder).execute(db)
}
```

### Update with Query

```swift
try database.write { db in
    // Update specific fields matching condition
    try Reminder
        .where { $0.id == reminderID }
        .update { $0.title = "New Title" }
        .execute(db)
}
```

### Bulk Update

```swift
try database.write { db in
    // Mark all as completed
    try Reminder
        .where { !$0.isCompleted }
        .update { $0.isCompleted = true }
        .execute(db)
}
```

### Update Multiple Fields

```swift
try database.write { db in
    try Reminder
        .where { $0.id == reminderID }
        .update {
            $0.title = "Updated"
            $0.isCompleted = true
            $0.completedAt = Date()
        }
        .execute(db)
}
```

## Delete

### Delete Single Record

```swift
try database.write { db in
    try Reminder.delete(reminder).execute(db)
}
```

### Delete by Condition

```swift
try database.write { db in
    // Delete completed reminders
    try Reminder
        .where(\.isCompleted)
        .delete()
        .execute(db)
}
```

### Delete by IDs

```swift
try database.write { db in
    try Reminder
        .where { $0.id.in(idsToDelete) }
        .delete()
        .execute(db)
}
```

### Delete All

```swift
try database.write { db in
    try Reminder.delete().execute(db)
}
```

## Transactions

All operations in a `write` block are in a transaction:

```swift
try database.write { db in
    // All or nothing - if any fails, all roll back
    try RemindersList.insert(list).execute(db)
    try Reminder.insert(reminder1).execute(db)
    try Reminder.insert(reminder2).execute(db)
}
```

## Raw SQL

Use `#sql` macro for raw SQL:

```swift
try database.write { db in
    try #sql("""
        UPDATE reminders
        SET isCompleted = 1
        WHERE dueDate < datetime('now')
        """).execute(db)
}

// With bindings
let title = "Search term"
try database.read { db in
    let results: [Reminder] = try #sql(
        "SELECT * FROM reminders WHERE title LIKE \(bind: "%\(title)%")"
    ).fetchAll(db)
}
```

## Aggregations

```swift
try database.read { db in
    let count = try Reminder.fetchCount(db)
    let maxPriority = try Reminder.select { $0.priority.max() }.fetchOne(db)
    let avgSeconds = try SyncUp.select { $0.seconds.avg() }.fetchOne(db)
}
```
