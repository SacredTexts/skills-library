---
name: speed-optimization
description: Performance optimization knowledge base with async SQLite-based tracking for measuring and improving agent execution speed.
---

# SPEED-OPTIMIZATION.md - Performance Engineering Knowledge Base

**Purpose**: Comprehensive performance optimization reference for speed agent with SQLite-based performance tracking and learning system.

**Zero-Overhead Design**: All performance tracking is ASYNC and NEVER blocks agent execution.

---

## 📊 SQLite Performance Tracking System

### Database Schema

**Location**: `.claude/performance.db` (gitignored, local-only)

**Schema**:
```sql
-- Agent execution records
CREATE TABLE IF NOT EXISTS agent_executions (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  agent_type TEXT NOT NULL, -- coder, tester, monitor, stuck, speed
  task_description TEXT NOT NULL,
  started_at REAL NOT NULL, -- Unix timestamp with milliseconds
  completed_at REAL, -- NULL if still running
  duration_ms INTEGER, -- Calculated on completion
  status TEXT NOT NULL, -- in_progress, completed, failed, timeout
  micro_task_count INTEGER DEFAULT 0,
  parallel_batch_size INTEGER DEFAULT 1,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Performance bottlenecks
CREATE TABLE IF NOT EXISTS bottlenecks (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  execution_id INTEGER NOT NULL,
  bottleneck_type TEXT NOT NULL, -- io, context, tool, network, computation
  location TEXT, -- file path, function name, tool name
  duration_ms INTEGER NOT NULL,
  percentage REAL, -- % of total execution time
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (execution_id) REFERENCES agent_executions(id)
);

-- Applied optimizations
CREATE TABLE IF NOT EXISTS optimizations (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  optimization_type TEXT NOT NULL, -- caching, parallel, streaming, skipping, predictive, context
  target_agent TEXT NOT NULL, -- which agent type
  applied_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  baseline_ms INTEGER NOT NULL, -- Before optimization
  optimized_ms INTEGER, -- After optimization (NULL if not measured yet)
  speedup_percent REAL, -- Calculated improvement
  quality_preserved BOOLEAN DEFAULT 1, -- Did quality metrics stay ≥99.5%?
  rollback_reason TEXT, -- If rolled back, why?
  success BOOLEAN DEFAULT 1
);

-- Quality metrics
CREATE TABLE IF NOT EXISTS quality_metrics (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  execution_id INTEGER NOT NULL,
  test_pass_rate REAL, -- Percentage (0-100)
  false_positive_rate REAL, -- Percentage (0-100)
  bug_introduction BOOLEAN DEFAULT 0,
  measured_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (execution_id) REFERENCES agent_executions(id)
);

-- Task patterns (for predictive loading)
CREATE TABLE IF NOT EXISTS task_patterns (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  current_task TEXT NOT NULL,
  next_task TEXT NOT NULL,
  frequency INTEGER DEFAULT 1, -- How often this sequence occurs
  avg_delay_ms INTEGER, -- Average time between tasks
  last_seen DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Indexes for performance
CREATE INDEX IF NOT EXISTS idx_agent_type ON agent_executions(agent_type);
CREATE INDEX IF NOT EXISTS idx_status ON agent_executions(status);
CREATE INDEX IF NOT EXISTS idx_started_at ON agent_executions(started_at);
CREATE INDEX IF NOT EXISTS idx_optimization_type ON optimizations(optimization_type);
CREATE INDEX IF NOT EXISTS idx_task_pattern ON task_patterns(current_task, next_task);
```

### Async Logging Pattern (ZERO OVERHEAD)

**Critical**: NEVER block agent execution for logging!

