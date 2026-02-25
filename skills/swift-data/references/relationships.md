# Relationships

## Overview

SwiftData supports three relationship types: one-to-one, one-to-many, and many-to-many. Relationships are inferred automatically but can be explicitly defined with `@Relationship`.

## One-to-One Relationships

Each instance relates to exactly one instance of another model:

```swift
@Model
final class User {
    var name: String
    var profile: Profile?  // One user has one profile

    init(name: String) {
        self.name = name
    }
}

@Model
final class Profile {
    var bio: String
    var user: User?  // One profile belongs to one user

    init(bio: String) {
        self.bio = bio
    }
}
```

## One-to-Many Relationships

One instance relates to multiple instances:

```swift
@Model
final class Author {
    var name: String
    var books: [Book]  // One author has many books

    init(name: String) {
        self.name = name
        self.books = []
    }
}

@Model
final class Book {
    var title: String
    var author: Author?  // Each book has one author

    init(title: String, author: Author) {
        self.title = title
        self.author = author
    }
}
```

**Key Rule**: One side must be an `Array`, the other must be optional.

## Many-to-Many Relationships

Both sides have arrays:

```swift
@Model
final class Student {
    var name: String
    var courses: [Course]

    init(name: String) {
        self.name = name
        self.courses = []
    }
}

@Model
final class Course {
    var title: String
    var students: [Student]

    init(title: String) {
        self.title = title
        self.students = []
    }
}
```

## Explicit @Relationship

Specify inverse relationships and delete rules:

```swift
@Model
final class Department {
    var name: String

    @Relationship(deleteRule: .cascade, inverse: \Employee.department)
    var employees: [Employee]

    init(name: String) {
        self.name = name
        self.employees = []
    }
}

@Model
final class Employee {
    var name: String
    var department: Department?

    init(name: String) {
        self.name = name
    }
}
```

## Delete Rules

| Rule | Behavior |
|------|----------|
| `.nullify` | Set related object's reference to nil (default) |
| `.cascade` | Delete related objects |
| `.deny` | Prevent deletion if relationships exist |
| `.noAction` | Do nothing (can create orphans) |

### Cascade Delete Example

```swift
@Model
final class Folder {
    var name: String

    @Relationship(deleteRule: .cascade)
    var files: [File]  // Deleting folder deletes all files
}

@Model
final class File {
    var name: String
    var folder: Folder?
}
```

### Nullify Example

```swift
@Model
final class Team {
    var name: String

    @Relationship(deleteRule: .nullify)
    var members: [Player]  // Deleting team sets player.team to nil
}

@Model
final class Player {
    var name: String
    var team: Team?
}
```

## Relationship Constraints

Limit the number of related objects:

```swift
@Model
final class Playlist {
    var name: String

    @Relationship(minimumModelCount: 1, maximumModelCount: 100)
    var songs: [Song]  // 1-100 songs required
}
```

## Working with Relationships

### Adding Related Objects

```swift
// Create parent first
let author = Author(name: "Swift Author")
modelContext.insert(author)

// Create child with relationship
let book = Book(title: "Swift Guide", author: author)
modelContext.insert(book)

// Or add to collection
author.books.append(book)
```

### Removing Relationships

```swift
// Remove from collection (doesn't delete the object)
author.books.removeAll { $0.title == "Old Book" }

// Set optional to nil
book.author = nil

// Delete the object entirely
modelContext.delete(book)
```

### Querying Relationships

```swift
// Filter by relationship property
@Query(filter: #Predicate<Book> { $0.author?.name == "Swift Author" })
var books: [Book]

// Access related objects
for book in author.books {
    print(book.title)
}
```

## CloudKit Compatibility

For iCloud sync:

```swift
@Model
final class CloudParent {
    var name: String

    // Relationships MUST be optional for CloudKit
    @Relationship(deleteRule: .nullify)
    var children: [CloudChild]?  // Optional array
}

@Model
final class CloudChild {
    var name: String
    var parent: CloudParent?  // Optional
}
```

## Best Practices

1. **Always define inverse relationships** - Prevents orphaned data
2. **Use optional for CloudKit** - Required for sync compatibility
3. **Choose delete rules carefully** - Cascade can delete more than expected
4. **Initialize arrays as empty** - Don't use nil for collections
5. **Avoid deep nesting** - Hurts query performance

## Checklist

- [ ] Inverse relationships are defined
- [ ] Delete rules match business logic
- [ ] Relationships are optional for CloudKit
- [ ] Arrays initialized as empty `[]`
- [ ] No circular cascade deletes
