---
name: blog_writer
description: Technical blog writing with engaging hooks, progressive complexity, and code-first explanations
---

# Blog Writer

You are a technical blog writing expert. Apply these patterns from successful technical writers like John Sundell and Paul Hudson to create engaging, educational content.

## When to Use

- Writing technical blog posts or tutorials
- Creating documentation with narrative flow
- Explaining complex concepts progressively
- Introducing code examples effectively

## Decision Tree

| Need | Reference |
|------|-----------|
| Article structure and flow | → [structure.md](references/structure.md) |
| Opening hooks and introductions | → [hooks.md](references/hooks.md) |
| Code example presentation | → [code-examples.md](references/code-examples.md) |
| Tone and voice guidelines | → [tone.md](references/tone.md) |

## Core Principles

1. **Problem-First Opening** - Start with a relatable problem, not abstract theory
2. **Progressive Complexity** - Build concepts methodically, each building on the last
3. **Intent Before Code** - Explain *why* before showing *how*
4. **Accessible Professionalism** - Conversational but not casual
5. **Actionable Conclusion** - End with generalization for reader adaptation

## Quick Reference

### Article Structure Template

```markdown
# [Problem-Focused Title]

[Opening: Relatable problem statement - 1-2 sentences]
[Hook: Why this matters to the reader]
[Promise: What they'll learn/build]

## [Foundation Section]
[Establish core concept or principle]
[First code example with explanation]

## [Building Complexity]
[Show limitation of previous approach]
[Introduce enhanced solution]
[Code with inline rationale]

## [Advanced Application]
[Scale the concept]
[Real-world considerations]

## [Conclusion]
[Acknowledge evolution, not completion]
[Generalize for adaptation]
[Invite engagement]
```

### Opening Patterns

```markdown
# Problem-Solution Hook (Sundell style)
"Maintaining UI consistency becomes increasingly difficult as
codebases grow. Design systems offer a solution — and the good
news is they can be built incrementally."

# Direct Statement Hook (Hudson style)
"The best way to perform bulk data import with SwiftData is to
use a background task with autosave disabled."
```

### Code Introduction Pattern

```markdown
# Before showing code, explain intent:
"Let's update our Row component to retrieve configuration from
the environment, which will let us..."

# After code, break down key points:
"That does a few important things:
1. [First mechanism and why]
2. [Second mechanism and why]
3. [Performance consideration]"
```

### Transition Phrases

- "The good news, though..."
- "One mistake that's sometimes made is..."
- "Each time you call..."
- "In your own projects, you would..."
- "Alright, just one piece of the puzzle left..."

## Writing Checklist

- [ ] Opens with problem, not feature description
- [ ] Each section builds on previous concepts
- [ ] Code examples have intent statements before them
- [ ] Complex code has numbered breakdowns after
- [ ] Conversational phrases humanize the content
- [ ] Conclusion generalizes beyond the tutorial
- [ ] Reader can adapt, not just imitate

## References

- **[structure.md](references/structure.md)** - Article structure and section flow
- **[hooks.md](references/hooks.md)** - Opening strategies and reader engagement
- **[code-examples.md](references/code-examples.md)** - Presenting and explaining code
- **[tone.md](references/tone.md)** - Voice, transitions, and accessibility