```bash
# Log execution start (background process)
log_execution_start() {
  local agent_type=$1
  local task_desc=$2
  local started_at=$(date +%s.%N)

  # Fire and forget (runs in background)
  sqlite3 .claude/performance.db "
    INSERT INTO agent_executions (agent_type, task_description, started_at, status)
    VALUES ('$agent_type', '$task_desc', $started_at, 'in_progress');
    SELECT last_insert_rowid();
  " &
}

# Log execution completion (background process)
log_execution_complete() {
  local execution_id=$1
  local completed_at=$(date +%s.%N)

  # Fire and forget
  sqlite3 .claude/performance.db "
    UPDATE agent_executions
    SET completed_at = $completed_at,
        duration_ms = CAST(($completed_at - started_at) * 1000 AS INTEGER),
        status = 'completed'
    WHERE id = $execution_id;
  " &
}

# Log bottleneck (background process)
log_bottleneck() {
  local execution_id=$1
  local type=$2
  local location=$3
  local duration_ms=$4

  # Fire and forget
  sqlite3 .claude/performance.db "
    INSERT INTO bottlenecks (execution_id, bottleneck_type, location, duration_ms)
    VALUES ($execution_id, '$type', '$location', $duration_ms);
  " &
}
```

**Usage in agents**:
```typescript
// Coder agent example
async function executeMicroTask(task: MicroTask) {
  // Non-blocking log start
  const executionId = await logExecutionStart("coder", task.content);

  const startTime = performance.now();

  try {
    // DO THE WORK
    await implementTask(task);

    // Non-blocking log completion
    logExecutionComplete(executionId, performance.now() - startTime);

  } catch (error) {
    // Non-blocking log failure
    logExecutionFailure(executionId, error);
    throw error;
  }
}
```

---

## 🔬 Performance Measurement Methodologies

### 1. Agent Execution Time Profiling

**Goal**: Measure actual vs target execution times per agent type

**Tools**:
```bash
# Measure coder micro-task
time Task({ subagent_type: "coder", ... })

# Hyperfine for statistical accuracy
hyperfine --warmup 3 --runs 10 'coder-task-command'

# Built-in timing
start=$(date +%s.%N)
# ... execute task ...
end=$(date +%s.%N)
duration=$(echo "$end - $start" | bc)
```

**Queries**:
```sql
-- Average execution time by agent type
SELECT
  agent_type,
  COUNT(*) as executions,
  AVG(duration_ms) as avg_ms,
  MIN(duration_ms) as min_ms,
  MAX(duration_ms) as max_ms,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY duration_ms) as p50_ms,
  PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY duration_ms) as p95_ms
FROM agent_executions
WHERE status = 'completed'
  AND started_at > strftime('%s', 'now', '-24 hours')
GROUP BY agent_type;

-- Slowest executions
SELECT
  agent_type,
  task_description,
  duration_ms,
  micro_task_count,
  parallel_batch_size,
  started_at
FROM agent_executions
WHERE status = 'completed'
ORDER BY duration_ms DESC
LIMIT 20;
```

### 2. Bottleneck Identification

**Goal**: Find specific operations consuming most time

**Tools**:
```bash
# Profile I/O bottlenecks
iostat -x 1 5 | tee io-profile.txt

# Profile CPU bottlenecks
top -b -n 5 | tee cpu-profile.txt

# Profile network bottlenecks (MCP servers)
time curl -X POST https://mcp-server.com/api

# Trace system calls
strace -c -f command
```

**Queries**:
```sql
-- Most common bottlenecks
SELECT
  bottleneck_type,
  COUNT(*) as occurrences,
  AVG(duration_ms) as avg_duration_ms,
  SUM(duration_ms) as total_duration_ms,
  AVG(percentage) as avg_percentage
FROM bottlenecks
WHERE created_at > datetime('now', '-7 days')
GROUP BY bottleneck_type
ORDER BY total_duration_ms DESC;

-- Bottlenecks by location
SELECT
  location,
  bottleneck_type,
  COUNT(*) as count,
  AVG(duration_ms) as avg_ms
FROM bottlenecks
WHERE location IS NOT NULL
GROUP BY location, bottleneck_type
ORDER BY count DESC
LIMIT 30;
```

### 3. Optimization Impact Analysis

**Goal**: Measure actual speedup from each optimization

