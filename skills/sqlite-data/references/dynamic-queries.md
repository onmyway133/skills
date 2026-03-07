# Dynamic Queries

Build queries with runtime parameters using FetchKeyRequest.

## FetchKeyRequest Protocol

For queries that depend on runtime values (search, filters, pagination):

```swift
struct RemindersRequest: FetchKeyRequest {
    var searchQuery = ""
    var showCompleted = false

    struct Value {
        var reminders: [Reminder] = []
        var totalCount = 0
        var filteredCount = 0
    }

    func fetch(_ db: Database) throws -> Value {
        var query = Reminder.all

        if !searchQuery.isEmpty {
            query = query.where { $0.title.contains(searchQuery) }
        }

        if !showCompleted {
            query = query.where { !$0.isCompleted }
        }

        query = query.order(by: \.title)

        return try Value(
            reminders: query.fetchAll(db),
            totalCount: Reminder.fetchCount(db),
            filteredCount: query.fetchCount(db)
        )
    }
}
```

## Using @Fetch with Dynamic Queries

```swift
struct SearchView: View {
    @Fetch(RemindersRequest())
    var data = RemindersRequest.Value()

    @State var searchText = ""
    @State var showCompleted = false

    var body: some View {
        List(data.reminders) { reminder in
            ReminderRow(reminder: reminder)
        }
        .searchable(text: $searchText)
        .toolbar {
            Toggle("Completed", isOn: $showCompleted)
        }
        .task(id: searchText) {
            await updateQuery()
        }
        .task(id: showCompleted) {
            await updateQuery()
        }
    }

    func updateQuery() async {
        let request = RemindersRequest(
            searchQuery: searchText,
            showCompleted: showCompleted
        )
        try? await $data.load(request, animation: .default).task
    }
}
```

## Search Implementation

Complete search implementation:

```swift
struct DynamicSearchDemo: View {
    @Fetch(SearchRequest()) var results = SearchRequest.Value()
    @State var query = ""

    var body: some View {
        List {
            Section {
                if query.isEmpty {
                    Text("Total: \(results.totalCount)")
                } else {
                    Text("Found: \(results.searchCount) of \(results.totalCount)")
                }
            }

            Section {
                ForEach(results.items) { item in
                    Text(item.title)
                }
            }
        }
        .searchable(text: $query)
        .task(id: query) {
            try? await $results.load(
                SearchRequest(query: query),
                animation: .default
            ).task
        }
    }
}

private struct SearchRequest: FetchKeyRequest {
    var query = ""

    struct Value {
        var items: [Item] = []
        var searchCount = 0
        var totalCount = 0
    }

    func fetch(_ db: Database) throws -> Value {
        let search = Item
            .where { $0.title.contains(query) }
            .order { $0.title.asc() }

        return try Value(
            items: search.fetchAll(db),
            searchCount: search.fetchCount(db),
            totalCount: Item.fetchCount(db)
        )
    }
}
```

## Pagination

```swift
struct PaginatedRequest: FetchKeyRequest {
    var page = 0
    var pageSize = 20

    struct Value {
        var items: [Item] = []
        var hasMore = false
        var totalCount = 0
    }

    func fetch(_ db: Database) throws -> Value {
        let total = try Item.fetchCount(db)
        let items = try Item
            .order(by: \.createdAt)
            .limit(pageSize, offset: page * pageSize)
            .fetchAll(db)

        return try Value(
            items: items,
            hasMore: (page + 1) * pageSize < total,
            totalCount: total
        )
    }
}
```

## Multiple Aggregations

Fetch multiple aggregations in a single transaction:

```swift
struct DashboardRequest: FetchKeyRequest {
    struct Value {
        var pendingCount = 0
        var completedCount = 0
        var overdueCount = 0
        var recentItems: [Reminder] = []
    }

    func fetch(_ db: Database) throws -> Value {
        let now = Date()

        return try Value(
            pendingCount: Reminder
                .where { !$0.isCompleted }
                .fetchCount(db),
            completedCount: Reminder
                .where(\.isCompleted)
                .fetchCount(db),
            overdueCount: Reminder
                .where { !$0.isCompleted && $0.dueDate < now }
                .fetchCount(db),
            recentItems: Reminder
                .order { $0.createdAt.desc() }
                .limit(5)
                .fetchAll(db)
        )
    }
}
```

## Complex Filters

```swift
struct AdvancedFilterRequest: FetchKeyRequest {
    var categories: Set<Category> = []
    var dateRange: ClosedRange<Date>?
    var minPriority: Priority?
    var sortOrder: SortOrder = .dateDesc

    enum SortOrder {
        case dateAsc, dateDesc, titleAsc, priorityDesc
    }

    struct Value {
        var items: [Item] = []
    }

    func fetch(_ db: Database) throws -> Value {
        var query = Item.all

        // Apply category filter
        if !categories.isEmpty {
            query = query.where { $0.category.in(categories) }
        }

        // Apply date range
        if let range = dateRange {
            query = query.where {
                $0.createdAt >= range.lowerBound &&
                $0.createdAt <= range.upperBound
            }
        }

        // Apply priority filter
        if let minPriority {
            query = query.where { $0.priority >= minPriority }
        }

        // Apply sort
        switch sortOrder {
        case .dateAsc:
            query = query.order { $0.createdAt.asc() }
        case .dateDesc:
            query = query.order { $0.createdAt.desc() }
        case .titleAsc:
            query = query.order { $0.title.asc() }
        case .priorityDesc:
            query = query.order { $0.priority.desc() }
        }

        return try Value(items: query.fetchAll(db))
    }
}
```

## Debouncing Search

```swift
struct SearchView: View {
    @Fetch(SearchRequest()) var results = SearchRequest.Value()
    @State var query = ""

    var body: some View {
        List(results.items) { item in
            Text(item.title)
        }
        .searchable(text: $query)
        .task(id: query) {
            // Debounce: wait before executing search
            try? await Task.sleep(for: .milliseconds(300))
            guard !Task.isCancelled else { return }

            try? await $results.load(
                SearchRequest(query: query)
            ).task
        }
    }
}
```

## Private FetchKeyRequest

Keep request types private when only used in one view:

```swift
struct RemindersView: View {
    @Fetch(Reminders()) var data = Reminders.Value()

    // Private nested type
    private struct Reminders: FetchKeyRequest {
        struct Value {
            var reminders: [Reminder] = []
            var count = 0
        }

        func fetch(_ db: Database) throws -> Value {
            try Value(
                reminders: Reminder.order(by: \.title).fetchAll(db),
                count: Reminder.fetchCount(db)
            )
        }
    }

    var body: some View {
        List(data.reminders) { reminder in
            Text(reminder.title)
        }
    }
}
```
