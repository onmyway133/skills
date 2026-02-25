# Queries and Filtering

## Overview

SwiftData provides two ways to fetch data: `@Query` property wrapper for SwiftUI views and `FetchDescriptor` for programmatic access.

## @Query in SwiftUI

The `@Query` property wrapper automatically fetches and updates data:

```swift
struct BookList: View {
    @Query var books: [Book]

    var body: some View {
        List(books) { book in
            Text(book.title)
        }
    }
}
```

### Sorting with @Query

```swift
// Single sort
@Query(sort: \Book.title) var books: [Book]

// Descending order
@Query(sort: \Book.publishedDate, order: .reverse) var books: [Book]

// Multiple sort descriptors
@Query(sort: [
    SortDescriptor(\Book.author),
    SortDescriptor(\Book.title)
]) var books: [Book]
```

### Filtering with @Query

```swift
// Simple predicate
@Query(filter: #Predicate<Book> { $0.rating > 3 })
var topBooks: [Book]

// Multiple conditions
@Query(filter: #Predicate<Book> {
    $0.rating > 3 && $0.isPublished == true
})
var publishedTopBooks: [Book]

// String contains
@Query(filter: #Predicate<Book> {
    $0.title.localizedStandardContains("Swift")
})
var swiftBooks: [Book]
```

### Combined Sort and Filter

```swift
@Query(
    filter: #Predicate<Book> { $0.rating > 3 },
    sort: \Book.title
) var topBooks: [Book]
```

### Limiting Results

```swift
@Query(sort: \Book.publishedDate, order: .reverse)
var books: [Book]

// In view, take first N
List(books.prefix(10)) { book in
    Text(book.title)
}
```

### Animation

```swift
@Query(sort: \Book.title, animation: .default)
var books: [Book]
```

## Dynamic Queries

Change query parameters at runtime:

```swift
struct BookList: View {
    @State private var searchText = ""
    @State private var sortOrder = SortOrder.title

    var body: some View {
        BookListContent(searchText: searchText, sortOrder: sortOrder)
    }
}

struct BookListContent: View {
    @Query var books: [Book]

    init(searchText: String, sortOrder: SortOrder) {
        let predicate = #Predicate<Book> { book in
            searchText.isEmpty || book.title.localizedStandardContains(searchText)
        }

        let sort: SortDescriptor<Book> = switch sortOrder {
        case .title: SortDescriptor(\Book.title)
        case .date: SortDescriptor(\Book.publishedDate, order: .reverse)
        }

        _books = Query(filter: predicate, sort: [sort])
    }

    var body: some View {
        List(books) { book in
            Text(book.title)
        }
    }
}
```

## FetchDescriptor

For programmatic fetching outside SwiftUI:

```swift
// Basic fetch
let descriptor = FetchDescriptor<Book>()
let books = try modelContext.fetch(descriptor)

// With predicate
let descriptor = FetchDescriptor<Book>(
    predicate: #Predicate { $0.rating > 3 }
)

// With sorting
let descriptor = FetchDescriptor<Book>(
    predicate: #Predicate { $0.isPublished },
    sortBy: [SortDescriptor(\.title)]
)
```

### Fetch Options

```swift
var descriptor = FetchDescriptor<Book>()

// Limit results
descriptor.fetchLimit = 10

// Skip results (pagination)
descriptor.fetchOffset = 20

// Prefetch relationships
descriptor.relationshipKeyPathsForPrefetching = [\.author]
```

### Counting Results

```swift
let descriptor = FetchDescriptor<Book>(
    predicate: #Predicate { $0.rating > 3 }
)
let count = try modelContext.fetchCount(descriptor)
```

## #Predicate Macro

Type-safe predicates with Swift syntax:

### Comparison Operators

```swift
#Predicate<Book> { $0.rating > 3 }
#Predicate<Book> { $0.rating >= 4 }
#Predicate<Book> { $0.rating == 5 }
#Predicate<Book> { $0.rating != 0 }
```

### Logical Operators

```swift
// AND
#Predicate<Book> { $0.rating > 3 && $0.isPublished }

// OR
#Predicate<Book> { $0.rating == 5 || $0.isFeatured }

// NOT
#Predicate<Book> { !$0.isArchived }
```

### String Operations

```swift
// Contains (case-insensitive)
#Predicate<Book> { $0.title.localizedStandardContains("swift") }

// Starts with
#Predicate<Book> { $0.title.starts(with: "The") }

// Case-sensitive contains
#Predicate<Book> { $0.title.contains("Swift") }
```

### Optional Handling

```swift
// Check for nil
#Predicate<Book> { $0.author != nil }

// Safe navigation
#Predicate<Book> { $0.author?.name == "Swift Author" }

// With nil coalescing
#Predicate<Book> { ($0.rating ?? 0) > 3 }
```

### Collections

```swift
// Array contains
#Predicate<Author> { $0.books.contains(where: { $0.rating > 4 }) }

// Array count
#Predicate<Author> { $0.books.count > 5 }
```

### Date Comparisons

```swift
let oneWeekAgo = Calendar.current.date(byAdding: .day, value: -7, to: .now)!

#Predicate<Book> { $0.publishedDate > oneWeekAgo }
```

## Find by ID

```swift
// Using PersistentIdentifier
let book = modelContext.model(for: bookID) as? Book

// Using fetch
let descriptor = FetchDescriptor<Book>(
    predicate: #Predicate { $0.persistentModelID == bookID }
)
let books = try modelContext.fetch(descriptor)
let book = books.first
```

## Batch Delete

```swift
let predicate = #Predicate<Book> { $0.isArchived }

try modelContext.delete(model: Book.self, where: predicate)
```

## Best Practices

1. **Use @Query for SwiftUI** - Automatic updates and animations
2. **Prefer predicates over fetching all** - Better performance
3. **Limit results when possible** - Use `fetchLimit`
4. **Prefetch relationships** - Avoid N+1 queries
5. **Use dynamic queries** - For search and filters

## Checklist

- [ ] Using @Query in SwiftUI views
- [ ] Predicates filter data appropriately
- [ ] Sort order matches UX expectations
- [ ] Large datasets use pagination
- [ ] Relationships prefetched when needed
