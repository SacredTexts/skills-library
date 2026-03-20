# The Library — skills.sacred-texts.com

## Document Control

- **Product**: The Library — unified skill/agent/hook/script distribution + visual workflow builder
- **Domain**: `skills.sacred-texts.com`
- **Hosting**: Vercel (TanStack Start with Vercel adapter)
- **Database**: PlanetScale Postgres (dedicated)
- **Real-time**: Electric SQL
- **Date**: 2026-03-20
- **Status**: Planning (code in a future session in a new project folder)

## Purpose

Production application at `skills.sacred-texts.com` with a **dual-source data layer**: `library.yaml` for static catalog definitions and PlanetScale Postgres for runtime state, both feeding into TanStack DB reactive collections on the client. This gives you the simplicity and git-trackability of YAML with the real-time multi-user capabilities of Postgres — unified through live queries on the client.

**The Library** replaces the defunct Goldy Go CLI as the canonical distribution system for all agent capabilities (skills, agents, prompts, hooks, scripts, orchestrators) across team members.

### The Dual-Source Principle

**YAML = what things ARE** (skill definitions, templates, config, pipeline structure)
- Developer-authored, git-tracked, deployed with code
- Reviewed in PRs, simple to read and edit
- Instant via in-memory cache on the server
- Changes deploy with the app (or hot-reload via file watcher)

**PlanetScale Postgres = what happens at RUNTIME** (execution history, user overrides, user-created skills, feature flags, analytics, state)
- Multi-user, queryable, transactional
- Real-time sync to client via Electric SQL
- User-generated content, runtime mutations

**TanStack DB = the unifying layer on the client**
- Both sources load into separate collections
- `useLiveQuery()` JOINs across collections in sub-millisecond
- Optimistic mutations for Postgres-backed data
- The UI doesn't know or care which source a piece of data came from

---

## Key References

### TanStack Core

- **TanStack Start:** <https://tanstack.com/start/latest>
- **TanStack Start Streaming Guide:** <https://tanstack.com/start/latest/docs/framework/react/guide/streaming-data-from-server-functions>
- **Streaming Data Example Repo:** <https://github.com/TanStack/router/tree/main/examples/react/start-streaming-data-from-server-functions>
- **TanStack Start Server Functions:** <https://tanstack.com/start/latest/docs/framework/react/guide/server-functions>
- **TanStack Start Databases Guide:** <https://tanstack.com/start/latest/docs/framework/react/guide/databases>
- **TanStack Start on Cloudflare Workers:** <https://developers.cloudflare.com/workers/framework-guides/web-apps/tanstack-start/>
- **TanStack Start Code Execution Patterns:** <https://tanstack.com/start/latest/docs/framework/react/guide/code-execution-patterns>
- **TanStack Router:** <https://tanstack.com/router/latest>
- **TanStack Query:** <https://tanstack.com/query/latest>

### TanStack DB (Reactive Client Store)

- **TanStack DB Overview:** <https://tanstack.com/db/latest/docs>
- **TanStack DB Quick Start:** <https://tanstack.com/db/latest/docs/quick-start>
- **TanStack DB Installation:** <https://tanstack.com/db/latest/docs/installation>
- **TanStack DB GitHub:** <https://github.com/TanStack/db>
- **Interactive Guide (Frontend at Scale):** <https://frontendatscale.com/blog/tanstack-db/>

### TanStack Intent (Agent Skills)

- **TanStack Intent Overview:** <https://tanstack.com/intent/latest/docs/overview>
- **TanStack Intent Registry:** <https://tanstack.com/intent/registry>
- **Intent Install CLI:** <https://tanstack.com/intent/latest/docs/cli/intent-install>
- **Blog: From Docs to Agents:** <https://tanstack.com/blog/from-docs-to-agents>

### TanStack Pacer (Timing & Scheduling)

- **TanStack Pacer Overview:** <https://tanstack.com/pacer/latest>
- **Debouncing Guide:** <https://tanstack.com/pacer/latest/docs/guides/debouncing>
- **Throttling Guide:** <https://tanstack.com/pacer/latest/docs/guides/throttling>
- **Rate Limiting Guide:** <https://tanstack.com/pacer/latest/docs/guides/rate-limiting>

### Other TanStack Libraries

- **TanStack Form:** <https://tanstack.com/form/latest> — Type-safe forms with validation for skill editing UI
- **TanStack Virtual:** <https://tanstack.com/virtual/latest> — Virtualized lists for large skill/event lists
- **TanStack Store:** <https://tanstack.com/store/latest> — Framework-agnostic reactive state (used internally by Pacer)
- **TanStack CLI:** <https://github.com/TanStack/cli> — Scaffolding with add-ons

### Electric SQL (Real-Time Postgres Sync)

- **Electric SQL Docs:** <https://electric-sql.com>
- **Electric Collection (TanStack DB):** <https://tanstack.com/db/latest/docs/collections/electric-collection> — The definitive reference for `electricCollectionOptions`, proxy setup, txid handshake, `awaitMatch`, debugging
- **Agent Skills Blog Post:** <https://electric-sql.com/blog/2026/03/06/agent-skills-now-shipping>
- **@electric-sql/client Registry Skills:** <https://tanstack.com/intent/registry/%40electric-sql__client>

### PlanetScale Postgres + Drizzle ORM

- **Drizzle ORM:** <https://orm.drizzle.team>
- **Drizzle with PlanetScale:** <https://orm.drizzle.team/docs/get-started/planetscale-new>

### Registry Skills (from TanStack Intent Registry)

- **@electric-sql/client** (8 skills): <https://tanstack.com/intent/registry/%40electric-sql__client> — Real-time Postgres sync, shape streams, proxy auth, deployment
- **@tanstack/db** (6 skills): <https://tanstack.com/intent/registry/%40tanstack__db> — Collections, live queries, optimistic mutations, custom adapters, meta-framework integration
- **@tanstack/cli** (5 skills): <https://tanstack.com/intent/registry/%40tanstack__cli> — App scaffolding, add-ons, doc search
- **@durable-streams/client** (5 skills): <https://tanstack.com/intent/registry/%40durable-streams__client> — Durable stream primitives for Cloudflare
- **@durable-streams/state** (2 skills): <https://tanstack.com/intent/registry/%40durable-streams__state>

---

## Architecture Overview

```
┌──────────────────────────────────────────────────────────────────┐
│                       Client (Browser)                           │
│                                                                  │
│  TanStack DB Collections (unified reactive layer)                │
│  ┌──────────────────────┐  ┌──────────────────────────────────┐  │
│  │ skillDefsCollection  │  │ executionHistoryCollection       │  │
│  │ (from YAML via       │  │ (from Postgres via Electric SQL) │  │
│  │  server function)    │  │                                  │  │
│  │ queryCollectionOpts  │  │ electricCollectionOptions         │  │
│  └──────────┬───────────┘  └──────────┬───────────────────────┘  │
│             │    useLiveQuery() JOINs across both    │            │
│             └──────────────┬────────────────────────┘            │
│                            ▼                                     │
│  Components: SkillSearch (Pacer) | EventLog (Virtual) | etc.     │
└──────────────────────────┬───────────────────────────────────────┘
                           │
        ┌──────────────────┼──────────────────┐
        │ Server Functions │                  │ Electric SQL
        │ (TanStack Start) │                  │ (Shape Streams)
        ▼                  │                  ▼
┌───────────────────┐      │      ┌─────────────────────────────┐
│  YAML Layer       │      │      │  PlanetScale Postgres       │
│                   │      │      │                             │
│  In-Memory Cache  │      │      │  Tables:                    │
│  ← fs.stat mtime  │      │      │  - execution_history        │
│  ← fs.watch       │      │      │  - user_skill_overrides     │
│                   │      │      │  - user_created_skills      │
│  data/skills.yaml │      │      │  - execution_state          │
│  data/config.yaml │      │      │  - feature_flags            │
│                   │      │      │                             │
│  Git-tracked      │      │      │  Electric SQL sync          │
│  Deploy-time      │      │      │  Real-time, multi-user      │
└───────────────────┘      │      └─────────────────────────────┘
                           │
                    ┌──────▼──────┐
                    │  Streaming  │
                    │  Execution  │
                    │  (async gen)│
                    │  reads YAML │
                    │  writes PG  │
                    └─────────────┘
```

---

## Project Setup

### Scaffold with TanStack CLI

```bash
# Create a new TanStack Start app with useful add-ons
npx @tanstack/cli create skills-app --add-ons tanstack-query

# Install agent skills for all TanStack dependencies
npx @tanstack/intent@latest install
```

### Dependencies

```json
{
  "dependencies": {
    "@tanstack/react-start": "latest",
    "@tanstack/react-router": "latest",
    "@tanstack/react-query": "latest",
    "@tanstack/react-db": "latest",
    "@tanstack/query-db-collection": "latest",
    "@tanstack/electric-db-collection": "latest",
    "@electric-sql/client": "latest",
    "@tanstack/react-pacer": "latest",
    "@tanstack/react-virtual": "latest",
    "@tanstack/react-form": "latest",
    "@tanstack/store": "latest",
    "drizzle-orm": "latest",
    "js-yaml": "^4.1.0",
    "zod": "^3.23.0"
  },
  "devDependencies": {
    "@types/js-yaml": "^4.0.9",
    "@tanstack/intent": "latest",
    "drizzle-kit": "latest"
  }
}
```

### Agent Skills Setup (CLAUDE.md / AGENTS.md)

After installing dependencies, run the Intent CLI to wire skills into your agent config:

```bash
# List all intent-enabled packages in your deps
npx @tanstack/intent list

# Generate skill mappings for your agent config
npx @tanstack/intent@latest install
```

This produces a block like:

```markdown
<!-- intent-skills:start -->
# Skill mappings — when working in these areas, load the linked skill file into context.
skills:
  - task: "TanStack DB collections, live queries, mutations"
    load: "node_modules/@tanstack/db/skills/db-core/SKILL.md"
  - task: "TanStack DB meta-framework integration (SSR, preloading)"
    load: "node_modules/@tanstack/db/skills/meta-framework/SKILL.md"
<!-- intent-skills:end -->
```

---

## Part 1: YAML Schema and Types

### Example YAML: `data/skills.yaml`

```yaml
skills:
  - id: git-commit
    name: "Git Commit Helper"
    description: "Generates conventional commit messages"
    version: "1.0.0"
    enabled: true
    steps:
      - name: analyze-diff
        action: shell
        command: "git diff --staged"
        timeout: 5000
      - name: generate-message
        action: llm
        prompt: "Generate a conventional commit message for this diff: {{analyze-diff.output}}"
        model: "claude-sonnet-4-20250514"
      - name: confirm
        action: user-confirm
        message: "Use this commit message?"

  - id: code-review
    name: "Code Review"
    description: "Reviews code changes and suggests improvements"
    version: "1.2.0"
    enabled: true
    steps:
      - name: get-changes
        action: shell
        command: "git diff HEAD~1"
        timeout: 10000
      - name: review
        action: llm
        prompt: "Review these changes for bugs, style issues, and improvements: {{get-changes.output}}"
        model: "claude-sonnet-4-20250514"
      - name: report
        action: output
        format: markdown
```

### Zod Schema and Types: `lib/types.ts`

