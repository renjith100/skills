# Renjith's Skills

Skills for AI-assisted development.

## Available Skills

| Skill | Description |
|-------|-------------|
| `commit-message` | Generate a conventional commit message (type, scope, bullet body) from staged or recent changes |
| `pino-logger` | Logging expert using Pino.js for Node.js and Next.js - structured logging, no console.log |
| `pnpm` | Always use pnpm as the package manager unless the project already uses npm or yarn |
| `worklog` | Track deep work sessions with goal, steps, notes, and review — uses the worklog CLI |

## Community Skills

For other skills, install directly from the original maintained sources:

| Skill | Source |
|-------|--------|
| `node` | [mcollina/skills](https://github.com/mcollina/skills) |
| `next-best-practices` | [vercel/nextjs-skills](https://github.com/vercel/nextjs-skills) |

## Usage

This package contains best practices and skills for AI-assisted development.

### Installing skills into a project

Run these commands from your project's root folder.

#### Next.js project

```bash
npx skills add https://github.com/mcollina/skills --skill node
npx skills add https://github.com/vercel/nextjs-skills --skill next-best-practices
npx skills add https://github.com/renjith100/skills --skill pnpm
npx skills add https://github.com/renjith100/skills --skill pino-logger
npx skills add https://github.com/renjith100/skills --skill commit-message
```

#### Node.js project

```bash
npx skills add https://github.com/mcollina/skills --skill node
npx skills add https://github.com/renjith100/skills --skill pnpm
npx skills add https://github.com/renjith100/skills --skill pino-logger
npx skills add https://github.com/renjith100/skills --skill commit-message
```

> Note: The `--skill` flag matches the `name` field in the skill's YAML frontmatter, not the folder name.
