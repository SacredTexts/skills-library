---
allowed-tools: Bash, Read
description: Disable WorkOS auth bypass - restore normal login flow
---

# Auth Off

Disable the auth bypass and restore normal WorkOS authentication.

## Steps

1. Run the toggle script:

```bash
python3 apps/web/scripts/auth-toggle.py off
```

2. Show the current status:

```bash
python3 apps/web/scripts/auth-toggle.py status
```

3. Tell the user the result and remind them to restart the dev server if it's running.
