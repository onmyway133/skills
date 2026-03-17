# Project Structure

## Folder Organization

Organize code by feature first, then by responsibility type in Library:

```
Project/
├── Features/           # Feature modules
│   ├── Home/
│   ├── Settings/
│   ├── Main/
│   └── [Feature]/
├── Library/            # Shared code
│   ├── Routes/         # Router + Route enum
│   ├── Extensions/     # View & type extensions
│   ├── Views/          # Reusable UI components
│   ├── Models/         # Data models
│   ├── Services/       # Business logic
│   ├── Database/       # Data persistence
│   └── Core/           # Core utilities
└── Resources/
    ├── Localizable.strings
    └── Assets
```

---

## Router and Navigation

Use a stack-based routing system with Router class and Route enum.

### Router Class

```swift
@Observable
class Router {
    var routes: [Route] = []

    func navigate(route: Route) {
        routes.append(route)
    }

    func pop() {
        _ = routes.popLast()
    }
}
```

### Route Enum

```swift
enum Route: Hashable {
    case home
    case settings
    case item(Item)
    case folder(Folder)
}
```

### RouteView

Map routes to views with a switch:

```swift
struct RouteView: View {
    let route: Route

    var body: some View {
        switch route {
        case .home: HomeView()
        case .settings: SettingsView()
        case let .item(item): ItemView(item: item)
        case let .folder(folder): FolderView(folder: folder)
        }
    }
}
```

### Route Handling Extension

```swift
extension View {
    func handleRoute(router: Router) -> some View {
        navigationDestination(for: Route.self) { route in
            RouteView(route: route)
        }
        .environment(router)
    }
}
```

### Usage in MainView

Each tab gets its own Router for independent navigation stacks:

```swift
struct MainView: View {
    @State var homeRouter = Router()
    @State var settingsRouter = Router()

    var body: some View {
        TabView {
            NavigationStack(path: $homeRouter.routes) {
                HomeView()
                    .handleRoute(router: homeRouter)
            }
            .tabItem { Label("Home", systemImage: "house") }

            NavigationStack(path: $settingsRouter.routes) {
                SettingsView()
                    .handleRoute(router: settingsRouter)
            }
            .tabItem { Label("Settings", systemImage: "gear") }
        }
    }
}
```

---

## Extension Patterns

### File Naming Convention

Split large views into extension files: `ViewName+section.swift`

```
Features/Home/
├── HomeView.swift              # Main view (~50-100 lines)
├── HomeView+hero.swift         # Hero section
├── HomeView+list.swift         # List section
└── HomeViewModel.swift
```

### Nested Structs for Sections

Use extensions with nested structs to organize subviews:

```swift
// HomeView+hero.swift
extension HomeView {
    struct HeroSection: View {
        @Environment(Router.self) private var router

        var body: some View {
            Section {
                // content
            }
        }
    }
}

// HomeView+list.swift
extension HomeView {
    struct ListSection: View {
        let items: [Item]

        var body: some View {
            ForEach(items) { item in
                ItemRow(item: item)
            }
        }
    }
}
```

### Custom Background and Cell Theming

When using custom backgrounds (sometimes with gradients) and custom cell colors, create background and secondary background modifiers:

```swift
// Custom background for the main view
extension View {
    func themeBackground() -> some View {
        self.background(Color.Theme.background)
    }

    func themeSecondaryBackground() -> some View {
        self.background(Color.Theme.secondaryBackground)
    }
}

// Usage with gradient background
struct HomeView: View {
    var body: some View {
        List { }
            .scrollContentBackground(.hidden)
            .background {
                LinearGradient(
                    colors: [.blue.opacity(0.3), .purple.opacity(0.2)],
                    startPoint: .top,
                    endPoint: .bottom
                )
                .ignoresSafeArea()
            }
    }
}

// Cell with themed background
struct ItemRow: View {
    var body: some View {
        content
            .listRowBackground(Color.Theme.secondaryBackground)
    }
}
```

Define theme colors in `Library/Extensions/Color+Theme.swift`:

```swift
extension Color {
    enum Theme {
        static let background = Color("Background")
        static let secondaryBackground = Color("SecondaryBackground")
    }
}
```

---

## SwiftUI View Organization

### Computed Properties for Extraction

Avoid deep nesting by extracting views to computed properties:

```swift
struct ItemView: View {
    var body: some View {
        list
            .toolbar { toolbarContent }
            .sheet(isPresented: $showsEdit) { editSheet }
    }

    private var list: some View {
        List {
            HeroSection(item: item)
            ContentSection(item: item)
        }
        .styleList()
    }

    private var toolbarContent: some ToolbarContent {
        ToolbarItem(placement: .primaryAction) {
            editButton
        }
    }
}
```

### When to Extract to Separate View

Extract to its own struct when a view has significant logic:

```swift
// Good candidates for extraction
struct BookRow: View { }
struct ColorCell: View { }
struct ReminderSection: View { }
```

Keep as computed property for simpler extractions:

```swift
private var cancelButton: some View {
    Button("Cancel") { dismiss() }
}
```

---

## MVVM

Use `@Observable` macro for iOS 17+:

```swift
@Observable
@MainActor
class HomeViewModel {
    var items: [Item] = []
    var isLoading = false

    func fetch() async { }
}
```

For Settings that need `AppStorage`, use `ObservableObject`:

```swift
class SettingsStore: ObservableObject {
    @AppStorage("theme") var theme: Theme = .system
    @AppStorage("enableSync") var enableSync = true
}
```

---

## Platform Adaptations

- **iPhone**: TabView with NavigationStack per tab
- **iPad**: NavigationSplitView with sidebar
- **Mac**: Form for settings (nice native UI)

```swift
struct MainView: View {
    @Environment(\.horizontalSizeClass) var sizeClass

    var body: some View {
        if sizeClass == .compact {
            MainTabView()
        } else {
            MainSplitView()
        }
    }
}
```

---

## SF Symbols (Type-Safe)

Use [SFSafeSymbols](https://github.com/SFSafeSymbols/SFSafeSymbols) for type-safe SF Symbol usage instead of string literals.

```swift
import SFSafeSymbols

// Use with Image
Image(systemSymbol: .starFill)
Image(systemSymbol: .gear)

// Group related icons with extensions
extension SFSymbol {
    static let folderIcons: [SFSymbol] = [
        .folder, .folderFill, .star, .bookmark, .heart
    ]
}

// Use in models
struct Item {
    var symbol: SFSymbol
}

// Use rawValue when needed for older APIs
Image(systemName: SFSymbol.xmark.rawValue)
```

---

## General Guidelines

### Comments
Only add comments for complex logic, hacks, or workarounds. Skip if code is self-explanatory.

### Previews
Don't add `#Preview` unless explicitly requested.

### String Literals
For many string literals (like JS messages), use an enum for type safety:

```swift
enum JSMessage: String {
    case reload
    case scrollToTop
}
```

### Database
Group database code in `Library/Database/`. Use SwiftData or CoreData depending on project needs.

### Parallax Headers
Use List with `.listStyle(.plain)` so header sections expand full width.
