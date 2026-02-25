# Core Data

## Overview

Production-focused Core Data guidance covering stack setup, fetch requests, threading, batch operations, migrations, and CloudKit sync.

## Core Principles

1. **Context awareness** - Distinguish view contexts (UI) from background contexts (heavy work)
2. **Safe object passing** - Use `NSManagedObjectID`, never pass `NSManagedObject` across contexts
3. **Migration strategy** - Prefer lightweight migration; use staged migration (iOS 17+) for complexity
4. **Batch operations** - Require persistent history tracking for UI synchronization
5. **CloudKit caveat** - Production schema is immutable

## Stack Setup

```swift
class PersistenceController {
    static let shared = PersistenceController()

    let container: NSPersistentContainer

    init(inMemory: Bool = false) {
        container = NSPersistentContainer(name: "Model")

        if inMemory {
            container.persistentStoreDescriptions.first?.url = URL(fileURLWithPath: "/dev/null")
        }

        container.loadPersistentStores { description, error in
            if let error = error {
                fatalError("Core Data failed: \(error)")
            }
        }

        container.viewContext.automaticallyMergesChangesFromParent = true
        container.viewContext.mergePolicy = NSMergeByPropertyObjectTrumpMergePolicy
    }

    var viewContext: NSManagedObjectContext {
        container.viewContext
    }

    func newBackgroundContext() -> NSManagedObjectContext {
        container.newBackgroundContext()
    }
}
```

## Threading Safety

```swift
// WRONG - passing object across contexts
let item = viewContext.fetch(request).first!
Task.detached {
    item.name = "Updated" // CRASH: wrong context
}

// CORRECT - pass object ID
let objectID = item.objectID
Task.detached {
    let context = container.newBackgroundContext()
    let item = try context.existingObject(with: objectID) as! Item
    item.name = "Updated"
    try context.save()
}
```

## Async/Await Pattern

```swift
func importItems(_ data: [ItemData]) async throws {
    let context = container.newBackgroundContext()

    try await context.perform {
        for itemData in data {
            let item = Item(context: context)
            item.configure(from: itemData)
        }
        try context.save()
    }
}
```

## Fetch Requests

```swift
// Basic fetch
let request = Item.fetchRequest()
request.predicate = NSPredicate(format: "isActive == %@", NSNumber(value: true))
request.sortDescriptors = [NSSortDescriptor(keyPath: \Item.date, ascending: false)]
request.fetchLimit = 50

let items = try viewContext.fetch(request)

// Fetch with relationships
request.relationshipKeyPathsForPrefetching = ["category", "tags"]

// Count only
let count = try viewContext.count(for: request)
```

## Batch Operations

```swift
// Batch insert (iOS 14+)
let batchInsert = NSBatchInsertRequest(entity: Item.entity()) { (dictionary: NSMutableDictionary) in
    guard index < data.count else { return true }
    dictionary["id"] = data[index].id
    dictionary["name"] = data[index].name
    index += 1
    return false
}
batchInsert.resultType = .objectIDs

// Batch delete
let batchDelete = NSBatchDeleteRequest(fetchRequest: fetchRequest)
batchDelete.resultType = .resultTypeObjectIDs

let result = try context.execute(batchDelete) as! NSBatchDeleteResult
let objectIDs = result.result as! [NSManagedObjectID]

// Merge changes to view context
NSManagedObjectContext.mergeChanges(
    fromRemoteContextSave: [NSDeletedObjectsKey: objectIDs],
    into: [viewContext]
)
```

## Migration

```swift
// Lightweight migration (automatic)
let description = NSPersistentStoreDescription()
description.shouldMigrateStoreAutomatically = true
description.shouldInferMappingModelAutomatically = true

// Staged migration (iOS 17+)
let stages = [
    NSMigrationStage.lightweight(fromVersionName: "Model", toVersionName: "Model 2")
]
```

## Common Errors

| Error | Solution |
|-------|----------|
| "Failed to find unique match for NSEntityDescription" | Check model in test target |
| `NSPersistentStoreIncompatibleVersionHashError` | Migration needed |
| Threading exception | Use `NSManagedObjectID` |
| Merge conflicts | Check merge policy |
| Batch ops skip UI | Enable persistent history |

## External References

For comprehensive Core Data guidance, see:

- **[Core-Data-Agent-Skill](https://github.com/AvdLee/Core-Data-Agent-Skill)** - Complete Core Data patterns with 14 reference modules
