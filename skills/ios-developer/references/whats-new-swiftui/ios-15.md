# SwiftUI in iOS 15

## Overview

iOS 15 brought significant async/await integration, simplified common UI patterns like alerts and action sheets, and introduced powerful focus management. This release emphasized developer productivity with cleaner APIs and better Markdown support.

## New Views

### Async Content
- **AsyncImage** - Loads and displays remote images from URLs with built-in placeholder and error handling support.

### Custom Drawing
- **TimelineView** - Schedules view updates on a timeline, enabling frame-by-frame animations and clock displays.
- **Canvas** - Immediate-mode drawing context for high-performance custom graphics without view overhead.

### Grouped Controls
- **ControlGroup** - Groups related controls together, automatically adapting presentation for different contexts.

### Location
- **LocationButton** - Standardized button for requesting one-time location access with consistent system appearance.

## New Modifiers

### Alerts and Sheets
- **alert(_:isPresented:actions:message:)** - Simplified alert API using ViewBuilder for actions instead of Alert struct.
- **confirmationDialog()** - Replaces actionSheet() with a cleaner, more flexible API.
- **interactiveDismissDisabled()** - Prevents sheet dismissal via swipe gesture when needed.

### Async Operations
- **task()** - Attaches an async task to view lifecycle, automatically cancelling when the view disappears.
- **refreshable()** - Adds pull-to-refresh capability to scrollable content with async action support.

### Search
- **searchable()** - Adds a search field to navigation containers with automatic suggestions support.

### Lists and Rows
- **swipeActions()** - Adds custom swipe actions to List rows with leading/trailing edge support.
- **listRowSeparator()** - Controls visibility and tint of row separators.
- **badge()** - Displays badges on TabView items and List rows.

### Keyboard
- **toolbar()** with keyboard placement - Adds accessory views above the keyboard.
- **submitLabel()** - Customizes the return key appearance for text inputs.
- **onSubmit()** - Handles text field submission events.

### Visual Effects
- **material** backgrounds - Apply blur and vibrancy effects using system materials.
- **foregroundStyle()** - Applies hierarchical styling (primary, secondary, tertiary) to content.

### Text Selection
- **textSelection()** - Enables user text selection in Text views.

### Button Styling
- **buttonStyle(.bordered)** - New bordered button appearance for prominent actions.
- **tint()** - Controls accent color for buttons and interactive elements.

### SF Symbols
- **symbolRenderingMode()** - Controls monochrome, hierarchical, palette, or multicolor rendering.

### Privacy
- **privacySensitive()** - Marks content to be hidden in system screenshots and screen recordings.

## Property Wrappers

- **@FocusState** - Tracks and controls keyboard focus across multiple text fields.

## Other Improvements

### Bindings in Lists
- ForEach and List now accept bindings directly, enabling two-way data flow in list contexts.

### Markdown in Text
- Text views now render inline Markdown syntax including bold, italic, links, and code.

### Debugging
- **Self._printChanges()** - Debug helper that prints which properties caused a view update.

### Preview Improvements
- Portrait and landscape preview orientations directly in preview modifiers.

### Text Formatting
- AttributedString support enables rich text styling within Text views.

## Migration Notes

iOS 15 deprecated the Alert struct in favor of the new alert() modifier with ViewBuilder. The new API is more consistent with SwiftUI patterns and supports more customization. Similarly, actionSheet() is replaced by confirmationDialog() with improved semantics.

The task() modifier is preferred over onAppear() for async work since it provides automatic cancellation.
