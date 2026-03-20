---
name: task-management
description: Standardized macro-task and micro-task system used by all agents (orchestrator, coder, tester, stuck) via TodoWrite.
---

# TASK-MANAGEMENT.md - Universal Task System for All Agents

**Purpose**: Standardized task management system for orchestrator and all subagents (coder, tester, stuck) with macro-task and micro-task support.

---

## 🎯 Core Principles

### 1. **Every Agent Uses TodoWrite**
- **Orchestrator**: Creates macro-tasks (features, workflows, multi-step operations)
- **Coder**: Breaks macro-tasks into micro-tasks (individual files, functions, schemas)
- **Tester**: Creates verification micro-tasks for each test scenario
- **Stuck**: Creates escalation micro-tasks for gathering context and human input

### 2. **Two-Level Task Hierarchy**

```
MACRO-TASK (Orchestrator creates)
├── MICRO-TASK 1 (Coder/Tester/Stuck creates)
├── MICRO-TASK 2
├── MICRO-TASK 3
└── MICRO-TASK 4
```

**Macro-Task Example**: "Implement user profile upload feature with RBAC"
**Micro-Tasks**:
- Create profile_images schema in /src/db/schema/
- Generate and apply migration
- Create uploadProfileImage server function
- Create ProfileImageUpload component
- Test upload flow with Playwright
- Test RBAC access control

### 3. **Task States**
- `pending` 📋 - Ready for execution
- `in_progress` 🔄 - Currently active (ONE per agent)
- `completed` ✅ - Successfully finished
- `blocked` 🚧 - Waiting on dependency or human input

---

## 📋 Orchestrator Task Management

### **When to Create Tasks**

**ALWAYS create tasks for**:
- Multi-step projects (3+ steps)
- Features requiring multiple files
- Database schema changes + migrations
- RBAC changes with cache invalidation
- Complex workflows involving multiple agents

### **Macro-Task Pattern**

```typescript
// Step 1: Analyze project requirements
// Step 2: Create comprehensive macro-task list

TodoWrite([
  {
    content: "Create notifications schema and migration",
    status: "pending",
    activeForm: "Creating notifications schema"
  },
  {
    content: "Implement notification server functions (create, read, mark as read)",
    status: "pending",
    activeForm: "Implementing notification server functions"
  },
  {
    content: "Create notification UI components",
    status: "pending",
    activeForm: "Creating notification UI components"
  },
  {
    content: "Test notification flow end-to-end with Playwright",
    status: "pending",
    activeForm: "Testing notification flow"
  }
]);
```

### **Delegation with Task Context**

```typescript
// When delegating to coder:
Task({
  subagent_type: "coder",
  description: "Create notifications schema",
  prompt: `
    Implement the first macro-task: "Create notifications schema and migration"

    Break this into micro-tasks:
    1. Create /src/db/schema/notifications.ts
    2. Export from /src/db/schema/index.ts
    3. Run pnpm db:generate
    4. Review migration SQL
    5. Run pnpm db:migrate

    Use TodoWrite to track your micro-tasks.
    Reference TASK-MANAGEMENT.md for micro-task patterns.
    Report completion when ALL micro-tasks are done.
  `
});
```

### **Progress Tracking**

```typescript
// After coder reports completion:
TodoWrite([
  {
    content: "Create notifications schema and migration",
    status: "completed", // ✅ Mark as completed
    activeForm: "Creating notifications schema"
  },
  {
    content: "Implement notification server functions",
    status: "in_progress", // 🔄 Mark next as in_progress
    activeForm: "Implementing notification server functions"
  },
  // ... rest of tasks
]);
```

---

## 🛠️ Coder Micro-Task Management

### **When to Create Micro-Tasks**

**ALWAYS break macro-tasks into micro-tasks for**:
- Database schema changes (schema file → export → migration → apply)
- Server function implementation (validator → handler → integration)
- Component creation (component file → types → tests)
- Route creation (route file → loader → server functions)

### **Micro-Task Pattern**

