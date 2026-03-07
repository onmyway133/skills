# Relationships and Joins

Query data from multiple tables using joins.

## Foreign Keys

Define relationships using foreign key columns:

```swift
@Table
struct RemindersList: Identifiable {
    let id: UUID
    var title = ""
}

@Table
struct Reminder: Identifiable {
    let id: UUID
    var title = ""
    var isCompleted = false
    var remindersListID: RemindersList.ID  // Foreign key
}
```

### Table Creation with Foreign Keys

```swift
migrator.registerMigration("Create tables") { db in
    try #sql("""
        CREATE TABLE "remindersLists" (
            "id" TEXT PRIMARY KEY NOT NULL,
            "title" TEXT NOT NULL DEFAULT ''
        ) STRICT
        """).execute(db)

    try #sql("""
        CREATE TABLE "reminders" (
            "id" TEXT PRIMARY KEY NOT NULL,
            "title" TEXT NOT NULL DEFAULT '',
            "isCompleted" INTEGER NOT NULL DEFAULT 0,
            "remindersListID" TEXT NOT NULL
                REFERENCES "remindersLists"("id") ON DELETE CASCADE
        ) STRICT
        """).execute(db)
}
```

## Basic Join

Join two tables and select combined data:

```swift
@Selection
struct ReminderWithList {
    let reminderTitle: String
    let listTitle: String
}

@FetchAll(
    Reminder
        .join(RemindersList.all) { $0.remindersListID.eq($1.id) }
        .select {
            ReminderWithList.Columns(
                reminderTitle: $0.title,
                listTitle: $1.title
            )
        }
)
var remindersWithLists
```

## Left Join

Include records even without matching related records:

```swift
@Selection
struct ListWithReminderCount {
    let list: RemindersList
    let reminderCount: Int
}

@FetchAll(
    RemindersList
        .group(by: \.id)
        .leftJoin(Reminder.all) { $0.id.eq($1.remindersListID) }
        .select {
            ListWithReminderCount.Columns(
                list: $0,
                reminderCount: $1.count()
            )
        }
)
var listsWithCounts
```

## Multiple Joins

Join more than two tables:

```swift
@Table
struct Tag: Identifiable {
    let id: UUID
    var name = ""
}

@Table
struct ReminderTag {
    let reminderID: Reminder.ID
    let tagID: Tag.ID
}

// Reminder with its list and tags
extension Reminder {
    static let withListAndTags = group(by: \.id)
        .join(RemindersList.all) { $0.remindersListID.eq($1.id) }
        .leftJoin(ReminderTag.all) { $0.id.eq($2.reminderID) }
        .leftJoin(Tag.all) { $2.tagID.eq($3.primaryKey) }
}
```

## Aggregations with Groups

Count, sum, average across groups:

```swift
// Count reminders per list
@Selection
struct ListStats {
    let listID: UUID
    let listTitle: String
    let totalReminders: Int
    let completedReminders: Int
}

@FetchAll(
    RemindersList
        .group(by: \.id)
        .leftJoin(Reminder.all) { $0.id.eq($1.remindersListID) }
        .select {
            ListStats.Columns(
                listID: $0.id,
                listTitle: $0.title,
                totalReminders: $1.count(),
                completedReminders: $1.isCompleted.sum()
            )
        }
)
var listStats
```

## Subqueries

Use subqueries for complex conditions:

```swift
// Lists that have at least one incomplete reminder
let listsWithIncomplete = RemindersList
    .where {
        $0.id.in(
            Reminder
                .where { !$0.isCompleted }
                .select(\.remindersListID)
        )
    }

// Delete reminders not in any active list
try Reminder
    .where {
        !$0.remindersListID.in(
            RemindersList.select(\.id)
        )
    }
    .delete()
    .execute(db)
```

## Static Relationship Helpers

Define reusable relationship queries:

```swift
extension RemindersList {
    // All reminders for this list
    func reminders() -> some TableQuery<Reminder> {
        Reminder.where { $0.remindersListID == id }
    }

    // Incomplete reminders count
    static let withIncompleteCount = group(by: \.id)
        .leftJoin(Reminder.where { !$0.isCompleted }) { $0.id.eq($1.remindersListID) }
        .select {
            ($0, $1.count())
        }
}

extension Reminder {
    // The list this reminder belongs to
    static let withList = join(RemindersList.all) { $0.remindersListID.eq($1.id) }

    // With all tags
    static let withTags = group(by: \.id)
        .leftJoin(ReminderTag.all) { $0.id.eq($1.reminderID) }
        .leftJoin(Tag.all) { $1.tagID.eq($2.primaryKey) }
}
```

## Cascade Delete

Handle automatic deletion of related records:

```sql
-- In migration
CREATE TABLE "reminders" (
    "id" TEXT PRIMARY KEY NOT NULL,
    "remindersListID" TEXT NOT NULL
        REFERENCES "remindersLists"("id") ON DELETE CASCADE
)
```

When a list is deleted, all its reminders are automatically deleted.

## Manual Relationship Loading

When you need to load relationships separately:

```swift
struct ListDetailRequest: FetchKeyRequest {
    let listID: UUID

    struct Value {
        var list: RemindersList?
        var reminders: [Reminder] = []
        var reminderCount = 0
    }

    func fetch(_ db: Database) throws -> Value {
        let list = try RemindersList.find(db, key: listID)
        let reminders = try Reminder
            .where { $0.remindersListID == listID }
            .order(by: \.title)
            .fetchAll(db)

        return Value(
            list: list,
            reminders: reminders,
            reminderCount: reminders.count
        )
    }
}
```

## Many-to-Many

Use a junction table:

```swift
@Table
struct Reminder: Identifiable {
    let id: UUID
    var title = ""
}

@Table
struct Tag: Identifiable {
    let id: UUID
    var name = ""
}

@Table
struct ReminderTag {
    let reminderID: Reminder.ID
    let tagID: Tag.ID
}

// Get all tags for a reminder
let tags = try Tag
    .join(ReminderTag.all) { $0.id.eq($1.tagID) }
    .where { $1.reminderID == reminderID }
    .select { $0 }
    .fetchAll(db)

// Get all reminders with a specific tag
let reminders = try Reminder
    .join(ReminderTag.all) { $0.id.eq($1.reminderID) }
    .where { $1.tagID == tagID }
    .select { $0 }
    .fetchAll(db)
```

## Avoiding N+1 Queries

Instead of loading relationships one by one:

```swift
// Bad: N+1 queries
for list in lists {
    let reminders = try Reminder
        .where { $0.remindersListID == list.id }
        .fetchAll(db)
}

// Good: Single query with join
@Selection
struct ListWithReminders {
    let list: RemindersList
    let reminders: [Reminder]
}

// Or use group_concat for simple cases
let listsWithReminders = try RemindersList
    .group(by: \.id)
    .leftJoin(Reminder.all) { $0.id.eq($1.remindersListID) }
    .select { ($0, $1) }
    .fetchAll(db)
```
