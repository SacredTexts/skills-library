---
allowed-tools: Bash, Read, Write, Edit, Grep, Glob, Task, TodoWrite, WebFetch, WebSearch
description: Fix a GitHub issue autonomously - validates, investigates, and implements the fix in a worktree
argument-hint: <github-issue-url>
---

# FIX GITHUB ISSUE

You are an autonomous issue-fixing agent. Given a GitHub issue URL, you will:
1. Fetch and document the issue
2. Validate the bug exists through parallel research
3. Critique and deeply understand the root cause
4. Fix the issue in an isolated worktree
5. Ensure cross-platform compatibility (Windows, Linux, macOS)

**CRITICAL: This is an Electron app that runs on all platforms. Every fix MUST work on Windows, Linux, AND macOS.**

---

## INPUT

GitHub Issue URL: `$ARGUMENTS`

If no URL provided, **STOP and ask the user for a GitHub issue URL**.

---

## PHASE 1: FETCH AND DOCUMENT THE ISSUE

### Step 1.1: Extract issue details

Parse the URL to get owner, repo, and issue number. The format is:
`https://github.com/{owner}/{repo}/issues/{number}`

### Step 1.2: Fetch the complete issue

Use the GitHub CLI to get the full issue with all comments:

```bash
gh issue view {number} --repo {owner}/{repo} --comments
```

### Step 1.3: Create local issue document

Create a file at `docs/issues/GITHUB_{number}.md` with the following format:

```markdown
# GitHub Issue #{number}: {title}

**URL:** {issue_url}
**Author:** {author}
**Created:** {created_date}
**Labels:** {labels}
**State:** {state}

## Description

{issue_body}

## Comments

{all_comments_with_authors_and_dates}

## Investigation Notes

_To be filled during investigation_

## Fix Summary

_To be filled after fix is implemented_
```

---

## PHASE 2: PARALLEL VALIDATION (SPAWN 2 RESEARCH AGENTS)

**CRITICAL: Before attempting any fix, validate that the issue is real and reproducible.**

Launch TWO research agents in parallel to independently validate the issue:

### Agent 1: Code Path Validation
Spawn using the Task tool with subagent_type='Explore':
```
Investigate GitHub Issue #{number}: {title}

Your mission: Validate whether this bug can exist in the codebase.

1. Search for the code paths mentioned in the issue
2. Trace the execution flow that would trigger this bug
3. Look for similar patterns that might also be affected
4. Document which files and functions are involved
5. Determine if the described behavior is possible given the current code

Provide a detailed report with:
- Files involved: [list of file paths with line numbers]
- Code flow: [how the bug manifests]
- Verdict: CONFIRMED / UNCONFIRMED / NEEDS MORE INFO
- Evidence: [specific code snippets that prove your verdict]
```

### Agent 2: Platform & Environment Validation
Spawn using the Task tool with subagent_type='Explore':
```
Investigate GitHub Issue #{number}: {title}

Your mission: Validate platform-specific and environmental aspects.

1. Check if this issue is platform-specific (Windows/Linux/macOS)
2. Look for platform-specific code paths that might cause this
3. Search for similar issues or patterns in the codebase
4. Check if there are existing tests that should catch this
5. Review recent changes that might have introduced this bug

Provide a detailed report with:
- Platform impact: [which platforms are affected]
- Environmental factors: [what conditions trigger this]
- Related code: [platform-specific implementations]
- Test coverage: [existing tests that relate to this]
- Recent changes: [commits that might have caused this]
- Verdict: CONFIRMED / UNCONFIRMED / NEEDS MORE INFO
```

**Wait for both agents to complete before proceeding.**

---

## PHASE 3: CRITIQUE AND ROOT CAUSE ANALYSIS

Based on the validation results, perform deep analysis:

### Step 3.1: Synthesize findings

Combine the reports from both research agents. If either agent returned UNCONFIRMED:
- Document why the bug might not exist
- Consider if the issue reporter might be experiencing something different
- Check if this was already fixed in a recent commit

### Step 3.2: Root cause critique

Answer these questions thoroughly:

1. **What exactly is happening?**
   - Describe the bug in technical terms
   - What is the expected behavior vs actual behavior?

2. **Why is it happening?**
   - What is the root cause, not just the symptom?
   - Is this a logic error, race condition, missing check, etc.?

3. **Where is it happening?**
   - Exact file(s) and line number(s)
   - Is it in frontend (Electron/React) or backend (Python)?

