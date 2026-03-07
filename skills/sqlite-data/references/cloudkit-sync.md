# CloudKit Synchronization

Sync your SQLite database to iCloud with SyncEngine.

## Requirements

- iOS 17+, macOS 14+, tvOS 17+, watchOS 10+
- iCloud entitlements configured
- Background Modes capability with "Remote notifications" enabled
- For sharing: `CKSharingSupported` key in Info.plist set to `true`

## Basic Setup

### Step 1: Configure App Entry Point

```swift
@main
struct MyApp: App {
    init() {
        try! prepareDependencies {
            $0.defaultDatabase = try appDatabase()
            $0.defaultSyncEngine = try SyncEngine(
                for: $0.defaultDatabase,
                tables: RemindersList.self, Reminder.self
            )
        }
    }

    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
```

### Step 2: Attach Metadata (Optional)

Access CloudKit metadata like `CKRecord` and `CKShare`:

```swift
func appDatabase() throws -> any DatabaseWriter {
    var configuration = Configuration()
    configuration.prepareDatabase { db in
        try db.attachMetadatabase()
    }
    // ...
}
```

## Schema Requirements

### Globally Unique Primary Keys

Primary keys must be UUIDs, not auto-incrementing integers:

```sql
CREATE TABLE "reminders" (
    "id" TEXT PRIMARY KEY NOT NULL ON CONFLICT REPLACE DEFAULT (uuid()),
    "title" TEXT NOT NULL DEFAULT ''
) STRICT
```

### Primary Key on Every Table

Including join tables:

```sql
CREATE TABLE "reminderTags" (
    "id" TEXT PRIMARY KEY NOT NULL ON CONFLICT REPLACE DEFAULT (uuid()),
    "reminderID" TEXT NOT NULL REFERENCES "reminders"("id") ON DELETE CASCADE,
    "tagID" TEXT NOT NULL REFERENCES "tags"("id") ON DELETE CASCADE
) STRICT
```

### No Unique Constraints

Unique constraints (except primary key) are not allowed for distributed schemas.

### Foreign Key Actions

Only `CASCADE`, `SET NULL`, and `SET DEFAULT` are supported for `ON DELETE`.

## Adding Columns (Migrations)

New columns must be nullable OR have a default with `ON CONFLICT REPLACE`:

```sql
-- Option 1: Nullable
ALTER TABLE "reminders"
ADD COLUMN "priority" INTEGER

-- Option 2: Default with ON CONFLICT REPLACE
ALTER TABLE "reminders"
ADD COLUMN "priority" INTEGER NOT NULL ON CONFLICT REPLACE DEFAULT 0
```

## Private Tables

Exclude tables from sync (e.g., FTS indexes, caches):

```swift
$0.defaultSyncEngine = try SyncEngine(
    for: $0.defaultDatabase,
    tables: RemindersList.self, Reminder.self,
    privateTables: ReminderSearchIndex.self
)
```

## Observing Sync State

```swift
struct ContentView: View {
    @Dependency(\.defaultSyncEngine) var syncEngine

    var body: some View {
        List {
            // ...
        }
        .toolbar {
            if syncEngine.isSyncing {
                ProgressView()
            }
        }
    }
}
```

## Record Sharing

Share records with other iCloud users:

### Present Share Sheet

```swift
@Dependency(\.defaultDatabase) var database
@Dependency(\.defaultSyncEngine) var syncEngine

func shareList(_ list: RemindersList) async throws {
    // Get or create CKShare
    let share = try await syncEngine.share(list)

    // Present UICloudSharingController
    let controller = UICloudSharingController(share: share, container: container)
    present(controller, animated: true)
}
```

### Check Sharing Status

```swift
@Selection
struct ListWithSharingInfo {
    let list: RemindersList
    let isShared: Bool
}

@FetchAll(
    RemindersList
        .leftJoin(SyncMetadata.all) { $0.syncMetadataID.eq($1.id) }
        .select {
            ListWithSharingInfo.Columns(
                list: $0,
                isShared: $1.isShared ?? false
            )
        }
)
var listsWithSharingInfo
```

### Get Share Participants

```swift
let share = try database.read { db in
    try SyncMetadata
        .find(list.syncMetadataID)
        .select(\.share)
        .fetchOne(db)
}

if let share {
    let participants = share.participants
    let owner = share.owner
}
```

## Accessing CloudKit Metadata

### Get CKRecord for a Row

```swift
let ckRecord = try database.read { db in
    try SyncMetadata
        .find(reminder.syncMetadataID)
        .select(\.lastKnownServerRecord)
        .fetchOne(db)
}
```

### Fetch Latest from CloudKit

```swift
let lastKnownRecord = try database.read { db in
    try SyncMetadata
        .find(reminder.syncMetadataID)
        .select(\.lastKnownServerRecord)
        .fetchOne(db)
}

if let lastKnownRecord {
    let latestRecord = try await container.privateCloudDatabase
        .record(for: lastKnownRecord.recordID)
}
```

## Conflict Resolution

Conflicts are resolved automatically using "last edit wins" per column. Each column tracks its edit timestamp, and the most recent edit wins.

## Reserved CloudKit Keywords

Avoid these column names:
- `creationDate`
- `creatorUserRecordID`
- `etag`
- `lastModifiedUserRecordID`
- `modificationDate`
- `recordChangeTag`
- `recordID`
- `recordType`

## Testing and Previews

### Bootstrap Helper

```swift
extension DependencyValues {
    mutating func bootstrapDatabase() throws {
        defaultDatabase = try appDatabase()
        defaultSyncEngine = try SyncEngine(
            for: defaultDatabase,
            tables: RemindersList.self, Reminder.self
        )
    }
}

// App entry
@main
struct MyApp: App {
    init() {
        try! prepareDependencies { try! $0.bootstrapDatabase() }
    }
}

// Tests
@Suite(.dependencies { try! $0.bootstrapDatabase() })
struct MySuite { }

// Previews
#Preview {
    let _ = try! prepareDependencies { try! $0.bootstrapDatabase() }
    ContentView()
}
```

## Migrating Existing Schema

If you have auto-incrementing integer IDs, migrate to UUIDs:

```swift
migrator.registerMigration("Migrate to UUID primary keys") { db in
    try SyncEngine.migratePrimaryKeys(
        db,
        tables: Reminder.self, RemindersList.self, Tag.self
    )
}
```

## Triggers with Sync

Skip triggers when sync engine is updating:

```swift
#sql("""
    CREATE TEMPORARY TRIGGER "update_timestamp"
    AFTER UPDATE ON "reminders"
    FOR EACH ROW WHEN NOT \(SyncEngine.$isSynchronizing)
    BEGIN
        UPDATE "reminders"
        SET "updatedAt" = datetime('now')
        WHERE "id" = NEW."id";
    END
    """).execute(db)
```

## Assets (BLOB Data)

BLOB columns automatically become CKAssets. Store large blobs in separate tables:

```swift
@Table
struct RemindersList: Identifiable {
    let id: UUID
    var title = ""
}

@Table
struct RemindersListCoverImage {
    @Column(primaryKey: true)
    let remindersListID: RemindersList.ID
    var image: Data
}
```

## Simulator Limitations

Simulators don't support push notifications. Force sync by:
1. Close and reopen the app
2. Kill and relaunch the app
