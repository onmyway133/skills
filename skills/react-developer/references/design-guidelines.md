# Design Guidelines

## Overview

UI code auditing patterns with 100+ rules covering accessibility, performance, and UX from Vercel's web-design-guidelines skill.

## Accessibility

### Focus States

```tsx
// Always visible focus indicators
<button
  className={cn(
    "px-4 py-2 rounded",
    "focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2",
    "focus-visible:ring-2" // Only show on keyboard focus
  )}
>
  Click me
</button>

// Skip links for keyboard users
<a
  href="#main-content"
  className="sr-only focus:not-sr-only focus:absolute focus:top-4 focus:left-4"
>
  Skip to main content
</a>
```

### Form Accessibility

```tsx
function FormField({ label, error, hint, ...props }) {
  const id = useId();
  const hintId = `${id}-hint`;
  const errorId = `${id}-error`;

  return (
    <div>
      <label htmlFor={id}>{label}</label>
      <input
        id={id}
        aria-describedby={hint ? hintId : undefined}
        aria-invalid={!!error}
        aria-errormessage={error ? errorId : undefined}
        {...props}
      />
      {hint && <span id={hintId}>{hint}</span>}
      {error && <span id={errorId} role="alert">{error}</span>}
    </div>
  );
}
```

### Icon Buttons

```tsx
// Icons need accessible labels
<button aria-label="Close dialog">
  <XIcon aria-hidden="true" />
</button>

// Or use visually hidden text
<button>
  <XIcon aria-hidden="true" />
  <span className="sr-only">Close dialog</span>
</button>
```

## Keyboard Navigation

### Focus Trapping

```tsx
// Trap focus in modals
function Dialog({ isOpen, onClose, children }) {
  const dialogRef = useRef<HTMLDivElement>(null);

  useEffect(() => {
    if (isOpen) {
      const focusable = dialogRef.current?.querySelectorAll(
        'button, [href], input, select, textarea, [tabindex]:not([tabindex="-1"])'
      );
      focusable?.[0]?.focus();
    }
  }, [isOpen]);

  return (
    <div
      ref={dialogRef}
      role="dialog"
      aria-modal="true"
      onKeyDown={(e) => e.key === 'Escape' && onClose()}
    >
      {children}
    </div>
  );
}
```

### Arrow Key Navigation

```tsx
// Lists and menus
function Menu({ items }) {
  const [activeIndex, setActiveIndex] = useState(0);

  const handleKeyDown = (e: KeyboardEvent) => {
    switch (e.key) {
      case 'ArrowDown':
        setActiveIndex(i => Math.min(i + 1, items.length - 1));
        break;
      case 'ArrowUp':
        setActiveIndex(i => Math.max(i - 1, 0));
        break;
      case 'Home':
        setActiveIndex(0);
        break;
      case 'End':
        setActiveIndex(items.length - 1);
        break;
    }
  };

  return (
    <ul role="menu" onKeyDown={handleKeyDown}>
      {items.map((item, i) => (
        <li
          key={item.id}
          role="menuitem"
          tabIndex={i === activeIndex ? 0 : -1}
        >
          {item.label}
        </li>
      ))}
    </ul>
  );
}
```

## Animations

### Reduced Motion

```tsx
// Respect user preferences
const prefersReducedMotion = window.matchMedia(
  '(prefers-reduced-motion: reduce)'
).matches;

// Tailwind approach
<div className="transition-transform motion-reduce:transition-none">
  Content
</div>

// CSS approach
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

## Typography

### Readable Text

```tsx
// Good defaults
<p className={cn(
  "text-base leading-relaxed", // 1.625 line height
  "max-w-prose",                // ~65 characters
  "text-gray-900 dark:text-gray-100"
)}>
  Long form content here...
</p>

// Responsive type scale
<h1 className="text-2xl md:text-3xl lg:text-4xl font-bold">
  Heading
</h1>
```

## Dark Mode

```tsx
// System preference + manual toggle
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState<'light' | 'dark' | 'system'>('system');

  useEffect(() => {
    const root = document.documentElement;
    if (theme === 'system') {
      const systemTheme = window.matchMedia('(prefers-color-scheme: dark)').matches
        ? 'dark' : 'light';
      root.classList.toggle('dark', systemTheme === 'dark');
    } else {
      root.classList.toggle('dark', theme === 'dark');
    }
  }, [theme]);

  return <ThemeContext.Provider value={{ theme, setTheme }}>{children}</ThemeContext.Provider>;
}
```

## External Reference

For comprehensive guidance, see [vercel-labs/agent-skills](https://github.com/vercel-labs/agent-skills).
