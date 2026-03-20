---
name: task-verification
description: Checklist for verifying the task management system is correctly implemented and referenced across all agents.
---

# TASK-VERIFICATION.md - Task Management System Verification

**Purpose**: Comprehensive checklist for verifying task management system is working correctly across all agents.

---

## ✅ System-Wide Verification

### Prerequisites Check
- [ ] TASK-MANAGEMENT.md exists at `.claude/agents/TASK-MANAGEMENT.md`
- [ ] All agent files reference TASK-MANAGEMENT.md
- [ ] Orchestrator CLAUDE.md includes task management in workflow
- [ ] SDK-INTEGRATION.md includes task management examples

---

## 🎯 Orchestrator Verification (CLAUDE.md)

### Task Creation
- [ ] **Creates macro-tasks** when receiving project from user
- [ ] Uses **TodoWrite** immediately after analysis
- [ ] Macro-tasks are **feature-level** (not too granular)
- [ ] Proper **sequencing** (migrations before code, code before tests)
- [ ] References **TASK-MANAGEMENT.md** in Platform Reference Links

### Delegation
- [ ] Delegates **ONE macro-task at a time** to coder
- [ ] Tells coder to "break into micro-tasks using TodoWrite"
- [ ] Tells coder to "reference TASK-MANAGEMENT.md"
- [ ] Waits for coder to complete **ALL micro-tasks**
- [ ] Tells tester to "create test micro-tasks using TodoWrite"

### Progress Tracking
- [ ] Marks macro-task as **completed** after tester passes
- [ ] Updates todo list with **TodoWrite** after each completion
- [ ] Moves to **next macro-task** only after current is done
- [ ] **Never skips** task tracking
- [ ] Reports to user **only when ALL macro-tasks complete**

### Example Workflow Check
```
✅ User gives project
✅ Orchestrator creates macro-task list with TodoWrite
✅ Orchestrator delegates macro-task #1 to coder
✅ Coder creates micro-tasks and completes them
✅ Orchestrator delegates verification to tester
✅ Tester creates test micro-tasks and executes them
✅ Orchestrator marks macro-task #1 completed
✅ Orchestrator moves to macro-task #2
```

---

## 🛠️ Coder Agent Verification (coder.md)

### Micro-Task Creation
- [ ] **Receives macro-task** from orchestrator
- [ ] **IMMEDIATELY creates micro-tasks** using TodoWrite
- [ ] Each micro-task is **atomic** (one file/command)
- [ ] Micro-tasks are **properly sequenced** (dependencies in order)
- [ ] References **TASK-MANAGEMENT.md** for patterns

### Execution Pattern
- [ ] Marks **ONE micro-task as in_progress** at a time
- [ ] Executes the micro-task action (write file, run command)
- [ ] **IMMEDIATELY marks as completed** after action
- [ ] Moves to **next micro-task** (mark as in_progress)
- [ ] **Never batches** completion updates

### Reporting
- [ ] Reports **"ALL micro-tasks completed"** to orchestrator
- [ ] Lists **completed micro-tasks** in report
- [ ] States **"Ready for testing agent verification"**
- [ ] **Never reports** completion before all micro-tasks done

### Example Micro-Task Execution
```
✅ Receives: "Create notifications schema and migration"
✅ Creates micro-tasks with TodoWrite:
   - Create schema file
   - Add indexes
   - Export from index.ts
   - Generate migration
   - Review SQL
   - Apply migration
✅ Executes each micro-task one at a time
✅ Marks completed immediately after each action
✅ Reports all completed to orchestrator
```

---

## 🧪 Tester Agent Verification (tester.md)

### Test Micro-Task Creation
- [ ] **Receives verification request** from orchestrator
- [ ] **IMMEDIATELY creates test micro-tasks** using TodoWrite
- [ ] Each test is **specific scenario** (auth, RBAC, visual, DB)
- [ ] Test micro-tasks include **evidence capture** (screenshots, queries)
- [ ] References **TASK-MANAGEMENT.md** for test patterns

### Test Execution
- [ ] Marks **ONE test micro-task as in_progress** at a time
- [ ] Executes test with **Playwright MCP** (visual verification)
- [ ] Captures **evidence** (screenshots, DOM snapshots, DB queries)
- [ ] **IMMEDIATELY marks as completed** if test passes
- [ ] **Invokes stuck agent** if test fails (with evidence)