```typescript
// Received macro-task: "Create notifications schema and migration"

// Step 1: Break into micro-tasks
TodoWrite([
  {
    content: "Create /src/db/schema/notifications.ts with proper fields",
    status: "in_progress",
    activeForm: "Creating notifications schema file"
  },
  {
    content: "Export notifications schema from /src/db/schema/index.ts",
    status: "pending",
    activeForm: "Exporting notifications schema"
  },
  {
    content: "Generate migration with pnpm db:generate",
    status: "pending",
    activeForm: "Generating migration"
  },
  {
    content: "Review generated SQL in /src/db/migrations/",
    status: "pending",
    activeForm: "Reviewing migration SQL"
  },
  {
    content: "Apply migration with pnpm db:migrate",
    status: "pending",
    activeForm: "Applying migration"
  }
]);

// Step 2: Execute micro-tasks one at a time
// Step 3: Mark each as completed immediately after finishing
// Step 4: Report to orchestrator when ALL micro-tasks done
```

### **Atomic Micro-Task Guidelines**

**Each micro-task should be**:
- ✅ **Atomic**: Complete in one action (create one file, run one command)
- ✅ **Verifiable**: Clear success criteria (file exists, command succeeds)
- ✅ **Sequential**: Proper dependency order (schema before migration)
- ✅ **Trackable**: Update status immediately after completion

**Example - Server Function Micro-Tasks**:
```typescript
TodoWrite([
  {
    content: "Create Zod validator schema for createNotification",
    status: "in_progress",
    activeForm: "Creating validator schema"
  },
  {
    content: "Implement createNotification.handler() with DB insert",
    status: "pending",
    activeForm: "Implementing handler"
  },
  {
    content: "Add withAuth() authentication check",
    status: "pending",
    activeForm: "Adding auth check"
  },
  {
    content: "Export createNotification from server functions",
    status: "pending",
    activeForm: "Exporting server function"
  }
]);
```

---

## 🧪 Tester Micro-Task Management

### **When to Create Micro-Tasks**

**ALWAYS create micro-tasks for**:
- Each test scenario (happy path, error cases, RBAC tests)
- Visual verification steps (screenshot captures, element checks)
- Database verification queries
- Auth flow testing steps

### **Test Micro-Task Pattern**

```typescript
// Received macro-task: "Test notification flow end-to-end"

// Step 1: Break into test micro-tasks
TodoWrite([
  {
    content: "Navigate to http://localhost:3000 and verify page loads",
    status: "in_progress",
    activeForm: "Verifying page load"
  },
  {
    content: "Create test notification via server function",
    status: "pending",
    activeForm: "Creating test notification"
  },
  {
    content: "Verify notification appears in UI",
    status: "pending",
    activeForm: "Verifying notification display"
  },
  {
    content: "Test mark-as-read functionality",
    status: "pending",
    activeForm: "Testing mark-as-read"
  },
  {
    content: "Verify database state after mark-as-read",
    status: "pending",
    activeForm: "Verifying database state"
  },
  {
    content: "Capture screenshot of notification flow",
    status: "pending",
    activeForm: "Capturing screenshot"
  }
]);

// Step 2: Execute tests using Playwright MCP
// Step 3: Mark each micro-task as completed
// Step 4: Report pass/fail to orchestrator
```

### **Platform-Specific Test Micro-Tasks**

**Auth Flow Testing**:
```typescript
TodoWrite([
  {
    content: "Test unauthenticated access redirects to login",
    status: "in_progress",
    activeForm: "Testing unauthenticated redirect"
  },
  {
    content: "Sign in with bodhimindflow@gmail.com via Google OAuth",
    status: "pending",
    activeForm: "Testing Google OAuth sign-in"
  },
  {
    content: "Verify WorkOS callback completes successfully",
    status: "pending",
    activeForm: "Verifying callback"
  },
  {
    content: "Verify user session cookie is set",
    status: "pending",
    activeForm: "Verifying session cookie"
  }
]);
```

**RBAC Testing**:
```typescript
TodoWrite([
  {
    content: "Sign in as admin user and access protected route",
    status: "in_progress",
    activeForm: "Testing admin access"
  },
  {
    content: "Verify admin can access RBAC-protected features",
    status: "pending",
    activeForm: "Verifying admin permissions"
  },
  {
    content: "Sign out and sign in as non-admin user",
    status: "pending",
    activeForm: "Switching to non-admin user"
  },
  {
    content: "Verify non-admin is blocked from protected features",
    status: "pending",
    activeForm: "Verifying access denial"
  },
  {
    content: "Capture screenshots of both access levels",
    status: "pending",
    activeForm: "Capturing access screenshots"
  }
]);
```

