# Opening Hooks

## Overview

The opening determines whether readers continue or bounce. Technical articles need hooks that establish relevance immediately.

## Hook Patterns

### Problem-Solution Hook

Start with a pain point, then promise relief:

```markdown
"Maintaining UI consistency becomes increasingly difficult as
codebases grow. The good news, though, is that design systems
can very often be built incrementally."
```

**Structure:**
1. State the problem (1 sentence)
2. Acknowledge difficulty (optional)
3. Promise the solution

### Direct Statement Hook

Lead with the answer for readers seeking quick solutions:

```markdown
"The best way to perform bulk data import with SwiftData is to
use a background task with autosave disabled."
```

**Best for:** Tutorial/how-to content where readers arrive via search.

### Question Hook

Pose a question the reader is likely asking:

```markdown
"Have you ever wondered why your SwiftUI app feels sluggish
when loading large datasets?"
```

**Use sparingly** - can feel manipulative if overused.

### Scenario Hook

Paint a specific situation:

```markdown
"You're reviewing a pull request and notice the same button
styling implemented three different ways. Sound familiar?"
```

## Hook Components

### The Promise

After the hook, clarify what readers will gain:

```markdown
"In this article, we'll build a compositional design system
that scales from a few components to an entire app."
```

### Skepticism Address

Preemptively handle objections:

```markdown
"Design systems sound complex, but they can be built
incrementally alongside your existing code."
```

## Anti-Patterns

```markdown
# Avoid generic openings
"SwiftUI is Apple's declarative UI framework..."

# Avoid autobiography
"I've been working with Swift for 10 years and..."

# Avoid throat-clearing
"Before we begin, let me explain why this topic is important..."
```

## Opening Checklist

- [ ] First sentence establishes problem or answer
- [ ] Reader relevance is clear within 2 sentences
- [ ] Promise of value is explicit
- [ ] No unnecessary preamble
- [ ] Tone matches article depth (casual for quick tips, measured for deep dives)
