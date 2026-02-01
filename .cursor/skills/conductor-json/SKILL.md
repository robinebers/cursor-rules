---
name: conductor-json
description: Generate conductor.json files for Conductor workspaces. Use when setting up a new project, creating conductor.json, or configuring workspace scripts for Tauri, Next.js, or React projects.
---

# Conductor JSON

Generate `conductor.json` to share workspace scripts with teammates.

## Schema

| Field | Type | Description |
|-------|------|-------------|
| `scripts.setup` | string | Command when setting up workspace |
| `scripts.run` | string | Command to start dev server |
| `scripts.archive` | string | Command when archiving workspace |
| `runScriptMode` | `"concurrent"` \| `"nonconcurrent"` | Kill in-progress run scripts before new one |

## Setup Script Pattern

Standard setup copies config dirs and env file from repo root to workspace:

```bash
bun install; for d in .claude .codex .cursor; do [ -d "$CONDUCTOR_ROOT_PATH/$d" ] && mkdir -p "$CONDUCTOR_WORKSPACE_PATH/$d" && rsync -a "$CONDUCTOR_ROOT_PATH/$d/." "$CONDUCTOR_WORKSPACE_PATH/$d/"; done; [ -f "$CONDUCTOR_ROOT_PATH/{ENV_FILE}" ] && cp "$CONDUCTOR_ROOT_PATH/{ENV_FILE}" "$CONDUCTOR_WORKSPACE_PATH/{ENV_FILE}"
```

Replace `{ENV_FILE}` based on project type.

## Templates

### Tauri

```json
{
    "scripts": {
        "setup": "bun install; for d in .claude .codex .cursor; do [ -d \"$CONDUCTOR_ROOT_PATH/$d\" ] && mkdir -p \"$CONDUCTOR_WORKSPACE_PATH/$d\" && rsync -a \"$CONDUCTOR_ROOT_PATH/$d/.\" \"$CONDUCTOR_WORKSPACE_PATH/$d/\"; done; [ -f \"$CONDUCTOR_ROOT_PATH/.env\" ] && cp \"$CONDUCTOR_ROOT_PATH/.env\" \"$CONDUCTOR_WORKSPACE_PATH/.env\"",
        "run": "bun run tauri dev"
    }
}
```

### Next.js / React

```json
{
    "scripts": {
        "setup": "bun install; for d in .claude .codex .cursor; do [ -d \"$CONDUCTOR_ROOT_PATH/$d\" ] && mkdir -p \"$CONDUCTOR_WORKSPACE_PATH/$d\" && rsync -a \"$CONDUCTOR_ROOT_PATH/$d/.\" \"$CONDUCTOR_WORKSPACE_PATH/$d/\"; done; [ -f \"$CONDUCTOR_ROOT_PATH/.env.local\" ] && cp \"$CONDUCTOR_ROOT_PATH/.env.local\" \"$CONDUCTOR_WORKSPACE_PATH/.env.local\"",
        "run": "bun run dev"
    }
}
```

## Environment Variables

- `$CONDUCTOR_ROOT_PATH` - Original repo root
- `$CONDUCTOR_WORKSPACE_PATH` - Current workspace directory

## Usage

1. Identify project type (check for `tauri.conf.json`, `next.config.*`, or `package.json` scripts)
2. Create `conductor.json` in workspace root using appropriate template
3. Commit to git so teammates get the same scripts