**Queries**:
```sql
-- Optimization success rate
SELECT
  optimization_type,
  COUNT(*) as applied,
  SUM(CASE WHEN success = 1 THEN 1 ELSE 0 END) as successes,
  ROUND(100.0 * SUM(CASE WHEN success = 1 THEN 1 ELSE 0 END) / COUNT(*), 2) as success_rate,
  AVG(speedup_percent) as avg_speedup,
  SUM(CASE WHEN quality_preserved = 0 THEN 1 ELSE 0 END) as quality_degradations
FROM optimizations
WHERE applied_at > datetime('now', '-30 days')
GROUP BY optimization_type
ORDER BY avg_speedup DESC;

-- Best optimizations
SELECT
  optimization_type,
  target_agent,
  baseline_ms,
  optimized_ms,
  speedup_percent,
  quality_preserved,
  applied_at
FROM optimizations
WHERE success = 1
  AND quality_preserved = 1
  AND speedup_percent > 20
ORDER BY speedup_percent DESC
LIMIT 20;
```

### 4. Quality Preservation Tracking

**Goal**: Ensure optimizations don't degrade quality

**Queries**:
```sql
-- Quality metrics over time
SELECT
  DATE(qm.measured_at) as date,
  AVG(qm.test_pass_rate) as avg_pass_rate,
  AVG(qm.false_positive_rate) as avg_false_positive,
  SUM(CASE WHEN qm.bug_introduction = 1 THEN 1 ELSE 0 END) as bugs_introduced,
  COUNT(*) as measurements
FROM quality_metrics qm
WHERE qm.measured_at > datetime('now', '-30 days')
GROUP BY DATE(qm.measured_at)
ORDER BY date DESC;

-- Quality degradation after optimizations
SELECT
  o.optimization_type,
  o.applied_at,
  qm.test_pass_rate,
  qm.false_positive_rate,
  qm.bug_introduction
FROM optimizations o
JOIN agent_executions ae ON ae.started_at > o.applied_at
LEFT JOIN quality_metrics qm ON qm.execution_id = ae.id
WHERE o.applied_at > datetime('now', '-7 days')
  AND qm.test_pass_rate < 99.5
ORDER BY o.applied_at DESC;
```

---

## 🎯 Optimization Techniques Catalog

### Technique 1: Intelligent Caching

**When to use**: Repeated reads of same data

**Speedup potential**: 30-50%

**Risk level**: Low

**Implementation**:
```typescript
// In-memory cache with TTL
const cache = new Map<string, { value: any; timestamp: number; hits: number }>();

async function cachedOperation<T>(
  key: string,
  fetcher: () => Promise<T>,
  ttl: number = 60000
): Promise<T> {
  const cached = cache.get(key);

  if (cached && Date.now() - cached.timestamp < ttl) {
    // Log cache hit (async, non-blocking)
    logCacheHit(key);
    cached.hits++;
    return cached.value;
  }

  // Cache miss - fetch data
  const value = await fetcher();
  cache.set(key, { value, timestamp: Date.now(), hits: 0 });

  // Log cache miss (async, non-blocking)
  logCacheMiss(key);

  return value;
}

// Invalidation on writes
function invalidateCache(pattern: string) {
  for (const [key] of cache) {
    if (key.match(pattern)) {
      cache.delete(key);
      logCacheInvalidation(key);
    }
  }
}
```

**Common cache targets**:
- Schema files (`/src/db/schema/*.ts`)
- Package.json
- RBAC permission constants
- Environment variables
- Configuration files

**Quality preservation**:
- Invalidate on file writes
- Use file checksums for staleness detection
- TTL prevents infinite staleness
- Monitor cache hit rate

**Tracking**:
```sql
-- Cache effectiveness
SELECT
  DATE(created_at) as date,
  COUNT(*) as cache_operations,
  SUM(CASE WHEN event_type = 'hit' THEN 1 ELSE 0 END) as hits,
  SUM(CASE WHEN event_type = 'miss' THEN 1 ELSE 0 END) as misses,
  ROUND(100.0 * SUM(CASE WHEN event_type = 'hit' THEN 1 ELSE 0 END) / COUNT(*), 2) as hit_rate
FROM cache_events
WHERE created_at > datetime('now', '-7 days')
GROUP BY DATE(created_at);
```

### Technique 2: Parallel Enhancement

**When to use**: Dependency analysis reveals hidden parallelism

**Speedup potential**: 2-5×

**Risk level**: Low

