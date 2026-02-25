# CloudKit Sync

## Overview

SwiftData integrates with CloudKit for automatic iCloud synchronization. Data syncs across devices signed into the same iCloud account.

## Enable CloudKit Sync

### 1. Project Configuration

In Xcode:
1. Select your target → Signing & Capabilities
2. Add "iCloud" capability
3. Check "CloudKit"
4. Create or select a container (e.g., `iCloud.com.yourcompany.appname`)

### 2. Code Configuration

```swift
let config = ModelConfiguration(
    cloudKitDatabase: .private("iCloud.com.yourcompany.appname")
)

let container = try ModelContainer(
    for: Book.self,
    configurations: config
)
```

### 3. App Setup

```swift
@main
struct MyApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
        .modelContainer(for: Book.self, inMemory: false, isAutosaveEnabled: true, isUndoEnabled: false) {
            // Use default CloudKit configuration
        }
    }
}
```

## Model Requirements

CloudKit has strict requirements for models:

### Required: Optional Relationships

```swift
// CORRECT
@Model
final class Author {
    var name: String
    var books: [Book]?  // Optional array for CloudKit

    init(name: String) {
        self.name = name
        self.books = []
    }
}

// WRONG - will fail with CloudKit
@Model
final class Author {
    var name: String
    var books: [Book]  // Non-optional fails
}
```

### Required: No @Attribute(.unique)

```swift
// WRONG - unique constraints conflict with CloudKit
@Model
final class User {
    @Attribute(.unique) var email: String  // Don't do this!
}

// CORRECT - use app logic for uniqueness
@Model
final class User {
    var email: String
    var id: UUID = UUID()
}
```

### Required: Default Values

```swift
@Model
final class Item {
    var name: String = ""           // Default value
    var count: Int = 0              // Default value
    var notes: String?              // Or optional
    var createdAt: Date = .now      // Default value
}
```

## Database Types

```swift
// Private database - user's own data
let config = ModelConfiguration(
    cloudKitDatabase: .private("iCloud.com.myapp")
)

// Shared database - shared with other users
// Requires additional CloudKit sharing setup

// No sync - local only
let config = ModelConfiguration(
    cloudKitDatabase: .none
)
```

## Disable Sync for Specific Models

Use separate configurations:

```swift
// Models that sync
let cloudSchema = Schema([UserData.self, Settings.self])
let cloudConfig = ModelConfiguration(
    "CloudStore",
    schema: cloudSchema,
    cloudKitDatabase: .private("iCloud.com.myapp")
)

// Models that stay local
let localSchema = Schema([CacheItem.self, TempData.self])
let localConfig = ModelConfiguration(
    "LocalStore",
    schema: localSchema,
    cloudKitDatabase: .none
)

let container = try ModelContainer(
    for: Schema([UserData.self, Settings.self, CacheItem.self, TempData.self]),
    configurations: [cloudConfig, localConfig]
)
```

## Sync Status

SwiftData doesn't expose sync status directly. Monitor changes:

```swift
// Observe changes in SwiftUI
struct ContentView: View {
    @Query var items: [Item]

    var body: some View {
        List(items) { item in
            ItemRow(item: item)
        }
        .onChange(of: items) { oldValue, newValue in
            // Items updated (possibly from sync)
        }
    }
}
```

## Conflict Resolution

CloudKit uses last-write-wins by default. SwiftData handles this automatically, but be aware:

- Local changes may be overwritten by remote changes
- Design models to minimize conflicts
- Use timestamps for custom conflict resolution

```swift
@Model
final class Document {
    var content: String
    var modifiedAt: Date = .now  // Track modifications
    var modifiedBy: String = ""  // Track who modified

    func update(content: String, by user: String) {
        self.content = content
        self.modifiedAt = .now
        self.modifiedBy = user
    }
}
```

## Testing CloudKit Sync

### Development Environment

1. Use CloudKit Dashboard to inspect data
2. Sign in with different iCloud accounts on simulator/devices
3. Test offline → online transitions

### Debug Launch Arguments

Add to scheme:
```
-com.apple.CoreData.CloudKitDebug 1
```

### Check Entitlements

Ensure your entitlements file includes:
```xml
<key>com.apple.developer.icloud-container-identifiers</key>
<array>
    <string>iCloud.com.yourcompany.appname</string>
</array>
<key>com.apple.developer.icloud-services</key>
<array>
    <string>CloudKit</string>
</array>
```

## Common Issues

### "CloudKit integration requires optional relationship"

Make all relationships optional:
```swift
var author: Author?        // Not: var author: Author
var books: [Book]?         // Not: var books: [Book]
```

### Data not syncing

1. Check iCloud is signed in
2. Check network connectivity
3. Verify CloudKit container exists in dashboard
4. Check entitlements match container ID

### Duplicate records

- Don't use `@Attribute(.unique)` with CloudKit
- Implement app-level deduplication if needed

## Best Practices

1. **Test on real devices** - Simulator has sync quirks
2. **Design for eventual consistency** - Data may take time to sync
3. **Handle offline gracefully** - App should work without network
4. **Minimize conflicts** - Small, focused models sync better
5. **Use timestamps** - Track when data was modified

## Checklist

- [ ] CloudKit capability added in Xcode
- [ ] Container ID matches in code and entitlements
- [ ] All relationships are optional
- [ ] No @Attribute(.unique) used
- [ ] All properties have defaults or are optional
- [ ] Tested on real devices with real iCloud accounts
