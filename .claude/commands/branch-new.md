---
allowed-tools: Bash, Read
description: Create a new git branch and switch to it properly
argument-hint: <branch-name> [optional: base-branch]
---

# CREATE NEW BRANCH

You are creating a new git branch for the user. **This is a critical workflow** - many issues occur when branches are created but not switched to.

**YOU MUST FOLLOW EVERY STEP. Do not skip any verification.**

---

## STEP 1: Parse the branch name

The user wants to create a branch named: `$ARGUMENTS`

If no branch name provided, **STOP and ask the user for a branch name**.

If a second argument is provided, use it as the base branch. Otherwise, default to the current branch.

---

## STEP 2: Check current state

**Run these commands NOW:**

```bash
git status
```

```bash
git branch --show-current
```

⚠️ **If there are uncommitted changes**, warn the user and ask if they want to:
1. Stash changes before switching
2. Commit changes first
3. Carry changes to the new branch (default for untracked files)

---

## STEP 3: Fetch latest from remote (if base is remote)

```bash
git fetch origin
```

---

## STEP 4: CREATE AND SWITCH to the new branch

**THIS IS THE CRITICAL STEP. Use `checkout -b` or `switch -c` which CREATES AND SWITCHES in one command.**

```bash
git checkout -b <branch-name>
```

Or if basing off a specific branch (e.g., `main`):

```bash
git checkout -b <branch-name> --no-track origin/main
```

**CRITICAL:** Always use `--no-track` when basing off a remote branch! Without it, git sets up tracking to the source branch (e.g., `origin/main`), and `git push` will push to THAT branch instead of creating a new remote branch.

**NEVER use just `git branch <name>`** - that creates without switching!

---

## STEP 5: VERIFY you are on the new branch

**This verification is MANDATORY. Run it NOW:**

```bash
git branch --show-current
```

**The output MUST match the new branch name.** If it doesn't, something went wrong.

---

## STEP 6: Confirm to user

Tell the user:

1. ✅ **Created branch:** `<branch-name>`
2. ✅ **Switched to:** `<branch-name>` (verified)
3. ✅ **Based on:** `<base-branch>`
4. 📝 **Next steps:** Make your changes and commit. When ready to push:
   ```bash
   git push -u origin <branch-name>
   ```

---

## COMMON PATTERNS

**Feature branch from main:**
```bash
git fetch origin
git checkout -b feature/my-feature --no-track origin/main
```

**Hotfix from main:**
```bash
git fetch origin
git checkout -b hotfix/2.7.3 --no-track origin/main
```

**Branch from current work (no --no-track needed):**
```bash
git checkout -b experiment/try-something
```

---

## ⚠️ CRITICAL RULES

1. **ALWAYS use `checkout -b` or `switch -c`** - never `git branch` alone
2. **ALWAYS use `--no-track`** when basing off a remote branch (e.g., `origin/main`, `origin/develop`)
3. **ALWAYS verify** with `git branch --show-current` after creating
4. **NEVER assume** the switch happened - verify it
5. **If verification fails**, run `git checkout <branch-name>` explicitly

---

## TROUBLESHOOTING

If the branch already exists:
```bash
git checkout <existing-branch>
```

If you need to reset and try again:
```bash
git checkout <original-branch>
git branch -d <failed-branch>  # delete local branch
git checkout -b <branch-name>
```

User arguments: $ARGUMENTS