# Common Issues

## Overview

This reference covers common SwiftData issues and their solutions.

## Model Issues

### "Cannot use @Model with structs"

**Problem**: Using @Model on a struct.

```swift
// WRONG
@Model
struct Book {  // Structs not supported
    var title: String
}
```

**Solution**: Use a final class.

```swift
// CORRECT
@Model
final class Book {
    var title: String
}
```

### "Property has no default value"

**Problem**: Non-optional property without default.

```swift
// WRONG
@Model
final class Book {
    var title: String
    var rating: Int  // No default, not optional
}
```

**Solution**: Add default or make optional.

```swift
// CORRECT
@Model
final class Book {
    var title: String
    var rating: Int = 0       // Default
    var notes: String?        // Or optional
}
```

### "Cannot convert value of type..."

**Problem**: Using unsupported property type.

```swift
// WRONG
@Model
final class Item {
    var data: CustomType  // Not Codable
}
```

**Solution**: Make type Codable or use supported types.

```swift
// CORRECT
struct CustomType: Codable {
    var value: String
}

@Model
final class Item {
    var data: CustomType
}
```

## Relationship Issues

### "CloudKit integration requires optional relationship"

**Problem**: Non-optional relationship with CloudKit.

```swift
// WRONG with CloudKit
@Model
final class Book {
    var author: Author  // Non-optional
    var tags: [Tag]     // Non-optional array
}
```

**Solution**: Make relationships optional.

```swift
// CORRECT
@Model
final class Book {
    var author: Author?
    var tags: [Tag]?
}
```

### "Circular reference detected"

**Problem**: Cascade delete creates infinite loop.

```swift
// WRONG
@Model final class A {
    @Relationship(deleteRule: .cascade)
    var b: B?
}

@Model final class B {
    @Relationship(deleteRule: .cascade)
    var a: A?  // Circular cascade!
}
```

**Solution**: Use `.nullify` for one side.

```swift
// CORRECT
@Model final class A {
    @Relationship(deleteRule: .cascade)
    var b: B?
}

@Model final class B {
    @Relationship(deleteRule: .nullify)
    var a: A?
}
```

## Query Issues

### "@Query used outside SwiftUI view"

**Problem**: Using @Query in non-view type.

```swift
// WRONG
class ViewModel: ObservableObject {
    @Query var books: [Book]  // Not in a View
}
```

**Solution**: Use FetchDescriptor in non-views.

```swift
// CORRECT
class ViewModel: ObservableObject {
    func fetchBooks(context: ModelContext) throws -> [Book] {
        try context.fetch(FetchDescriptor<Book>())
    }
}
```

### "Predicate type mismatch"

**Problem**: Wrong type in predicate.

```swift
// WRONG
#Predicate<Book> { $0.rating > "3" }  // String vs Int
```

**Solution**: Use correct types.

```swift
// CORRECT
#Predicate<Book> { $0.rating > 3 }
```

### "Cannot use key path in predicate"

**Problem**: Complex expressions in predicate.

```swift
// WRONG
#Predicate<Book> {
    $0.title.count > 10  // Complex operation
}
```

**Solution**: Simplify or use fetch + filter.

```swift
// CORRECT - use localizedStandardContains or similar
// Or fetch all and filter in Swift
let books = try context.fetch(FetchDescriptor<Book>())
let filtered = books.filter { $0.title.count > 10 }
```

## Context Issues

### "Context accessed from wrong thread"

**Problem**: Passing ModelContext between threads.

```swift
// WRONG
let context = modelContext
Task.detached {
    let books = try context.fetch(descriptor)  // Wrong thread!
}
```

**Solution**: Create new context for each thread.

```swift
// CORRECT
Task.detached {
    let context = ModelContext(container)
    let books = try context.fetch(descriptor)
}
```

### "Cannot save context"

**Problem**: Save fails without apparent error.

**Debug Steps**:
1. Check for validation errors
2. Ensure all required properties have values
3. Check relationship constraints

```swift
do {
    try modelContext.save()
} catch {
    print("Save error: \(error)")
    // Check specific error type
}
```

### "Object was deleted"

**Problem**: Accessing deleted object.

```swift
let book = books.first!
modelContext.delete(book)

print(book.title)  // May crash or show stale data
```

**Solution**: Check before accessing.

```swift
if !book.isDeleted {
    print(book.title)
}
```

## CloudKit Issues

### "Data not syncing"

**Checklist**:
1. iCloud signed in?
2. Network available?
3. CloudKit container exists?
4. Entitlements correct?
5. All relationships optional?
6. No @Attribute(.unique)?

### "Duplicate records after sync"

**Problem**: @Attribute(.unique) with CloudKit.

**Solution**: Remove unique constraint, handle in app logic.

```swift
// Instead of unique constraint
func findOrCreate(email: String, context: ModelContext) throws -> User {
    let predicate = #Predicate<User> { $0.email == email }
    let existing = try context.fetch(FetchDescriptor(predicate: predicate))

    if let user = existing.first {
        return user
    }

    let user = User(email: email)
    context.insert(user)
    return user
}
```

## Migration Issues

### "Store migration failed"

**Problem**: Schema change without migration plan.

**Solution**: Define migration stages.

```swift
enum MigrationPlan: SchemaMigrationPlan {
    static var schemas: [any VersionedSchema.Type] {
        [SchemaV1.self, SchemaV2.self]
    }

    static var stages: [MigrationStage] {
        [.lightweight(fromVersion: SchemaV1.self, toVersion: SchemaV2.self)]
    }
}
```

### "Data lost after update"

**Problem**: Renamed property without originalName.

```swift
// WRONG - data lost
// V1: var name: String
// V2: var title: String  // New property, name data lost
```

**Solution**: Use originalName.

```swift
// CORRECT
@Attribute(originalName: "name")
var title: String
```

## Debug Launch Arguments

Add to scheme's Arguments:

| Argument | Purpose |
|----------|---------|
| `-com.apple.CoreData.SQLDebug 1` | SQL logging |
| `-com.apple.CoreData.CloudKitDebug 1` | CloudKit logging |
| `-com.apple.CoreData.MigrationDebug 1` | Migration logging |
| `-com.apple.CoreData.ConcurrencyDebug 1` | Thread issues |

## Finding the Database

```swift
// Print store location
if let url = container.configurations.first?.url {
    print("Database: \(url.path)")
}

// Default location
// ~/Library/Application Support/<BundleID>/default.store
```

View with DB Browser for SQLite or similar tool.

## Best Practices for Debugging

1. **Enable SQL debug** - See what queries run
2. **Check thread** - Context must stay on creation thread
3. **Validate models** - All properties have defaults/optional
4. **Test migrations** - Use in-memory containers
5. **Monitor CloudKit** - Use CloudKit Dashboard

## Checklist When Stuck

- [ ] Model is final class with @Model
- [ ] All properties have defaults or are optional
- [ ] Relationships are optional (for CloudKit)
- [ ] No @Attribute(.unique) with CloudKit
- [ ] Context used on same thread it was created
- [ ] Migration plan defined for schema changes
- [ ] Debug arguments enabled for logging
