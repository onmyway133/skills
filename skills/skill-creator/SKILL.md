---
name: "skill-creator"
description: "Creates new Claude Code skills with proper structure, references, and documentation"
---

# Skill Creator

You are a skill creation expert. Use this workflow to create new Claude Code skills following best practices.

## When to Use

- Creating a new skill for a framework, library, or pattern
- Converting documentation into a skill format
- Building comprehensive reference-based skills

## Skill Structure (AvdLee Pattern)

```
skills/<skill-name>/
├── SKILL.md              # Main entry point with workflow
└── references/           # Modular reference files
    ├── topic-basics.md
    ├── topic-advanced.md
    ├── patterns.md
    └── common-issues.md
```

## Creation Process

### Step 1: Define Scope

Before creating, clarify:
- **Name**: kebab-case folder name (e.g., `swift-data`)
- **Purpose**: What problem does this skill solve?
- **Audience**: Who uses it and at what skill level?
- **Sources**: Documentation, articles, or repos to learn from

### Step 2: Research Content

Gather comprehensive information:
1. Fetch official documentation
2. Read tutorials and articles
3. Identify key concepts and patterns
4. Note common issues and solutions

### Step 3: Plan Reference Files

Break content into focused modules:

| File Pattern | Content Type |
|--------------|--------------|
| `*-basics.md` | Fundamentals and getting started |
| `*-advanced.md` | Complex patterns and edge cases |
| `*-patterns.md` | Best practices and common patterns |
| `*-integration.md` | Working with other frameworks |
| `common-issues.md` | Troubleshooting and gotchas |

**Naming Convention**: lowercase kebab-case (e.g., `model-basics.md`)

### Step 4: Write Main SKILL.md

Structure:
```markdown
---
name: "Skill Name"
description: "One-line description for marketplace"
---

# Skill Name

Brief intro paragraph about what this skill covers.

## When to Use

Trigger scenarios for this skill.

## Decision Tree

Guide for which reference to consult:
- Need X? → See references/topic-a.md
- Need Y? → See references/topic-b.md

## Core Principles

3-5 key rules that apply everywhere.

## Quick Reference

Most commonly needed patterns with code.

## References

Link to each reference file with description.
```

### Step 5: Write Reference Files

Each reference file structure:
```markdown
# Topic Name

## Overview
Brief context for this topic.

## Patterns

### Pattern 1
Code example with explanation.

### Pattern 2
Code example with explanation.

## Best Practices
- Practice 1
- Practice 2

## Common Mistakes
- Mistake 1: Why it's wrong and how to fix

## Checklist
- [ ] Verification item 1
- [ ] Verification item 2
```

### Step 6: Update Marketplace

Add to `.claude-plugin/marketplace.json`:
```json
{
  "name": "skill-name",
  "description": "Description for marketplace",
  "source": "./skills/skill-name"
}
```

## Quality Checklist

Before completing:
- [ ] SKILL.md has YAML frontmatter (name, description)
- [ ] Folder name matches skill name in kebab-case
- [ ] References cover all major topics
- [ ] Code examples are correct and tested
- [ ] No opinions - only facts and patterns
- [ ] Accessibility from beginner to advanced
- [ ] Marketplace entry added

## Example: Creating a SwiftData Skill

```bash
# 1. Create structure
mkdir -p skills/swift-data/references

# 2. Create main SKILL.md
# 3. Create reference files:
#    - model-basics.md
#    - relationships.md
#    - queries-filtering.md
#    - containers-contexts.md
#    - cloudkit-sync.md
#    - migration.md
#    - performance.md
#    - common-issues.md

# 4. Update marketplace.json
```

## Guidelines

- **Focused scope**: Each skill covers one framework/topic
- **Fact-based**: Document patterns, not opinions
- **Modular**: References should be independently useful
- **Scannable**: Use headers, lists, and code blocks
- **Actionable**: Include concrete code examples
- **Complete**: Cover basics to advanced
