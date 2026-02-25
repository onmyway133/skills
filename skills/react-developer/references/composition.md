# Composition Patterns

## Overview

Component architecture patterns for scaling React applications, addressing boolean prop proliferation through compound components, state lifting, and internal composition.

## The Problem

```tsx
// Prop explosion - hard to maintain
<Button
  primary
  large
  disabled
  withIcon
  iconPosition="left"
  loading
  fullWidth
  rounded="lg"
/>

// Which props conflict? What combinations are valid?
```

## Compound Components

### Basic Pattern

```tsx
// Define the compound component
function Select({ children, value, onChange }) {
  return (
    <SelectContext.Provider value={{ value, onChange }}>
      <div role="listbox">{children}</div>
    </SelectContext.Provider>
  );
}

Select.Option = function Option({ value, children }) {
  const ctx = useContext(SelectContext);
  const isSelected = ctx.value === value;

  return (
    <div
      role="option"
      aria-selected={isSelected}
      onClick={() => ctx.onChange(value)}
    >
      {children}
    </div>
  );
};

// Usage - clear structure
<Select value={selected} onChange={setSelected}>
  <Select.Option value="react">React</Select.Option>
  <Select.Option value="vue">Vue</Select.Option>
  <Select.Option value="angular">Angular</Select.Option>
</Select>
```

### With Slots

```tsx
function Card({ children }) {
  const header = Children.toArray(children).find(
    child => child.type === Card.Header
  );
  const body = Children.toArray(children).find(
    child => child.type === Card.Body
  );
  const footer = Children.toArray(children).find(
    child => child.type === Card.Footer
  );

  return (
    <div className="card">
      {header && <div className="card-header">{header}</div>}
      <div className="card-body">{body}</div>
      {footer && <div className="card-footer">{footer}</div>}
    </div>
  );
}

Card.Header = ({ children }) => children;
Card.Body = ({ children }) => children;
Card.Footer = ({ children }) => children;

// Usage
<Card>
  <Card.Header>
    <h2>Title</h2>
  </Card.Header>
  <Card.Body>
    <p>Content here</p>
  </Card.Body>
  <Card.Footer>
    <Button>Action</Button>
  </Card.Footer>
</Card>
```

## Render Props

```tsx
// Expose internal state without prop drilling
function Toggle({ children }) {
  const [on, setOn] = useState(false);
  const toggle = () => setOn(prev => !prev);

  return children({ on, toggle });
}

// Usage
<Toggle>
  {({ on, toggle }) => (
    <div>
      <button onClick={toggle}>
        {on ? 'Hide' : 'Show'}
      </button>
      {on && <Content />}
    </div>
  )}
</Toggle>
```

## State Lifting

### Extract Shared State

```tsx
// BAD: Duplicated state
function FilterPanel() {
  const [filters, setFilters] = useState({});
  return <FilterForm filters={filters} onChange={setFilters} />;
}

function ResultsList() {
  const [filters, setFilters] = useState({}); // Duplicate!
  const results = useFilteredResults(filters);
  return <List items={results} />;
}

// GOOD: Lifted state
function SearchPage() {
  const [filters, setFilters] = useState({});

  return (
    <>
      <FilterPanel filters={filters} onChange={setFilters} />
      <ResultsList filters={filters} />
    </>
  );
}
```

### Custom Hooks for Reuse

```tsx
function useFilters(initialFilters = {}) {
  const [filters, setFilters] = useState(initialFilters);

  const updateFilter = useCallback((key, value) => {
    setFilters(prev => ({ ...prev, [key]: value }));
  }, []);

  const clearFilters = useCallback(() => {
    setFilters(initialFilters);
  }, [initialFilters]);

  return { filters, updateFilter, clearFilters };
}

// Reusable across components
function ProductsPage() {
  const { filters, updateFilter, clearFilters } = useFilters({
    category: 'all',
    priceRange: [0, 100]
  });

  // ...
}
```

## Polymorphic Components

```tsx
type ButtonProps<T extends ElementType> = {
  as?: T;
  children: ReactNode;
} & ComponentPropsWithoutRef<T>;

function Button<T extends ElementType = 'button'>({
  as,
  children,
  ...props
}: ButtonProps<T>) {
  const Component = as || 'button';
  return <Component {...props}>{children}</Component>;
}

// Renders as <button>
<Button onClick={handleClick}>Click me</Button>

// Renders as <a>
<Button as="a" href="/about">About</Button>

// Renders as Link (Next.js)
<Button as={Link} href="/dashboard">Dashboard</Button>
```

## Headless Components

```tsx
// Logic only, no UI
function useCombobox({ items, onSelect }) {
  const [isOpen, setIsOpen] = useState(false);
  const [query, setQuery] = useState('');
  const [activeIndex, setActiveIndex] = useState(0);

  const filtered = items.filter(item =>
    item.label.toLowerCase().includes(query.toLowerCase())
  );

  const getInputProps = () => ({
    value: query,
    onChange: (e) => setQuery(e.target.value),
    onFocus: () => setIsOpen(true),
  });

  const getMenuProps = () => ({
    role: 'listbox',
  });

  const getItemProps = (index) => ({
    role: 'option',
    'aria-selected': index === activeIndex,
    onClick: () => onSelect(filtered[index]),
  });

  return { isOpen, filtered, getInputProps, getMenuProps, getItemProps };
}

// Consumer provides all UI
function MyCombobox({ items, onSelect }) {
  const { isOpen, filtered, getInputProps, getMenuProps, getItemProps } =
    useCombobox({ items, onSelect });

  return (
    <div>
      <input {...getInputProps()} className="my-input-styles" />
      {isOpen && (
        <ul {...getMenuProps()} className="my-menu-styles">
          {filtered.map((item, i) => (
            <li key={item.id} {...getItemProps(i)} className="my-item-styles">
              {item.label}
            </li>
          ))}
        </ul>
      )}
    </div>
  );
}
```

## External Reference

For comprehensive guidance, see [vercel-labs/agent-skills](https://github.com/vercel-labs/agent-skills).