---

## 🚨 Stuck Agent Micro-Task Management

### **When to Create Micro-Tasks**

**ALWAYS create micro-tasks for**:
- Context gathering (DB state, auth state, RBAC state)
- Diagnostic queries
- Human escalation preparation
- Resolution verification

### **Escalation Micro-Task Pattern**

```typescript
// Received error: "Migration failed with foreign key constraint violation"

// Step 1: Break into diagnostic micro-tasks
TodoWrite([
  {
    content: "Query drizzle.__drizzle_migrations for applied migrations",
    status: "in_progress",
    activeForm: "Checking migration history"
  },
  {
    content: "Check for existing foreign key constraints on target table",
    status: "pending",
    activeForm: "Checking constraints"
  },
  {
    content: "Verify referenced table exists with correct schema",
    status: "pending",
    activeForm: "Verifying referenced table"
  },
  {
    content: "Gather full error context and attempted solutions",
    status: "pending",
    activeForm: "Gathering error context"
  },
  {
    content: "Escalate to human with comprehensive context",
    status: "pending",
    activeForm: "Escalating to human"
  }
]);

// Step 2: Execute diagnostics
// Step 3: Present findings to human
// Step 4: Apply human's decision
```

### **Platform Diagnostic Micro-Tasks**

**Database Diagnostics**:
```typescript
TodoWrite([
  {
    content: "Query information_schema.tables for schema verification",
    status: "in_progress",
    activeForm: "Verifying table schema"
  },
  {
    content: "Check indexes with information_schema.statistics",
    status: "pending",
    activeForm: "Checking indexes"
  },
  {
    content: "Verify foreign key constraints",
    status: "pending",
    activeForm: "Verifying constraints"
  }
]);
```

**RBAC Diagnostics**:
```typescript
TodoWrite([
  {
    content: "Query roles table for user's current role",
    status: "in_progress",
    activeForm: "Checking user role"
  },
  {
    content: "Query user_access for additional/denied permissions",
    status: "pending",
    activeForm: "Checking user access"
  },
  {
    content: "Verify access cache state (check recent invalidations)",
    status: "pending",
    activeForm: "Checking access cache"
  }
]);
```

---

## 🔄 Cross-Agent Task Flow

### **Complete Workflow Example**

**User Request**: "Add user profile image upload feature"

```
ORCHESTRATOR:
┌─────────────────────────────────────────────────────────────┐
│ TodoWrite([                                                 │
│   "Create profile_images schema and migration",            │
│   "Implement uploadProfileImage server function",          │
│   "Create ProfileImageUpload component",                   │
│   "Test upload flow with Playwright"                       │
│ ])                                                          │
└─────────────────────────────────────────────────────────────┘
        │
        │ Delegate macro-task 1 to CODER
        ↓
CODER:
┌─────────────────────────────────────────────────────────────┐
│ TodoWrite([                                                 │
│   "Create /src/db/schema/profileImages.ts",                │
│   "Export from /src/db/schema/index.ts",                   │
│   "Generate migration with pnpm db:generate",              │
│   "Review and apply migration"                             │
│ ])                                                          │
│                                                             │
│ Execute micro-tasks → Mark completed → Report to Orchestrator│
└─────────────────────────────────────────────────────────────┘
        │
        │ Delegate verification to TESTER
        ↓
TESTER:
┌─────────────────────────────────────────────────────────────┐
│ TodoWrite([                                                 │
│   "Verify profile_images table exists in DB",              │
│   "Check table schema matches expectations",               │
│   "Verify indexes are created correctly"                   │
│ ])                                                          │
│                                                             │
│ Execute tests → Pass/Fail → Report to Orchestrator         │
└─────────────────────────────────────────────────────────────┘
        │
        │ If FAIL → Escalate to STUCK
        ↓
STUCK (if needed):
┌─────────────────────────────────────────────────────────────┐
│ TodoWrite([                                                 │
│   "Gather DB error context",                               │
│   "Check migration conflicts",                             │
│   "Prepare human escalation with full context"             │
│ ])                                                          │
│                                                             │
│ Execute diagnostics → Escalate to Human → Apply decision   │
└─────────────────────────────────────────────────────────────┘
        │
        │ Resolution → Back to ORCHESTRATOR
        ↓
ORCHESTRATOR:
┌─────────────────────────────────────────────────────────────┐
│ Mark macro-task 1 completed                                │
│ Move to macro-task 2                                       │
│ Continue workflow...                                       │
└─────────────────────────────────────────────────────────────┘
```

