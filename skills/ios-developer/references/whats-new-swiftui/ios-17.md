# SwiftUI in iOS 17

## Overview

iOS 17 focused heavily on ScrollView enhancements, advanced animation capabilities, and improved developer ergonomics. This release introduced paging, scroll transitions, phase animators, and Metal shader integration for custom visual effects.

## New Views

### Content States
- **ContentUnavailableView** - Standardized empty state presentation with icon, title, description, and actions. System-provided variants for search results and network errors.

### In-App Purchases
- **StoreKit views** - ProductView and StoreView for displaying and purchasing products directly in SwiftUI.

## New Modifiers

### ScrollView Paging and Snapping
- **scrollTargetBehavior(.paging)** - Full-page snapping like UIPageViewController.
- **scrollTargetBehavior(.viewAligned)** - Snap to individual child view boundaries.
- **scrollTargetLayout()** - Mark container for scroll target alignment.
- **scrollPosition(id:)** - Bind to current scroll position by ID.

### ScrollView Appearance
- **scrollClipDisabled()** - Allow content to overflow scroll bounds for visual effects.
- **defaultScrollAnchor(.bottom)** - Start scroll position at bottom (chat interfaces).
- **scrollIndicatorsFlash()** - Trigger scroll indicator flash programmatically.
- **contentMargins()** - Inset content or scroll indicators independently.

### Scroll Transitions
- **scrollTransition()** - Apply custom transitions to items as they scroll in/out of view.

### Vertical Paging
- **containerRelativeFrame()** - Size views relative to scroll container for vertical paging layouts.

### Animation

#### Phase Animators
- **phaseAnimator()** - Multi-step animations that cycle through defined phases automatically.
- Custom phase sequences with per-phase animation curves.

#### Animation Callbacks
- **withAnimation() { } completion: { }** - Execute code when animation completes.

#### SF Symbol Animation
- **symbolEffect()** - Built-in animations: bounce, pulse, scale, variableColor, replace.
- **contentTransition(.symbolEffect)** - Animated symbol changes.

### Metal Shaders
- **layerEffect()** - Apply Metal shader functions to view layers.
- **colorEffect()** - Pixel-level color transformations via Metal.
- **distortionEffect()** - Geometry distortion using Metal shaders.

### Visual Adjustments
- **visualEffect()** - Modify view appearance based on geometry (size, position in scroll view).

### Shape Composition
- **union()**, **intersection()**, **subtracting()**, **symmetricDifference()** - Combine shapes using boolean operations.

### Container Relative Sizing
- **containerRelativeFrame()** - Size views as fractions of their container, replacing many GeometryReader uses.

### Inspector
- **inspector()** - Side panel presentation for supplementary content, adapts to sheet on compact sizes.

### Input Events
- **onKeyPress()** - Respond to hardware keyboard input with key and modifier detection.

### Navigation
- **NavigationSplitView** preferred column - Control which column appears in compact width.

### Haptics
- **sensoryFeedback()** - Trigger haptic feedback tied to value changes or events.

### Safe Area
- **safeAreaPadding()** - Add padding that respects safe area insets.

## Other Improvements

### Spring Animations
- **spring()** - Simplified spring animation with duration and bounce parameters.
- **spring(duration:bounce:)** - More intuitive than mass/stiffness/damping.

### onChange Improvements
- **onChange(of:initial:)** - Optionally trigger on initial value.
- **onChange { }** - Zero-parameter variant when you only need the trigger.

### Shape Fill and Stroke
- Shapes now support simultaneous fill and stroke in a single modifier call.

### Color
- **resolve(in:)** - Extract RGB components from Color in a specific environment.

### Buttons
- **buttonRepeatBehavior(.enabled)** - Buttons fire repeatedly while held.

## Migration Notes

iOS 17 significantly reduces the need for GeometryReader:
- Use `containerRelativeFrame()` for proportional sizing
- Use `visualEffect()` for geometry-based visual changes
- Use `scrollPosition(id:)` instead of preference keys for scroll tracking

The new spring animation API is preferred over the mass/stiffness/damping variant for most use cases.