### Reporting
- [ ] Reports **"ALL test micro-tasks completed"** to orchestrator
- [ ] Provides **pass/fail status** with evidence
- [ ] **INCLUDES SCREENSHOTS** as proof
- [ ] Lists any **visual issues** discovered
- [ ] States **"Ready for next step"** or **"Test failures found"**

### Example Test Execution
```
✅ Receives: "Test notification flow end-to-end"
✅ Creates test micro-tasks with TodoWrite:
   - Navigate and verify page load
   - Create test notification
   - Verify UI display with screenshot
   - Test mark-as-read
   - Verify DB state
   - Capture final screenshot
✅ Executes each test one at a time
✅ Marks completed immediately after each passes
✅ Reports all completed with screenshots to orchestrator
```

---

## 🚨 Stuck Agent Verification (stuck.md)

### Diagnostic Micro-Task Creation
- [ ] **Receives error/problem** from coder/tester
- [ ] **IMMEDIATELY creates diagnostic micro-tasks** using TodoWrite
- [ ] Each diagnostic is **specific check** (DB query, file read, log check)
- [ ] Diagnostic micro-tasks **gather context** for human
- [ ] References **TASK-MANAGEMENT.md** for diagnostic patterns

### Diagnostic Execution
- [ ] Marks **ONE diagnostic micro-task as in_progress** at a time
- [ ] Executes diagnostic (query DB, read files, check state)
- [ ] **IMMEDIATELY marks as completed** after gathering info
- [ ] Compiles **comprehensive context** from all diagnostics
- [ ] **Escalates to human** with full context

### Human Escalation
- [ ] **ALL diagnostic micro-tasks completed** before escalation
- [ ] Presents **clear problem description** to human
- [ ] Includes **results from all diagnostics** in escalation
- [ ] Offers **2-4 specific options** when possible
- [ ] Returns **human's decision** to calling agent

### Example Diagnostic Execution
```
✅ Receives: "Migration failed with FK constraint violation"
✅ Creates diagnostic micro-tasks with TodoWrite:
   - Query migration history
   - Check existing constraints
   - Verify referenced table
   - Review SQL migration
   - Gather error context
✅ Executes each diagnostic one at a time
✅ Marks completed immediately after each
✅ Escalates to human with full context
✅ Returns human decision to coder
```

---

## 📊 Cross-Agent Integration Verification

### Complete Feature Implementation Check
```
ORCHESTRATOR:
✅ Creates macro-tasks: [schema, server functions, UI, testing]
✅ Delegates "Create schema" to CODER

CODER:
✅ Creates micro-tasks: [create file, indexes, export, generate, apply]
✅ Executes all micro-tasks
✅ Reports completion to ORCHESTRATOR

ORCHESTRATOR:
✅ Marks "Create schema" completed
✅ Delegates "Verify schema" to TESTER

TESTER:
✅ Creates test micro-tasks: [verify table, check schema, test operations]
✅ Executes all tests with Playwright MCP
✅ Reports pass with screenshots to ORCHESTRATOR

ORCHESTRATOR:
✅ Marks "Verify schema" completed
✅ Moves to next macro-task: "Create server functions"
```

### Error Handling Flow Check
```
CODER:
✅ Encounters migration error during micro-task execution
✅ Does NOT use fallback or workaround
✅ Invokes STUCK agent immediately

STUCK:
✅ Creates diagnostic micro-tasks
✅ Executes diagnostics, marks completed
✅ Escalates to human with context
✅ Returns human decision to CODER

CODER:
✅ Applies human decision
✅ Completes remaining micro-tasks
✅ Reports completion to ORCHESTRATOR
```

---

## 🎯 Success Criteria

### Task System is Working When:
- ✅ **Every agent** uses TodoWrite for their task level
- ✅ **Orchestrator** creates macro-tasks, **subagents** create micro-tasks
- ✅ Tasks are marked **completed IMMEDIATELY** after finishing
- ✅ **Only ONE task** per agent is in_progress at a time
- ✅ Task descriptions are **specific and actionable**
- ✅ Dependencies are **properly sequenced**
- ✅ Human can **see complete progress** at any time
- ✅ **Zero tasks** are left in limbo (all completed or blocked)
- ✅ **No fallbacks** used (stuck agent escalates all issues)

