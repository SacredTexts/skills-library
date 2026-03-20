---
allowed-tools: Bash, Read
description: Enable WorkOS auth bypass - log in as Super Admin (bodhimindflow@gmail.com)
---

# Auth On (Super Admin Bypass)

Enable the auth bypass to log in as Super Admin (bodhimindflow@gmail.com) without a real WorkOS login session. This is for local development, Playwright testing, and AI-assisted bug testing.

The bypass user's email is resolved against the Neon production database via `resolveWorkosId` + `getAuth`, so the session gets the real `dbUser` record with `super_admin` role.

## Steps

1. Run the toggle script:

```bash
python3 apps/web/scripts/auth-toggle.py on
```

2. Show the current status:

```bash
python3 apps/web/scripts/auth-toggle.py status
```

3. Tell the user the result and remind them to restart the dev server if it's running. They can also use `pnpm dev:noauth` which always enables bypass regardless of `.env.local`.
