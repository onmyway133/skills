# Code Examples

## Overview

Code is the centerpiece of technical articles. How you introduce and explain it determines comprehension.

## The Intent-Code-Explain Pattern

### 1. Intent Statement (Before Code)

Explain *why* before showing *how*:

```markdown
"Let's update our Row component to retrieve configuration from
the environment, which will let us customize styling without
passing parameters through every view."
```

**Not:**
```markdown
"Here's the code:"
```

### 2. The Code Block

Present clean, focused code:

```swift
struct Row<Content: View>: View {
    @Environment(\.designSystem.row) var config
    let content: () -> Content

    var body: some View {
        HStack(spacing: config.spacing) {
            content()
        }
        .padding(config.padding)
    }
}
```

### 3. Explanation (After Code)

Break down key points systematically:

```markdown
"That does a few important things:

1. We read the row configuration from the environment rather
   than accepting it as a parameter
2. The spacing comes from our design system, ensuring consistency
3. Padding is also centralized, so changes propagate everywhere"
```

## Inline Comments

Use comments for non-obvious decisions:

```swift
struct BackgroundImporter {
    func importData(_ items: [Item]) async {
        let context = ModelContext(container)
        // Disable autosave to batch all inserts into one transaction
        context.autosaveEnabled = false

        for item in items {
            context.insert(item)
        }

        // Single save at the end for performance
        try? context.save()
    }
}
```

## Code Progression

Show evolution, not just final state:

```markdown
## Version 1: Basic Implementation

[Simple code]

## Version 2: Adding Flexibility

"While our basic version works, we need to handle cases where..."

[Enhanced code]

## Version 3: Production Ready

"For real-world use, we should also consider..."

[Final code]
```

## Callouts and Warnings

Highlight critical information:

```markdown
> **Important:** Disabling autosave is crucial here. Without it,
> SwiftData will save after each insert, making bulk operations
> dramatically slower.

> **Tip:** For datasets over 5,000 objects, consider using batch
> insert requests instead.
```

## Code Block Guidelines

| Aspect | Guideline |
|--------|-----------|
| Length | 10-30 lines ideal; split larger blocks |
| Context | Include enough to be runnable |
| Focus | Remove unrelated code |
| Naming | Use clear, descriptive names |
| Comments | Explain *why*, not *what* |

## Anti-Patterns

```markdown
# Avoid unexplained code dumps
"Here's the implementation:"
[50 lines of code]
"Moving on..."

# Avoid over-commenting obvious code
let name = "John" // Set name to John

# Avoid incomplete snippets without context
.padding() // Won't work without struct context
```
