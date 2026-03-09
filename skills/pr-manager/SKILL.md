---
name: pr-manager
description: 'Manage GitHub pull requests with `gh` and `git`: create a pull request from the current branch, wait for CI checks, and fetch and organize review comments. Use when the user says things like "create pr", "create the pr", "open a pr", "open the pr", "pull comments", "check for new comments", "check PR comments", or "fetch review comments".'
---

# PR Manager

Use this skill to create a GitHub pull request or to fetch and summarize review comments for an existing pull request.

## Prerequisites

- Use `gh` for all GitHub operations.
- Use `git` for local commit and push operations.
- Work in the current repository unless the user points to a different one.
- Surface `gh` or API failures clearly and suggest `gh auth status` when authentication may be the cause.

## Create PR Workflow

Run this workflow when the user asks to create or open a PR.

1. Verify prerequisites.
   - Run `GIT_EDITOR=true git status` to inspect the working tree.
   - Run `GIT_EDITOR=true git branch --show-current` to get the current branch.
   - Determine the repo default branch with `gh repo view --json defaultBranchRef -q .defaultBranchRef.name`.
   - If that call fails, fall back to `main`, then `master`.
   - If the current branch is the default branch, create or switch to a feature branch before continuing so the PR is never opened from `main` or `master`.
2. Handle local changes.
   - If unstaged or staged tracked changes exist, commit only the relevant files before creating the PR.
   - Keep commit scope focused. Do not commit unrelated files.
3. Check branch tracking.
   - Run `GIT_EDITOR=true git rev-parse --abbrev-ref @{upstream}`.
   - If no upstream exists, prepare to push with `GIT_EDITOR=true git push -u origin HEAD`.
4. Analyze changes from base to `HEAD`.
   - Run `GIT_EDITOR=true git log <base_branch>..HEAD --oneline`.
   - Run `GIT_EDITOR=true git diff <base_branch>...HEAD --stat`.
   - Run `GIT_EDITOR=true git diff <base_branch>...HEAD` when more detail is needed to draft the PR body.
5. Find a PR template.
   - Check in this order:
     - `.github/pull_request_template.md`
     - `.github/PULL_REQUEST_TEMPLATE.md`
     - `docs/pull_request_template.md`
     - `pull_request_template.md`
   - Also check for multiple templates under `.github/PULL_REQUEST_TEMPLATE/`.
6. Draft PR content.
   - Generate a concise title from the actual changes.
   - Fill every section of the discovered template with concrete details.
   - If no template exists, use this fallback format:
     - `## Summary` with 1-3 bullets
     - `## Test plan` as a checklist
   - Keep the TL;DR to no more than 2 sentences.
7. Push and create the PR.
   - Immediately before the push, record both `review_cycle_started_at=$(date -u +"%Y-%m-%dT%H:%M:%SZ")` and `current_review_commit=$(git rev-parse HEAD)`.
   - Push with `GIT_EDITOR=true git push -u origin HEAD` when needed.
   - Create the PR with `gh pr create` and a heredoc body.
   - Always return the PR URL.

## Post-PR Checks And Comment Pull

After creating a PR, continue automatically without asking the user.

- Run the wait, polling, and follow-up comment pull in the primary thread.
- Do not move `sleep`, check polling, or the follow-up comment workflow into a background terminal, detached process, or subagent.
- Treat `current_review_commit` as the primary marker for the active review cycle.
- Treat `review_cycle_started_at` as a fallback marker for reviews or issue comments that do not include commit metadata.
- After the push completes, wait `sleep 30` in the primary thread before starting the post-push review cycle.
- Use the same post-push review cycle for `create pr` and for any standalone review check that performs a push.

1. Wait 5 minutes for CI and automated reviewers to start.
   - Use `sleep 300` directly in the primary thread.
2. Poll checks every 30 seconds.
   - Run `gh pr checks <PR_NUMBER> --json name,state,startedAt,completedAt,link`.
   - Treat a check as done only when `completedAt` is a real completion timestamp, not an empty value and not `0001-01-01T00:00:00Z`.
   - Keep polling every 30 seconds until all checks are done.
   - Show a brief status update each cycle such as `3/5 checks done, waiting...`.
   - If implementing the polling loop in shell, do not use a non-zero exit status to mean `still waiting`; some shells in agent environments stop immediately on any non-zero exit.
   - Use explicit loop control instead: print the progress, `break` when all checks are done, otherwise `sleep 30` and continue polling.
   - Reserve non-zero exits for real failures such as `gh` returning an error.
   - Stop after 40 cycles (20 minutes) and report any checks still pending.
3. Use the check results only to decide whether to keep waiting or proceed.
   - Do not add a separate checks section to the pull-comments response unless the user explicitly asks for check results.
4. Run the full pull-comments workflow against the created PR.
   - Follow the pull-comments output rules exactly, including the fallback to older comments when the current review cycle is empty.

## Pull Comments Workflow

Run this workflow when the user asks to pull comments, check for new comments, check PR status, or review an existing PR discussion.