### Red Flags (System NOT Working):
- ❌ Agent completes multiple tasks before updating todo list
- ❌ Multiple tasks marked as in_progress simultaneously
- ❌ Vague task descriptions ("Fix issue", "Update code")
- ❌ Dependencies out of order (code before migration)
- ❌ Agent skips TodoWrite tool usage
- ❌ Agent uses fallback instead of invoking stuck
- ❌ Tasks marked complete before actually finished
- ❌ Orchestrator reports to user before all tasks done

---

## 🔍 Manual Verification Test

### Test Scenario: "Add User Notifications Feature"

**1. Give orchestrator this request**:
```
"Add a user notifications feature with database schema, server functions, and UI components. Test with Playwright."
```

**2. Verify orchestrator**:
- [ ] Creates macro-task list with TodoWrite
- [ ] Macro-tasks include: schema, migration, server functions, UI, testing
- [ ] Proper sequencing: schema → migration → code → tests
- [ ] Delegates first macro-task to coder

**3. Verify coder**:
- [ ] Creates micro-task breakdown with TodoWrite
- [ ] Micro-tasks are atomic: create file, add indexes, export, generate, apply
- [ ] Marks in_progress → executes → marks completed for EACH
- [ ] Reports "ALL micro-tasks completed" to orchestrator

**4. Verify tester**:
- [ ] Creates test micro-tasks with TodoWrite
- [ ] Tests include: DB verification, UI rendering, interaction testing
- [ ] Captures screenshots as evidence
- [ ] Reports pass/fail with screenshots to orchestrator

**5. Verify orchestrator final**:
- [ ] Marks macro-task completed after tester passes
- [ ] Moves to next macro-task
- [ ] Continues until ALL macro-tasks done
- [ ] Reports to user only when complete

---

## 📋 Quick Verification Commands

### Check if TASK-MANAGEMENT.md is referenced:
```bash
grep -r "TASK-MANAGEMENT.md" /Volumes/Coding/Code/platform/.claude/agents/
```

**Expected results**:
- CLAUDE.md: References in Platform Reference Links
- coder.md: References in workflow section
- tester.md: References in workflow section
- stuck.md: References in workflow section
- SDK-INTEGRATION.md: References in task management section

### Check if agents use TodoWrite:
```bash
grep -r "TodoWrite" /Volumes/Coding/Code/platform/.claude/agents/
```

**Expected results**:
- TASK-MANAGEMENT.md: Multiple examples
- coder.md: Micro-task execution pattern
- tester.md: Test micro-task pattern
- stuck.md: Diagnostic micro-task pattern

---

## ✅ Final Verification Checklist

### Files Created/Updated:
- [ ] `.claude/agents/TASK-MANAGEMENT.md` - Complete guide created
- [ ] `.claude/CLAUDE.md` - Orchestrator references task management
- [ ] `.claude/agents/coder.md` - Micro-task patterns integrated
- [ ] `.claude/agents/tester.md` - Test micro-task patterns integrated
- [ ] `.claude/agents/stuck.md` - Diagnostic micro-task patterns integrated
- [ ] `.claude/agents/SDK-INTEGRATION.md` - Task management examples added
- [ ] `.claude/agents/TASK-VERIFICATION.md` - This verification guide created

### System Integration:
- [ ] All agents reference TASK-MANAGEMENT.md
- [ ] Orchestrator delegates with task context
- [ ] Subagents create micro-tasks immediately
- [ ] Progress tracked with TodoWrite throughout
- [ ] Cross-agent communication includes task references

### Testing:
- [ ] Manual test scenario completed successfully
- [ ] All agents followed task patterns
- [ ] Zero fallbacks used
- [ ] Human can see progress at all times
- [ ] Tasks completed in correct sequence

---

## 🧹 Cleanup Verification Checklist

