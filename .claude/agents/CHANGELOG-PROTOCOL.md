---
name: changelog-protocol
description: Defines when and how to update CHANGELOG.md, enforcing updates only after full features are complete and tested.
---

# Changelog Protocol

Update `/CHANGELOG.md` only after **FULL features complete and tested**.

---

## When to Update

| ✅ DO Update | ❌ DON'T Update |
|--------------|-----------------|
| Full feature complete + tested | Micro-tasks, single files |
| Major features, schema changes | In-progress work |
| Infrastructure changes | Bug fixes during dev |

---

## Format

```markdown
- [Feature] ([components]) | @[git-user] [timestamp]
```

**Example**:
```markdown
### Added
- User Management (RBAC, roles, uploads) | @deploy 2025-01-22T14:32:15Z
```

---

## Commands

```bash
git config user.name                    # Get git user
date -u +"%Y-%m-%dT%H:%M:%SZ"          # Get UTC timestamp
```

---

## Rules

1. **Orchestrator** updates changelog (not coder, tester, stuck)
2. Update ONLY after tester reports success
3. Keep entries concise (2-5 word feature + 3-5 word components)
4. Always include git user + ISO timestamp
