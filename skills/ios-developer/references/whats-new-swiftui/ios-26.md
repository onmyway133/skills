# SwiftUI in iOS 26

## Overview

iOS 26 introduces the Liquid Glass design language, native web content embedding, enhanced rich text editing, and new animation macros. This release marks a significant visual refresh with the glass-based UI paradigm and continues improving developer productivity.

## New Views

### Web Content
- **WebView** - Native SwiftUI view for displaying web content powered by WebKit.
- Supports loading from URLs, HTML strings, or raw data.
- **WebPage** - Observable model type for customizing and interacting with web content.
- No more UIViewRepresentable wrappers needed for basic web display.

```swift
struct ContentView: View {
    var body: some View {
        WebView(url: URL(string: "https://apple.com")!)
    }
}

// With WebPage for interaction
@State private var page = WebPage()

WebView(page: page)
    .onAppear {
        page.load(URLRequest(url: url))
    }
```

### 3D Charts
- **Chart3D** - Display data plots in three dimensions for enhanced data visualization.

## New Modifiers

### List Navigation
- **listSectionIndexLabels()** - Adds alphabetical section index for quick navigation in long lists (like Contacts app).

### Labels
- **labelIconWidth(.fixed)** - Ensures consistent icon widths across labels for aligned visual appearance.

### TabView Enhancements
- **tabViewMinimizeBehavior()** - TabView minimizes when scrolling content, expanding available screen space.
- **tabViewAccessory()** - Add supplementary content alongside the tab bar.

### Navigation
- **navigationSubtitle()** - Display secondary text below the navigation title.

### Scroll Effects
- **scrollEdgeEffect()** - Customize bounce and edge effects for ScrollView and List. Control overscroll behavior and visual feedback.

### Scene Padding
- **scenePadding()** - Automatically applies appropriate padding based on the current scene context (window, sheet, etc.).

## Macros

### @Animatable
Simplifies creating animatable views and modifiers by generating AnimatableData conformance automatically.

```swift
@Animatable
struct PulseModifier: ViewModifier {
    var scale: Double  // Automatically animatable

    func body(content: Content) -> some View {
        content.scaleEffect(scale)
    }
}
```

The macro eliminates manual AnimatableData implementation for common animation patterns.

## Rich Text Editing

### TextView with AttributedString
- **TextView** - Native rich text editing view supporting AttributedString.
- Full formatting support: bold, italic, fonts, colors, links.
- Two-way binding to AttributedString for live updates.
- Replaces UITextView wrapping for rich text scenarios.

## Web Links

### Enhanced openURL
- **openURL** environment action now supports in-app browser presentation.
- Configure with `.inAppBrowser` option to keep users in your app.
- Maintains Safari-like experience without context switching.

## SF Symbols

### Self-Drawing Symbols
- **symbolEffect(.drawingGroup)** - SF Symbols animate their own drawing path.
- Creates elegant appear/disappear animations following symbol stroke order.

## List Improvements

### Section Spacing
- **listSectionSpacing()** - Custom spacing between list sections.
- Fine-grained control over visual density in lists.

### Toolbar
- **ToolbarSpacer** - Add flexible space in toolbars for custom layouts.

## Liquid Glass Design

iOS 26 introduces Liquid Glass, Apple's most significant design evolution since iOS 7. It's a dynamic material combining optical properties of glass with fluidity.

### Characteristics
- **Refraction**: Content beneath glass surfaces is visibly refracted
- **Reflection**: Surfaces reflect light from surrounding areas
- **Lensing**: Edge effects create depth and motion perception
- **Adaptive**: Surfaces respond to content behind them in real-time

### Key APIs

#### GlassEffectContainer
Container view that combines multiple glass shapes with morphing capabilities:

```swift
GlassEffectContainer {
    VStack {
        Button("Action 1") { }
            .glassEffect()
        Button("Action 2") { }
            .glassEffect()
    }
}
// Individual glass shapes can morph into one another
```

#### .glassEffect() Modifier
Apply reflective, depth-aware glass surfaces:

```swift
Text("Hello")
    .padding()
    .glassEffect()
```

#### Toolbar Integration
Toolbar items automatically float on Liquid Glass surfaces that adapt to content beneath.

#### Sheet Presentation
Partial-height sheets use inset Liquid Glass backgrounds. At smaller heights, bottom edges curve to nest in display corners. Full-height transitions gradually become opaque.

### Adoption
- Apps rebuilt with Xcode 26 automatically get Liquid Glass styling
- No redesign required for basic adoption
- One-year grace period to disable Liquid Glass if needed
- Tinting available for prominent toolbar items using bordered button styles

## Migration Notes

**Rich Text**: If you were wrapping UITextView for rich text editing, consider migrating to the native TextView with AttributedString support.

**Web Content**: For simple web display needs, WebView eliminates the need for UIViewRepresentable wrappers around WKWebView.

**Animations**: The @Animatable macro significantly reduces boilerplate for custom animations. Migrate manual AnimatableData implementations where possible.

**Design Updates**: Test your app with the new Liquid Glass appearance. System controls automatically adapt, but custom UI may need adjustments to complement the new aesthetic.
