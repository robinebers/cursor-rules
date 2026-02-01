# pull-comments

Fetch, organize, and explain all review comments from a GitHub PR.

## Instructions

1. **Determine PR**: Check if there's a known PR context (branch tracking, recent `gh pr` commands, or user-provided URL/number). If unknown, ask user: "Which PR? (URL, number, or 'current' for this branch)"

2. **Fetch ALL comments** using `gh`:
   ```bash
   # Get PR reviews with comments
   gh pr view <PR> --json reviews,comments,url,title,author --jq '.'

   # Get review comments (inline code comments)
   gh api repos/{owner}/{repo}/pulls/{number}/comments --paginate

   # Get issue-style comments (general discussion)
   gh api repos/{owner}/{repo}/issues/{number}/comments --paginate
   ```

3. **Process & Organize**:
   - Group comments by **reviewer** (author login)
   - **Dedupe**: Merge duplicate/similar comments (same file+line, or >80% text similarity)
   - For threads, collapse into single item with context

4. **Categorize by Severity** (infer from tone/content):
   - 🔴 **Blocking**: "must", "required", "can't merge", security issues, bugs
   - 🟡 **Should Fix**: "should", "please", style violations, better approaches
   - 🟢 **Suggestions**: "consider", "optional", "nit", "minor", questions

5. **Explain Simply** for each comment:
   - What the reviewer wants (plain English, no jargon)
   - Why it matters (1 sentence)
   - Quick fix hint if obvious

## Output Format

Number each comment sequentially (1, 2, 3...) across all categories so the user can easily reference them like "fix #3" or "skip #7".

```
## PR #123: {title}
By @{author} | {n} reviewers | {total} comments

---

### 🔴 Blocking ({count})

**#C1** · @reviewer1 · `src/file.tsx:42`
Original comment text summarized in simple paragraph

**Translation**: [Plain English explanation]
**Why**: [Plain English brief reason this matters]
**Fix**: [Plain English suggestion if clear]

**#C2** · @reviewer2 · `src/api.ts:88`
[...]

---

### 🟡 Should Fix ({count})

**#C3** · @reviewer1 · `src/utils.ts:12`
[...]

### 🟢 Suggestions ({count})

**#5** · @reviewer3 · `src/index.ts:5`
[...]

---

## Quick Reference
| # | File | Reviewer | Severity |
|---|------|----------|----------|
| 1 | src/file.tsx:42 | @reviewer1 | 🔴 |
| 2 | src/api.ts:88 | @reviewer2 | 🔴 |
| 3 | src/utils.ts:12 | @reviewer1 | 🟡 |
[...]

## Summary
- {blocking} blocking issues to resolve
- {should_fix} recommended changes
- {suggestions} optional improvements
- Top concerns: [1-2 sentence summary of main themes]
```

## Edge Cases
- If PR has no comments: "No review comments yet"
- If API fails: Show error, suggest checking `gh auth status`
- Very long threads: Summarize, show first/last comment + count