```typescript
import { z } from 'zod'

export const StepSchema = z.object({
  name: z.string(),
  action: z.enum(['shell', 'llm', 'user-confirm', 'output', 'fetch', 'transform']),
  command: z.string().optional(),
  prompt: z.string().optional(),
  model: z.string().optional(),
  timeout: z.number().optional(),
  message: z.string().optional(),
  format: z.string().optional(),
})

export const SkillSchema = z.object({
  id: z.string(),
  name: z.string(),
  description: z.string(),
  version: z.string(),
  enabled: z.boolean().default(true),
  steps: z.array(StepSchema),
})

export const SkillsFileSchema = z.object({
  skills: z.array(SkillSchema),
})

export type Step = z.infer<typeof StepSchema>
export type Skill = z.infer<typeof SkillSchema>
export type SkillsFile = z.infer<typeof SkillsFileSchema>

// Event types streamed to the client during skill execution
export type SkillEvent =
  | { type: 'started'; skillName: string; totalSteps: number }
  | { type: 'step_started'; stepName: string; stepIndex: number }
  | { type: 'step_output'; stepName: string; output: string }
  | { type: 'step_complete'; stepName: string; durationMs: number }
  | { type: 'awaiting_confirm'; stepName: string; message: string }
  | { type: 'error'; stepName: string; error: string }
  | { type: 'complete'; totalDurationMs: number }

// ─── Postgres-backed types (runtime data) ───

export const ExecutionRecordSchema = z.object({
  id: z.string().uuid(),
  skill_id: z.string(),
  user_id: z.string(),
  status: z.enum(['running', 'completed', 'failed', 'cancelled']),
  duration_ms: z.number().nullable(),
  executed_at: z.string().datetime(),
  step_results: z.record(z.unknown()).nullable(),
  error: z.string().nullable(),
})

export const UserOverrideSchema = z.object({
  skill_id: z.string(),
  user_id: z.string(),
  enabled: z.boolean().nullable(),
  custom_params: z.record(z.unknown()).nullable(),
  notes: z.string().nullable(),
  updated_at: z.string().datetime(),
})

export type ExecutionRecord = z.infer<typeof ExecutionRecordSchema>
export type UserOverride = z.infer<typeof UserOverrideSchema>
```

---

## Part 2: In-Memory YAML Cache with File-Watch Invalidation

### `lib/yaml-store.ts`

```typescript
import fs from 'fs'
import path from 'path'
import yaml from 'js-yaml'
import { SkillsFileSchema, type Skill, type SkillsFile } from './types'

const SKILLS_PATH = path.resolve('data/skills.yaml')

let cache: SkillsFile | null = null
let lastModified = 0

/**
 * Load skills from YAML with in-memory caching.
 * Uses fs.statSync mtime as the invalidation gate.
 * After first load, this is a pure memory read (~0ms)
 * unless the file has been modified.
 */
export function loadSkills(): SkillsFile {
  const stat = fs.statSync(SKILLS_PATH)

  if (cache && stat.mtimeMs <= lastModified) {
    return cache
  }

  const raw = fs.readFileSync(SKILLS_PATH, 'utf-8')
  const parsed = yaml.load(raw)
  const validated = SkillsFileSchema.parse(parsed)

  cache = validated
  lastModified = stat.mtimeMs

  console.log(`[yaml-store] Loaded ${validated.skills.length} skills from YAML`)
  return validated
}

export function getSkillById(id: string): Skill | undefined {
  const { skills } = loadSkills()
  return skills.find((s) => s.id === id)
}

export function getEnabledSkills(): Skill[] {
  const { skills } = loadSkills()
  return skills.filter((s) => s.enabled)
}

export function invalidateCache(): void {
  cache = null
  lastModified = 0
  console.log('[yaml-store] Cache invalidated')
}
```

### Optional: File Watcher for Auto-Invalidation (Fly.io / Hetzner only)

```typescript
// lib/yaml-watcher.ts
// Only use on persistent server processes. NOT for Cloudflare Workers.

import fs from 'fs'
import path from 'path'
import { invalidateCache } from './yaml-store'

const SKILLS_PATH = path.resolve('data/skills.yaml')
let watcher: fs.FSWatcher | null = null

export function startWatching(): void {
  if (watcher) return

  watcher = fs.watch(SKILLS_PATH, (eventType) => {
    if (eventType === 'change') {
      console.log('[yaml-watcher] skills.yaml changed, invalidating cache')
      invalidateCache()
    }
  })
}

export function stopWatching(): void {
  watcher?.close()
  watcher = null
}
```

---

## Part 3: Server Functions (Instant Reads)

### `server/skills.ts`

```typescript
import { createServerFn } from '@tanstack/react-start'
import { setResponseHeaders } from '@tanstack/react-start/server'
import { z } from 'zod'
import { loadSkills, getSkillById, getEnabledSkills, invalidateCache } from '../lib/yaml-store'
import type { Skill } from '../lib/types'

/**
 * GET all enabled skills — instant from memory cache.
 * Use in route loaders for instant page loads.
 * Adds Cache-Control headers for CDN layer caching.
 */
export const fetchSkills = createServerFn({ method: 'GET' })
  .handler(async (): Promise<Skill[]> => {
    setResponseHeaders(
      new Headers({
        'Cache-Control': 'public, max-age=10, s-maxage=60, stale-while-revalidate=300',
      })
    )
    return getEnabledSkills()
  })

/**
 * GET a single skill by ID.
 */
export const fetchSkill = createServerFn({ method: 'GET' })
  .inputValidator(z.object({ id: z.string() }))
  .handler(async ({ data }): Promise<Skill | null> => {
    return getSkillById(data.id) ?? null
  })

/**
 * POST to force cache invalidation.
 * Call from a git webhook or admin UI.
 */
export const reloadSkills = createServerFn({ method: 'POST' })
  .handler(async () => {
    invalidateCache()
    const fresh = loadSkills()
    return { reloaded: true, count: fresh.skills.length }
  })
```

---

## Part 4: Streaming Server Functions (Real-Time Execution)

This is the key pattern from the TanStack Start streaming docs. Server functions can return async generators, and each `yield` sends a typed chunk to the client.

### `server/execute-skill.ts`

```typescript
import { createServerFn } from '@tanstack/react-start'
import { z } from 'zod'
import { getSkillById } from '../lib/yaml-store'
import type { SkillEvent, Step } from '../lib/types'

/**
 * Execute a skill and stream progress events to the client.
 *
 * Uses TanStack Start's async generator pattern:
 * https://tanstack.com/start/latest/docs/framework/react/guide/streaming-data-from-server-functions
 *
 * Each `yield` sends a typed SkillEvent chunk over the wire.
 * The client consumes with `for await (const event of stream)`.
 */
export const executeSkill = createServerFn()
  .inputValidator(
    z.object({
      skillId: z.string(),
      params: z.record(z.unknown()).default({}),
    })
  )
  .handler(async function* ({ data }): AsyncGenerator<SkillEvent> {
    const skill = getSkillById(data.skillId)

    if (!skill) {
      yield { type: 'error', stepName: 'init', error: `Skill "${data.skillId}" not found` }
      return
    }

    const startTime = performance.now()

    yield {
      type: 'started',
      skillName: skill.name,
      totalSteps: skill.steps.length,
    }

    const stepOutputs: Record<string, string> = {}

    for (let i = 0; i < skill.steps.length; i++) {
      const step = skill.steps[i]
      const stepStart = performance.now()

      yield { type: 'step_started', stepName: step.name, stepIndex: i }

      try {
        const output = await runStep(step, stepOutputs, data.params)
        stepOutputs[step.name] = output

        yield { type: 'step_output', stepName: step.name, output }

        const durationMs = Math.round(performance.now() - stepStart)
        yield { type: 'step_complete', stepName: step.name, durationMs }
      } catch (err) {
        yield {
          type: 'error',
          stepName: step.name,
          error: err instanceof Error ? err.message : String(err),
        }
        return
      }
    }

    yield {
      type: 'complete',
      totalDurationMs: Math.round(performance.now() - startTime),
    }
  })

/**
 * Execute a single step. Replace simulated delays with real
 * shell execution, LLM calls, etc. in production.
 */
async function runStep(
  step: Step,
  previousOutputs: Record<string, string>,
  params: Record<string, unknown>
): Promise<string> {
  const interpolate = (template: string): string => {
    return template.replace(/\{\{(\S+?)\.output\}\}/g, (_, ref) => {
      return previousOutputs[ref] ?? `[missing: ${ref}]`
    })
  }

  switch (step.action) {
    case 'shell': {
      const command = interpolate(step.command ?? '')
      // TODO: Replace with actual child_process.exec
      await simulateDelay(step.timeout ?? 2000)
      return `[shell output for: ${command}]`
    }
    case 'llm': {
      const prompt = interpolate(step.prompt ?? '')
      // TODO: Replace with actual LLM API call
      await simulateDelay(1500)
      return `[LLM response for: ${prompt.slice(0, 80)}...]`
    }
    case 'user-confirm': {
      await simulateDelay(500)
      return 'confirmed'
    }
    case 'output': {
      return Object.entries(previousOutputs)
        .map(([k, v]) => `## ${k}\n${v}`)
        .join('\n\n')
    }
    default:
      return `[unknown action: ${step.action}]`
  }
}

function simulateDelay(ms: number): Promise<void> {
  return new Promise((resolve) => setTimeout(resolve, Math.min(ms, 3000)))
}
```

### Alternative: ReadableStream Pattern

Both async generators and ReadableStream work in TanStack Start:

```typescript
export const executeSkillStream = createServerFn()
  .inputValidator(z.object({ skillId: z.string() }))
  .handler(async ({ data }) => {
    const skill = getSkillById(data.skillId)

    return new ReadableStream<SkillEvent>({
      async start(controller) {
        if (!skill) {
          controller.enqueue({ type: 'error', stepName: 'init', error: 'Skill not found' })
          controller.close()
          return
        }

        controller.enqueue({
          type: 'started',
          skillName: skill.name,
          totalSteps: skill.steps.length,
        })

        // ... run steps, enqueue events ...

        controller.enqueue({ type: 'complete', totalDurationMs: 1000 })
        controller.close()
      },
    })
  })
```

---

## Part 5: TanStack DB — Dual-Source Reactive Collections

TanStack DB is a reactive client store that extends TanStack Query with collections, live queries, and optimistic mutations. The key insight: **you can have multiple collections from different sources and JOIN them with live queries.** YAML-backed and Postgres-backed data unify on the client.

**Key concept from the @tanstack/db meta-framework skill:** SSR is NOT supported for TanStack DB — routes using DB must disable SSR and preload collections in the route loader.

### `lib/collections.ts` — Both Sources

```typescript
import { createCollection, eq } from '@tanstack/react-db'
import { queryCollectionOptions } from '@tanstack/query-db-collection'
import { electricCollectionOptions } from '@tanstack/electric-db-collection'
import { fetchSkills } from '../server/skills'
import type { Skill } from './types'

// ─────────────────────────────────────────────────────────
// YAML-BACKED COLLECTION: Skill Definitions
// Source: data/skills.yaml → in-memory cache → server function
// These are developer-authored, git-tracked, deploy-time.
// ─────────────────────────────────────────────────────────
export const skillDefsCollection = createCollection(
  queryCollectionOptions({
    queryKey: ['skill-definitions'],
    queryFn: async () => {
      // Calls TanStack Start server function → reads from YAML cache
      return await fetchSkills()
    },
    getKey: (skill: Skill) => skill.id,
    // No onUpdate/onDelete — YAML definitions are read-only on the client.
    // Edit YAML in your editor, commit, deploy.
  })
)

