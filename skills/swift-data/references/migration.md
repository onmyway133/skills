# Migration

## Overview

SwiftData supports two migration types:
- **Lightweight migration**: Automatic, handles simple changes
- **Complex migration**: Manual, for data transformations

## Lightweight Migration

SwiftData automatically handles:
- Adding new properties (with defaults)
- Removing properties
- Renaming properties (with `@Attribute(originalName:)`)
- Adding new models
- Non-destructive relationship changes

### Adding Properties

```swift
// Version 1
@Model
final class Book {
    var title: String
}

// Version 2 - Add property with default
@Model
final class Book {
    var title: String
    var rating: Int = 0          // New property with default
    var notes: String?           // New optional property
}
```

### Renaming Properties

Use `originalName` to preserve data:

```swift
// Version 1
@Model
final class Book {
    var name: String
}

// Version 2 - Rename property
@Model
final class Book {
    @Attribute(originalName: "name")
    var title: String  // Renamed from "name"
}
```

### Removing Properties

Simply remove the property - data is discarded:

```swift
// Version 1
@Model
final class Book {
    var title: String
    var oldField: String  // Will be removed
}

// Version 2 - Property removed
@Model
final class Book {
    var title: String
    // oldField removed - data lost
}
```

## Complex Migration

For data transformations, use `VersionedSchema` and `SchemaMigrationPlan`.

### Define Schema Versions

```swift
enum BookSchemaV1: VersionedSchema {
    static var versionIdentifier = Schema.Version(1, 0, 0)

    static var models: [any PersistentModel.Type] {
        [Book.self]
    }

    @Model
    final class Book {
        var title: String
        var authorName: String  // Single string

        init(title: String, authorName: String) {
            self.title = title
            self.authorName = authorName
        }
    }
}

enum BookSchemaV2: VersionedSchema {
    static var versionIdentifier = Schema.Version(2, 0, 0)

    static var models: [any PersistentModel.Type] {
        [Book.self, Author.self]
    }

    @Model
    final class Book {
        var title: String
        var author: Author?  // Now a relationship

        init(title: String) {
            self.title = title
        }
    }

    @Model
    final class Author {
        var name: String
        var books: [Book]?

        init(name: String) {
            self.name = name
        }
    }
}
```

### Create Migration Plan

```swift
enum BookMigrationPlan: SchemaMigrationPlan {
    static var schemas: [any VersionedSchema.Type] {
        [BookSchemaV1.self, BookSchemaV2.self]
    }

    static var stages: [MigrationStage] {
        [migrateV1toV2]
    }

    static let migrateV1toV2 = MigrationStage.custom(
        fromVersion: BookSchemaV1.self,
        toVersion: BookSchemaV2.self
    ) { context in
        // Fetch all V1 books
        let books = try context.fetch(FetchDescriptor<BookSchemaV1.Book>())

        // Create authors and update relationships
        var authorCache: [String: BookSchemaV2.Author] = [:]

        for oldBook in books {
            // Get or create author
            let author: BookSchemaV2.Author
            if let existing = authorCache[oldBook.authorName] {
                author = existing
            } else {
                author = BookSchemaV2.Author(name: oldBook.authorName)
                context.insert(author)
                authorCache[oldBook.authorName] = author
            }

            // Create new book with relationship
            let newBook = BookSchemaV2.Book(title: oldBook.title)
            newBook.author = author
            context.insert(newBook)

            // Delete old book
            context.delete(oldBook)
        }

        try context.save()
    }
}
```

### Use Migration Plan

```swift
let container = try ModelContainer(
    for: BookSchemaV2.Book.self, BookSchemaV2.Author.self,
    migrationPlan: BookMigrationPlan.self
)
```

## Lightweight vs Custom Migration Stages

```swift
enum MyMigrationPlan: SchemaMigrationPlan {
    static var schemas: [any VersionedSchema.Type] {
        [SchemaV1.self, SchemaV2.self, SchemaV3.self]
    }

    static var stages: [MigrationStage] {
        [
            // V1 → V2: Lightweight (just add property)
            .lightweight(fromVersion: SchemaV1.self, toVersion: SchemaV2.self),

            // V2 → V3: Custom (transform data)
            .custom(fromVersion: SchemaV2.self, toVersion: SchemaV3.self) { context in
                // Custom migration logic
            }
        ]
    }
}
```

## Testing Migrations

```swift
func testMigration() throws {
    // Create V1 container with test data
    let v1Config = ModelConfiguration(isStoredInMemoryOnly: true)
    let v1Container = try ModelContainer(
        for: SchemaV1.Book.self,
        configurations: v1Config
    )

    let context = v1Container.mainContext
    context.insert(SchemaV1.Book(title: "Test", authorName: "Author"))
    try context.save()

    // Migrate to V2
    let v2Container = try ModelContainer(
        for: SchemaV2.Book.self,
        migrationPlan: BookMigrationPlan.self,
        configurations: v1Config
    )

    // Verify migration
    let books = try v2Container.mainContext.fetch(FetchDescriptor<SchemaV2.Book>())
    XCTAssertEqual(books.count, 1)
    XCTAssertNotNil(books.first?.author)
}
```

## Version Numbering

Use semantic versioning:

```swift
// Major.Minor.Patch
Schema.Version(1, 0, 0)  // Initial
Schema.Version(1, 1, 0)  // Minor change
Schema.Version(2, 0, 0)  // Breaking change
```

## Best Practices

1. **Test migrations thoroughly** - Use in-memory containers
2. **Keep schemas immutable** - Don't modify old versions
3. **Use lightweight when possible** - Simpler and safer
4. **Back up before migration** - Especially for production
5. **Plan for rollback** - Store backups if migration fails

## Common Patterns

### Rename Model

```swift
// Create new model, migrate data, delete old
static let migrateV1toV2 = MigrationStage.custom(...) { context in
    let oldItems = try context.fetch(FetchDescriptor<OldModel>())
    for old in oldItems {
        let new = NewModel(...)
        context.insert(new)
        context.delete(old)
    }
}
```

### Split Model

```swift
// One model becomes two
static let splitModel = MigrationStage.custom(...) { context in
    let items = try context.fetch(FetchDescriptor<CombinedModel>())
    for item in items {
        let part1 = ModelA(...)
        let part2 = ModelB(...)
        context.insert(part1)
        context.insert(part2)
        context.delete(item)
    }
}
```

### Merge Models

```swift
// Two models become one
static let mergeModels = MigrationStage.custom(...) { context in
    let itemsA = try context.fetch(FetchDescriptor<ModelA>())
    let itemsB = try context.fetch(FetchDescriptor<ModelB>())

    for (a, b) in zip(itemsA, itemsB) {
        let merged = CombinedModel(from: a, and: b)
        context.insert(merged)
        context.delete(a)
        context.delete(b)
    }
}
```

## Checklist

- [ ] Schema version incremented
- [ ] Migration stage defined (lightweight or custom)
- [ ] Old schemas preserved (don't modify)
- [ ] Migration tested with real data
- [ ] Rollback strategy in place
