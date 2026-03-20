---
allowed-tools: Bash, Read, Grep, Glob
description: Commit the work you've been doing with a good commit message
argument-hint: [optional: specific files to include]
---

# COMMIT YOUR WORK

You are committing the changes from the work you've been doing in this session.

**YOU MUST TAKE ACTION. Do not just analyze or suggest - actually run the commands.**

---

## STEP 1: Review what you changed

Look at the unstaged and staged changes to understand what you worked on:

```bash
git status
```

```bash
git diff --stat
```

```bash
git diff --cached --stat
```

---

## STEP 2: Stage the changes

**Run this command NOW:**

```bash
git add -A
```

If user specified files in `$ARGUMENTS`, stage only those instead:
```bash
git add $ARGUMENTS
```

---

## STEP 3: Verify what's staged

```bash
git diff --cached --stat
```

---

## STEP 4: Generate commit message

Based on the work you did, create a commit message:

**Format:**
```
<type>(<scope>): <imperative description>

<Why this change was needed. What problem it solves.>
```

**Types:** `feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `perf`, `ci`

**Scope examples:** `ui`, `api`, `core`, `cli`, `auth`, `merge`, `deps`

**Rules:**
- First line: lowercase imperative verb (add, fix, update, remove)
- First line: max 72 characters
- Body: explain WHY, not just WHAT
- Reference issues if applicable: `Closes #123`

---

## STEP 5: COMMIT NOW

**You MUST run this command with your generated message:**

```bash
git commit -m "$(cat <<'EOF'
type(scope): your description here

Your explanation of why this change was made.
What problem does it solve?
EOF
)"
```

**Replace the placeholder text with your actual commit message.**

---

## STEP 6: Confirm success

```bash
git log --oneline -1
```

---

## IMPORTANT

1. **DO NOT just output a suggested message** - you must run `git commit`
2. **DO NOT ask for permission** - commit the changes
3. **If nothing to commit** - tell the user there are no changes
4. **If changes should be split** - ask user which to commit first, then commit that subset

User context: $ARGUMENTS