**Implementation**:
```typescript
// Enhanced dependency analysis
function findAdditionalParallelism(tasks: MicroTask[]) {
  const graph = buildDependencyGraph(tasks);

  // Level 1: File-level independence (current)
  const fileLevel = findIndependentFiles(tasks);

  // Level 2: Function-level independence (NEW)
  const functionLevel = findIndependentFunctions(tasks);

  // Level 3: Region-level independence (NEW)
  const regionLevel = findIndependentRegions(tasks);

  // Combine all levels
  const allIndependent = [...fileLevel, ...functionLevel, ...regionLevel];

  // Log parallelization opportunity
  logParallelizationOpportunity(
    tasks.length,
    allIndependent.length,
    allIndependent.length / tasks.length
  );

  return allIndependent;
}
```

**Quality preservation**:
- Conservative dependency detection (assume dependent if unsure)
- Static analysis validation
- Conflict detection and rollback
- Test all parallel groups independently

**Tracking**:
```sql
-- Parallelization effectiveness
SELECT
  agent_type,
  AVG(parallel_batch_size) as avg_batch_size,
  AVG(duration_ms / micro_task_count) as avg_ms_per_task,
  COUNT(*) as executions
FROM agent_executions
WHERE parallel_batch_size > 1
  AND status = 'completed'
GROUP BY agent_type;
```

### Technique 3: Streaming Operations

**When to use**: Large file reads, database queries, test results

**Speedup potential**: 40-60%

**Risk level**: Low

**Implementation**:
```typescript
// Streaming test execution
async function* streamTestExecution(scenarios: TestScenario[]) {
  for (const scenario of scenarios) {
    const startTime = performance.now();
    const result = await executeTest(scenario);
    const duration = performance.now() - startTime;

    // Log test result (async)
    logTestResult(scenario.name, duration, result.passed);

    yield result;

    // Early termination on failure
    if (!result.passed && scenario.critical) {
      break;
    }
  }
}

// Consumer can process as results arrive
for await (const result of streamTestExecution(testScenarios)) {
  updateProgress(result);
  if (!result.passed) {
    // Early feedback
    notifyFailure(result);
  }
}
```

**Quality preservation**:
- All data still processed (just incrementally)
- Early termination preserves failure detection
- Progress updates improve UX

**Tracking**:
```sql
-- Streaming vs batch performance
SELECT
  task_description,
  execution_mode, -- streaming or batch
  AVG(duration_ms) as avg_duration,
  AVG(duration_ms / micro_task_count) as avg_per_task
FROM agent_executions
WHERE agent_type = 'tester'
GROUP BY execution_mode;
```

### Technique 4: Smart Skipping

**When to use**: Redundant operations on unchanged code

**Speedup potential**: 20-30%

**Risk level**: Medium

**Implementation**:
```typescript
// Content-addressable result caching
const resultCache = new Map<string, { result: any; timestamp: number }>();

async function smartExecute<T>(
  files: string[],
  operation: () => Promise<T>
): Promise<T> {
  // Hash file contents
  const contentHash = await hashFiles(files);

  // Check cache
  const cached = resultCache.get(contentHash);
  if (cached && Date.now() - cached.timestamp < 3600000) {
    // Log skip (async)
    logSmartSkip(files, contentHash);
    return cached.result;
  }

  // Execute operation
  const result = await operation();

  // Cache result
  resultCache.set(contentHash, { result, timestamp: Date.now() });

  // Log execution (async)
  logSmartExecution(files, contentHash);

  return result;
}
```

**Quality preservation**:
- Use cryptographic hashes (SHA-256)
- Include dependencies in hash (imports, configs)
- Clear cache on external changes
- Validate cache correctness periodically

**Tracking**:
```sql
-- Smart skip effectiveness
SELECT
  DATE(created_at) as date,
  COUNT(*) as total_operations,
  SUM(CASE WHEN skipped = 1 THEN 1 ELSE 0 END) as skipped,
  ROUND(100.0 * SUM(CASE WHEN skipped = 1 THEN 1 ELSE 0 END) / COUNT(*), 2) as skip_rate,
  AVG(CASE WHEN skipped = 1 THEN 0 ELSE duration_ms END) as avg_execution_ms
FROM smart_operations
WHERE created_at > datetime('now', '-30 days')
GROUP BY DATE(created_at);
```

