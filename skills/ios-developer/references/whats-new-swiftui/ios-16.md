# SwiftUI in iOS 16

## Overview

iOS 16 delivered a complete navigation system overhaul, replacing NavigationView with NavigationStack and NavigationSplitView. This release also introduced powerful new layout capabilities, enhanced photo/media selection, and bottom sheet presentations.

## Navigation (Major Overhaul)

### NavigationStack
Replaces NavigationView for stack-based navigation with value-driven, programmatic control.

- **NavigationStack** - Container for push/pop navigation with path state management
- **navigationDestination(for:)** - Type-safe destination mapping for navigation values
- **NavigationPath** - Type-erased collection for heterogeneous navigation state
- **Codable path support** - Save and restore navigation state across app launches

### NavigationSplitView
Multi-column navigation for iPad and Mac with adaptive behavior.

- **NavigationSplitView** - Two or three column layouts with sidebar, content, and detail
- **navigationSplitViewColumnWidth()** - Custom column width constraints
- **navigationSplitViewStyle()** - Control balanced, prominent, or automatic column sizing
- **Programmatic sidebar visibility** - Show/hide sidebar columns with bindings

### Navigation Customization
- **navigationTitle()** with binding - User-editable navigation titles
- **toolbarBackground()** - Custom toolbar background colors and materials
- **toolbar(.hidden)** - Hide navigation bar, tab bar, or other toolbars

## New Views

### Layout
- **Grid** - Fixed grid layout with explicit row/column positioning (unlike LazyVGrid).
- **GridRow** - Defines a row within a Grid with cell spanning support.
- **ViewThatFits** - Automatically selects the first child that fits available space.

### Tables
- **Table** - Multi-column data display for macOS and iPadOS with sorting support.
- **TableColumn** - Defines sortable, resizable columns with key path bindings.

### Media Selection
- **PhotosPicker** - System photo library picker supporting images and videos.
- **Transferable** protocol - Type-safe data transfer for drag/drop and copy/paste.

### Sharing
- **ShareLink** - Presents system share sheet for Transferable content.
- **PasteButton** - Standardized paste action with type-safe content handling.

### Date Selection
- **MultiDatePicker** - Calendar-based selection of multiple dates.

## New Modifiers

### Sheets and Presentations
- **presentationDetents()** - Bottom sheet with customizable height stops (medium, large, fraction, height).
- **presentationDragIndicator()** - Show/hide the sheet drag handle.
- **presentationBackgroundInteraction()** - Control interaction with content behind sheets.

### Interaction
- **onTapGesture(coordinateSpace:)** - Capture tap location within a view.
- **dropDestination()** - Enhanced drag and drop with transfer types.
- **scrollDismissesKeyboard()** - Automatically dismiss keyboard on scroll.

### Search
- **searchScopes()** - Filter search by categories or domains.
- **searchSuggestions()** - Provide search completion suggestions.

### Scroll and List
- **scrollIndicators()** - Control scroll indicator visibility.
- **scrollContentBackground()** - Customize or hide list/scroll view backgrounds.
- **listRowSeparator(.hidden, edges:)** - Fine-grained separator control.

### Find and Replace
- **findNavigator()** - System find and replace interface for text content.

### System UI
- **persistentSystemOverlays()** - Control home indicator visibility.
- **defersSystemGestures()** - Prevent system edge gestures from interfering.

### SF Symbols
- **symbolVariant()** - Apply fill, circle, square variants.
- **symbolEffect()** with variable color - Animate symbol layers.

## Other Improvements

### Custom Layouts
- **Layout** protocol - Build entirely custom layout containers with full control over sizing and positioning.
- **AnyLayout** - Type-erase layouts for conditional layout switching (VStack/HStack swap).

### Rendering
- **ImageRenderer** - Convert SwiftUI views to UIImage or CGImage.
- **render()** with GraphicsContext - Export views to PDF.

### Visual Effects
- **shadow(.inner)** - Inner shadow support for shapes and views.

### Alerts
- **TextField in alerts** - Add text input fields directly in alert dialogs.

### App Store
- **RequestReviewAction** - Environment-based app review request.

## Migration Notes

**NavigationView is deprecated.** Migrate to:
- **NavigationStack** - For iPhone-style push navigation
- **NavigationSplitView** - For iPad/Mac sidebar navigation

Key migration patterns:
- Replace `NavigationLink(destination:)` with `NavigationLink(value:)` + `navigationDestination(for:)`
- Use `NavigationPath` for programmatic navigation instead of isActive bindings
- NavigationSplitView handles adaptive behavior automatically - no need for manual iPhone/iPad switching
