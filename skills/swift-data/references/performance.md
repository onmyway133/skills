# Performance

## Overview

SwiftData is built on Core Data with optimizations for Swift. Follow these patterns for best performance.

## Lazy Loading

SwiftData loads data lazily by default:

```swift
@Model
final class Author {
    var name: String
    var books: [Book]?  // Not loaded until accessed
}

// Books loaded only when accessed
let author = authors.first!
print(author.name)       // Loaded
print(author.books)      // NOW loaded
```

## Prefetching Relationships

Avoid N+1 queries by prefetching:

```swift
var descriptor = FetchDescriptor<Author>()
descriptor.relationshipKeyPathsForPrefetching = [\.books]

let authors = try modelContext.fetch(descriptor)

// Books already loaded - no additional queries
for author in authors {
    print(author.books?.count ?? 0)
}
```

## Indexing Properties

Add indexes for frequently queried properties:

```swift
@Model
final class User {
    @Attribute(.index)
    var email: String  // Indexed for fast queries

    @Attribute(.index)
    var createdAt: Date  // Indexed for sorting

    var name: String  // Not indexed
}
```

### Compound Indexes (iOS 18+)

```swift
@Model
final class Order {
    var customerId: String
    var orderDate: Date
    var status: String
}

// In schema definition
#Index<Order>([\.customerId, \.orderDate])
```

## Batch Operations

### Batch Insert

```swift
// WRONG - slow for large datasets
for item in largeArray {
    modelContext.insert(Item(data: item))
}
try modelContext.save()

// CORRECT - batch insert
modelContext.autosaveEnabled = false

for item in largeArray {
    modelContext.insert(Item(data: item))
}

try modelContext.save()
modelContext.autosaveEnabled = true
```

### Batch Delete

```swift
// Delete matching predicate - efficient
try modelContext.delete(
    model: Item.self,
    where: #Predicate { $0.isArchived }
)

// vs. fetching and deleting one by one - slow
let items = try modelContext.fetch(FetchDescriptor<Item>())
for item in items.filter({ $0.isArchived }) {
    modelContext.delete(item)
}
```

## Pagination

Fetch data in pages for large datasets:

```swift
class PaginatedFetcher {
    let pageSize = 50
    var currentPage = 0

    func fetchNextPage(context: ModelContext) throws -> [Item] {
        var descriptor = FetchDescriptor<Item>(
            sortBy: [SortDescriptor(\.createdAt, order: .reverse)]
        )
        descriptor.fetchLimit = pageSize
        descriptor.fetchOffset = currentPage * pageSize

        currentPage += 1
        return try context.fetch(descriptor)
    }
}
```

### With @Query

```swift
struct ItemList: View {
    @Query(sort: \Item.createdAt, order: .reverse)
    var allItems: [Item]

    @State private var displayCount = 50

    var body: some View {
        List(allItems.prefix(displayCount)) { item in
            ItemRow(item: item)
        }
        .onAppear {
            // Load more as user scrolls
        }
    }
}
```

## Background Processing

Process data on background threads:

```swift
actor DataImporter {
    let container: ModelContainer

    init(container: ModelContainer) {
        self.container = container
    }

    func importData(_ items: [ImportItem]) async throws {
        let context = ModelContext(container)
        context.autosaveEnabled = false

        for item in items {
            let model = Item(from: item)
            context.insert(model)
        }

        try context.save()
    }
}

// Usage
let importer = DataImporter(container: container)
try await importer.importData(largeDataset)
```

## Fetch Count

Count without loading objects:

```swift
// EFFICIENT - just count
let descriptor = FetchDescriptor<Item>(
    predicate: #Predicate { $0.isActive }
)
let count = try modelContext.fetchCount(descriptor)

// INEFFICIENT - loads all objects
let items = try modelContext.fetch(descriptor)
let count = items.count
```

## Limit Fetched Properties

Fetch only what you need (when available):

```swift
// Future SwiftData may support this like Core Data
// For now, design models to avoid large properties in lists

@Model
final class Article {
    var title: String           // Light - used in lists
    var summary: String         // Light - used in lists
    @Attribute(.externalStorage)
    var fullContent: Data?      // Heavy - loaded separately
}
```

## Avoiding Memory Issues

### Release References

```swift
// In long-running operations
func processLargeDataset() {
    autoreleasepool {
        let items = try modelContext.fetch(descriptor)
        for item in items {
            process(item)
        }
    }
    // Memory released here
}
```

### Use Pagination for Display

```swift
struct LargeList: View {
    @Query var items: [Item]

    var body: some View {
        // LazyVStack only loads visible items
        ScrollView {
            LazyVStack {
                ForEach(items) { item in
                    ItemRow(item: item)
                }
            }
        }
    }
}
```

## Measuring Performance

### Instruments

Use Core Data instrument (SwiftData uses Core Data under the hood):
1. Profile with Instruments
2. Select "Core Data" template
3. Look for:
   - Fetch count and duration
   - Faults and fault resolution
   - Save operations

### Debug Logging

Add launch argument:
```
-com.apple.CoreData.SQLDebug 1
```

Levels:
- `1` - Basic SQL logging
- `2` - Include parameters
- `3` - Verbose

## Common Performance Issues

### N+1 Query Problem

```swift
// PROBLEM: Each author access triggers query
for author in authors {
    print(author.books?.count)  // Query per author!
}

// SOLUTION: Prefetch
var descriptor = FetchDescriptor<Author>()
descriptor.relationshipKeyPathsForPrefetching = [\.books]
```

### Fetching Too Much

```swift
// PROBLEM: Fetch all, use few
let allItems = try modelContext.fetch(FetchDescriptor<Item>())
let recent = allItems.prefix(10)

// SOLUTION: Limit fetch
var descriptor = FetchDescriptor<Item>(
    sortBy: [SortDescriptor(\.date, order: .reverse)]
)
descriptor.fetchLimit = 10
let recent = try modelContext.fetch(descriptor)
```

### Saving Too Often

```swift
// PROBLEM: Save after each insert
for item in items {
    modelContext.insert(item)
    try modelContext.save()  // Slow!
}

// SOLUTION: Batch save
for item in items {
    modelContext.insert(item)
}
try modelContext.save()  // Once
```

## Best Practices

1. **Prefetch relationships** - Use `relationshipKeyPathsForPrefetching`
2. **Index queried properties** - Add `@Attribute(.index)`
3. **Batch operations** - Save once, not per item
4. **Use pagination** - Don't fetch everything
5. **Background for heavy work** - Separate ModelContext
6. **Count without fetch** - Use `fetchCount`

## Checklist

- [ ] Frequently queried properties indexed
- [ ] Relationships prefetched when needed
- [ ] Large operations use batch save
- [ ] Lists use pagination or lazy loading
- [ ] Heavy processing on background context
- [ ] Profile with Instruments before shipping