---

## 📊 Task Management Best Practices

### **1. Immediate Status Updates**
```typescript
// ❌ BAD: Batch updates
// Complete 3 tasks then update all at once

// ✅ GOOD: Immediate updates
Write("file.ts", content);
TodoWrite([{ content: "Create file.ts", status: "completed", ... }]);
// Immediately mark completed after action
```

### **2. One Active Task Per Agent**
```typescript
// ❌ BAD: Multiple in_progress tasks
{
  { content: "Create schema", status: "in_progress", ... },
  { content: "Create component", status: "in_progress", ... }
}

// ✅ GOOD: Single in_progress task
{
  { content: "Create schema", status: "in_progress", ... },
  { content: "Create component", status: "pending", ... }
}
```

### **3. Clear Task Descriptions**
```typescript
// ❌ BAD: Vague descriptions
{ content: "Fix database", status: "pending", ... }

// ✅ GOOD: Specific descriptions
{ content: "Create notifications schema in /src/db/schema/notifications.ts", status: "pending", ... }
```

### **4. Proper Task Sequencing**
```typescript
// ✅ GOOD: Dependencies in correct order
TodoWrite([
  { content: "Create schema file", status: "in_progress", ... },
  { content: "Export from index.ts", status: "pending", ... },  // Depends on schema
  { content: "Generate migration", status: "pending", ... },    // Depends on export
  { content: "Apply migration", status: "pending", ... }        // Depends on generation
]);
```

---

## 🎯 Platform-Specific Task Patterns

### **Database Schema + Migration**
```typescript
// Macro-task (Orchestrator)
"Create [feature] database schema with migration"

// Micro-tasks (Coder)
[
  "Create /src/db/schema/[feature].ts with id, created_at, updated_at",
  "Add proper indexes on foreign keys and WHERE columns",
  "Export schema from /src/db/schema/index.ts",
  "Generate migration: pnpm db:generate",
  "Review SQL in /src/db/migrations/[timestamp]_[name].sql",
  "Apply migration: pnpm db:migrate"
]

// Verification (Tester)
[
  "Query information_schema.tables to verify table exists",
  "Verify schema matches expectations",
  "Check indexes are created correctly",
  "Test insert/update/delete operations"
]
```

### **Server Function Implementation**
```typescript
// Macro-task (Orchestrator)
"Implement [action] server function with auth and validation"

// Micro-tasks (Coder)
[
  "Create Zod validator schema",
  "Implement handler function with DB operations",
  "Add withAuth() for authentication",
  "Add RBAC permission checks if needed",
  "Handle errors with proper error messages",
  "Export from server functions"
]

// Verification (Tester)
[
  "Test server function with valid input",
  "Test with invalid input (expect validation error)",
  "Test without auth (expect 401)",
  "Test with wrong permissions (expect 403)",
  "Verify DB state after operation"
]
```

### **Protected Route Creation**
```typescript
// Macro-task (Orchestrator)
"Create [name] protected route with RBAC"

// Micro-tasks (Coder)
[
  "Create route file in /src/routes/_authenticated/[path]/index.tsx",
  "Add getAuth() in loader function",
  "Add userHasAppAccess() RBAC check",
  "Implement route component with UI",
  "Register access entity in access_entities table"
]

// Verification (Tester)
[
  "Navigate to route URL",
  "Verify unauthenticated access redirects",
  "Sign in as admin, verify access granted",
  "Sign in as non-admin, verify access denied",
  "Capture screenshots of access levels"
]
```

---

## ✅ Success Criteria

