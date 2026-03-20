---
allowed-tools: Bash, Read, Write, Edit, Grep, Glob, Task, TodoWrite, WebFetch
description: Review PR feedback (CodeRabbit, CI, comments) and fix any issues before merge
argument-hint: [optional: PR number or URL, defaults to current branch's PR]
---

# FOLLOW UP ON PULL REQUEST

You are following up on a pull request that was previously created. This is typically used after:
- AI review bots (CodeRabbit, etc.) have finished reviewing (~5 minutes)
- CI/CD tests have completed
- Human reviewers have left comments

Your job is to read ALL feedback and fix any issues so the PR is ready to merge.

**MANDATORY FIRST STEP:** Before analyzing anything, you MUST fetch ALL comment types:
1. Reviews (`gh pr view --json reviews`)
2. **Inline code comments** (`gh api .../pulls/{n}/comments`) - Cursor Bot posts here!
3. Issue comments (`gh api .../issues/{n}/comments`)

Do NOT skip any of these. Run them in parallel at the start of every follow-up.

---

## INPUT

PR identifier: `$ARGUMENTS`

If not provided, detect the PR for the current branch automatically.

---

## PHASE 1: IDENTIFY THE PR

### Step 1.1: Find the PR

```bash
# Get current branch
git branch --show-current

# Find PR for current branch (if no argument provided)
gh pr view --json number,title,url,state,reviewDecision,statusCheckRollup
```

If `$ARGUMENTS` is provided:
- If it's a number: `gh pr view {number}`
- If it's a URL: Extract the PR number and use that

### Step 1.2: Verify PR state

```bash
gh pr view --json state,mergeable,mergeStateStatus
```

**Check if PR is still open.** If merged or closed, inform the user and stop.

---

## PHASE 2: GATHER ALL PR FEEDBACK

### Step 2.1: Get PR overview

```bash
# Full PR details
gh pr view --json number,title,body,author,baseRefName,headRefName,createdAt,updatedAt

# Get the diff (what we changed)
gh pr diff
```

### Step 2.2: Get all reviews and comments

**CRITICAL: You MUST run ALL THREE of these commands to catch all feedback types:**

```bash
# 1. REVIEWS - Get all reviews (approved, changes requested, commented)
gh pr view {number} --repo {owner}/{repo} --json reviews --jq '.reviews[] | "[\(.state)] @\(.author.login): \(.body)"'

# 2. INLINE CODE COMMENTS - PR review comments attached to specific lines
# This is where Cursor Bot, CodeRabbit inline suggestions, and human line comments appear!
gh api repos/{owner}/{repo}/pulls/{number}/comments --jq '.[] | "[\(.path):\(.line // .original_line)] @\(.user.login): \(.body)"'

# 3. GENERAL DISCUSSION - Issue-style comments (PR conversation thread)
gh api repos/{owner}/{repo}/issues/{number}/comments --jq '.[] | "@\(.user.login): \(.body)"'
```

**Run all three in parallel** to ensure no feedback is missed.

### Step 2.3: Get CI/CD status

```bash
# Get all check runs and their status
gh pr checks {number} --repo {owner}/{repo}

# Get detailed check info if any failed
gh pr view {number} --repo {owner}/{repo} --json statusCheckRollup --jq '.statusCheckRollup[] | "\(.name): \(.status) - \(.conclusion)"'
```

### Step 2.4: Check for AI bot feedback

**Known AI review bots to look for:**
- `coderabbitai[bot]` - CodeRabbit (inline + issue comments)
- `cursor[bot]` - Cursor Bugbot (inline comments only!)
- `gemini-code-assist[bot]` - Gemini Code Assist
- `github-actions[bot]` - Auto Claude PR Review
- Any username containing `[bot]`

The inline PR comments (Step 2.2 command #2) is especially important because **Cursor Bot ONLY posts inline comments**, not issue comments.

---

## PHASE 3: ANALYZE FEEDBACK

### Step 3.1: Categorize the feedback

Create a structured summary of all feedback:

```markdown
## PR Feedback Summary

### CI/CD Status
- [ ] Build: {PASS/FAIL}
- [ ] Tests: {PASS/FAIL}
- [ ] Lint: {PASS/FAIL}
- [ ] Type Check: {PASS/FAIL}

### Review Status
- Approved by: {list}
- Changes Requested by: {list}
- Pending review from: {list}

### CodeRabbit/AI Feedback
{Categorize by severity}
- **Critical:** {issues that must be fixed}
- **Suggestions:** {optional improvements}
- **Nitpicks:** {minor style/formatting}

### Human Comments
{List all human comments with context}

### Required Actions
1. {action 1}
2. {action 2}
...
```

### Step 3.2: Prioritize issues

**Must Fix (blocking merge):**
- CI/CD failures
- Changes requested by required reviewers
- Critical security issues
- Breaking changes identified

**Should Fix (good practice):**
- CodeRabbit suggestions that improve code quality
- Performance improvements
- Missing error handling

**Optional (nice to have):**
- Style nitpicks already handled by linters
- Alternative approaches that aren't necessarily better
- Suggestions that would require significant refactoring

---

## PHASE 4: FIX ISSUES

### Step 4.1: Create fix plan

Use TodoWrite to track all fixes needed:

```
- [ ] Fix CI failure: {description}
- [ ] Address review comment: {description}
- [ ] Fix CodeRabbit issue: {description}
...
```

### Step 4.2: Make fixes

For each issue:

1. **Understand the context** - Read the relevant code
2. **Apply the fix** - Make minimal, targeted changes
3. **Verify locally** - Run tests/lint if applicable
4. **Mark as done** - Update todo list

**IMPORTANT RULES:**
- Fix one issue at a time
- Don't introduce new issues while fixing
- Maintain cross-platform compatibility
- Follow existing code patterns

### Step 4.3: Handle disagreements

If you disagree with feedback:

1. **CodeRabbit suggestions** - Use `/coderabbitreview` to analyze and respond
2. **Human comments** - Document your reasoning but don't ignore valid concerns
3. **CI failures** - These must be fixed, no exceptions

For suggestions you're not implementing, prepare a response explaining why.

---

## PHASE 5: COMMIT AND PUSH

### Step 5.1: Stage and commit fixes

```bash
# Check what changed
git status
git diff

# Stage all changes
git add -A

# Commit with descriptive message
git commit -m "$(cat <<'EOF'
fix: address PR feedback

- {fix 1}
- {fix 2}
- {fix 3}

Co-authored-by: CodeRabbit <coderabbit@users.noreply.github.com>
EOF
)"
```

### Step 5.2: Push updates

```bash
git push
```

### Step 5.3: Respond to comments (if needed)

For inline comments that were addressed:

```bash
# Reply to a specific review comment
gh api repos/{owner}/{repo}/pulls/{number}/comments/{comment_id}/replies -f body="Fixed in latest commit."
```

For general discussion:

```bash
# Add a comment to the PR
gh pr comment {number} --body "Addressed all feedback:
- ✅ Fixed CI failures
- ✅ Applied CodeRabbit suggestions
- ✅ Resolved review comments

Ready for re-review!"
```

---

## PHASE 6: VERIFY AND REPORT

### Step 6.1: Wait for CI (optional)

```bash
# Check if CI is running
gh pr checks --watch
```

### Step 6.2: Report to user

```
## PR Follow-up Complete

**PR:** #{number} - {title}
**URL:** {url}

### Issues Fixed
{List of issues that were fixed}

### Issues Not Fixed (with reasoning)
{List of issues skipped and why}

### CI Status
{Current CI status after push}

### Review Status
{Current review status}

### Next Steps
{What needs to happen for merge - more reviews, CI to pass, etc.}
```

---

## HANDLING SPECIFIC FEEDBACK TYPES

### CodeRabbit Feedback

CodeRabbit typically provides:
1. **Summary** - Overview of changes
2. **Walkthrough** - File-by-file breakdown
3. **Inline comments** - Specific code suggestions

For inline comments, use the existing `/coderabbitreview` command:
```
/coderabbitreview In {file_path} around lines {start} to {end}, {problem}; {recommendation}.
```

### CI Failures

Common CI failures and fixes:

| Failure | How to Fix |
|---------|-----------|
| Type errors | Run `npm run typecheck` locally, fix errors |
| Test failures | Run tests locally, fix failing tests |
| Lint errors | Run `npm run lint:fix` or fix manually |
| Build errors | Check build logs, fix compilation issues |

### Human Review Comments

Types of human feedback:
1. **Questions** - Answer in PR comment
2. **Suggestions** - Implement or explain why not
3. **Required changes** - Must be fixed before merge
4. **Approvals** - No action needed

---

## MULTIPLE FOLLOW-UPS

If you need to follow up multiple times (e.g., after each review cycle):

1. Each follow-up reads fresh state from GitHub
2. Previous fixes are already committed
3. Focus on NEW feedback since last follow-up

```bash
# See commits since PR was opened
gh pr view --json commits --jq '.commits[-5:] | .[] | "\(.oid[:7]) \(.messageHeadline)"'
```

---

## ERROR HANDLING

### If PR not found
- Check if on correct branch
- Verify PR wasn't closed/merged
- Ask user for PR number explicitly

### If no write access
- Can only analyze and report
- User needs to make fixes manually

### If CI is still running
- Report current status
- Suggest waiting and running again

### If conflicts detected
```bash
gh pr view --json mergeable
```
If not mergeable, need to resolve conflicts first.

---

## IMPORTANT NOTES

1. **Fetch ALL comment types** - Reviews, inline PR comments, AND issue comments (3 separate API calls!)
2. **Check for ALL bots** - CodeRabbit, Cursor Bot, Gemini, Auto Claude Review, etc.
3. **Fix blocking issues first** - CI failures, required changes
4. **Be thorough** - Address all actionable feedback
5. **Respond professionally** - Leave helpful comments
6. **Don't over-engineer** - Fix what's asked, nothing more
7. **Cross-platform** - Ensure fixes work on all OS

User input: $ARGUMENTS
