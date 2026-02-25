# Containers and Contexts

## Overview

SwiftData uses three core types for data management:
- **ModelContainer**: Manages the database file and schema
- **ModelContext**: Tracks in-memory changes (insert, update, delete)
- **ModelConfiguration**: Configures storage options

## ModelContainer

The container creates and manages the database:

```swift
// Simple setup - creates default database
let container = try ModelContainer(for: Book.self)

// Multiple models
let container = try ModelContainer(for: Book.self, Author.self, Publisher.self)

// Or use schema
let schema = Schema([Book.self, Author.self])
let container = try ModelContainer(for: schema)
```

### App Setup

```swift
@main
struct MyApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(for: Book.self)
    }
}
```

### Custom Configuration

```swift
let config = ModelConfiguration(
    "MyStore",
    schema: Schema([Book.self]),
    isStoredInMemoryOnly: false,
    allowsSave: true
)

let container = try ModelContainer(
    for: Book.self,
    configurations: config
)
```

## ModelContext

The context tracks changes and provides CRUD operations:

```swift
// Access in SwiftUI
@Environment(\.modelContext) var modelContext

// Create from container
let context = ModelContext(container)

// Or use main context
let context = container.mainContext
```

### CRUD Operations

```swift
// Create
let book = Book(title: "Swift Guide")
modelContext.insert(book)

// Read
let descriptor = FetchDescriptor<Book>()
let books = try modelContext.fetch(descriptor)

// Update - just modify properties
book.title = "Updated Title"

// Delete
modelContext.delete(book)
```

### Saving

```swift
// Autosave is ON by default
// Manual save if needed
try modelContext.save()

// Check for unsaved changes
if modelContext.hasChanges {
    try modelContext.save()
}
```

### Rollback Changes

```swift
// Discard all unsaved changes
modelContext.rollback()
```

### Undo/Redo

```swift
// Enable undo manager
modelContext.undoManager = UndoManager()

// Undo last change
modelContext.undoManager?.undo()

// Redo
modelContext.undoManager?.redo()
```

## ModelConfiguration

Configure storage behavior:

```swift
// In-memory only (for previews, testing)
let config = ModelConfiguration(isStoredInMemoryOnly: true)

// Custom file name
let config = ModelConfiguration("MyDatabase")

// Read-only
let config = ModelConfiguration(allowsSave: false)

// CloudKit sync
let config = ModelConfiguration(cloudKitDatabase: .private("iCloud.com.myapp"))
```

### Multiple Configurations

Store different models in different databases:

```swift
let localConfig = ModelConfiguration(
    "LocalData",
    schema: Schema([CacheItem.self]),
    cloudKitDatabase: .none
)

let cloudConfig = ModelConfiguration(
    "CloudData",
    schema: Schema([UserData.self]),
    cloudKitDatabase: .private("iCloud.com.myapp")
)

let container = try ModelContainer(
    for: Schema([CacheItem.self, UserData.self]),
    configurations: [localConfig, cloudConfig]
)
```

## Thread Safety

**Critical Rule**: ModelContext is NOT thread-safe.

```swift
// WRONG - context on different thread
Task.detached {
    let books = try await modelContext.fetch(descriptor)  // Crash risk!
}

// CORRECT - create context on background thread
Task.detached {
    let context = ModelContext(container)
    let books = try context.fetch(descriptor)

    // Process books here

    // If needed on main thread
    await MainActor.run {
        // Update UI
    }
}
```

### Background Context Pattern

```swift
actor DataProcessor {
    let container: ModelContainer

    init(container: ModelContainer) {
        self.container = container
    }

    func processBooks() async throws {
        let context = ModelContext(container)

        let descriptor = FetchDescriptor<Book>()
        let books = try context.fetch(descriptor)

        for book in books {
            book.processedAt = .now
        }

        try context.save()
    }
}
```

## Autosave Behavior

```swift
// Autosave is ON by default
// Disable for manual control
modelContext.autosaveEnabled = false

// With disabled autosave, must call:
try modelContext.save()
```

### When Autosave Triggers

- View disappears
- Scene goes to background
- After short delay following changes
- NOT immediately after each change

## Container in Previews

```swift
struct BookList_Previews: PreviewProvider {
    static var previews: some View {
        BookList()
            .modelContainer(previewContainer)
    }

    static var previewContainer: ModelContainer = {
        let config = ModelConfiguration(isStoredInMemoryOnly: true)
        let container = try! ModelContainer(for: Book.self, configurations: config)

        // Add sample data
        let context = container.mainContext
        context.insert(Book(title: "Sample Book"))

        return container
    }()
}
```

## Reset Container

Delete all data:

```swift
try container.mainContext.delete(model: Book.self)
try container.mainContext.save()
```

Or delete the store file:

```swift
let storeURL = container.configurations.first?.url
try FileManager.default.removeItem(at: storeURL!)
```

## Best Practices

1. **One container per app** - Usually set at app level
2. **Use @Environment for context** - In SwiftUI views
3. **Create new context for background** - Never share across threads
4. **Rely on autosave** - Disable only when necessary
5. **Use in-memory for previews** - Fast and isolated

## Checklist

- [ ] Container set up at app level
- [ ] Context accessed via @Environment in views
- [ ] Background work uses separate context
- [ ] Autosave behavior matches requirements
- [ ] Preview uses in-memory configuration
