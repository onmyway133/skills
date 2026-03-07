# Fetching Data

Use property wrappers to fetch and observe data from your SQLite database.

## @FetchAll

Fetch a collection of records with automatic observation:

```swift
struct ReminderList: View {
    // Fetch all reminders sorted by title
    @FetchAll(Reminder.order(by: \.title))
    var reminders

    var body: some View {
        List(reminders) { reminder in
            Text(reminder.title)
        }
    }
}
```

### With Filtering

```swift
// Completed reminders, descending by title
@FetchAll(Reminder.where(\.isCompleted).order { $0.title.desc() })
var completedReminders

// Using closure syntax
@FetchAll(Reminder.where { !$0.isCompleted })
var incompleteReminders
```

### With Animation

```swift
@FetchAll(Reminder.order(by: \.title), animation: .default)
var reminders
```

### With Raw SQL

```swift
@FetchAll(#sql("SELECT * FROM reminders WHERE isCompleted ORDER BY title DESC"))
var completedReminders: [Reminder]

// Schema-safe SQL interpolation
@FetchAll(
    #sql("""
        SELECT \(Reminder.columns)
        FROM \(Reminder.self)
        WHERE \(Reminder.isCompleted)
        ORDER BY \(Reminder.title) DESC
        """)
)
var completedReminders: [Reminder]
```

## @FetchOne

Fetch a single value (aggregates, counts, single records):

```swift
// Count
@FetchOne(Reminder.count())
var totalCount = 0

// Filtered count
@FetchOne(Reminder.where(\.isCompleted).count())
var completedCount = 0

// With animation
@FetchOne(Reminder.count(), animation: .default)
var count = 0

// Single record (optional)
@FetchOne(Reminder.where { $0.id == someID })
var reminder: Reminder?

// Raw SQL
@FetchOne(#sql("SELECT count(*) FROM reminders WHERE isCompleted"))
var completedCount = 0
```

## @Fetch

Execute multiple queries in a single transaction using `FetchKeyRequest`:

```swift
struct RemindersView: View {
    @Fetch(RemindersData()) var data = RemindersData.Value()

    var body: some View {
        VStack {
            Text("Total: \(data.totalCount)")
            Text("Completed: \(data.completedCount)")
            List(data.incompleteReminders) { reminder in
                Text(reminder.title)
            }
        }
    }
}

struct RemindersData: FetchKeyRequest {
    struct Value {
        var incompleteReminders: [Reminder] = []
        var completedCount = 0
        var totalCount = 0
    }

    func fetch(_ db: Database) throws -> Value {
        try Value(
            incompleteReminders: Reminder.where { !$0.isCompleted }
                .order(by: \.title)
                .fetchAll(db),
            completedCount: Reminder.where(\.isCompleted).fetchCount(db),
            totalCount: Reminder.fetchCount(db)
        )
    }
}
```

## Observable Models

Property wrappers work in `@Observable` models (not just SwiftUI views):

```swift
@Observable
@MainActor
class RemindersModel {
    @ObservationIgnored
    @FetchAll(Reminder.order(by: \.title))
    var reminders

    @ObservationIgnored
    @FetchOne(Reminder.count())
    var count = 0

    @ObservationIgnored
    @Dependency(\.defaultDatabase) var database

    func deleteReminder(_ reminder: Reminder) throws {
        try database.write { db in
            try Reminder.delete(reminder).execute(db)
        }
    }
}
```

**Note:** Use `@ObservationIgnored` with `@Observable` macro. The property wrappers handle their own observation.

## UIKit Integration

```swift
final class RemindersViewController: UICollectionViewController {
    @FetchAll(Reminder.order(by: \.title))
    private var reminders

    @Dependency(\.defaultDatabase) var database

    override func viewDidLoad() {
        super.viewDidLoad()

        // Observe changes
        observe { [weak self] in
            guard let self else { return }
            var snapshot = NSDiffableDataSourceSnapshot<Section, Reminder>()
            snapshot.appendSections([.main])
            snapshot.appendItems(reminders, toSection: .main)
            dataSource.apply(snapshot, animatingDifferences: true)
        }
    }
}
```

## Combine Publisher

Access a Combine publisher for the fetched data:

```swift
@FetchAll(Reminder.order(by: \.title))
var reminders

// Access publisher
$reminders.publisher
    .sink { reminders in
        print("Reminders updated: \(reminders.count)")
    }
    .store(in: &cancellables)
```

## Loading State

Check loading state and errors:

```swift
@FetchAll(Reminder.all)
var reminders

var body: some View {
    Group {
        if $reminders.isLoading {
            ProgressView()
        } else if let error = $reminders.loadError {
            Text("Error: \(error.localizedDescription)")
        } else {
            List(reminders) { reminder in
                Text(reminder.title)
            }
        }
    }
}
```

## Manual Reload

Manually trigger a reload:

```swift
@FetchAll(Reminder.all)
var reminders

func refresh() async {
    try? await $reminders.load()
}

// Or with a new query
try? await $reminders.load(.fetchAll(Reminder.where(\.isCompleted)))
```

## Custom Database

Use a specific database instead of the default:

```swift
@FetchAll(Reminder.all, database: customDatabase)
var reminders
```

## Joins and Selections

Fetch data from multiple tables:

```swift
@Selection
struct ReminderWithList {
    let reminderTitle: String
    let listTitle: String
}

@FetchAll(
    Reminder
        .join(RemindersList.all) { $0.remindersListID.eq($1.id) }
        .select {
            ReminderWithList.Columns(
                reminderTitle: $0.title,
                listTitle: $1.title
            )
        }
)
var remindersWithLists
```