### Technique 5: Predictive Loading

**When to use**: Predictable task sequences

**Speedup potential**: 15-25%

**Risk level**: Low

**Implementation**:
```typescript
// Learn task patterns
function recordTaskSequence(currentTask: string, nextTask: string) {
  // Async update (non-blocking)
  sqlite3('.claude/performance.db', `
    INSERT INTO task_patterns (current_task, next_task, frequency)
    VALUES ('${currentTask}', '${nextTask}', 1)
    ON CONFLICT (current_task, next_task)
    DO UPDATE SET
      frequency = frequency + 1,
      last_seen = CURRENT_TIMESTAMP;
  `);
}

// Predict and prefetch
async function prefetchForTask(currentTask: string) {
  // Query likely next tasks (async)
  const likelyNext = await sqlite3('.claude/performance.db', `
    SELECT next_task, frequency
    FROM task_patterns
    WHERE current_task = '${currentTask}'
    ORDER BY frequency DESC
    LIMIT 5;
  `);

  // Prefetch in background (non-blocking)
  for (const { next_task } of likelyNext) {
    Promise.resolve().then(() => warmCacheForTask(next_task));
  }
}
```

**Quality preservation**:
- Prefetch is async (never blocks)
- Cache warming is best-effort (failures ignored)
- No impact if prediction wrong

**Tracking**:
```sql
-- Prefetch accuracy
SELECT
  current_task,
  next_task,
  frequency,
  SUM(CASE WHEN actually_used = 1 THEN 1 ELSE 0 END) as correct_predictions,
  ROUND(100.0 * SUM(CASE WHEN actually_used = 1 THEN 1 ELSE 0 END) / frequency, 2) as accuracy
FROM task_patterns tp
LEFT JOIN prefetch_events pe ON pe.current_task = tp.current_task AND pe.next_task = tp.next_task
WHERE tp.last_seen > datetime('now', '-7 days')
GROUP BY current_task, next_task
ORDER BY frequency DESC
LIMIT 50;
```

### Technique 6: Context Optimization

**When to use**: Large context windows, verbose prompts

**Speedup potential**: 10-20%

**Risk level**: Medium

**Implementation**:
```typescript
// Smart context compression
function optimizeContext(context: string, taskType: string): string {
  // Remove redundant sections
  let optimized = removeRedundantExamples(context);

  // Summarize verbose explanations
  optimized = summarizeVerboseSections(optimized, 500); // Max 500 chars per section

  // Keep only relevant sections
  const relevantSections = getRelevantSections(taskType);
  optimized = extractSections(optimized, relevantSections);

  // Log compression ratio (async)
  const originalSize = context.length;
  const compressedSize = optimized.length;
  logContextOptimization(originalSize, compressedSize, taskType);

  return optimized;
}
```

**Quality preservation**:
- A/B test optimized vs full context
- Measure task success rate (must be ≥99.5%)
- Rollback if quality degrades
- Human review for risky optimizations

**Tracking**:
```sql
-- Context optimization impact
SELECT
  task_type,
  AVG(original_size) as avg_original,
  AVG(compressed_size) as avg_compressed,
  ROUND(100.0 * (1 - AVG(compressed_size) / AVG(original_size)), 2) as compression_ratio,
  AVG(task_success_rate) as avg_success_rate
FROM context_optimizations
WHERE created_at > datetime('now', '-30 days')
GROUP BY task_type;
```

---

## 🎯 Performance Targets & Thresholds

### Agent-Specific Targets

**Coder Agent**:
```yaml
Current: 2-3s per micro-task
Target: 1-2s per micro-task
Speedup: 33-50%

Bottleneck priorities:
1. Schema file reads (30% of time)
2. Tool initialization (20% of time)
3. File I/O operations (15% of time)
4. Context loading (10% of time)
```

**Tester Agent**:
```yaml
Current: 5-8s per scenario
Target: 3-4s per scenario
Speedup: 37-60%

Bottleneck priorities:
1. Playwright browser startup (40% of time)
2. Screenshot capture (20% of time)
3. Database queries (15% of time)
4. Console log parsing (10% of time)
```

