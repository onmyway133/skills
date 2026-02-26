---
name: feature_plan
description: Designs comprehensive implementation plans for new features with architecture considerations
---

# Feature Plan

When asked to plan a new feature, create a structured implementation blueprint.

## Planning Process

### Step 1: Understand Requirements

Before planning, clarify:
- What problem does this feature solve?
- Who are the users affected?
- What's the expected behavior?
- Are there edge cases or constraints?

Ask clarifying questions if requirements are ambiguous.

### Step 2: Assess Current State

Explore the codebase to understand:
- Existing patterns and architecture
- Related features or components
- Reusable code that can be leveraged
- Potential conflicts or dependencies

### Step 3: Design the Solution

Create a plan covering:

#### Architecture Overview
```
[User Action]
     ↓
[View Layer] ← What UI components
     ↓
[ViewModel/Logic] ← Business rules
     ↓
[Data Layer] ← Storage, network, etc.
```

#### Component Breakdown

For each component:
- **Purpose**: What it does
- **Location**: Where it lives in the codebase
- **Dependencies**: What it needs
- **Interface**: Public API it exposes

#### Data Flow

Describe how data moves:
- User inputs and actions
- State transformations
- Persistence points
- Output/display

### Step 4: Identify Files to Modify

Create a concrete list:

| File | Action | Changes |
|------|--------|---------|
| `path/to/file` | Create | New view model |
| `path/to/existing` | Modify | Add new method |

### Step 5: Define Implementation Sequence

Order tasks by dependencies:

```
1. [ ] Data models (no dependencies)
2. [ ] Repository/Service layer (depends on models)
3. [ ] ViewModel (depends on service)
4. [ ] Views (depends on view model)
5. [ ] Integration and testing
```

### Step 6: Risk Assessment

Identify potential issues:
- Breaking changes to existing features
- Performance considerations
- Security implications
- Migration requirements

## Plan Output Format

```markdown
# Feature: [Name]

## Overview
Brief description of what we're building.

## Requirements
- [ ] Requirement 1
- [ ] Requirement 2

## Architecture

### Components
- **ComponentA**: Purpose and responsibility
- **ComponentB**: Purpose and responsibility

### Data Flow
[Diagram or description]

## Implementation Steps

### Phase 1: Foundation
- [ ] Task 1.1
- [ ] Task 1.2

### Phase 2: Core Logic
- [ ] Task 2.1

### Phase 3: UI
- [ ] Task 3.1

### Phase 4: Polish
- [ ] Testing
- [ ] Edge cases

## Files to Create/Modify
[Table of files]

## Risks and Considerations
- Risk 1: Mitigation strategy
- Risk 2: Mitigation strategy

## Open Questions
- Question needing user input
```

## Guidelines

- Keep plans actionable and specific
- Reference existing patterns in the codebase
- Break large features into phases
- Include testing as part of the plan
- Identify blockers or unknowns upfront
- Suggest incremental delivery when possible
