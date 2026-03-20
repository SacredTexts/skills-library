<claude-mem-context>
# Recent Activity

### Feb 17, 2026

| ID | Time | T | Title | Read |
|----|------|---|-------|------|
| #6280 | 5:11 AM | 🔵 | Project-Level Goldy Commands Also Exist | ~250 |
</claude-mem-context>
## AI Gateway (Platform Standard)

Use the shared AI gateway at `apps/web/src/lib/ai/` for provider integrations.
If this folder is not present in the current branch yet, create it before migrating feature code.

- Keep workflow endpoints/serverFns separate, but route provider/model/env logic through the shared gateway.
- Do not read API keys directly in feature modules (`process.env.*_API_KEY`).
- Do not initialize provider SDK clients in feature modules/routes (`new Anthropic`, `GoogleGenAI`, `createOpenRouter`).
- Do not hardcode provider base URLs/headers/models in feature modules/routes.
- Use gateway abstractions and model registry keys from `apps/web/src/lib/ai/`.
- Source-of-truth migration plan: `apps/web/docs/ai-chat-api-strategy.md`.