### **Task System is Working When**:
- ✅ Every agent uses TodoWrite for their task level
- ✅ Orchestrator creates macro-tasks, subagents create micro-tasks
- ✅ Tasks are marked completed IMMEDIATELY after finishing
- ✅ Only ONE task per agent is in_progress at a time
- ✅ Task descriptions are specific and actionable
- ✅ Dependencies are properly sequenced
- ✅ Human can see complete progress at any time
- ✅ Zero tasks are left in limbo (all completed or blocked)

### **Verification Checklist**:
```typescript
// Check orchestrator
- [ ] Created macro-task list with TodoWrite
- [ ] Each macro-task is feature-level (not too granular)
- [ ] Proper sequencing (migrations before code)

// Check coder
- [ ] Created micro-task breakdown for macro-task
- [ ] Each micro-task is atomic (one file/command)
- [ ] Marked completed immediately after each action
- [ ] Reported completion when ALL micro-tasks done

// Check tester
- [ ] Created test micro-tasks for verification
- [ ] Each test scenario has specific micro-tasks
- [ ] Captured evidence (screenshots, queries)
- [ ] Reported pass/fail with evidence

// Check stuck
- [ ] Created diagnostic micro-tasks
- [ ] Gathered comprehensive context
- [ ] Escalated to human with full details
```

---

## ⚡ PARALLEL EXECUTION PATTERNS

### **Why Parallel Execution Matters**

**Sequential Execution** (Traditional - SLOW):
```
Task 1 (30s) → Task 2 (30s) → Task 3 (30s) = 90 seconds total
```

**Parallel Execution** (Ultra-Fast):
```
Task 1 (30s) ┐
Task 2 (30s) ├→ Execute simultaneously
Task 3 (30s) ┘
= 30 seconds total (3× FASTER!)
```

### **Orchestrator-Level: Parallel Macro-Task Delegation**

**When to Parallelize**: Independent macro-tasks with NO dependencies

**Independent Tasks** (Safe to parallelize):
- ✅ Creating multiple schemas (different tables)
- ✅ Creating multiple components (different features)
- ✅ Creating multiple routes (different paths)
- ✅ Implementing multiple server functions (different endpoints)

**Example - Create 3 Schemas in Parallel**:
```typescript
// ❌ SLOW: Sequential delegation (90 seconds)
Task({ subagent_type: "coder", prompt: "Create notifications schema..." });
// Wait for completion...
Task({ subagent_type: "coder", prompt: "Create alerts schema..." });
// Wait for completion...
Task({ subagent_type: "coder", prompt: "Create messages schema..." });
// Total: 30s + 30s + 30s = 90s

// ✅ FAST: Parallel delegation (30 seconds) - 3× FASTER!
// Single message with multiple Task calls:
Task({ subagent_type: "coder", prompt: "Create notifications schema..." });
Task({ subagent_type: "coder", prompt: "Create alerts schema..." });
Task({ subagent_type: "coder", prompt: "Create messages schema..." });
// All execute simultaneously
// Total: max(30s, 30s, 30s) = 30s
```

**Dependency Analysis Decision Tree**:
```
For each macro-task:
  ├─ Does it modify the SAME file as another task?
  │  ├─ YES → SEQUENTIAL (file conflict risk)
  │  └─ NO → Continue analysis
  │
  ├─ Does it require OUTPUT from another task?
  │  ├─ YES → SEQUENTIAL (dependency exists)
  │  └─ NO → Continue analysis
  │
  ├─ Does it operate on SAME resource as another?
  │  ├─ YES → SEQUENTIAL (resource conflict risk)
  │  └─ NO → PARALLEL! ⚡
```

**Dependent Tasks** (Must be sequential):
```typescript
// ❌ CANNOT parallelize these - they have dependencies
Task({ subagent_type: "coder", prompt: "Create all schemas..." });
// MUST WAIT for schemas to be created
Task({ subagent_type: "coder", prompt: "Generate and apply migrations..." });
// MUST WAIT for migrations
Task({ subagent_type: "coder", prompt: "Create server functions using new tables..." });
```

### **Agent-Level: Parallel Micro-Task Execution**

