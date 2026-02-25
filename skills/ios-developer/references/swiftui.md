# SwiftUI

## Overview

SwiftUI best practices for building, reviewing, and improving SwiftUI code with modern patterns.

## Core Principles

1. **Use `@Observable`** - Prefer `@Observable` over `ObservableObject` for new code
2. **Keep `@State` private** - State properties should be private to the view
3. **Use `@Binding` sparingly** - Only when children need to modify parent state
4. **Leverage `@Bindable`** - For injected observable objects requiring bindings

## State Management

```swift
// Modern state management with @Observable
@Observable
@MainActor
final class ViewModel {
    var items: [Item] = []
    var isLoading = false

    func load() async {
        isLoading = true
        items = await fetchItems()
        isLoading = false
    }
}

struct ContentView: View {
    @State private var viewModel = ViewModel()

    var body: some View {
        List(viewModel.items) { item in
            ItemRow(item: item)
        }
        .overlay {
            if viewModel.isLoading {
                ProgressView()
            }
        }
        .task {
            await viewModel.load()
        }
    }
}
```

## Modern API Replacements

| Deprecated | Modern |
|------------|--------|
| `foregroundColor()` | `foregroundStyle()` |
| `NavigationView` | `NavigationStack` |
| `GeometryReader` (for sizing) | `containerRelativeFrame()` |
| `.onChange(of:perform:)` | `.onChange(of:) { old, new in }` |

## View Composition

```swift
// Keep views small and focused
struct UserCard: View {
    let user: User

    var body: some View {
        VStack(alignment: .leading, spacing: 8) {
            AvatarView(url: user.avatarURL)
            UserInfo(user: user)
        }
        .padding()
        .background(.regularMaterial)
        .clipShape(RoundedRectangle(cornerRadius: 12))
    }
}

// Extract subviews for reusability
struct AvatarView: View {
    let url: URL?

    var body: some View {
        AsyncImage(url: url) { image in
            image.resizable().scaledToFill()
        } placeholder: {
            Color.gray
        }
        .frame(width: 60, height: 60)
        .clipShape(Circle())
    }
}
```

## Performance

- **Stable identity** - Use proper `id` in ForEach loops
- **Pass only needed values** - Don't pass entire models when few properties needed
- **Avoid state in hot paths** - Minimize state updates during scrolling
- **Use lazy containers** - `LazyVStack`, `LazyHStack` for large lists

## Animation

```swift
// Explicit value-based animation
.animation(.spring, value: isExpanded)

// Event-driven animation
withAnimation(.easeInOut) {
    isExpanded.toggle()
}

// Prefer transforms over layout changes
.scaleEffect(isPressed ? 0.95 : 1.0)
```

## External References

For comprehensive SwiftUI guidance, see:

- **[SwiftUI-Agent-Skill](https://github.com/AvdLee/SwiftUI-Agent-Skill)** - Expert SwiftUI best practices
- **[Dimillian/Skills](https://github.com/Dimillian/Skills)** - SwiftUI patterns:
  - `swiftui-ui-patterns` - View composition and state
  - `swiftui-view-refactor` - Code consistency
  - `swiftui-performance-audit` - Performance optimization
  - `swiftui-liquid-glass` - iOS 26+ Liquid Glass API
