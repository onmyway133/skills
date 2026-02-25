# Performance

## Overview

React and Next.js performance optimization patterns based on Vercel's react-best-practices skill with 40+ rules across 8 categories.

## Priority Categories

1. **Waterfalls** - Highest impact
2. **Bundle Size** - High impact
3. **Server-Side** - Medium-high impact
4. **Client-Side** - Medium impact
5. **Re-renders** - Medium impact
6. **Rendering** - Low-medium impact
7. **Micro-optimizations** - Low impact

## Avoiding Waterfalls

### Parallel Data Fetching

```tsx
// BAD: Sequential - creates waterfall
async function Page() {
  const user = await fetchUser();
  const posts = await fetchPosts(user.id);
  const comments = await fetchComments(posts[0].id);
  return <Dashboard user={user} posts={posts} comments={comments} />;
}

// GOOD: Parallel where possible
async function Page() {
  const [user, posts] = await Promise.all([
    fetchUser(),
    fetchPosts()
  ]);
  return <Dashboard user={user} posts={posts} />;
}
```

### Streaming with Suspense

```tsx
async function Page() {
  return (
    <div>
      <Header />
      <Suspense fallback={<PostsSkeleton />}>
        <Posts />
      </Suspense>
      <Suspense fallback={<CommentsSkeleton />}>
        <Comments />
      </Suspense>
    </div>
  );
}
```

## Bundle Size

### Dynamic Imports

```tsx
// BAD: Imports entire library
import { Chart } from 'heavy-chart-library';

// GOOD: Dynamic import
const Chart = dynamic(() => import('heavy-chart-library').then(m => m.Chart), {
  loading: () => <ChartSkeleton />,
  ssr: false
});
```

### Tree Shaking

```tsx
// BAD: Imports entire lodash
import _ from 'lodash';
_.debounce(fn, 300);

// GOOD: Import specific function
import debounce from 'lodash/debounce';
debounce(fn, 300);
```

## Server Components

### Minimize Client JS

```tsx
// Default to Server Components
// app/page.tsx
async function Page() {
  const data = await fetchData(); // Runs on server
  return <DataTable data={data} />;
}

// Only add 'use client' when needed
'use client'
function InteractiveChart({ data }) {
  const [selected, setSelected] = useState(null);
  return <Chart data={data} onSelect={setSelected} />;
}
```

### Component Boundaries

```tsx
// BAD: Entire page is client
'use client'
function Page() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <Header /> {/* Could be server */}
      <Sidebar /> {/* Could be server */}
      <Counter count={count} setCount={setCount} />
    </div>
  );
}

// GOOD: Only interactive parts are client
function Page() {
  return (
    <div>
      <Header />
      <Sidebar />
      <Counter /> {/* 'use client' in Counter file */}
    </div>
  );
}
```

## Minimizing Re-renders

### Memoization

```tsx
// Memoize expensive computations
const sortedItems = useMemo(
  () => items.sort((a, b) => a.name.localeCompare(b.name)),
  [items]
);

// Memoize callbacks passed to children
const handleClick = useCallback((id: string) => {
  setSelected(id);
}, []);

// Memoize components
const MemoizedList = memo(function List({ items }) {
  return items.map(item => <Item key={item.id} {...item} />);
});
```

### State Colocation

```tsx
// BAD: State too high, causes sibling re-renders
function Parent() {
  const [search, setSearch] = useState('');
  return (
    <>
      <SearchInput value={search} onChange={setSearch} />
      <ExpensiveComponent /> {/* Re-renders on every keystroke */}
    </>
  );
}

// GOOD: State colocated where needed
function Parent() {
  return (
    <>
      <SearchSection /> {/* Contains its own state */}
      <ExpensiveComponent />
    </>
  );
}
```

## Image Optimization

```tsx
import Image from 'next/image';

// Automatic optimization
<Image
  src="/hero.jpg"
  alt="Hero image"
  width={1200}
  height={600}
  priority // Above the fold
  placeholder="blur"
  blurDataURL={blurHash}
/>
```

## External Reference

For comprehensive guidance, see [vercel-labs/agent-skills](https://github.com/vercel-labs/agent-skills).
