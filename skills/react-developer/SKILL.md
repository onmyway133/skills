---
name: "react-developer"
description: "Comprehensive React development with performance, design, composition, and React Native patterns"
---

# React Developer

You are a React development expert. This skill aggregates best practices from Vercel's agent skills for comprehensive React and Next.js development.

## When to Use

- Building React/Next.js applications
- Optimizing React performance
- Implementing accessible UI patterns
- Working with React Native mobile apps
- Designing scalable component architectures

## Decision Tree

| Need | Reference |
|------|-----------|
| Performance optimization | → [performance.md](references/performance.md) |
| UI/UX and accessibility | → [design-guidelines.md](references/design-guidelines.md) |
| Component architecture | → [composition.md](references/composition.md) |
| React Native mobile | → [react-native.md](references/react-native.md) |

## Core Principles

1. **Performance First** - Avoid waterfalls, optimize bundles, minimize re-renders
2. **Accessible by Default** - Focus states, ARIA, keyboard navigation
3. **Composition Over Props** - Compound components, slots, render props
4. **Server Components** - Use RSC where possible, minimize client JS
5. **Platform Aware** - iOS/Android considerations for React Native

## Quick Reference

### Performance Patterns

```tsx
// Avoid data fetching waterfalls
// BAD: Sequential fetches
async function Page() {
  const user = await fetchUser();
  const posts = await fetchPosts(user.id); // Waits for user
  return <Feed user={user} posts={posts} />;
}

// GOOD: Parallel fetches
async function Page() {
  const userPromise = fetchUser();
  const postsPromise = fetchPosts();
  const [user, posts] = await Promise.all([userPromise, postsPromise]);
  return <Feed user={user} posts={posts} />;
}
```

### Component Composition

```tsx
// Avoid boolean prop explosion
// BAD
<Button primary large disabled withIcon iconPosition="left" />

// GOOD: Compound components
<Button variant="primary" size="lg" disabled>
  <Button.Icon position="left">
    <StarIcon />
  </Button.Icon>
  <Button.Label>Featured</Button.Label>
</Button>
```

### Accessibility Essentials

```tsx
// Focus management
<button
  className="focus:ring-2 focus:ring-offset-2 focus:ring-blue-500"
  aria-label="Close dialog"
>
  <XIcon aria-hidden="true" />
</button>

// Form accessibility
<label htmlFor="email">Email</label>
<input
  id="email"
  type="email"
  aria-describedby="email-hint"
  aria-invalid={!!errors.email}
/>
<span id="email-hint">We'll never share your email</span>
```

### React Native Patterns

```tsx
// Use FlashList for performance
import { FlashList } from "@shopify/flash-list";

<FlashList
  data={items}
  renderItem={({ item }) => <ItemCard item={item} />}
  estimatedItemSize={100}
/>

// Platform-specific code
import { Platform } from 'react-native';

const styles = {
  shadow: Platform.select({
    ios: { shadowOpacity: 0.2 },
    android: { elevation: 4 },
  }),
};
```

## References

- **[performance.md](references/performance.md)** - React and Next.js performance optimization
- **[design-guidelines.md](references/design-guidelines.md)** - UI patterns and accessibility
- **[composition.md](references/composition.md)** - Component architecture and scaling
- **[react-native.md](references/react-native.md)** - Mobile development patterns

## External Resources

For comprehensive React development guidance, see:

- [vercel-labs/agent-skills](https://github.com/vercel-labs/agent-skills) - Performance, design, composition, and React Native patterns
