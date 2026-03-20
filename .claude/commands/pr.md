---
allowed-tools: Bash, Read, Grep, Glob
description: Create a pull request from the current branch to develop (works for issue fixes, features, or any changes)
argument-hint: [optional: target-branch, defaults to develop]
---

# CREATE PULL REQUEST

You are creating a pull request from the current branch. This works for:
- **Issue fixes** - After using `/fix-issue` (auto-links and closes the issue)
- **New features** - General feature development
- **Bug fixes** - Fixes not tied to a GitHub issue
- **Refactoring** - Code improvements
- **Any other changes** - Documentation, tests, etc.

**CRITICAL: PRs should target the `develop` branch by default (per CONTRIBUTING.md guidelines).**

---

## INPUT

Target branch: `$ARGUMENTS` (defaults to `develop` if not specified)

---

## PHASE 1: PRE-FLIGHT CHECKS

### Step 1.1: Verify current state

Run these commands to understand the current state:

```bash
# Current branch
git branch --show-current

# Check for uncommitted changes
git status

# Check if we're ahead of remote
git status -sb
```

**STOP CONDITIONS:**
- If on `main` or `develop` branch: STOP and warn user they should be on a feature branch
- If there are uncommitted changes: STOP and ask user to commit first
- If no commits ahead of target: STOP and inform user there's nothing to PR

### Step 1.2: Verify commits

```bash
# Show commits that will be in the PR
git log --oneline develop..HEAD
```

If no commits are shown, inform the user there's nothing to create a PR for.

---

## PHASE 2: DETECT PR TYPE AND GATHER CONTEXT

### Step 2.1: Determine the type of PR

Check the branch name and context to determine what type of PR this is:

```bash
# Get branch name
BRANCH=$(git branch --show-current)
echo "Branch: $BRANCH"
```

**Detect PR type from branch name patterns:**

| Pattern | Type | Example |
|---------|------|---------|
| `fix/issue-{N}` or `fix/gh-{N}` | GitHub Issue Fix | `fix/issue-586` |
| `fix/*` (no issue number) | Bug Fix | `fix/memory-leak` |
| `feat/*` or `feature/*` | New Feature | `feat/dark-mode` |
| `refactor/*` | Refactoring | `refactor/auth-module` |
| `docs/*` | Documentation | `docs/api-guide` |
| `test/*` | Tests | `test/add-coverage` |
| `chore/*` | Maintenance | `chore/update-deps` |
| `perf/*` | Performance | `perf/optimize-queries` |
| Other | General | Determine from commits |

### Step 2.2: Extract GitHub issue number (if applicable)

**Check multiple sources for issue references:**

1. **Branch name:** Look for patterns like `issue-123`, `gh-123`, `#123`
2. **Commit messages:** Search for `Fixes #123`, `Closes #123`, `Resolves #123`
3. **Issue documentation:** Check `docs/issues/GITHUB_*.md` files
4. **Conversation context:** If you worked on an issue in this session, you know the number

```bash
# Check branch name for issue number
BRANCH=$(git branch --show-current)
echo "$BRANCH" | grep -oE '[0-9]+' | head -1

# Check recent commit messages for issue references
git log --oneline -10 | grep -oE '(Fixes|Closes|Resolves|fixes|closes|resolves) #[0-9]+' | head -1

# List issue documentation
ls docs/issues/GITHUB_*.md 2>/dev/null
```

**If an issue number is found:**
- Read the issue documentation if it exists
- Ensure the PR body includes `Fixes #{number}` to auto-close
- Include issue context in the PR description

### Step 2.3: Analyze the changes

```bash
# Get diff stats
git diff --stat develop..HEAD

# Get full diff for context (for understanding what changed)
git diff develop..HEAD
```

### Step 2.4: Gather additional context

```bash
# See if any tests were added or modified
git diff --name-only develop..HEAD | grep -E "(test_|_test\.py|\.test\.ts|\.spec\.ts)"

# Check what types of files changed
git diff --name-only develop..HEAD
```

---

## PHASE 3: PUSH BRANCH

### Step 3.1: Push to remote

```bash
# Get current branch name
BRANCH=$(git branch --show-current)

# Push with upstream tracking
git push -u origin $BRANCH
```

If push fails due to authentication, inform the user to authenticate with GitHub first.

---

## PHASE 4: CREATE PULL REQUEST

### Step 4.1: Generate PR content based on type

**IMPORTANT:** The PR body format depends on the type of change.

---

#### TYPE A: GitHub Issue Fix

**Detected when:** Branch contains issue number OR commits reference an issue OR issue documentation exists.

**CRITICAL:** Include `Fixes #123` on its own line to auto-close the issue when merged.

```bash
gh pr create --base develop --title "fix: {short_description} (#${ISSUE_NUMBER})" --body "$(cat <<'EOF'
## Summary

{Brief description of what was fixed}

Fixes #{ISSUE_NUMBER}

## Root Cause

{Explanation of why the bug occurred - pull from issue documentation if available}

## Solution

{What the fix does and why this approach was chosen}

## Changes

{List of key changes made, organized by file/area}

## Testing

- [ ] Tested on macOS
- [ ] Tested on Windows
- [ ] Tested on Linux
- [ ] Existing tests pass
- [ ] New tests added (if applicable)

## Cross-Platform Compatibility

{Notes on platform-specific considerations, if applicable}

EOF
)"
```

