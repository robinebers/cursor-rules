# Cursor Rules

A collection of rules, agents, commands, and skills for [Cursor](https://cursor.com) that I use in my workflows.

## Structure

```
.cursor/
├── agents/
│   └── code-auditor.md          # Codebase analysis & refactoring specialist
├── commands/
│   ├── create-pr.md             # Open a PR with auto-generated description
│   └── pull-comments.md         # Fetch & organize PR review comments
├── rules/
│   ├── convex_rules.mdc
│   ├── decision-making.mdc
│   ├── file-length-modularity.mdc
│   ├── general.mdc
│   ├── research-guidelines.mdc
│   └── tailwind-v4.mdc
└── skills/
    └── conductor-json/          # Generate conductor.json for workspaces
```

## What's Included

### Agents

- **code-auditor** — Expert code auditor that finds duplicate code, unused code, DRY violations, technical debt, and unused dependencies. Ideal after major feature work or before refactoring.

### Commands

- **pull-comments** — Fetches all review comments from a GitHub PR, dedupes them, categorizes by severity (blocking / should fix / suggestions), and explains each in plain English.
- **create-pr** — Creates a PR from your current branch with a structured description, auto-committing any unstaged work first.

### Rules

- **convex_rules** — Convex-specific patterns and best practices
- **decision-making** — Structured decision-making guidelines
- **file-length-modularity** — File length and modularity standards
- **general** — General coding conventions
- **research-guidelines** — Research and exploration guidelines
- **tailwind-v4** — Tailwind CSS v4 patterns

### Skills

- **conductor-json** — Generates `conductor.json` files for Conductor workspaces, with templates for Tauri, Next.js, and React projects.

## Usage

### Manual Install

Copy the `.cursor/` directory into your project root. Cursor will automatically pick up rules, agents, commands, and skills from this location.

### Automatic

As of January 2026, the Cursor Nightly version has a "From GitHub" functionality. So you can just link it and apply them to your project that way.

Just put in this repo URL:
`https://github.com/robinebers/cursor-rules.git`

Good luck!