// ─────────────────────────────────────────────────────────
// POSTGRES-BACKED COLLECTION: Execution History
// Source: PlanetScale Postgres → Electric SQL → real-time sync
// These are runtime, multi-user, written by the execution engine.
//
// shapeOptions.url points to the PROXY ROUTE, not directly to Electric.
// Electric is never exposed to the client.
// ─────────────────────────────────────────────────────────
export const executionHistoryCollection = createCollection(
  electricCollectionOptions({
    id: 'execution-history',
    shapeOptions: {
      url: '/api/execution-history',  // Proxy route, NOT direct Electric URL
    },
    getKey: (record: ExecutionRecord) => record.id,
    schema: ExecutionRecordSchema,
    // Read-only on the client — writes happen via server functions
    // which return txids that Electric syncs automatically
  })
)

// ─────────────────────────────────────────────────────────
// POSTGRES-BACKED COLLECTION: User Skill Overrides
// Users can override YAML-defined skill settings (e.g. disable,
// change default params, pin a version). These layer on top of
// the YAML definitions via a live query JOIN.
// ─────────────────────────────────────────────────────────
export const userOverridesCollection = createCollection(
  electricCollectionOptions({
    id: 'user-overrides',
    shapeOptions: {
      url: '/api/user-overrides',  // Proxy route
    },
    getKey: (override: UserOverride) => override.skill_id,
    schema: UserOverrideSchema,

    // Optimistic mutations — instant UI, synced to Postgres via txid
    onUpdate: async ({ transaction }) => {
      const { original, modified } = transaction.mutations[0]
      const res = await fetch(`/api/overrides/${original.skill_id}`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(modified),
      })
      const { txid } = await res.json()
      // Return txid so TanStack DB waits for Electric to confirm sync
      // This prevents the optimistic update from flickering
      return { txid }
    },
  })
)

// ─────────────────────────────────────────────────────────
// POSTGRES-BACKED COLLECTION: User-Created Skills
// Skills that users create at runtime (not in YAML).
// Same Skill shape, different source.
// ─────────────────────────────────────────────────────────
export const userSkillsCollection = createCollection(
  electricCollectionOptions({
    id: 'user-skills',
    shapeOptions: {
      url: '/api/user-skills',  // Proxy route
    },
    getKey: (skill: Skill) => skill.id,
    schema: SkillSchema,

    onInsert: async ({ transaction }) => {
      const newItem = transaction.mutations[0].modified
      const res = await fetch('/api/skills', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(newItem),
      })
      const { txid } = await res.json()
      return { txid }
    },

    onUpdate: async ({ transaction }) => {
      const { original, changes } = transaction.mutations[0]
      const res = await fetch(`/api/skills/${original.id}`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(changes),
      })
      const { txid } = await res.json()
      return { txid }
    },

    onDelete: async ({ transaction }) => {
      const { original } = transaction.mutations[0]
      const res = await fetch(`/api/skills/${original.id}`, {
        method: 'DELETE',
      })
      const { txid } = await res.json()
      return { txid }
    },
  })
)
```

### Cross-Collection Live Queries (The Killer Feature)

This is where the dual-source architecture pays off. TanStack DB's live queries can JOIN across collections from completely different backends:

```typescript
import { useLiveQuery, from, leftJoin, eq, fn } from '@tanstack/react-db'
import {
  skillDefsCollection,
  executionHistoryCollection,
  userOverridesCollection,
  userSkillsCollection,
} from '../lib/collections'

/**
 * Unified skill list: YAML definitions + user overrides + execution stats.
 * The UI gets a single reactive view — it doesn't know YAML from Postgres.
 */
function SkillDashboard() {
  // JOIN YAML skill definitions with Postgres user overrides
  const { data: skills } = useLiveQuery(
    from(skillDefsCollection)
      .leftJoin(
        userOverridesCollection,
        eq(skillDefsCollection, 'id', userOverridesCollection, 'skill_id')
      )
      .select({
        id: skillDefsCollection.id,
        name: skillDefsCollection.name,
        description: skillDefsCollection.description,
        version: skillDefsCollection.version,
        // User override wins if it exists, otherwise YAML default
        enabled: fn.coalesce(userOverridesCollection.enabled, skillDefsCollection.enabled),
        userNotes: userOverridesCollection.notes,
      })
      .orderBy('name', 'asc')
  )

  return (
    <ul>
      {skills.map((skill) => (
        <li key={skill.id}>
          {skill.name} v{skill.version}
          {skill.userNotes && <span> — {skill.userNotes}</span>}
          {!skill.enabled && <span> (disabled)</span>}
        </li>
      ))}
    </ul>
  )
}

/**
 * Skill detail with execution history from Postgres.
 * YAML tells us what the skill IS. Postgres tells us how it's been USED.
 */
function SkillWithHistory({ skillId }: { skillId: string }) {
  // Recent executions for this skill (Postgres, real-time)
  const { data: history } = useLiveQuery(
    from(executionHistoryCollection)
      .where(eq('skill_id', skillId))
      .orderBy('executed_at', 'desc')
      .limit(20)
  )

  // Aggregate stats across all executions
  const { data: stats } = useLiveQuery(
    from(executionHistoryCollection)
      .where(eq('skill_id', skillId))
      .select({
        totalRuns: fn.count(),
        avgDuration: fn.avg('duration_ms'),
        lastRun: fn.max('executed_at'),
      })
  )

  return (
    <div>
      <h3>Execution History</h3>
      <p>Total runs: {stats[0]?.totalRuns} | Avg: {stats[0]?.avgDuration}ms</p>
      <ul>
        {history.map((run) => (
          <li key={run.id}>
            {run.status} — {run.duration_ms}ms — {run.executed_at}
          </li>
        ))}
      </ul>
    </div>
  )
}

/**
 * Combined view: YAML-defined skills + user-created skills.
 * Two different collections, same type, unified in one query.
 */
function AllSkills() {
  const { data: yamlSkills } = useLiveQuery(
    from(skillDefsCollection).where(eq('enabled', true))
  )

  const { data: userSkills } = useLiveQuery(
    from(userSkillsCollection).where(eq('enabled', true))
  )

  // Combine both sources into one list
  const allSkills = [...yamlSkills, ...userSkills].sort((a, b) =>
    a.name.localeCompare(b.name)
  )

  return (
    <ul>
      {allSkills.map((skill) => (
        <li key={skill.id}>
          {skill.name}
          {userSkills.some((s) => s.id === skill.id) && <span> (custom)</span>}
        </li>
      ))}
    </ul>
  )
}
```

### Route with SSR Disabled + Collection Preload

```typescript
// routes/skills.tsx
import { createFileRoute } from '@tanstack/react-router'
import {
  skillDefsCollection,
  executionHistoryCollection,
  userOverridesCollection,
} from '../lib/collections'

export const Route = createFileRoute('/skills')({
  // TanStack DB requires SSR disabled per the meta-framework skill
  ssr: false,

  loader: async () => {
    // Preload all collections in parallel so data is ready at mount
    await Promise.all([
      skillDefsCollection.preload(),
      executionHistoryCollection.preload(),
      userOverridesCollection.preload(),
    ])
    return {}
  },

  component: SkillsPage,
})
```

---

## Part 6: TanStack Pacer — Timing Control

Use Pacer for debounced search, throttled event streaming, and rate-limited API calls.

### Debounced Skill Search

```typescript
import { useState, useCallback } from 'react'
import { debounce } from '@tanstack/react-pacer/debouncer'
import { useLiveQuery, from, like } from '@tanstack/react-db'
import { skillsCollection } from '../lib/collections'

function SkillSearch() {
  const [searchTerm, setSearchTerm] = useState('')
  const [debouncedTerm, setDebouncedTerm] = useState('')

  // Debounce search input — only query after 300ms of inactivity
  const debouncedSetTerm = useCallback(
    debounce(setDebouncedTerm, { wait: 300 }),
    []
  )

  const { data: results } = useLiveQuery(
    from(skillsCollection)
      .where(like('name', `%${debouncedTerm}%`))
  )

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const value = e.target.value
    setSearchTerm(value)       // Instant UI update
    debouncedSetTerm(value)    // Debounced query update
  }

  return (
    <div>
      <input value={searchTerm} onChange={handleChange} placeholder="Search skills..." />
      <ul>
        {results.map((skill) => (
          <li key={skill.id}>{skill.name}</li>
        ))}
      </ul>
    </div>
  )
}
```

### Throttled Execution Event Handler

```typescript
import { throttle } from '@tanstack/react-pacer'

// Throttle event logging to max once per 200ms during streaming execution
const throttledLogEvent = throttle(
  (event: SkillEvent) => {
    analytics.track('skill_event', event)
  },
  { wait: 200 }
)
```

---

## Part 7: TanStack Virtual — Virtualized Event Lists

For long execution logs or large skill lists, use TanStack Virtual for smooth scrolling:

```typescript
import { useVirtualizer } from '@tanstack/react-virtual'
import { useRef } from 'react'
import type { SkillEvent } from '../lib/types'

function VirtualizedEventLog({ events }: { events: SkillEvent[] }) {
  const parentRef = useRef<HTMLDivElement>(null)

  const virtualizer = useVirtualizer({
    count: events.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 40,
    overscan: 5,
  })

  return (
    <div ref={parentRef} style={{ height: '400px', overflow: 'auto' }}>
      <div style={{ height: `${virtualizer.getTotalSize()}px`, position: 'relative' }}>
        {virtualizer.getVirtualItems().map((virtualItem) => (
          <div
            key={virtualItem.key}
            style={{
              position: 'absolute',
              top: 0,
              left: 0,
              width: '100%',
              height: `${virtualItem.size}px`,
              transform: `translateY(${virtualItem.start}px)`,
            }}
          >
            <EventDisplay event={events[virtualItem.index]} />
          </div>
        ))}
      </div>
    </div>
  )
}
```

---

## Part 8: TanStack Form — Skill Editing

Use TanStack Form for type-safe, validated skill editing:

```typescript
import { useForm } from '@tanstack/react-form'
import { z } from 'zod'
import { SkillSchema } from '../lib/types'
import { skillsCollection } from '../lib/collections'

function SkillEditor({ skill }: { skill: Skill }) {
  const form = useForm({
    defaultValues: skill,
    onSubmit: async ({ value }) => {
      // Optimistic update via TanStack DB
      skillsCollection.update(skill.id, (draft) => {
        Object.assign(draft, value)
      })
    },
  })

  return (
    <form
      onSubmit={(e) => {
        e.preventDefault()
        form.handleSubmit()
      }}
    >
      <form.Field
        name="name"
        children={(field) => (
          <div>
            <label>Name</label>
            <input
              value={field.state.value}
              onChange={(e) => field.handleChange(e.target.value)}
            />
          </div>
        )}
      />
      <form.Field
        name="description"
        children={(field) => (
          <div>
            <label>Description</label>
            <textarea
              value={field.state.value}
              onChange={(e) => field.handleChange(e.target.value)}
            />
          </div>
        )}
      />
      <button type="submit">Save</button>
    </form>
  )
}
```

---

## Part 9: Skill Detail Page with Streaming Execution

### `routes/skills.$skillId.tsx`

```typescript
import { createFileRoute } from '@tanstack/react-router'
import { useState } from 'react'
import { fetchSkill } from '../server/skills'
import { executeSkill } from '../server/execute-skill'
import type { SkillEvent } from '../lib/types'

export const Route = createFileRoute('/skills/$skillId')({
  loader: async ({ params }) => {
    const skill = await fetchSkill({ data: { id: params.skillId } })
    if (!skill) throw new Error('Skill not found')
    return { skill }
  },
  component: SkillDetailPage,
})