### For Coder Agents
- [ ] Track all test files created during task (keep list: `created_files=()`)
- [ ] Delete ONLY your created files, not all test files (scoped cleanup)
- [ ] Verify your test files deleted: `ls -1 "${created_files[@]}" 2>/dev/null` (should be empty)
- [ ] Confirm other test files untouched (don't delete legitimate tests or other agents' files)
- [ ] Pattern: `rm -f test-myfeature.mjs check-myfeature.mjs` (specific files only)

**Example Scoped Cleanup for Coder**:
```bash
# Track files created during task
created_files=(
  "test-notifications.mjs"
  "check-notifications.mjs"
)

# Delete ONLY your created files (scoped cleanup)
rm -f "${created_files[@]}"

# Verify deletion successful
remaining=$(ls -1 "${created_files[@]}" 2>/dev/null | wc -l)
if [ "$remaining" -eq 0 ]; then
  echo "✅ Cleanup successful: All created test files deleted"
else
  echo "⚠️ Cleanup incomplete: $remaining files remain"
fi

# Confirm legitimate tests preserved
echo "Preserved legitimate tests:"
ls -1 apps/web/src/**/__tests__/**/*.test.ts 2>/dev/null
```

### For Tester Agents
- [ ] Track all screenshots/artifacts created during test (keep list: `created_screenshots=()`)
- [ ] Delete ONLY your created artifacts, not all screenshots (scoped cleanup)
- [ ] Verify your artifacts deleted: `ls -d "${created_screenshots[@]}" 2>/dev/null` (should be empty)
- [ ] Confirm other test artifacts untouched (don't delete other tests' screenshots)
- [ ] Pattern: `rm -rf screenshots-mytest/ test-myfeature.png` (specific files only)

**Example Scoped Cleanup for Tester**:
```bash
# Track artifacts created during test
created_screenshots=(
  "screenshots-notifications/"
  "test-notification-flow.png"
)

# Delete ONLY your created artifacts (scoped cleanup)
rm -rf "${created_screenshots[@]}"

# Verify deletion successful
remaining=$(ls -d "${created_screenshots[@]}" 2>/dev/null | wc -l)
if [ "$remaining" -eq 0 ]; then
  echo "✅ Cleanup successful: All created test artifacts deleted"
else
  echo "⚠️ Cleanup incomplete: $remaining artifacts remain"
fi

# Confirm other test artifacts preserved
echo "Preserved other test artifacts:"
ls -1d screenshots-*/ 2>/dev/null | grep -v screenshots-notifications
```

### For Orchestrator
- [ ] Verify agents report deleting ONLY their own files (not global cleanup)
- [ ] Check git status shows no new untracked test files from completed tasks
- [ ] Confirm legitimate test files (`src/**/__tests__/**`) preserved
- [ ] Confirm other agents' in-progress test files preserved

**Orchestrator Cleanup Verification Command**:
```bash
# Check for untracked test files after agent cleanup
git status --short | grep "^??" | grep -E "\.(mjs|js|png)$"

# Expected: Empty output (all temp test files cleaned up)
# Red flag: Any test-*.mjs, check-*.mjs, or screenshots-*/ files
```

### Red Flags (Cleanup NOT Working):
- ❌ Agent deletes `apps/web/src/**/__tests__/**` directory (legitimate tests)
- ❌ Agent uses `rm -rf test-*.mjs` (too broad, deletes other agents' files)
- ❌ Agent uses `rm -rf screenshots-*/` (deletes all screenshot directories)
- ❌ Agent doesn't verify deletion with `ls` command
- ❌ Git status shows untracked test files after agent reports cleanup complete
- ❌ Other agents' in-progress test files deleted

### Success Criteria:
- ✅ Agent tracks specific files created during task execution
- ✅ Cleanup deletes ONLY tracked files (scoped to agent's work)
- ✅ Verification confirms deletion successful (zero remaining tracked files)
- ✅ Legitimate test files preserved (`src/**/__tests__/**/*.test.ts`)
- ✅ Other agents' test files preserved (not deleted by mistake)
- ✅ Git status shows zero new untracked test files after cleanup
- ✅ Orchestrator can verify cleanup was scoped correctly

---

**System Status**: ✅ READY FOR PRODUCTION USE

All agents now follow standardized macro/micro-task system with complete visibility, zero fallbacks, and scoped cleanup! 🚀