---

#### TYPE B: New Feature

**Detected when:** Branch starts with `feat/` or `feature/`

```bash
gh pr create --base develop --title "feat: {feature_name}" --body "$(cat <<'EOF'
## Summary

{Brief description of the new feature}

## Motivation

{Why this feature is needed}

## Changes

{List of key changes, new files, etc.}

## How It Works

{Brief explanation of the implementation}

## Testing

- [ ] Tested locally
- [ ] Existing tests pass
- [ ] New tests added for the feature

## Screenshots/Demo

{If applicable, add visual proof or usage examples}

EOF
)"
```

---

#### TYPE C: Bug Fix (No GitHub Issue)

**Detected when:** Branch starts with `fix/` but has no issue number

```bash
gh pr create --base develop --title "fix: {description}" --body "$(cat <<'EOF'
## Summary

{Brief description of the bug and fix}

## Problem

{What was broken and how it manifested}

## Solution

{How the fix resolves the issue}

## Changes

{List of key changes}

## Testing

- [ ] Tested locally
- [ ] Existing tests pass
- [ ] Verified fix works

EOF
)"
```

---

#### TYPE D: Refactoring

**Detected when:** Branch starts with `refactor/`

```bash
gh pr create --base develop --title "refactor: {description}" --body "$(cat <<'EOF'
## Summary

{Brief description of the refactoring}

## Motivation

{Why this refactoring was needed}

## Changes

{What was reorganized/improved}

## Impact

- No functional changes (behavior is identical)
- {Or list any intentional behavior changes}

## Testing

- [ ] Existing tests pass
- [ ] Manually verified behavior is unchanged

EOF
)"
```

---

#### TYPE E: General Changes

**Detected when:** None of the above patterns match

Determine the type from the commit messages and changes:

```bash
gh pr create --base develop --title "{type}: {description}" --body "$(cat <<'EOF'
## Summary

{Brief description of what this PR does}

## Changes

{List of key changes}

## Testing

- [ ] Tested locally
- [ ] Existing tests pass

## Notes

{Any additional context}

EOF
)"
```

### Step 4.2: Verify PR was created

After `gh pr create` succeeds, it will output the PR URL.

---

## PHASE 5: REPORT TO USER

Provide a summary:

```
## Pull Request Created

**PR URL:** {url}
**Branch:** {branch_name} -> develop
**Title:** {pr_title}
**Type:** {Issue Fix / Feature / Bug Fix / Refactor / etc.}

### Issue Auto-Close
{If issue number found: "GitHub Issue #{number} will be automatically closed when this PR is merged."}
{If no issue: "No GitHub issue linked to this PR."}

### Commits Included
{list of commits}

### Files Changed
{diff stats}

### Next Steps
1. Wait for CI checks to pass
2. Request review if needed
3. Address any CodeRabbit feedback using `/coderabbitreview`
4. Merge when approved
```

---

## ISSUE LINKING KEYWORDS

GitHub recognizes these keywords to auto-close issues when the PR is merged:

| Keyword | Example | Effect |
|---------|---------|--------|
| `Fixes` | `Fixes #123` | Closes issue #123 |
| `Closes` | `Closes #123` | Closes issue #123 |
| `Resolves` | `Resolves #123` | Closes issue #123 |
| `fix` | `fix #123` | Closes issue #123 |
| `close` | `close #123` | Closes issue #123 |
| `resolve` | `resolve #123` | Closes issue #123 |

**Always use `Fixes #{number}` on its own line in the PR body for reliable auto-closing.**

---

## ERROR HANDLING

### If not authenticated with GitHub
```
gh auth login
```
Then retry.

### If PR already exists
```bash
gh pr view
```
Show the existing PR to the user.

### If branch doesn't exist on remote
Push it first, then create PR.

### If develop is behind main
```bash
git fetch origin
git log --oneline origin/main..origin/develop
```
Warn user if develop is significantly behind main.

---

## ALTERNATIVE TARGETS

While `develop` is the default, user can specify different targets:

- `/pr main` - Direct to main (for hotfixes)
- `/pr feature/other-branch` - To another feature branch

Always confirm with user if targeting `main` directly:
```
WARNING: You're creating a PR directly to main.
This should only be done for critical hotfixes.
Continue? (y/n)
```

---

## IMPORTANT NOTES

1. **Always target develop** unless explicitly overridden
2. **Auto-detect issue references** from branch, commits, and documentation
3. **Use `Fixes #123`** to auto-close linked issues on merge
4. **Adapt PR format** based on the type of change (feature, fix, refactor, etc.)
5. **Document cross-platform testing** for UI/Electron changes
6. **Don't merge** - just create the PR for review

User input: $ARGUMENTS
