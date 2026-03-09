---
name: agents-md
description: Install or update Robin Ebers's AGENTS.md from github.com/robinebers/agents.md. Use when the user asks to add AGENTS.md to a workspace, refresh it to the latest upstream version, merge upstream changes into a customized local AGENTS.md, or make the repo follow Robin's agent protocol file.
---

# Agents.md

Use this skill to manage the workspace-root `AGENTS.md` file from `robinebers/agents.md`.

## Source Of Truth

- Treat `https://raw.githubusercontent.com/robinebers/agents.md/main/AGENTS.md` as the canonical upstream file.
- Work on the workspace-root `AGENTS.md` unless the user explicitly asks for another path.
- Preserve intentional local edits during updates, especially repo-specific notes and the `## User Notes` section.
- If the workspace already has `AGENTS.md`, do not overwrite it during install. Switch to the update workflow instead.

## Install Workflow

Run this when the user asks to install, add, or bootstrap `AGENTS.md`.

1. Check whether `AGENTS.md` already exists at the workspace root.
2. If it does not exist, install it with:

```bash
curl -fsSL https://raw.githubusercontent.com/robinebers/agents.md/main/AGENTS.md -o AGENTS.md
```

3. Verify the file exists and report the version line if present.
4. If `AGENTS.md` already exists, do not clobber it. Explain that the file is already installed and use the update workflow if the user wants latest upstream content.

## Update Workflow

Run this when the user asks to update `AGENTS.md` to the latest version.

1. Fetch the upstream file into a temporary path, not directly over the local file.

```bash
tmpfile="$(mktemp)"
curl -fsSL https://raw.githubusercontent.com/robinebers/agents.md/main/AGENTS.md -o "$tmpfile"
```

2. Compare the upstream file with the local `AGENTS.md`.
3. If there is no effective change, say so and leave the file untouched.
4. If upstream changed, merge the latest upstream content into the local file without losing intentional local edits.
5. Preserve local notes by default, especially under `## User Notes`.
6. When a rule changed upstream and the local file also changed in the same area, prefer the user's local customization only when it is clearly intentional. Otherwise prefer upstream wording.
7. Replace the local file only after the merge result is ready, then verify the final file on disk.

## Merge Rules

- Keep the file as a single `AGENTS.md`; do not split it into helper docs.
- Preserve repo-specific local notes, custom commands, and user-maintained sections.
- Keep the upstream version header current after a successful update unless the user intentionally changed it.
- If the merge is ambiguous, surface the conflict briefly and ask one blocking question instead of guessing.

## Typical Triggers

- "Install Robin's AGENTS.md here."
- "Add your AGENTS file to this repo."
- "Update the agents to the latest version."
- "Refresh AGENTS.md without losing our local notes."

## Done Criteria

- `AGENTS.md` exists at the intended root path.
- Installs use the upstream raw URL above.
- Updates preserve intentional local content while bringing in upstream changes.
- The final response states whether the operation was install, no-op, or merged update.
