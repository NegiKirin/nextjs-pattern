# nextjs-pattern

Claude Code skill for analyzing, scaffolding, and reviewing Next.js feature-based modular architecture.

## What this repo is

This is a standalone skill authoring repo. It is not installed into `.claude/` by default.

```text
nextjs-pattern/
├── SKILL.md
└── references/
```

## Skill contents

- [SKILL.md](SKILL.md) — skill entrypoint and core rules
- [references/architecture.md](references/architecture.md) — layer and dependency rules
- [references/scaffolding.md](references/scaffolding.md) — feature scaffolding examples
- [references/core-infrastructure.md](references/core-infrastructure.md) — API client and core setup
- [references/fullstack-boundary.md](references/fullstack-boundary.md) — FE-only vs fullstack classification
- [references/review-checklist.md](references/review-checklist.md) — review checklist

## Use cases

Use this skill to help Claude Code:

- classify a Next.js project as frontend-only or fullstack
- scaffold a feature module with API, hooks, components, and barrel exports
- review App Router boundaries
- enforce feature-based imports and dependency direction
- avoid `core/` depending on `features/`

## Install later

When you want to use the skill, copy or symlink this repo to a Claude Code skill discovery location, for example:

```bash
~/.claude/skills/nextjs-pattern
```

Do not install it yet if you only want to keep authoring the skill.