**Coder Parallel Micro-Tasks**:
```typescript
// Macro-task: "Create 5 server functions"

// ❌ SLOW: Sequential micro-tasks (200 seconds)
TodoWrite([
  { content: "Create createNotification server function", status: "in_progress", ... },
  { content: "Create updateNotification server function", status: "pending", ... },
  { content: "Create deleteNotification server function", status: "pending", ... },
  { content: "Create listNotifications server function", status: "pending", ... },
  { content: "Create markAsRead server function", status: "pending", ... }
]);
// Execute one at a time: 40s × 5 = 200s

// ✅ FAST: Parallel micro-task execution (40 seconds) - 5× FASTER!
// These are independent files - can be created simultaneously
TodoWrite([
  { content: "Create createNotification server function", status: "in_progress", ... },
  { content: "Create updateNotification server function", status: "in_progress", ... },
  { content: "Create deleteNotification server function", status: "in_progress", ... },
  { content: "Create listNotifications server function", status: "in_progress", ... },
  { content: "Create markAsRead server function", status: "in_progress", ... }
]);
// All execute simultaneously via parallel tool calls
// Execute: Write file 1, Write file 2, Write file 3, Write file 4, Write file 5
// Total: max(40s) = 40s
```

**Tester Parallel Micro-Tasks**:
```typescript
// Macro-task: "Test 5 different features"

// ✅ FAST: Parallel test execution (20 seconds) - 5× FASTER!
TodoWrite([
  { content: "Test notifications feature with Playwright", status: "in_progress", ... },
  { content: "Test alerts feature with Playwright", status: "in_progress", ... },
  { content: "Test messages feature with Playwright", status: "in_progress", ... },
  { content: "Test profile feature with Playwright", status: "in_progress", ... },
  { content: "Test settings feature with Playwright", status: "in_progress", ... }
]);
// All tests run in parallel with separate Playwright sessions
// Total: max(20s) = 20s instead of 100s
```

### **Real-World Performance Example**

**Project**: Create notifications, alerts, and messages systems

**Sequential Execution (OLD - SLOW)**:
```
Orchestrator creates macro-tasks:
├─ Create notifications schema (30s)
├─ Create alerts schema (30s)
├─ Create messages schema (30s)
├─ Generate + apply migrations (20s)
├─ Create 3 notification server functions (40s each = 120s)
├─ Create 3 alert server functions (40s each = 120s)
├─ Create 3 message server functions (40s each = 120s)
├─ Test notifications (20s)
├─ Test alerts (20s)
└─ Test messages (20s)

Total: 30+30+30+20+120+120+120+20+20+20 = 530 seconds (8.8 minutes)
```

**Parallel Execution (NEW - ULTRA-FAST)**:
```
Orchestrator creates macro-tasks:

Phase 1 (PARALLEL): Create 3 schemas simultaneously
├─ Create notifications schema (30s) ┐
├─ Create alerts schema (30s)       ├→ Parallel
└─ Create messages schema (30s)     ┘
= max(30s) = 30s

Phase 2 (SEQUENTIAL): Migration must wait for schemas
└─ Generate + apply migrations (20s)

Phase 3 (PARALLEL): Create 9 server functions simultaneously
├─ 3 notification functions (40s) ┐
├─ 3 alert functions (40s)        ├→ Parallel
└─ 3 message functions (40s)      ┘
= max(40s) = 40s

Phase 4 (PARALLEL): Test 3 features simultaneously
├─ Test notifications (20s) ┐
├─ Test alerts (20s)        ├→ Parallel
└─ Test messages (20s)      ┘
= max(20s) = 20s

Total: 30 + 20 + 40 + 20 = 110 seconds (1.8 minutes)
Speedup: 530s → 110s = 4.8× FASTER! ⚡
```

### **Micro-Task Parallelization Safety Rules**

**✅ SAFE to Parallelize**:
- Creating different files
- Querying different database tables
- Testing different features
- Independent calculations/operations

**❌ UNSAFE to Parallelize**:
- Modifying the same file
- Sequential database operations (write then read)
- Dependent test scenarios (auth then protected route)
- Operations requiring previous results

### **Performance Calculation Formula**

**Sequential Time**: `SUM(all task times)`
```
T_sequential = t1 + t2 + t3 + ... + tn
```

**Parallel Time**: `MAX(parallel batch times)`
```
T_parallel = max(parallel_batch_1) + max(parallel_batch_2) + ...
```