1. Determine the PR.
   - Use an explicit PR URL or PR number when the user provides one.
   - Otherwise infer PR context from current branch tracking or recent `gh pr` activity.
   - If the PR still cannot be determined, ask: `Which PR? (URL, number, or "current" for this branch)`
2. Check whether a new review cycle should start before fetching comments.
   - If this workflow is running immediately after `create pr` or another user-requested push, reuse that pushed commit as the active review cycle and continue with the same wait/poll/check flow.
   - If the branch is ahead of its upstream with committed changes during a standalone `pull comments`, `check PR status`, or similar read/check request, ask for confirmation before pushing anything.
   - If the user approves the push, record `review_cycle_started_at=$(date -u +"%Y-%m-%dT%H:%M:%SZ")` and `current_review_commit=$(git rev-parse HEAD)`, push those committed changes, wait `sleep 30`, and then run the same wait/poll/check cycle used by `create pr`.
   - If the user does not approve the push, do not push. Set `current_review_commit=$(git rev-parse @{upstream})` and continue against the currently pushed `HEAD`, making it clear that unpushed local commits are excluded from the review cycle.
   - If there are only unstaged changes, ignore them for review checking, set `current_review_commit=$(git rev-parse HEAD)`, and continue against the currently pushed `HEAD`.
   - If nothing needs to be pushed, set `current_review_commit=$(git rev-parse HEAD)` and continue immediately against the currently pushed `HEAD`.
   - Treat `review_cycle_started_at` as optional unless a push in the active review cycle actually set it.
3. Fetch all comment sources with `gh`.

```bash
gh pr view <PR> --json reviews,comments,url,title,author --jq '.'
gh api repos/{owner}/{repo}/pulls/{number}/comments --paginate
gh api repos/{owner}/{repo}/issues/{number}/comments --paginate
```

4. Select the comment set to summarize.
   - Build `currentCycleComments` first.
   - Prefer comments tied to `current_review_commit` when the API exposes commit metadata such as `commit.oid`, `commit_id`, or `original_commit_id`.
   - For review summaries or issue comments without commit metadata, use `review_cycle_started_at` as the fallback lower bound only when it is known for the active review cycle.
   - If `review_cycle_started_at` is unknown, do not treat it as a required filter; prefer commit-linked comments first, then fall back to older PR comments instead of excluding comments on a missing timestamp filter.
   - If `currentCycleComments` is empty but older PR comments still exist, fall back to those older comments instead of saying `No new review comments yet`.
   - Only say `No new review comments yet` when there are no current-cycle comments and no older comments worth surfacing.
5. Process and organize the fetched data.
   - Filter out comments and threads whose latest state clearly shows they are already addressed, resolved, fixed, or superseded.
   - Prefer unresolved inline comments over generic review-summary comments when both refer to the same issue.
   - Group comments by reviewer login.
   - Deduplicate duplicate or near-duplicate comments.
   - Collapse comment threads into a single item with enough context to understand the issue.
   - If no actionable comments remain after filtering, use the output-format empty states instead of resurfacing already fixed comments.
6. Categorize severity from the reviewer tone and content.
   - `Critical`: required changes, bugs, security problems, or anything that prevents merge.
   - `Should Fix`: recommended changes, style issues, cleaner approaches, or repeated requests.
   - `Suggestions`: optional ideas, nits, questions, or minor improvements.
7. Explain each comment for non-technical readers.
   - Start with a single plain-language summary paragraph that combines the reviewer request and the practical meaning.
   - `Why it matters`: explain the impact in simple everyday language for a non-technical person and avoid jargon.
   - `How to fix`: explain the likely fix in simple everyday language when the fix is obvious.
   - If a technical term is unavoidable, translate it immediately into plain language.

## Output Format

Number comments sequentially as `C1`, `C2`, `C3`, and so on across all categories so the user can refer to them easily.

Limit the response to exactly:
- One title line in the form `PR #123 - {title}`
- `### Critical`
- `### Should Fix`
- `### Suggestions`

Do not include reviewer counts, comment counts, check results, quick-reference tables, summaries, metadata headers, or any extra sections before or after those categories.

If none are available for a category (e.g., 0 suggestions) show an empty state like `No suggestions found.` under `### Suggestions`.

```markdown
PR #123 - {title}

### Critical

🔴 **#C1** · @reviewer1 · `src/file.tsx:42`
{single plain-language summary of the reviewer comment and what it means in practice}

**Why it matters**: {simple impact explanation for a non-technical person}
**How to fix**: {simple likely fix when obvious}

---

### Should Fix

🟠 **#C2** · @reviewer2 · `src/api.ts:88`
...

### Suggestions

🟡 **#C3** · @reviewer3 · `src/index.ts:5`
...
```

## Edge Cases

- If the current review cycle has no comments but older PR comments still exist, summarize the older comments.
- If the user explicitly asks for all comments instead of the current review cycle, skip the review-cycle selection step and summarize the full discussion.
- If comment threads are very long, summarize the thread into one item and keep only the essential context.
- If the GitHub API fails, show the error and suggest checking `gh auth status`.