4. **When does it happen?**
   - What sequence of actions triggers it?
   - Is it consistent or intermittent?

5. **Who is affected?**
   - All users or specific platforms?
   - Does it depend on configuration?

6. **What could go wrong with a fix?**
   - What existing functionality might break?
   - Are there edge cases to consider?
   - What are the cross-platform implications?

### Step 3.3: Update the issue document

Add your investigation notes to `docs/issues/GITHUB_{number}.md`.

---

## PHASE 4: CREATE WORKTREE AND IMPLEMENT FIX

### Step 4.1: Create a new branch and worktree

```bash
# Create branch name from issue
BRANCH_NAME="fix/issue-{number}"

# Check current branch
git branch --show-current

# Fetch latest
git fetch origin

# Create new branch from current branch
git checkout -b $BRANCH_NAME

# Verify
git branch --show-current
```

### Step 4.2: Plan the fix

Before writing any code, create a checklist:

```markdown
## Fix Checklist

- [ ] Fix the root cause, not just the symptom
- [ ] Ensure fix works on Windows
- [ ] Ensure fix works on Linux
- [ ] Ensure fix works on macOS
- [ ] No breaking changes to existing features
- [ ] Add/update tests if applicable
- [ ] Follow existing code patterns
- [ ] Use i18n for any user-facing strings
```

### Step 4.3: Implement the fix

**CRITICAL RULES:**

1. **Minimal changes** - Only change what's necessary to fix the bug
2. **Cross-platform** - Test paths, commands, and APIs work on all OS
3. **No regressions** - Don't break existing functionality
4. **Follow patterns** - Match the existing code style
5. **Document** - Add comments if the fix is non-obvious

Common cross-platform considerations:
- Use `path.join()` instead of string concatenation for paths
- Use `process.platform` checks when needed
- Avoid shell commands that differ between OS
- Test file paths with spaces
- Consider case sensitivity (macOS/Windows vs Linux)

### Step 4.4: Verify the fix

Run relevant tests:
```bash
# Backend tests
apps/backend/.venv/bin/pytest tests/ -v -k "related_test_pattern"

# Frontend type checking
cd apps/frontend && npm run typecheck
```

### Step 4.5: Update issue document

Complete the "Fix Summary" section in `docs/issues/GITHUB_{number}.md`:

```markdown
## Fix Summary

**Branch:** fix/issue-{number}
**Files Changed:**
- {file1}: {what was changed}
- {file2}: {what was changed}

**Root Cause:** {explanation}

**Solution:** {what the fix does}

**Testing:**
- [ ] Tested on macOS
- [ ] Tested on Windows
- [ ] Tested on Linux
- [ ] Existing tests pass

**Cross-platform Notes:** {any platform-specific considerations}
```

---

## PHASE 5: FINAL VALIDATION

### Step 5.1: Review all changes

```bash
git diff --stat
git diff
```

### Step 5.2: Commit the fix

```bash
git add -A
git commit -m "$(cat <<'EOF'
fix: resolve issue #{number} - {short description}

{Longer explanation of what was fixed and why}

Fixes #{number}
EOF
)"
```

### Step 5.3: Report to user

Provide a summary:

```
## Issue Fix Complete

**Issue:** #{number} - {title}
**Branch:** fix/issue-{number}
**Status:** Ready for review

### Changes Made
{list of files and changes}

### What Was Fixed
{explanation of the fix}

### Cross-Platform Compatibility
- Windows: {status}
- Linux: {status}
- macOS: {status}

### Next Steps
1. Review the changes
2. Run `/pr` to create a pull request
3. Or run additional tests as needed
```

---

## ERROR HANDLING

### If the issue URL is invalid
- Inform the user and ask for correction

### If the issue is already closed
- Check if it was fixed, inform the user

### If validation agents cannot confirm the bug
- Document findings
- Ask user if they want to proceed anyway
- Consider if this might be user error or environmental

### If the fix introduces test failures
- Roll back and investigate
- Do not proceed with a broken fix

### If platform-specific code is needed
- Implement for all platforms, not just one
- Use proper OS detection and branching

---

## IMPORTANT NOTES

1. **Never guess** - If unsure about something, investigate first
2. **Always validate** - The research agents ensure we understand the problem
3. **Cross-platform is mandatory** - This app runs on Windows, Linux, and macOS
4. **Minimal footprint** - Only change what's needed for the fix
5. **Document everything** - Future developers need to understand this fix

User input: $ARGUMENTS
