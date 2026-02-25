# Super Skills

A collection of Claude Code skills for iOS, Swift, React, and frontend development.

## Skills

| Skill | Description |
|-------|-------------|
| **swift** | Swift 6+ with modern concurrency and SwiftUI patterns |
| **swiftui** | SwiftUI component design, composition, and styling |
| **swift-data** | SwiftData persistence with models, queries, relationships, and CloudKit sync |
| **swift-testing** | Swift Testing framework patterns and best practices |
| **ios-developer** | Comprehensive iOS development with SwiftUI, Concurrency, Core Data, Testing |
| **react** | React 18+, TypeScript, Tailwind, shadcn/ui |
| **react-developer** | Comprehensive React with performance, design, composition, React Native |
| **frontend-design** | Distinctive, production-grade frontend interfaces |
| **feature-plan** | Implementation planning and architecture design |
| **skill-creator** | Creates new Claude Code skills with proper structure and documentation |
| **blog-writer** | Technical blog writing with hooks, progressive complexity, code examples |

## Installation

### Using skills.sh (Recommended)

```bash
npx skills add https://github.com/onmyway113/skills --skill swift
npx skills add https://github.com/onmyway113/skills --skill swiftui
npx skills add https://github.com/onmyway113/skills --skill react
```

### Manual Installation

Clone this repository and add skills to your Claude Code configuration:

```bash
git clone https://github.com/onmyway113/skills.git
```

Then reference the skills in your Claude Code settings.

## Usage

Skills are automatically applied based on context. For example:

- When working on Swift files, the `swift` skill provides modern patterns
- When building SwiftUI views, the `swiftui` skill offers component patterns
- When planning features, use the `feature-plan` skill for structured blueprints

## Structure

```
skills/
├── skills/
│   ├── swift/SKILL.md
│   ├── swiftui/SKILL.md
│   ├── swift-data/
│   │   ├── SKILL.md
│   │   └── references/
│   ├── ios-developer/
│   │   ├── SKILL.md
│   │   └── references/
│   ├── swift-testing/SKILL.md
│   ├── react/SKILL.md
│   ├── react-developer/
│   │   ├── SKILL.md
│   │   └── references/
│   ├── frontend-design/SKILL.md
│   ├── feature-plan/SKILL.md
│   ├── skill-creator/SKILL.md
│   └── blog-writer/
│       ├── SKILL.md
│       └── references/
├── .claude-plugin/
│   └── marketplace.json
├── .claude/
│   └── CLAUDE.md
├── .mcp.json
├── README.md
└── LICENSE
```

## Marketplace

This repository is configured as a Claude Code skills marketplace named `super-skills`.

## Contributing

1. Fork the repository
2. Create a new skill folder under `skills/`
3. Add a `SKILL.md` file with YAML frontmatter
4. Submit a pull request

## References

### Skills Frameworks
- [Anthropic Skills](https://github.com/anthropics/skills) - Official Anthropic skills documentation
- [Vercel Agent Skills](https://github.com/vercel-labs/agent-skills) - Vercel Labs agent skills examples

### iOS Development Skills
- [AvdLee/SwiftUI-Agent-Skill](https://github.com/AvdLee/SwiftUI-Agent-Skill) - SwiftUI best practices
- [AvdLee/Swift-Concurrency-Agent-Skill](https://github.com/AvdLee/Swift-Concurrency-Agent-Skill) - Swift Concurrency patterns
- [AvdLee/Core-Data-Agent-Skill](https://github.com/AvdLee/Core-Data-Agent-Skill) - Core Data guidance
- [AvdLee/Swift-Testing-Agent-Skill](https://github.com/AvdLee/Swift-Testing-Agent-Skill) - Swift Testing framework
- [Dimillian/Skills](https://github.com/Dimillian/Skills) - iOS development skills collection
## License

MIT License - see [LICENSE](LICENSE)
