# Open a PR

Use `gh` for all GitHub operations. Use `git` for local commits.

## Pre-PR Workflow

1. **Verify prerequisites**
   - Run `GIT_EDITOR=true git status` to check working tree state.
   - Run `GIT_EDITOR=true git branch --show-current` to get current branch.
   - Determine repo default branch with:
     - `gh repo view --json defaultBranchRef -q .defaultBranchRef.name`
     - Fallback order if needed: `main`, then `master`.
   - If current branch is the default branch, bail and explain.

2. **Handle local changes**
   - If there are unstaged/staged changes, commit relevant changes first using `git` (not `gh`).
   - Keep commit scope focused; do not commit unrelated files.

3. **Check branch tracking**
   - Run `GIT_EDITOR=true git rev-parse --abbrev-ref @{upstream}`.
   - If no upstream exists, plan to push with `git push -u origin HEAD` before creating PR.

4. **Analyze changes from base to HEAD**
   - Run `GIT_EDITOR=true git log <base_branch>..HEAD --oneline`.
   - Run `GIT_EDITOR=true git diff <base_branch>...HEAD --stat`.
   - Run detailed diff as needed: `GIT_EDITOR=true git diff <base_branch>...HEAD`.

5. **Find PR template**
   - Check, in order:
     - `.github/pull_request_template.md`
     - `.github/PULL_REQUEST_TEMPLATE.md`
     - `docs/pull_request_template.md`
     - `pull_request_template.md`
   - Also check for multiple templates under `.github/PULL_REQUEST_TEMPLATE/`.
   - If no template exists, use fallback:
     - `## Summary` with 1-3 bullets
     - `## Test plan` as a checklist

6. **Draft PR content**
   - Generate a concise title from actual branch changes.
   - Fill every section of the template (or fallback) with concrete details.
   - Keep TL;DR to <=2 sentences.

7. **Push and create PR**
   - Push branch if needed: `GIT_EDITOR=true git push -u origin HEAD`.
   - Create PR with `gh pr create` and a heredoc body.
   - Always paste the PR URL in your response.

## Fallback PR Format (only if no repo template exists)

`<feature_area>: <Title>` (80 characters or less)

`<TLDR>` (no more than 2 sentences)

`<Description>`
- 1-3 bullets explaining what changed

`<Test plan>`
- checklist of validation steps performed

## Post-PR: Wait for CI & Pull Comments

After the PR is successfully created, run the full end-to-end workflow below **automatically** (do not ask):

### 1. Initial wait (5 minutes)

Wait 5 minutes for CI checks to start and make progress. Use `sleep 300` in a backgrounded shell, then check back.

### 2. Poll checks every 30s until complete

Use `gh pr checks <PR_NUMBER> --json name,state,status,conclusion` to poll.

- A check is **done** when its `status` is `"completed"` (regardless of conclusion).
- Keep polling every 30 seconds (`sleep 30`) until **all** checks have `status: "completed"`.
- While polling, show a brief status update each cycle (e.g. "3/5 checks done, waiting...").
- Safety cap: after 20 minutes of polling (40 cycles), stop and report which checks are still pending.

### 3. Report check results

Once all checks finish, summarize outcomes:
- List each check name + conclusion (success/failure/skipped/etc.)
- If any checks failed, call that out prominently.

### 4. Pull review comments

After checks complete, run the **full `/pull-comments` workflow** against this PR:

1. **Fetch ALL comments** using `gh`:
   ```bash
   gh pr view <PR_NUMBER> --json reviews,comments,url,title,author --jq '.'
   gh api repos/{owner}/{repo}/pulls/{number}/comments --paginate
   gh api repos/{owner}/{repo}/issues/{number}/comments --paginate
   ```

2. **Process & Organize**:
   - Group comments by **reviewer** (author login)
   - **Dedupe** merge duplicate/similar comments
   - For threads, collapse into single item with context

3. **Categorize by Severity** (infer from tone/content):
   - 🔴 **Blocking**: "must", "required", "can't merge", security issues, bugs
   - 🟡 **Should Fix**: "should", "please", style violations, better approaches
   - 🟢 **Suggestions**: "consider", "optional", "nit", "minor", questions

4. **Explain Simply** for each comment:
   - What the reviewer wants (plain English)
   - Why it matters (1 sentence)
   - Quick fix hint if obvious

5. **Output** using the numbered format (C1, C2, C3...) with Quick Reference table and Summary — same format as `/pull-comments`.

If there are no review comments yet, just say "No review comments yet" and you're done.
