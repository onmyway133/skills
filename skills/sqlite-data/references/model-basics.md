# Model Basics

Define your data models using the @Table macro from StructuredQueries.

## @Table Macro

The @Table macro works with **structs** (not classes like SwiftData's @Model):

```swift
import SQLiteData

@Table
struct Item: Identifiable {
    let id: UUID
    var title = ""
    var isInStock = true
    var notes = ""
}
```

**Key differences from SwiftData:**
- Works with structs (value types), not classes
- No automatic `persistentIdentifier` - you must define your own `id`
- Default values specified inline, no initializer required
- Add `nonisolated` for Sendable conformance in concurrent contexts

## Column Types

### Supported Types

| Swift Type | SQLite Type |
|------------|-------------|
| `String` | TEXT |
| `Int`, `Int64` | INTEGER |
| `Double`, `Float` | REAL |
| `Bool` | INTEGER (0/1) |
| `Date` | TEXT (ISO8601) |
| `Data` | BLOB |
| `UUID` | TEXT |
| `URL` | TEXT |
| Optional<T> | NULL or underlying type |

### Custom Column Mapping

Use `@Column` for custom representations:

```swift
@Table
struct RemindersList: Identifiable {
    let id: UUID

    @Column(as: Color.HexRepresentation.self)
    var color: Color = .blue
}

extension Color {
    struct HexRepresentation: QueryBindable, QueryDecodable {
        let color: Color

        init(_ color: Color) {
            self.color = color
        }

        // QueryBindable
        func queryBinding() -> QueryBinding {
            .text(color.hexString)
        }

        // QueryDecodable
        init(decoder: inout some QueryDecoder) throws {
            let hex = try String(decoder: &decoder)
            self.color = Color(hex: hex)
        }
    }
}
```

## Enums

Enums must conform to `QueryBindable`:

```swift
@Table
struct Reminder {
    let id: UUID
    var priority: Priority?

    enum Priority: Int, QueryBindable {
        case low, medium, high
    }
}

// Usage - enums work in queries unlike SwiftData
@FetchAll(Reminder.where { $0.priority == .high })
var highPriorityReminders
```

## Primary Keys

### Auto-incrementing Integer

```swift
@Table
struct Fact: Identifiable {
    let id: Int  // Auto-incremented by SQLite
    var body: String
}

// Table creation
try #sql("""
    CREATE TABLE "facts" (
        "id" INTEGER PRIMARY KEY AUTOINCREMENT,
        "body" TEXT NOT NULL
    ) STRICT
    """).execute(db)
```

### UUID Primary Key

```swift
@Table
struct SyncUp: Identifiable {
    let id: UUID
    var title = ""
}

// Table creation with default UUID
try #sql("""
    CREATE TABLE "syncUps" (
        "id" TEXT PRIMARY KEY NOT NULL DEFAULT (uuid()),
        "title" TEXT NOT NULL DEFAULT ''
    ) STRICT
    """).execute(db)
```

## Draft Types

For inserting records without specifying all fields (especially auto-generated ones):

```swift
@Table
struct Fact: Identifiable {
    let id: Int  // Auto-generated
    var body: String
}

// Insert using Draft (omits id)
try Fact.insert {
    Fact.Draft(body: "A new fact")
}.execute(db)
```

The `@Table` macro automatically generates a `Draft` type that excludes `let` properties.

## Computed Properties

Computed properties are not persisted:

```swift
@Table
struct Reminder: Identifiable {
    let id: UUID
    var title = ""
    var dueDate: Date?
    var isCompleted = false

    // Not persisted - computed on access
    var isOverdue: Bool {
        guard let dueDate else { return false }
        return !isCompleted && dueDate < Date()
    }

    var displayTitle: String {
        isCompleted ? "✓ \(title)" : title
    }
}
```

## Static Query Helpers

Define reusable query filters as static properties:

```swift
extension Reminder {
    static let incomplete = Self.where { !$0.isCompleted }
    static let overdue = Self.where {
        $0.dueDate != nil && !$0.isCompleted
    }
    static let sortedByDueDate = Self.order(by: \.dueDate)
}

// Usage
@FetchAll(Reminder.incomplete.order(by: \.title))
var incompleteReminders
```

## Sendable Conformance

For Swift 6 strict concurrency, use `nonisolated`:

```swift
@Table
nonisolated struct Item: Identifiable, Hashable, Sendable {
    let id: UUID
    var title = ""
}
```

## Table Name Customization

By default, the table name is the pluralized, camelCase struct name. Customize with:

```swift
@Table("custom_items")
struct Item {
    let id: UUID
    var title = ""
}
```