**Monitor Agent**:
```yaml
Current: 0.5-1s detection
Target: 0.3-0.5s detection
Speedup: 40-67%

Bottleneck priorities:
1. Polling interval (50% of time)
2. Solver spawn overhead (25% of time)
3. Result aggregation (15% of time)
```

**Orchestrator**:
```yaml
Current: 1-2s planning
Target: 0.5-1s planning
Speedup: 50%

Bottleneck priorities:
1. TodoWrite serialization (35% of time)
2. Dependency analysis (30% of time)
3. Context window management (20% of time)
```

### Quality Thresholds

**Test Pass Rate**:
```yaml
Baseline: ≥99.5%
Warning: <99.5%
Critical: <99.0%
Action: Rollback optimization
```

**False Positive Rate**:
```yaml
Baseline: ≤0.1%
Warning: >0.1%
Critical: >0.5%
Action: Investigate optimization impact
```

**Bug Introduction Rate**:
```yaml
Baseline: ≤0.01%
Warning: >0.01%
Critical: >0.05%
Action: Rollback all recent optimizations
```

---

## 🔍 SQLite Query Patterns for Speed Agent

### Daily Performance Summary

```sql
-- Daily performance summary
SELECT
  DATE(ae.started_at) as date,
  ae.agent_type,
  COUNT(*) as executions,
  AVG(ae.duration_ms) as avg_duration_ms,
  MIN(ae.duration_ms) as min_duration_ms,
  MAX(ae.duration_ms) as max_duration_ms,
  AVG(qm.test_pass_rate) as avg_test_pass_rate
FROM agent_executions ae
LEFT JOIN quality_metrics qm ON qm.execution_id = ae.id
WHERE ae.started_at > strftime('%s', 'now', '-7 days')
  AND ae.status = 'completed'
GROUP BY DATE(ae.started_at), ae.agent_type
ORDER BY date DESC, ae.agent_type;
```

### Optimization Opportunities

```sql
-- Find optimization opportunities
SELECT
  ae.agent_type,
  ae.task_description,
  COUNT(*) as frequency,
  AVG(ae.duration_ms) as avg_duration_ms,
  b.bottleneck_type,
  AVG(b.duration_ms) as bottleneck_avg_ms,
  AVG(b.percentage) as bottleneck_percentage
FROM agent_executions ae
JOIN bottlenecks b ON b.execution_id = ae.id
WHERE ae.started_at > strftime('%s', 'now', '-24 hours')
  AND ae.duration_ms > (
    SELECT AVG(duration_ms) * 1.5
    FROM agent_executions
    WHERE agent_type = ae.agent_type
  )
GROUP BY ae.agent_type, b.bottleneck_type
HAVING COUNT(*) >= 3
ORDER BY frequency DESC, bottleneck_avg_ms DESC
LIMIT 20;
```

### Learning System Queries

```sql
-- Most effective optimizations
SELECT
  optimization_type,
  COUNT(*) as applications,
  AVG(speedup_percent) as avg_speedup,
  MIN(baseline_ms) as min_baseline,
  MAX(baseline_ms) as max_baseline,
  SUM(CASE WHEN quality_preserved = 1 THEN 1 ELSE 0 END) as quality_preserved_count
FROM optimizations
WHERE success = 1
  AND applied_at > datetime('now', '-90 days')
GROUP BY optimization_type
ORDER BY avg_speedup DESC;

-- Task sequences for predictive loading
SELECT
  current_task,
  next_task,
  frequency,
  AVG(avg_delay_ms) as avg_delay
FROM task_patterns
WHERE frequency >= 5
  AND last_seen > datetime('now', '-30 days')
ORDER BY frequency DESC
LIMIT 100;
```

---

## 📚 References

**Speed Agent**: `.claude/agents/speed.md` - Primary optimization agent
**Task Management**: `.claude/agents/TASK-MANAGEMENT.md` - Task tracking patterns
**SDK Integration**: `.claude/agents/SDK-INTEGRATION.md` - Tool creation and MCP servers
**Orchestrator**: `.claude/CLAUDE.md` - Master orchestration and parallelization

---

**This knowledge base enables data-driven performance optimization through continuous measurement, learning, and improvement - all with ZERO overhead to agent execution!** ⚡📊🚀
