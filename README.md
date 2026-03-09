# Skills

A small public repo of reusable agent skills, packaged for installation with [`npx skills`](https://skills.sh/).

## Included Skills

- `agents-md` - Install or update Robin Ebers's `AGENTS.md` in a workspace without losing local notes.
- `code-auditor` - Audit codebases for duplicate code, dead code, dependency bloat, and refactoring opportunities.
- `conductor-json` - Generate `conductor.json` files for Conductor workspaces.
- `pr-manager` - Create pull requests with `gh`, wait for checks, and fetch or summarize review comments.

## Install

Install everything from GitHub:

```bash
npx skills add robinebers/skills
```

Install a single skill:

```bash
npx skills add robinebers/skills --skill agents-md
npx skills add robinebers/skills --skill code-auditor
npx skills add robinebers/skills --skill pr-manager
npx skills add robinebers/skills --skill conductor-json
```

List what the package exposes:

```bash
npx skills add robinebers/skills --list
```

The `skills` CLI will install each skill into the right location for your agent, including Cursor, Claude Code, Codex, and others.