function SkillDetailPage() {
  const { skill } = Route.useLoaderData()
  const [events, setEvents] = useState<SkillEvent[]>([])
  const [isRunning, setIsRunning] = useState(false)

  const run = async () => {
    setEvents([])
    setIsRunning(true)

    try {
      // executeSkill returns an async iterable (from the async generator)
      const stream = await executeSkill({
        data: { skillId: skill.id, params: {} },
      })

      // Each iteration yields a typed SkillEvent
      for await (const event of stream) {
        setEvents((prev) => [...prev, event])
      }
    } catch (err) {
      setEvents((prev) => [
        ...prev,
        { type: 'error', stepName: 'client', error: String(err) },
      ])
    } finally {
      setIsRunning(false)
    }
  }

  return (
    <div>
      <h1>{skill.name}</h1>
      <p>{skill.description}</p>
      <p>Version: {skill.version}</p>

      <h2>Steps</h2>
      <ol>
        {skill.steps.map((step) => (
          <li key={step.name}>
            <strong>{step.name}</strong> ({step.action})
          </li>
        ))}
      </ol>

      <button onClick={run} disabled={isRunning}>
        {isRunning ? 'Running...' : 'Execute Skill'}
      </button>

      {/* Use VirtualizedEventLog from Part 7 for long event lists */}
      {events.length > 0 && <VirtualizedEventLog events={events} />}
    </div>
  )
}
```

---

## Part 10: Admin Reload Endpoint

For invalidating the cache after deploying new YAML (git push → webhook → this endpoint):

```typescript
// routes/api/reload-skills.ts
import { json } from '@tanstack/react-start'
import { reloadSkills } from '../../server/skills'

