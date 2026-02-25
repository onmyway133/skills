# Model Basics

## Overview

SwiftData uses the `@Model` macro to define persistent data types. Models must be classes, not structs, because SwiftData needs reference semantics for change tracking.

## The @Model Macro

```swift
import SwiftData

@Model
final class User {
    var name: String
    var email: String
    var createdAt: Date

    init(name: String, email: String) {
        self.name = name
        self.email = email
        self.createdAt = .now
    }
}
```

## Supported Property Types

| Type | Support |
|------|---------|
| `String`, `Int`, `Double`, `Bool` | Native |
| `Date`, `Data`, `URL`, `UUID` | Native |
| `Optional<T>` | Native |
| `Array<T>` | Native (for primitives and relationships) |
| `Codable` structs/enums | Automatic encoding |
| Other `@Model` classes | Relationships |

### Codable Types

Structs and enums conforming to `Codable` are automatically encoded:

```swift
enum Priority: String, Codable {
    case low, medium, high
}

struct Address: Codable {
    var street: String
    var city: String
    var zipCode: String
}

@Model
final class Task {
    var title: String
    var priority: Priority      // Enum - encoded automatically
    var address: Address?       // Struct - encoded automatically

    init(title: String, priority: Priority) {
        self.title = title
        self.priority = priority
    }
}
```

## Property Attributes

### @Attribute(.unique)

Ensures property values are unique across all instances:

```swift
@Model
final class User {
    @Attribute(.unique) var email: String
    var name: String
}
```

**Warning**: Don't use `@Attribute(.unique)` with CloudKit sync - it causes conflicts.

### @Attribute(.externalStorage)

Stores large data externally (files, images):

```swift
@Model
final class Photo {
    var name: String
    @Attribute(.externalStorage) var imageData: Data?
}
```

### @Attribute(.spotlight)

Enables Spotlight indexing:

```swift
@Model
final class Note {
    @Attribute(.spotlight) var title: String
    @Attribute(.spotlight) var content: String
}
```

### @Attribute(.encrypt)

Encrypts sensitive data:

```swift
@Model
final class Secret {
    @Attribute(.encrypt) var apiKey: String
}
```

### @Transient

Excludes property from persistence:

```swift
@Model
final class Document {
    var title: String
    var content: String
    @Transient var wordCount: Int = 0  // Not saved
}
```

## Computed Properties

Computed properties are automatically transient:

```swift
@Model
final class Person {
    var firstName: String
    var lastName: String

    var fullName: String {  // Not persisted
        "\(firstName) \(lastName)"
    }
}
```

## Default Values

Always provide default values or make properties optional:

```swift
@Model
final class Item {
    var name: String
    var quantity: Int = 1           // Default value
    var notes: String?              // Optional
    var createdAt: Date = .now      // Default value
}
```

## CloudKit Compatibility

For iCloud sync, follow these rules:

```swift
@Model
final class CloudItem {
    // NO @Attribute(.unique) with CloudKit
    var id: UUID = UUID()  // Use default instead

    // All relationships must be optional
    var category: Category?

    // Properties need defaults or be optional
    var name: String = ""
    var count: Int = 0
    var notes: String?
}
```

## Best Practices

1. **Use `final class`** - Prevents subclassing issues
2. **Provide initializers** - Don't rely on defaults only
3. **Use optionals for nullable data** - Not empty strings
4. **Mark large data as external** - Images, files, blobs
5. **Avoid unique constraints with CloudKit** - Use app logic instead

## Checklist

- [ ] Model is a `final class` with `@Model`
- [ ] All properties have defaults or are optional
- [ ] Large data uses `@Attribute(.externalStorage)`
- [ ] No `@Attribute(.unique)` if using CloudKit
- [ ] Codable types conform to `Codable`