**Speedup Factor**:
```
Speedup = T_sequential / T_parallel

Example:
- Sequential: 30 + 30 + 30 = 90s
- Parallel: max(30, 30, 30) = 30s
- Speedup: 90 / 30 = 3× faster
```

### **Quick Parallelization Decision**

**Ask These Questions**:
1. **Same file?** → NO = Can parallelize ✅
2. **Needs output?** → NO = Can parallelize ✅
3. **Same resource?** → NO = Can parallelize ✅
4. **All YES?** → PARALLELIZE! ⚡

**Example Decisions**:
```yaml
"Create 3 schemas" → Different files, no dependencies → PARALLEL ✅
"Create schema then migration" → Output dependency → SEQUENTIAL ❌
"Create 5 components" → Different files, independent → PARALLEL ✅
"Test auth then test protected route" → Needs auth first → SEQUENTIAL ❌
```

---

## 🧹 Test File Cleanup Patterns (SCOPED - Only Delete Your Own Files)

### Coder Agent Cleanup Pattern

**Track files YOU create**:
```bash
# At task start, initialize tracking
created_test_files=()

# When creating test file
echo "console.log('test')" > test-myfeature.mjs
created_test_files+=("test-myfeature.mjs")

# When creating check script
echo "// check" > check-myfeature.mjs
created_test_files+=("check-myfeature.mjs")
```

**Delete ONLY your files at task end**:
```bash
# Scoped cleanup (only YOUR files)
if [ ${#created_test_files[@]} -gt 0 ]; then
  rm -f "${created_test_files[@]}"
  echo "Deleted ${#created_test_files[@]} test files I created"
fi

# Verify YOUR files deleted
for file in "${created_test_files[@]}"; do
  if [ -f "$file" ]; then
    echo "WARNING: Failed to delete $file"
  fi
done
```

### Tester Agent Cleanup Pattern

**Track artifacts YOU create**:
```bash
# At test start, initialize tracking
created_artifacts=()

# When creating screenshot directory
mkdir screenshots-mytest
created_artifacts+=("screenshots-mytest/")

# When saving screenshot
# (Playwright saves to screenshots-mytest/)
created_artifacts+=("test-output.png")
```

**Delete ONLY your artifacts at test end**:
```bash
# Scoped cleanup (only YOUR artifacts)
if [ ${#created_artifacts[@]} -gt 0 ]; then
  rm -rf "${created_artifacts[@]}"
  echo "Deleted ${#created_artifacts[@]} test artifacts I created"
fi

# Verify YOUR artifacts deleted
for artifact in "${created_artifacts[@]}"; do
  if [ -e "$artifact" ]; then
    echo "WARNING: Failed to delete $artifact"
  fi
done
```

### ❌ WRONG: Global Cleanup (Deletes ALL test files - DON'T DO THIS)

```bash
# WRONG - Deletes all test files (including other agents' files)
rm -f *.test.mjs *-test.mjs test-*.mjs

# WRONG - Deletes all screenshots (including other tests' screenshots)
rm -rf screenshots*/
```

### ✅ CORRECT: Scoped Cleanup (Only YOUR files)

```bash
# CORRECT - Only delete specific files YOU created
rm -f test-myfeature.mjs check-myfeature.mjs

# CORRECT - Only delete specific directory YOU created
rm -rf screenshots-mytest/
```

---

## 🚀 Quick Reference

**Orchestrator**: Macro-tasks (features) → Analyze dependencies → Delegate in PARALLEL when safe
**Coder**: Micro-tasks (files/commands) → Execute in PARALLEL when safe → Report completion → SCOPED cleanup
**Tester**: Test micro-tasks (scenarios) → Run tests in PARALLEL when independent → Report pass/fail → SCOPED cleanup
**Stuck**: Diagnostic micro-tasks (context) → Gather diagnostics in PARALLEL → Escalate to human

**Parallel Execution Goal**: Transform minutes into seconds through intelligent parallelization! ⚡

**Every agent uses TodoWrite. Every agent analyzes dependencies. Every agent parallelizes when safe. Every agent cleans up ONLY their own files. Every agent reports completion.**

This is how we build amazing features with perfect visibility AND ultra-fast performance! 🎯