export const APIRoute = createAPIFileRoute('/api/reload-skills')({
  POST: async ({ request }) => {
    const secret = request.headers.get('x-webhook-secret')
    if (secret !== process.env.WEBHOOK_SECRET) {
      return json({ error: 'unauthorized' }, { status: 401 })
    }

    const result = await reloadSkills()
    return json(result)
  },
})
```

---

## File Structure

```
project/
├── data/
│   ├── skills.yaml              # YAML source of truth (skill definitions)
│   └── config.yaml              # App config, feature flags
├── lib/
│   ├── types.ts                 # Zod schemas for YAML + Postgres types
│   ├── yaml-store.ts            # In-memory YAML cache with mtime invalidation
│   ├── yaml-watcher.ts          # Optional fs.watch (Fly.io/Hetzner only)
│   ├── collections.ts           # TanStack DB collections (YAML + Postgres)
│   ├── planetscale.ts           # Postgres client for PlanetScale
│   └── txid.ts                  # generateTxId helper (MUST be inside transaction)
├── db/
│   ├── schema.ts                # Drizzle schema (execution_history, overrides, etc.)
│   └── migrations/              # Drizzle Kit migrations
├── server/
│   ├── skills.ts                # GET server functions (YAML → instant reads)
│   ├── execute-skill.ts         # Streaming server function (reads YAML, writes PG)
│   └── persist-execution.ts     # Write execution results to Postgres with txid
├── routes/
│   ├── skills.tsx               # Skills list (DB live queries across both sources)
│   ├── skills.$skillId.tsx      # Skill detail + streaming execution
│   └── api/
│       ├── reload-skills.ts     # Webhook endpoint for YAML cache busting
│       ├── execution-history.ts # Electric proxy: execution_history table
│       ├── user-overrides.ts    # Electric proxy: user_skill_overrides table
│       └── user-skills.ts       # Electric proxy: user_skills table
├── components/
│   ├── SkillSearch.tsx           # Debounced search (Pacer + DB live query)
│   ├── SkillDashboard.tsx        # Cross-collection JOINed view
│   ├── VirtualizedEventLog.tsx   # Virtualized event list (Virtual)
│   └── SkillEditor.tsx           # Form-based skill editing (Form)
├── AGENTS.md                    # Intent skill mappings (auto-generated)
├── app.config.ts
└── package.json
```

---

## Dual-Source Data Flow: What Lives Where

### Decision Guide

| Data | Source | Why |
|------|--------|-----|
| Skill definitions (name, steps, version) | **YAML** | Developer-authored, reviewed in PRs, deployed with code |
| Skill pipeline config (prompts, commands) | **YAML** | Same — these are code, not user data |
| App config, feature flags | **YAML** | Simple, git-tracked, environment-specific |
| Execution history / logs | **Postgres** | Runtime, append-only, queryable, multi-user |
| User skill overrides (disable, custom params) | **Postgres** | Per-user, mutable, real-time sync |
| User-created skills | **Postgres** | User-generated content, CRUD |
| User preferences / settings | **Postgres** | Per-user, mutable |
| Analytics / metrics | **Postgres** | Aggregatable, time-series |

### PlanetScale Postgres Schema

```sql
-- Execution history: written by the streaming execution engine
CREATE TABLE execution_history (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  skill_id TEXT NOT NULL,          -- References YAML skill ID
  user_id TEXT NOT NULL,
  status TEXT NOT NULL DEFAULT 'running',  -- running | completed | failed | cancelled
  duration_ms INTEGER,
  executed_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  step_results JSONB,              -- Output from each step
  error TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- IMPORTANT for Electric SQL: set REPLICA IDENTITY FULL on all synced tables
ALTER TABLE execution_history REPLICA IDENTITY FULL;

-- User overrides: layer on top of YAML skill definitions
CREATE TABLE user_skill_overrides (
  skill_id TEXT NOT NULL,          -- References YAML skill ID
  user_id TEXT NOT NULL,
  enabled BOOLEAN,                 -- NULL = use YAML default
  custom_params JSONB,             -- Override default step params
  notes TEXT,                      -- User's notes on the skill
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  PRIMARY KEY (skill_id, user_id)
);

ALTER TABLE user_skill_overrides REPLICA IDENTITY FULL;

-- User-created skills: same shape as YAML skills, but user-generated
CREATE TABLE user_skills (
  id TEXT PRIMARY KEY,
  user_id TEXT NOT NULL,
  name TEXT NOT NULL,
  description TEXT NOT NULL DEFAULT '',
  version TEXT NOT NULL DEFAULT '0.1.0',
  enabled BOOLEAN NOT NULL DEFAULT true,
  steps JSONB NOT NULL DEFAULT '[]',  -- Array of Step objects
  created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

ALTER TABLE user_skills REPLICA IDENTITY FULL;

-- Indexes for common queries
CREATE INDEX idx_exec_history_skill ON execution_history (skill_id, executed_at DESC);
CREATE INDEX idx_exec_history_user ON execution_history (user_id, executed_at DESC);
CREATE INDEX idx_user_overrides_user ON user_skill_overrides (user_id);
CREATE INDEX idx_user_skills_user ON user_skills (user_id);
```

### Writing Execution Results to Postgres (from Streaming Server Function)

The streaming execution engine reads skill definitions from YAML but writes results to Postgres. **Critical: `pg_current_xact_id()` MUST be called inside the same transaction as the mutation**, otherwise the txid won't match and Electric sync will stall.

```typescript
// lib/txid.ts — Reusable txid helper
// The ::xid cast strips off the epoch, giving you the raw 32-bit value
// that matches what PostgreSQL sends in logical replication streams
// (and then exposed through Electric which we'll match against
// in the client).
export async function generateTxId(tx: any): Promise<number> {
  const result = await tx`SELECT pg_current_xact_id()::xid::text as txid`
  const txid = result[0]?.txid

  if (txid === undefined) {
    throw new Error('Failed to get transaction ID')
  }

  return parseInt(txid, 10)
}
```

```typescript
// server/persist-execution.ts — Write execution results to Postgres
import { sql } from '../lib/planetscale'
import { generateTxId } from '../lib/txid'

/**
 * Persist an execution record and return the txid for Electric sync.
 *
 * ✅ CORRECT: txid queried INSIDE the transaction
 * ❌ WRONG: querying txid outside the transaction (will never match!)
 *
 * See: https://tanstack.com/db/latest/docs — Electric Collection debugging section
 */
export async function persistExecution(params: {
  skillId: string
  userId: string
  status: string
  durationMs: number
  stepResults: Record<string, unknown>
}) {
  let txid!: number

  const result = await sql.begin(async (tx) => {
    // MUST be inside the transaction
    txid = await generateTxId(tx)

    const [record] = await tx`
      INSERT INTO execution_history (skill_id, user_id, status, duration_ms, step_results)
      VALUES (${params.skillId}, ${params.userId}, ${params.status}, ${params.durationMs}, ${JSON.stringify(params.stepResults)})
      RETURNING *
    `
    return record
  })

  return { record: result, txid }
}
```

Then in the streaming execution handler, call this after the final yield:

```typescript
// Inside executeSkill handler, after yielding 'complete':
yield { type: 'complete', totalDurationMs }

// Write to Postgres — the Electric-synced executionHistoryCollection
// on all connected clients will update automatically
await persistExecution({
  skillId: data.skillId,
  userId: 'current-user',  // from auth context
  status: 'completed',
  durationMs: totalDurationMs,
  stepResults: stepOutputs,
})
```

### Electric SQL Proxy Routes (TanStack Start)

Electric is deployed behind proxy routes that handle shape configuration, auth, and security. Each synced table gets its own proxy route. Uses `createServerFileRoute` from TanStack Start:

```typescript
// routes/api/execution-history.ts
import { createServerFileRoute } from '@tanstack/react-start/server'
import { ELECTRIC_PROTOCOL_QUERY_PARAMS } from '@electric-sql/client'

// Electric service URL (server-side only, never exposed to client)
const ELECTRIC_BASE_URL = process.env.ELECTRIC_URL + '/v1/shape'

const serve = async ({ request }: { request: Request }) => {
  // TODO: Check user authorization here
  // const session = await getSession(request)
  // if (!session) return new Response('Unauthorized', { status: 401 })

  const url = new URL(request.url)
  const originUrl = new URL(ELECTRIC_BASE_URL)

  // Passthrough Electric protocol params (offset, handle, etc.)
  url.searchParams.forEach((value, key) => {
    if (ELECTRIC_PROTOCOL_QUERY_PARAMS.includes(key)) {
      originUrl.searchParams.set(key, value)
    }
  })

  // Set shape parameters — table and optional filtering
  originUrl.searchParams.set('table', 'execution_history')
  // Optional: filter by user for tenant isolation
  // originUrl.searchParams.set('where', `user_id = '${session.userId}'`)

  const response = await fetch(originUrl)

  // Clean up headers that break streaming through proxies
  const headers = new Headers(response.headers)
  headers.delete('content-encoding')
  headers.delete('content-length')

  return new Response(response.body, {
    status: response.status,
    statusText: response.statusText,
    headers,
  })
}

export const ServerRoute = createServerFileRoute('/api/execution-history').methods({
  GET: serve,
})
```

```typescript
// routes/api/user-overrides.ts — Same pattern, different table
import { createServerFileRoute } from '@tanstack/react-start/server'
import { ELECTRIC_PROTOCOL_QUERY_PARAMS } from '@electric-sql/client'

const ELECTRIC_BASE_URL = process.env.ELECTRIC_URL + '/v1/shape'

const serve = async ({ request }: { request: Request }) => {
  const url = new URL(request.url)
  const originUrl = new URL(ELECTRIC_BASE_URL)

  url.searchParams.forEach((value, key) => {
    if (ELECTRIC_PROTOCOL_QUERY_PARAMS.includes(key)) {
      originUrl.searchParams.set(key, value)
    }
  })

  originUrl.searchParams.set('table', 'user_skill_overrides')
  // Tenant isolation: only sync this user's overrides
  // originUrl.searchParams.set('where', `user_id = '${session.userId}'`)

  const response = await fetch(originUrl)
  const headers = new Headers(response.headers)
  headers.delete('content-encoding')
  headers.delete('content-length')

  return new Response(response.body, {
    status: response.status,
    statusText: response.statusText,
    headers,
  })
}

export const ServerRoute = createServerFileRoute('/api/user-overrides').methods({
  GET: serve,
})
```

```typescript
// routes/api/user-skills.ts — Same pattern for user-created skills
import { createServerFileRoute } from '@tanstack/react-start/server'
import { ELECTRIC_PROTOCOL_QUERY_PARAMS } from '@electric-sql/client'

const ELECTRIC_BASE_URL = process.env.ELECTRIC_URL + '/v1/shape'

const serve = async ({ request }: { request: Request }) => {
  const url = new URL(request.url)
  const originUrl = new URL(ELECTRIC_BASE_URL)

  url.searchParams.forEach((value, key) => {
    if (ELECTRIC_PROTOCOL_QUERY_PARAMS.includes(key)) {
      originUrl.searchParams.set(key, value)
    }
  })

  originUrl.searchParams.set('table', 'user_skills')

  const response = await fetch(originUrl)
  const headers = new Headers(response.headers)
  headers.delete('content-encoding')
  headers.delete('content-length')

  return new Response(response.body, {
    status: response.status,
    statusText: response.statusText,
    headers,
  })
}

export const ServerRoute = createServerFileRoute('/api/user-skills').methods({
  GET: serve,
})
```

### Why This is Better Than YAML-Only or Postgres-Only

**YAML-only problems:**
- Can't store user data, execution history, or runtime state
- No multi-user — everyone sees the same YAML
- No real-time sync between clients

**Postgres-only problems:**
- Skill definitions become opaque database rows instead of readable files
- Can't review skill changes in a PR
- Need a migration for every schema change to skill structure
- Harder for developers to quickly edit and test skill pipelines
- Lose git history of who changed what and when

**Dual-source solves both:**
- Developers edit YAML in their IDE, commit, get PR review
- Users get real-time, per-user state in Postgres
- TanStack DB JOINs both on the client — the UI is unified
- The streaming execution engine reads YAML (what to do) and writes Postgres (what happened)
- Electric SQL gives you real-time sync for the Postgres side with zero WebSocket plumbing

---

## TanStack Library Quick Reference

| Library | Role in This App | Status |
|---------|-----------------|--------|
| **TanStack Start** | Full-stack framework, server functions, streaming | RC |
| **TanStack Router** | File-based routing, loaders, prefetching | Stable |
| **TanStack Query** | Server state management, caching, refetching | Stable |
| **TanStack DB** | Reactive client store, live queries, optimistic mutations | Beta |
| **TanStack Pacer** | Debounce, throttle, rate limit, queue, batch | Beta |
| **TanStack Virtual** | Virtualized lists for large skill/event lists | Stable |
| **TanStack Form** | Type-safe forms with validation | Stable |
| **TanStack Store** | Framework-agnostic reactive state (internal to Pacer) | Alpha |
| **TanStack Intent** | Agent skill discovery from npm packages | Alpha |
| **TanStack CLI** | Project scaffolding with add-ons | Alpha |

---

## Notes for Claude Code CLI

### Build order
1. `data/skills.yaml` + `lib/types.ts` — get the YAML schema and all Zod types (YAML + Postgres) right first
2. `lib/yaml-store.ts` — verify YAML loads and caches correctly
3. `server/skills.ts` — GET server functions, test instant reads
4. `server/execute-skill.ts` — streaming execution with simulated delays, test `for await` on client
5. `db/schema.ts` + `lib/planetscale.ts` — Drizzle schema and PlanetScale client
6. `lib/collections.ts` — TanStack DB collections for both YAML and Postgres sources
7. `routes/api/electric/[...path].ts` — Electric SQL proxy route
8. Components: SkillDashboard (cross-collection JOINs), SkillSearch (Pacer), VirtualizedEventLog (Virtual), SkillEditor (Form)

### Key architectural points
- **YAML is read-only on the client** — `skillDefsCollection` has no mutation handlers. Developers edit YAML in their IDE, commit, deploy.
- **Postgres is read-write on the client** — `userOverridesCollection`, `userSkillsCollection`, `executionHistoryCollection` all have optimistic mutation handlers with Electric SQL txid handshake.
- **The streaming execution engine bridges both** — it reads skill definitions from YAML (what to do) and writes execution results to Postgres (what happened).
- **Cross-collection JOINs are the killer feature** — `useLiveQuery()` can JOIN YAML-backed and Postgres-backed collections. The UI doesn't know which source the data came from.
- **TanStack DB routes must have `ssr: false`** and use `collection.preload()` (or `Promise.all` for multiple collections) in the loader.
- All Postgres tables synced via Electric SQL must have `REPLICA IDENTITY FULL` set.
- Electric SQL proxy route must forward `electric-offset`, `electric-handle`, `electric-schema`, `electric-cursor` headers.

### Library-specific reminders
- Run `npx @tanstack/cli create skills-app --add-ons tanstack-query` to scaffold, then add the other TanStack packages manually
- Run `npx @tanstack/intent@latest install` after installing deps to wire up agent skills from node_modules
- Use `npx @tanstack/cli search-docs "loaders" --library router --framework react --json` for agent-friendly doc lookup during development
- TanStack Pacer's `debounce()` requires a stable reference (useCallback) in React
- TanStack DB's `collection.update()` uses an Immer-style draft proxy — mutate the draft directly, don't return a new object
- All code uses the current TanStack Start API (`createServerFn().handler()` / `createServerFn().inputValidator().handler()`)
- The `for await (const event of stream)` pattern on the client is how TanStack Start exposes the async generator — no extra libraries needed

### Electric SQL critical gotchas
- **`shapeOptions.url` points to YOUR proxy route** (e.g. `/api/execution-history`), never directly to Electric. Electric is server-side only.
- **Proxy routes use `createServerFileRoute`** from `@tanstack/react-start/server`, NOT `createAPIFileRoute`
- **Always delete `content-encoding` and `content-length`** from the proxied response headers — these break streaming through proxies
- **Forward only `ELECTRIC_PROTOCOL_QUERY_PARAMS`** from the client request to Electric — don't blindly pass all query params
- **`pg_current_xact_id()` MUST be called INSIDE the same transaction as the mutation.** If called outside, the txid won't match and `awaitTxId` will stall forever. This is the #1 debugging issue.
- **Use `::xid::text` cast** (not just `::text`) when querying `pg_current_xact_id()` — this strips the epoch to match Electric's replication stream format
- **All synced tables need `REPLICA IDENTITY FULL`** — run `ALTER TABLE xxx REPLICA IDENTITY FULL` on every table Electric syncs
- **Debug txid issues with `localStorage.debug = 'ts/db:electric'`** in the browser console to see when txids are expected vs. when they arrive
- **Persistence handlers should return `{ txid }`** — TanStack DB blocks sync data until the mutation is confirmed, preventing optimistic update flicker
- **For prototyping without txids**, you can use `awaitMatch()` with a custom match function, or even a simple `setTimeout` (crude but works for POC)

---

## Part 11: Deployment & Infrastructure

### Production Target

- **Domain**: `skills.sacred-texts.com`
- **Hosting**: Vercel (TanStack Start with Vercel adapter)
- **Database**: PlanetScale Postgres (new database, dedicated to this app)
- **Real-time sync**: Electric SQL (hosted alongside PlanetScale)
- **Repo**: New standalone repository (code moved from `platform/research/the-library-main/`)

### Environment Variables

```bash
# PlanetScale
DATABASE_URL="mysql://..."

# Electric SQL
ELECTRIC_URL="https://electric.sacred-texts.com"

# Webhook (for YAML cache invalidation on deploy)
WEBHOOK_SECRET="..."

# Optional: GitHub token for /library push/sync commands
GITHUB_TOKEN="ghp_..."
```

### Vercel Configuration

```json
// vercel.json
{
  "framework": null,
  "buildCommand": "pnpm build",
  "outputDirectory": ".output",
  "regions": ["iad1"],
  "headers": [
    {
      "source": "/api/execution-history",
      "headers": [{ "key": "Cache-Control", "value": "no-cache" }]
    },
    {
      "source": "/api/user-overrides",
      "headers": [{ "key": "Cache-Control", "value": "no-cache" }]
    }
  ]
}
```

---

## Part 12: The Library Integration — library.yaml as Core Data Source

### The Library Replaces Goldy CLI

The Library (from `research/the-library-main/`) becomes the **canonical distribution system** for all agent capabilities. The defunct Goldy Go binary is removed. The `/goldy` planning skill and `/goldy-loop` orchestration continue as Library-managed entries.

### library.yaml Is the Database

`library.yaml` is the **single source of truth** for what capabilities exist. It maps directly to the "YAML = what things ARE" principle from Part 1. All catalog data lives in this file — it is git-tracked, PR-reviewed, and deployed with the app.

PlanetScale Postgres stores **runtime data only**: execution history, user overrides, user-created skills, workflow execution state.

### Source of Truth Mapping

| Data | Source | Rationale |
|------|--------|-----------|
| Skill definitions (name, type, source, deps) | **library.yaml** | Developer-authored, git-tracked |
| Skill groupings and relationships | **library.yaml** | Structural data, reviewed in PRs |
| Orchestrator workflow definitions | **library.yaml** | Composed from skills, git-tracked |
| Slash command mappings | **library.yaml** | Configuration, deployed with code |
| Hook/script associations | **library.yaml** | Structural metadata |
| Execution history | **Postgres** | Runtime, append-only, multi-user |
| User skill overrides | **Postgres** | Per-user, mutable |
| User-created skills (runtime) | **Postgres** | User-generated content |
| Workflow execution state | **Postgres** | Runtime, resumable |

---

## Part 13: Extended YAML Schema

The Library's original schema supports skills, agents, and prompts. We extend it to include **hooks**, **scripts**, **orchestrators**, and **slash command mappings**.

### Extended `library.yaml`

```yaml
# library.yaml — The Database
# All catalog data lives here. Runtime data lives in Postgres.

default_dirs:
  skills:
    default: .claude/skills/
    global: ~/.claude/skills/
  agents:
    default: .claude/agents/
    global: ~/.claude/agents/
  prompts:
    default: .claude/commands/
    global: ~/.claude/commands/
  hooks:
    default: .claude/hooks/
    global: ~/.claude/hooks/
  scripts:
    default: .claude/scripts/
    global: ~/.claude/scripts/

library:
  skills:
    - name: tanstack-query
      description: "TanStack Query v5 best practices for data fetching and caching"
      source: https://github.com/anthropics/claude-skills/blob/main/skills/tanstack-query/SKILL.md
      slash_command: /tanstack-query
      type: skill
      enabled: true
      tags: [frontend, data-fetching, react]
      group: tanstack-ecosystem
      requires: []

    - name: tanstack-router
      description: "Type-safe file-based routing with TanStack Router"
      source: https://github.com/anthropics/claude-skills/blob/main/skills/tanstack-router/SKILL.md
      slash_command: /tanstack-router
      type: skill
      enabled: true
      tags: [frontend, routing, react]
      group: tanstack-ecosystem
      requires: []

    - name: goldy-planning
      description: "Gold Standard planning and phased execution"
      source: /Users/forest/.goldy/SKILL.md
      slash_command: /goldy
      type: skill
      enabled: true
      tags: [planning, orchestration]
      group: goldy-system
      hooks:
        - name: goldy-auto-invoke
          trigger: SessionStart
          script: scripts/goldy_auto_invoke.py
      scripts:
        - name: goldy.py
          path: scripts/goldy.py
          description: "Planning script"
        - name: goldy_loop.py
          path: scripts/goldy_loop.py
          description: "Phase loop execution"
      requires: []

  agents:
    - name: code-reviewer
      description: "Expert code review for quality, security, and maintainability"
      source: ~/.claude/agents/code-reviewer/AGENT.md
      type: agent
      enabled: true
      tags: [review, quality]
      requires: []

  prompts:
    - name: commit-message
      description: "Generate conventional commit messages"
      source: ~/.claude/commands/commit-message.md
      slash_command: /commit-message
      type: prompt
      enabled: true
      tags: [git, workflow]
      requires: []

  hooks:
    - name: session-start-audit
      description: "Run session audit on SessionStart"
      trigger: SessionStart
      script: hooks/session-start-audit.sh
      type: hook
      enabled: true
      requires: []

  scripts:
    - name: goldy_tavily
      description: "Tavily web search integration for Goldy"
      path: scripts/goldy_tavily.py
      type: script
      language: python
      enabled: true
      requires: [skill:goldy-planning]

  orchestrators:
    - name: full-review-pipeline
      description: "Run code review → test → lint → commit in sequence"
      slash_command: /full-review
      type: orchestrator
      enabled: true
      tags: [workflow, review, ci]
      steps:
        - skill: code-reviewer
          action: review
        - script: run-tests.py
          action: execute
        - skill: commit-message
          action: generate
      requires: [skill:code-reviewer, prompt:commit-message, script:run-tests]
```

### Extended Zod Schema: `lib/types.ts`

```typescript
import { z } from 'zod'

// ─── Entry Types ───
export const EntryType = z.enum([
  'skill', 'agent', 'prompt', 'hook', 'script', 'orchestrator'
])
export type EntryType = z.infer<typeof EntryType>

export const InstallStatus = z.enum([
  'installed-default', 'installed-global', 'not-installed'
])
export type InstallStatus = z.infer<typeof InstallStatus>

export const SourceType = z.enum(['local', 'github-browser', 'github-raw'])
export type SourceType = z.infer<typeof SourceType>

// ─── Hook Definition (embedded in skills) ───
export const HookDefSchema = z.object({
  name: z.string(),
  trigger: z.enum(['SessionStart', 'PreToolUse', 'PostToolUse', 'Stop', 'SubAgentStart']),
  script: z.string(),
})

// ─── Script Definition (embedded in skills or standalone) ───
export const ScriptDefSchema = z.object({
  name: z.string(),
  path: z.string(),
  description: z.string().optional(),
  language: z.enum(['python', 'bash', 'typescript', 'javascript']).optional(),
})

// ─── Orchestrator Step ───
export const OrchestratorStepSchema = z.object({
  skill: z.string().optional(),
  script: z.string().optional(),
  prompt: z.string().optional(),
  action: z.string(),
  params: z.record(z.unknown()).optional(),
})

// ─── Base Entry (shared fields) ───
const BaseEntrySchema = z.object({
  name: z.string(),
  description: z.string(),
  type: EntryType,
  enabled: z.boolean().default(true),
  tags: z.array(z.string()).optional(),
  group: z.string().optional(),
  requires: z.array(z.string()).optional(),
  slash_command: z.string().optional(),
})

// ─── Skill Entry ───
export const SkillEntrySchema = BaseEntrySchema.extend({
  type: z.literal('skill'),
  source: z.string(),
  hooks: z.array(HookDefSchema).optional(),
  scripts: z.array(ScriptDefSchema).optional(),
})

// ─── Agent Entry ───
export const AgentEntrySchema = BaseEntrySchema.extend({
  type: z.literal('agent'),
  source: z.string(),
})

// ─── Prompt Entry ───
export const PromptEntrySchema = BaseEntrySchema.extend({
  type: z.literal('prompt'),
  source: z.string(),
})

// ─── Hook Entry ───
export const HookEntrySchema = BaseEntrySchema.extend({
  type: z.literal('hook'),
  trigger: z.string(),
  script: z.string(),
})

// ─── Script Entry ───
export const ScriptEntrySchema = BaseEntrySchema.extend({
  type: z.literal('script'),
  path: z.string(),
  language: z.string().optional(),
})

// ─── Orchestrator Entry ───
export const OrchestratorEntrySchema = BaseEntrySchema.extend({
  type: z.literal('orchestrator'),
  steps: z.array(OrchestratorStepSchema),
})

// ─── Union Entry ───
export const LibraryEntrySchema = z.discriminatedUnion('type', [
  SkillEntrySchema,
  AgentEntrySchema,
  PromptEntrySchema,
  HookEntrySchema,
  ScriptEntrySchema,
  OrchestratorEntrySchema,
])
export type LibraryEntry = z.infer<typeof LibraryEntrySchema>

// ─── Default Dirs ───
export const DefaultDirsSchema = z.record(
  EntryType,
  z.object({ default: z.string(), global: z.string() })
)

// ─── Full Library Catalog ───
export const LibraryCatalogSchema = z.object({
  default_dirs: DefaultDirsSchema,
  library: z.object({
    skills: z.array(SkillEntrySchema).optional().default([]),
    agents: z.array(AgentEntrySchema).optional().default([]),
    prompts: z.array(PromptEntrySchema).optional().default([]),
    hooks: z.array(HookEntrySchema).optional().default([]),
    scripts: z.array(ScriptEntrySchema).optional().default([]),
    orchestrators: z.array(OrchestratorEntrySchema).optional().default([]),
  }),
})
export type LibraryCatalog = z.infer<typeof LibraryCatalogSchema>

// ─── Computed Entry (with runtime info) ───
export interface ComputedEntry extends LibraryEntry {
  installStatus: InstallStatus
  installPath?: string
  sourceInfo: {
    type: SourceType
    raw: string
    org?: string
    repo?: string
    branch?: string
    filePath: string
    parentDir: string
    displayUrl: string
  }
  attachedHooks?: HookDefSchema[]
  attachedScripts?: ScriptDefSchema[]
}
```

---

## Part 14: UI Design System

### Design Language

- **Theme**: Dark mode (slate/zinc background palette)
- **Accent colors**: Sky-blue (#0ea5e9) for connections/lines, Green (#22c55e) for active/installed status
- **Typography**: Inter or system font stack
- **Border radius**: 8px for cards, 4px for badges
- **Spacing**: 4px grid system

### Skill Card Component

Each skill is a compact card showing essential info at a glance:

```
┌─────────────────────────────────────────────┐
│ ┌─────┐                           ✏️ 🔗     │
│ │badge│  Skill Name                          │
│ │skill│  One-line description text...        │
│ └─────┘                                      │
│                                              │
│  /slash-command          ┌──────┐ ┌────────┐ │
│  [editable field]        │hook🪝│ │script📜│ │
│                          └──────┘ └────────┘ │
└─────────────────────────────────────────────┘

Active card: border-2 border-green-500 (2px green)
Inactive card: border border-zinc-700 (1px default)
```

**Card elements:**
- **Type badge** (top-left): Colored pill — `skill` (blue), `agent` (purple), `prompt` (amber), `hook` (pink), `script` (cyan), `orchestrator` (emerald)
- **Edit pencil** (✏️, top-right): Opens popover editor. If source is a GitHub repo, links to the original repo
- **External link** (🔗, top-right): Opens source URL in new tab
- **Slash command field**: Click-to-edit inline input. Shows `/command-name` when set, "Add slash command" placeholder when empty
- **Attached sub-cards**: Tiny cards below the main card for associated hooks and scripts. Connected to the main card with a thin line

### Attached Sub-Cards (Hooks & Scripts)

When a skill has `hooks` or `scripts` defined in library.yaml, show them as small attached cards:

```
┌─────────────────────────────────────┐
│  Goldy Planning                      │ ← Main skill card
│  /goldy                              │
└────────────┬──────────┬─────────────┘
             │          │
     ┌───────┴──┐  ┌────┴──────┐
     │🪝 auto-  │  │📜 goldy.py│  ← Attached sub-cards
     │  invoke  │  │           │
     └──────────┘  └───────────┘
```

- Sub-cards are 60% width of main card
- Connected by 1px `sky-500` vertical line
- Click a sub-card to see its details in the popover

### Grouped Skills (Sky-Blue Connection Lines)

Skills sharing the same `group` field in library.yaml are visually connected:

```
┌───────────────────┐     ┌───────────────────┐     ┌───────────────────┐
│  TanStack Query   │─────│  TanStack Router  │─────│  TanStack Start   │
│  /tanstack-query  │     │  /tanstack-router  │     │  /tanstack-start  │
└───────────────────┘     └───────────────────┘     └───────────────────┘
         ↑                                                    ↑
         └────────── 1px solid sky-500 line ──────────────────┘
         Group: "tanstack-ecosystem"
```

- Groups are rendered as horizontal rows
- Cards within a group are connected by 1px `sky-500` (`#0ea5e9`) horizontal lines
- Group label shown above the row
- Ungrouped skills render in a standard grid

### Edit Popover

Clicking the pencil (✏️) on any card opens a popover with editable fields:

```
┌──────────────────────────────────────────────┐
│  Edit: Goldy Planning                    ✕   │
│──────────────────────────────────────────────│
│  Name        [Goldy Planning          ]      │
│  Description [Gold Standard planning...]     │
│  Source       /Users/forest/.goldy/SKILL.md  │
│              [Open in GitHub ↗]              │
│  Slash Cmd   [/goldy                  ]      │
│  Type        [skill ▾]                       │
│  Group       [goldy-system            ]      │
│  Tags        [planning] [orchestration] [+]  │
│  Enabled     [✓]                             │
│                                              │
│  Dependencies                                │
│  (none)                                      │
│                                              │
│  Hooks                                       │
│  🪝 goldy-auto-invoke (SessionStart)         │
│                                              │
│  Scripts                                     │
│  📜 goldy.py — Planning script               │
│  📜 goldy_loop.py — Phase loop execution     │
│                                              │
│  [Save to library.yaml]    [Cancel]          │
└──────────────────────────────────────────────┘
```

- Uses `shadcn/ui Popover` or `Dialog` component
- Uses `TanStack Form` for type-safe field validation
- **Save writes to library.yaml** via a server function that:
  1. Reads the current file
  2. Parses YAML
  3. Updates the specific entry
  4. Writes back to disk (preserving comments where possible)
  5. File watcher triggers TanStack DB collection refresh
- If source is a GitHub URL, the "Open in GitHub" link navigates to the repo

---

## Part 15: Orchestrator Agent Creator (Workflow Builder)

### Overview

The "Create" button opens a full-screen workflow builder where users compose orchestrator agents by dragging skills, hooks, and scripts into a flowchart.

### Layout

```
┌──────────────────────────────────────────────────────────────────────┐
│  Create Orchestrator Agent                           [Save] [Cancel] │
│  Name: [My Review Pipeline        ]                                  │
│  Slash Command: [/review-pipeline  ]                                 │
├──────────────────────┬───────────────────────────────────────────────┤
│  Available Items     │  Workflow Canvas                               │
│                      │                                               │
│  🔍 Search...        │      ┌──────────────────┐                     │
│                      │      │  Start            │                     │
│  ┌────┬──────┬─────┐ │      └────────┬─────────┘                     │
│  │Name│ Type │ ⠿  │ │               │ (sky-blue 1px)                │
│  ├────┼──────┼─────┤ │      ┌────────┴─────────┐                     │
│  │code│skill │ ⠿  │ │      │  code-reviewer    │                     │
│  │rev.│      │drag │ │      │  skill · review   │                     │
│  ├────┼──────┼─────┤ │      └────────┬─────────┘                     │
│  │run │script│ ⠿  │ │               │                               │
│  │test│      │drag │ │      ┌────────┴─────────┐                     │
│  ├────┼──────┼─────┤ │      │  run-tests.py     │                     │
│  │comm│prompt│ ⠿  │ │      │  script · execute  │                     │
│  │msg │      │drag │ │      └────────┬─────────┘                     │
│  ├────┼──────┼─────┤ │               │                               │
│  │lint│hook  │ ⠿  │ │      ┌────────┴─────────┐                     │
│  │chk │      │drag │ │      │  commit-message   │                     │
│  └────┴──────┴─────┘ │      │  prompt · generate │                     │
│                      │      └────────┬─────────┘                     │
│                      │               │                               │
│                      │      ┌────────┴─────────┐                     │
│                      │      │  End              │                     │
│                      │      └──────────────────┘                     │
├──────────────────────┴───────────────────────────────────────────────┤
│  Description: [Runs full code review, tests, then commits         ]  │
│  Tags: [workflow] [review] [ci] [+]                                  │
└──────────────────────────────────────────────────────────────────────┘
```

### Tech Implementation

**React Flow (@xyflow/react)** powers the workflow canvas:

```typescript
import { ReactFlow, Background, Controls, MiniMap } from '@xyflow/react'
import '@xyflow/react/dist/style.css'

// Custom node types for the workflow
const nodeTypes = {
  'start-node': StartNode,
  'end-node': EndNode,
  'skill-node': SkillNode,
  'script-node': ScriptNode,
  'hook-node': HookNode,
  'prompt-node': PromptNode,
}

// Custom edge style: 1px sky-blue
const defaultEdgeOptions = {
  style: { stroke: '#0ea5e9', strokeWidth: 1 },
  type: 'smoothstep',
  animated: false,
}

function WorkflowCanvas({ nodes, edges, onNodesChange, onEdgesChange }) {
  return (
    <ReactFlow
      nodes={nodes}
      edges={edges}
      onNodesChange={onNodesChange}
      onEdgesChange={onEdgesChange}
      nodeTypes={nodeTypes}
      defaultEdgeOptions={defaultEdgeOptions}
      fitView
      snapToGrid
      snapGrid={[16, 16]}
    >
      <Background color="#27272a" gap={16} />
      <Controls />
      <MiniMap />
    </ReactFlow>
  )
}
```

**@dnd-kit** handles drag from the left panel to the canvas:

```typescript
import { DndContext, DragOverlay } from '@dnd-kit/core'
import { SortableContext, verticalListSortingStrategy } from '@dnd-kit/sortable'

function OrchestratorCreator() {
  const [nodes, setNodes, onNodesChange] = useNodesState(initialNodes)
  const [edges, setEdges, onEdgesChange] = useEdgesState(initialEdges)

  const handleDragEnd = (event: DragEndEvent) => {
    const { active, over } = event
    if (over?.id === 'workflow-canvas') {
      // Add dragged item as a new node in the flowchart
      const entry = active.data.current as LibraryEntry
      const newNode = createWorkflowNode(entry, nodes.length)
      setNodes((prev) => [...prev, newNode])
      // Auto-connect to previous node
      if (nodes.length > 1) {
        const lastNode = nodes[nodes.length - 1]
        setEdges((prev) => [
          ...prev,
          { id: `e-${lastNode.id}-${newNode.id}`, source: lastNode.id, target: newNode.id },
        ])
      }
    }
  }

  return (
    <DndContext onDragEnd={handleDragEnd}>
      <div className="flex h-full">
        {/* Left panel: available items */}
        <ItemList entries={allEntries} />
        {/* Right panel: workflow canvas */}
        <WorkflowCanvas
          nodes={nodes}
          edges={edges}
          onNodesChange={onNodesChange}
          onEdgesChange={onEdgesChange}
        />
      </div>
    </DndContext>
  )
}
```

### Left Panel — Available Items (3-Column Table)

```typescript
function ItemList({ entries }: { entries: ComputedEntry[] }) {
  const [search, setSearch] = useState('')

  const filtered = entries.filter((e) =>
    e.name.toLowerCase().includes(search.toLowerCase())
  )

  return (
    <div className="w-80 border-r border-zinc-700 p-4">
      <SearchBar value={search} onChange={setSearch} />
      <div className="mt-4 space-y-1">
        {filtered.map((entry) => (
          <DraggableItem key={entry.name} entry={entry}>
            <div className="grid grid-cols-[1fr_auto_auto] gap-2 items-center
                            px-3 py-2 rounded-md hover:bg-zinc-800 cursor-grab">
              {/* Column 1: Name */}
              <span className="text-sm text-zinc-200 truncate">{entry.name}</span>
              {/* Column 2: Type badge */}
              <TypeBadge type={entry.type} />
              {/* Column 3: Drag handle */}
              <GripVertical className="w-4 h-4 text-zinc-500" />
            </div>
          </DraggableItem>
        ))}
      </div>
    </div>
  )
}
```

### Custom Workflow Nodes

```typescript
import { Handle, Position, type NodeProps } from '@xyflow/react'

function SkillNode({ data }: NodeProps) {
  return (
    <div className="rounded-lg border border-zinc-600 bg-zinc-800 px-4 py-3 min-w-[200px]">
      <Handle type="target" position={Position.Top} />
      <div className="flex items-center gap-2">
        <TypeBadge type={data.type} size="sm" />
        <span className="text-sm font-medium text-zinc-100">{data.name}</span>
      </div>
      <span className="text-xs text-zinc-400 mt-1 block">
        {data.type} · {data.action}
      </span>
      <Handle type="source" position={Position.Bottom} />
    </div>
  )
}
```

### Saving an Orchestrator

When the user clicks "Save":

1. Serialize the React Flow nodes + edges into the `orchestrators` schema:
   ```yaml
   orchestrators:
     - name: my-review-pipeline
       description: "Runs full code review, tests, then commits"
       slash_command: /review-pipeline
       type: orchestrator
       enabled: true
       tags: [workflow, review, ci]
       steps:
         - skill: code-reviewer
           action: review
         - script: run-tests.py
           action: execute
         - prompt: commit-message
           action: generate
       requires: [skill:code-reviewer, script:run-tests, prompt:commit-message]
   ```
2. Server function appends to `library.yaml`
3. File watcher triggers collection refresh
4. UI auto-updates via TanStack DB live query

### Create Button Options

The "Create" button opens a dropdown with options:

| Option | Creates | Opens |
|--------|---------|-------|
| Orchestrator Agent | New orchestrator entry | Workflow builder (full screen) |
| Skill | New skill entry | Edit popover with empty fields |
| Hook | New hook entry | Edit popover with trigger selector |
| Script | New script entry | Edit popover with language/path fields |
| Slash Command | New prompt entry | Edit popover with command name |

---

## Part 16: Additional Dependencies

Add these to the existing `package.json` from the Project Setup section:

```json
{
  "dependencies": {
    "@xyflow/react": "^12.0.0",
    "@dnd-kit/core": "^6.1.0",
    "@dnd-kit/sortable": "^8.0.0",
    "@dnd-kit/utilities": "^3.2.0",
    "lucide-react": "^0.400.0",
    "class-variance-authority": "^0.7.0",
    "clsx": "^2.1.0",
    "tailwind-merge": "^2.3.0"
  },
  "devDependencies": {
    "@types/node": "^22.0.0"
  }
}
```

**shadcn/ui components to install:**

```bash
npx shadcn@latest init
npx shadcn@latest add badge button dialog input label popover select separator tabs textarea tooltip
```

### Full Dependency Map

| Package | Layer | Purpose |
|---------|-------|---------|
| @tanstack/react-start | Framework | Full-stack React, server functions, streaming |
| @tanstack/react-router | Routing | File-based type-safe routing |
| @tanstack/react-query | Data | Server state, caching, refetching |
| @tanstack/react-db | Data | Reactive collections, live queries |
| @tanstack/query-db-collection | Data | YAML → TanStack DB bridge |
| @tanstack/electric-db-collection | Data | Postgres → TanStack DB bridge |
| @electric-sql/client | Sync | Real-time Postgres sync |
| @tanstack/react-pacer | Timing | Debounced search, throttled events |
| @tanstack/react-virtual | Perf | Virtualized large lists |
| @tanstack/react-form | Forms | Type-safe skill editing forms |
| @tanstack/store | State | Internal reactive state |
| @xyflow/react | UI | Workflow builder flowchart canvas |
| @dnd-kit/core | UI | Drag-and-drop from list to canvas |
| @dnd-kit/sortable | UI | Reorderable workflow steps |
| drizzle-orm | DB | PlanetScale Postgres ORM |
| js-yaml | Parse | library.yaml parsing |
| zod | Validation | Schema validation for all types |
| lucide-react | UI | Icons (pencil, grip, link, etc.) |
| shadcn/ui (Radix) | UI | Popover, Dialog, Badge, Button, Tabs |
| tailwindcss | Styling | Utility-first CSS |

---

## Part 17: Updated File Structure

```
skills-app/
├── data/
│   └── library.yaml              # THE database — all catalog data
├── lib/
│   ├── types.ts                  # Extended Zod schemas (Part 13)
│   ├── yaml-store.ts             # In-memory cache with mtime invalidation
│   ├── yaml-watcher.ts           # fs.watch for auto-invalidation
│   ├── yaml-writer.ts            # Write-back to library.yaml (for edits)
│   ├── collections.ts            # TanStack DB collections (YAML + Postgres)
│   ├── source-parser.ts          # Parse source URLs (local/github-browser/github-raw)
│   ├── install-checker.ts        # Check install status against default_dirs
│   ├── planetscale.ts            # Postgres client
│   └── txid.ts                   # Transaction ID helper for Electric
├── db/
│   ├── schema.ts                 # Drizzle schema (execution_history, overrides, etc.)
│   └── migrations/
├── server/
│   ├── library.ts                # Server functions: fetchCatalog, fetchEntry, updateEntry
│   ├── skills.ts                 # GET server functions (YAML → instant reads)
│   ├── execute-skill.ts          # Streaming execution engine
│   ├── persist-execution.ts      # Write results to Postgres with txid
│   └── yaml-mutator.ts           # Safe YAML read-modify-write for edit popover saves
├── routes/
│   ├── __root.tsx                # Root layout with dark theme
│   ├── index.tsx                 # Catalog browser (main page)
│   ├── create.tsx                # Orchestrator creator (workflow builder)
│   ├── skills.$skillId.tsx       # Skill detail + streaming execution
│   └── api/
│       ├── reload-skills.ts      # Webhook endpoint for YAML cache busting
│       ├── execution-history.ts  # Electric proxy: execution_history table
│       ├── user-overrides.ts     # Electric proxy: user_skill_overrides table
│       └── user-skills.ts        # Electric proxy: user_skills table
├── components/
│   ├── catalog/
│   │   ├── CatalogView.tsx       # Main grid with type tabs + group rendering
│   │   ├── SkillCard.tsx         # Individual skill card with badges
│   │   ├── AttachedSubCard.tsx   # Tiny hook/script card attached to parent
│   │   ├── GroupRow.tsx          # Horizontal row with sky-blue connecting lines
│   │   ├── TypeBadge.tsx         # Colored pill badge (skill/agent/prompt/hook/script)
│   │   ├── StatusIndicator.tsx   # Green border for active, default for inactive
│   │   └── SlashCommandField.tsx # Click-to-edit inline slash command input
│   ├── editor/
│   │   ├── EditPopover.tsx       # Full edit popover with TanStack Form
│   │   ├── TagEditor.tsx         # Tag chips with add/remove
│   │   └── DependencyList.tsx    # Dependency display with status
│   ├── creator/
│   │   ├── OrchestratorCreator.tsx  # Full-screen workflow builder
│   │   ├── WorkflowCanvas.tsx    # React Flow canvas with custom nodes
│   │   ├── ItemList.tsx          # Left panel: 3-column draggable list
│   │   ├── DraggableItem.tsx     # Individual draggable row
│   │   ├── SkillNode.tsx         # React Flow custom node for skills
│   │   ├── ScriptNode.tsx        # React Flow custom node for scripts
│   │   ├── HookNode.tsx          # React Flow custom node for hooks
│   │   ├── StartNode.tsx         # Workflow start marker
│   │   ├── EndNode.tsx           # Workflow end marker
│   │   └── CreateButton.tsx      # Create dropdown (orchestrator/skill/hook/script/command)
│   ├── shared/
│   │   ├── SearchBar.tsx         # Debounced search with Pacer
│   │   ├── ConnectionLine.tsx    # SVG sky-blue connection line component
│   │   └── Layout.tsx            # App shell with dark theme
│   ├── SkillDashboard.tsx        # Cross-collection JOINed view
│   ├── VirtualizedEventLog.tsx   # Virtualized execution event list
│   └── SkillEditor.tsx           # Form-based skill editing (TanStack Form)
├── ui/                           # shadcn/ui generated components
│   ├── badge.tsx
│   ├── button.tsx
│   ├── dialog.tsx
│   ├── input.tsx
│   ├── popover.tsx
│   ├── select.tsx
│   ├── tabs.tsx
│   └── tooltip.tsx
├── styles/
│   └── globals.css               # Tailwind directives + custom properties
├── CLAUDE.md                     # Agent instructions for this project
├── AGENTS.md                     # TanStack Intent skill mappings
├── app.config.ts                 # TanStack Start config with Vercel adapter
├── vercel.json
├── drizzle.config.ts
└── package.json
```

---

## Part 18: Implementation Phases

### Phase 1: Project Scaffold + YAML Layer
**Checklist rule: mark `[x]` only when implementation is complete and validated with tests/evidence.**

- [ ] Scaffold with `npx @tanstack/cli create skills-app --add-ons tanstack-query`
- [ ] Install all dependencies (TanStack ecosystem + React Flow + dnd-kit + shadcn)
- [ ] Create `data/library.yaml` with extended schema (skills, agents, prompts, hooks, scripts, orchestrators)
- [ ] Create `lib/types.ts` with full Zod schema (Part 13)
- [ ] Create `lib/yaml-store.ts` with in-memory cache + mtime invalidation
- [ ] Create `lib/yaml-watcher.ts` with fs.watch auto-invalidation
- [ ] Create `lib/source-parser.ts` for source URL detection
- [ ] Create `lib/install-checker.ts` for install status checking
- [ ] Create `server/library.ts` with `fetchCatalog` and `fetchEntry` server functions
- [ ] Verify: `fetchCatalog()` returns parsed library.yaml with computed install status

### Phase 2: Database + Electric SQL
**Checklist rule: mark `[x]` only when implementation is complete and validated with tests/evidence.**

- [ ] Create PlanetScale database for `skills.sacred-texts.com`
- [ ] Create `db/schema.ts` with Drizzle schema (execution_history, user_skill_overrides, user_skills)
- [ ] Run migrations: `npx drizzle-kit push`
- [ ] Set `REPLICA IDENTITY FULL` on all synced tables
- [ ] Set up Electric SQL service
- [ ] Create `lib/planetscale.ts` Postgres client
- [ ] Create `lib/txid.ts` transaction ID helper
- [ ] Create `lib/collections.ts` with dual-source TanStack DB collections
- [ ] Create Electric proxy routes: `/api/execution-history`, `/api/user-overrides`, `/api/user-skills`
- [ ] Verify: Electric SQL syncs changes from Postgres to client in real-time

### Phase 3: Catalog Browser UI
**Checklist rule: mark `[x]` only when implementation is complete and validated with tests/evidence.**

- [ ] Create `styles/globals.css` with dark theme variables
- [ ] Create `components/shared/Layout.tsx` — dark theme shell
- [ ] Create `components/catalog/TypeBadge.tsx` — colored type pills
- [ ] Create `components/catalog/StatusIndicator.tsx` — green border for active
- [ ] Create `components/catalog/SkillCard.tsx` — main card with all elements
- [ ] Create `components/catalog/AttachedSubCard.tsx` — tiny hook/script sub-cards
- [ ] Create `components/catalog/SlashCommandField.tsx` — click-to-edit inline input
- [ ] Create `components/catalog/GroupRow.tsx` — horizontal row with sky-blue lines
- [ ] Create `components/catalog/CatalogView.tsx` — type tabs + grouped grid
- [ ] Create `components/shared/SearchBar.tsx` — debounced search with Pacer
- [ ] Create `routes/index.tsx` — main catalog page (SSR disabled, collection preload)
- [ ] Verify: Browse all catalog entries, search filters in real-time, groups connected by lines

### Phase 4: Edit Popover + YAML Write-Back
**Checklist rule: mark `[x]` only when implementation is complete and validated with tests/evidence.**

- [ ] Create `lib/yaml-writer.ts` — safe read-modify-write for library.yaml
- [ ] Create `server/yaml-mutator.ts` — server function for YAML mutations
- [ ] Create `components/editor/EditPopover.tsx` — full edit form with TanStack Form
- [ ] Create `components/editor/TagEditor.tsx` — tag chips with add/remove
- [ ] Create `components/editor/DependencyList.tsx` — dependency display
- [ ] Wire pencil icon click → open EditPopover with entry data
- [ ] Wire "Save" → server function → write to library.yaml → file watcher triggers refresh
- [ ] Wire "Open in GitHub" link for GitHub-sourced entries
- [ ] Verify: Edit a skill's description, save, UI auto-refreshes with new data

### Phase 5: Orchestrator Workflow Builder
**Checklist rule: mark `[x]` only when implementation is complete and validated with tests/evidence.**

- [ ] Create `components/creator/CreateButton.tsx` — dropdown with create options
- [ ] Create `components/creator/ItemList.tsx` — 3-column draggable list (name, type, grip)
- [ ] Create `components/creator/DraggableItem.tsx` — @dnd-kit draggable wrapper
- [ ] Create `components/creator/WorkflowCanvas.tsx` — React Flow canvas with sky-blue edges
- [ ] Create custom nodes: `SkillNode.tsx`, `ScriptNode.tsx`, `HookNode.tsx`, `StartNode.tsx`, `EndNode.tsx`
- [ ] Create `components/creator/OrchestratorCreator.tsx` — full-screen builder layout
- [ ] Create `routes/create.tsx` — workflow builder route
- [ ] Implement drag from ItemList → WorkflowCanvas (adds node + auto-connects)
- [ ] Implement node reordering via drag on the canvas
- [ ] Implement "Save" → serialize to orchestrator YAML → write to library.yaml
- [ ] Verify: Create a 3-step orchestrator, save, see it appear in the catalog

### Phase 6: Streaming Execution + History
**Checklist rule: mark `[x]` only when implementation is complete and validated with tests/evidence.**

- [ ] Create `server/execute-skill.ts` — streaming async generator execution
- [ ] Create `server/persist-execution.ts` — write results to Postgres with txid
- [ ] Create `components/VirtualizedEventLog.tsx` — TanStack Virtual event list
- [ ] Create `routes/skills.$skillId.tsx` — detail page with execute button
- [ ] Create `components/SkillDashboard.tsx` — cross-collection JOIN view (YAML defs + Postgres history)
- [ ] Wire execute button → streaming events → virtualized log
- [ ] Wire execution results → persist to Postgres → Electric syncs to all clients
- [ ] Verify: Execute a skill, see streaming events, verify history appears in dashboard

### Phase 7: Deployment
**Checklist rule: mark `[x]` only when implementation is complete and validated with tests/evidence.**

- [ ] Configure `app.config.ts` with Vercel adapter
- [ ] Create `vercel.json` with headers and regions
- [ ] Set all environment variables in Vercel dashboard
- [ ] Configure `skills.sacred-texts.com` DNS → Vercel
- [ ] Deploy and verify: full app loads at production URL
- [ ] Verify: Electric SQL syncs in production
- [ ] Verify: library.yaml edits persist across deploys (git-tracked)

### Phase 8: Populate Catalog + Team Onboarding
**Checklist rule: mark `[x]` only when implementation is complete and validated with tests/evidence.**

- [ ] Inventory all existing skills from `~/.claude/skills/`, `~/.goldy/skills/`, `~/.agents/skills/`
- [ ] Deduplicate and catalog all entries in library.yaml
- [ ] Remove Goldy Go binary (`~/.goldy/goldy`, `/opt/homebrew/bin/goldy`, `.goreleaser.yml`)
- [ ] Verify: `/library list` shows all cataloged entries
- [ ] Create team onboarding guide (how to fork, clone, sync)
- [ ] Set up shared GitHub repo for library.yaml
- [ ] Verify: team member can access `skills.sacred-texts.com` and browse the catalog

---

## Routes

| Route | Purpose | SSR |
|-------|---------|-----|
| `/` | Catalog browser (cards, search, groups, tabs) | `false` (uses TanStack DB) |
| `/create` | Orchestrator workflow builder | `false` |
| `/skills/:id` | Skill detail + streaming execution | `false` |
| `/api/reload-skills` | Webhook for YAML cache invalidation | N/A (API) |
| `/api/execution-history` | Electric proxy for execution_history | N/A (API) |
| `/api/user-overrides` | Electric proxy for user_skill_overrides | N/A (API) |
| `/api/user-skills` | Electric proxy for user_skills | N/A (API) |

---

## Acceptance Targets

1. `skills.sacred-texts.com` loads the catalog browser
2. library.yaml contains all existing skills (~70+ entries) with extended schema
3. Cards show correct type badges (skill/agent/prompt/hook/script/orchestrator)
4. Active/enabled cards have green 2px border
5. Grouped skills connected by 1px sky-blue horizontal lines
6. Attached sub-cards shown for skills with hooks/scripts
7. Click-to-edit slash command field on each card
8. Pencil icon opens edit popover with all fields
9. Edit popover saves to library.yaml, UI auto-refreshes
10. "Open in GitHub" link works for GitHub-sourced entries
11. Search filters entries by name/description with debounce
12. Type tabs filter by skill/agent/prompt/hook/script/orchestrator
13. "Create" button offers: Orchestrator, Skill, Hook, Script, Slash Command
14. Workflow builder: drag from 3-column list to canvas
15. Workflow nodes snap into flowchart with sky-blue 1px connecting lines
16. Workflow nodes reorderable via drag
17. Save orchestrator → serializes to library.yaml
18. Streaming execution shows real-time events in virtualized log
19. Execution history persists to Postgres, syncs via Electric SQL
20. Cross-collection live query JOINs YAML defs + Postgres history
21. Go binary removed, /library commands functional
22. Vercel deployment stable with Electric SQL sync
