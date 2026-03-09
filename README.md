# Skills

A small public repo of reusable agent skills, packaged for installation with [`npx skills`](https://skills.sh/).

## Structure

```text
.cursor/
└── agents/
    └── code-auditor.md
skills/
├── conductor-json/
│   └── SKILL.md
└── pr-manager/
    ├── SKILL.md
    └── agents/
        └── openai.yaml
```

## Included Skills

- `pr-manager` - Create pull requests with `gh`, wait for checks, and fetch or summarize review comments.
- `conductor-json` - Generate `conductor.json` files for Conductor workspaces.

## Install

Install everything from GitHub:

```bash
npx skills add robinebers/skills
```

Install a single skill:

```bash
npx skills add robinebers/skills --skill pr-manager
npx skills add robinebers/skills --skill conductor-json
```

List what the package exposes:

```bash
npx skills add robinebers/skills --list
```

The `skills` CLI will install each skill into the right location for your agent, including Cursor, Claude Code, Codex, and others.

## Other Files

- `.cursor/agents/code-auditor.md` is a Cursor agent prompt I still keep in this repo, but it is not part of the installable `npx skills` package.