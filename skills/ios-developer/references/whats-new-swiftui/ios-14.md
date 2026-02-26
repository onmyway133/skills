# SwiftUI in iOS 14

## Overview

iOS 14 marked a significant expansion of SwiftUI's capabilities, introducing lazy loading for performance optimization, new input controls, and crucial property wrappers for app lifecycle management. This release made SwiftUI viable for more complex, production-ready applications.

## New Views

### Text and Input
- **TextEditor** - Multi-line text editing view that replaces the need for UITextView wrapping. Supports text binding and styling.

### Layout and Performance
- **LazyVGrid / LazyHGrid** - Grid layouts that only instantiate visible cells, enabling efficient display of large collections in grid format.
- **LazyVStack / LazyHStack** - Stack containers that defer view creation until needed, essential for scrollable content with many items.

### Media and Visualization
- **ColorPicker** - Native color selection interface, providing both compact and expanded presentation modes.
- **Map** - MapKit integration directly in SwiftUI for displaying geographic content.
- **ProgressView** - Unified progress indicator supporting both determinate (with value) and indeterminate (spinning) states.
- **VideoPlayer** - AVKit video playback with built-in controls and transport UI.
- **SpriteView** - Direct embedding of SpriteKit scenes within SwiftUI view hierarchies.

### Disclosure and Labeling
- **DisclosureGroup** - Expandable/collapsible content container with built-in disclosure indicator.
- **Label** - Standardized view combining an icon and text, respecting system label styles.
- **Link** - Opens URLs in Safari or the appropriate app handler.

## New Modifiers

### Navigation and Presentation
- **toolbar()** - Unified API for adding toolbar items across navigation bars and bottom bars.
- **fullScreenCover()** - Full-screen modal presentation alternative to sheet().
- **appStoreOverlay()** - Displays App Store product recommendations within your app.

### Scrolling and Paging
- **tabViewStyle(.page)** - Converts TabView into a horizontal paging scroll view.

### Animation
- **matchedGeometryEffect()** - Enables smooth animated transitions between views by synchronizing their geometry.

### State and Events
- **onChange(of:)** - Executes code when a specified value changes, essential for side effects.
- **onContinueUserActivity()** - Handles Handoff and Spotlight continuation.

### Files and Data
- **fileExporter()** - Presents a document export interface for saving files.
- **textCase()** - Controls text capitalization in TextField and other text inputs.

### List Styling
- **listStyle(.insetGrouped)** - New list appearance matching iOS Settings app style.

## Property Wrappers

- **@AppStorage** - Direct read/write access to UserDefaults with automatic view updates.
- **@ScaledMetric** - Scales numeric values according to the user's Dynamic Type setting.
- **@StateObject** - Creates and owns an ObservableObject instance with view lifetime.
- **@UIApplicationDelegateAdaptor** - Bridges UIKit app delegate methods into SwiftUI lifecycle.

## Other Improvements

### Document-Based Apps
- **FileDocument** protocol and **DocumentGroup** scene type enable building document-centric applications entirely in SwiftUI.

### Scroll Control
- **ScrollViewReader** - Programmatic scrolling to specific views using proxy-based scroll control.

### Lists
- **Expanding lists** - Hierarchical data display with built-in expand/collapse for nested content.
- **ContainerRelativeShape** - Shape that adapts to its container's corner radius, useful for widget backgrounds.

### App Lifecycle
- **@main** and **App** protocol - Pure SwiftUI app entry point without UIKit scaffolding.
- **WindowGroup** - Multi-window scene management for iPadOS and macOS.

### Text Formatting
- Date formatting directly in Text views using the `format:` parameter.

## Migration Notes

iOS 14 introduced the SwiftUI app lifecycle as an alternative to UIKit's UIApplicationDelegate. New projects should prefer the SwiftUI lifecycle for simpler architecture, but @UIApplicationDelegateAdaptor provides an escape hatch for UIKit integration when needed.
