# Start Dev Server

Kill all platform and plugin worker services, clear caches, and start web + worker.

## Instructions

### Step 1: Kill all services and free ports

```bash
pkill -9 -f "concurrently.*ista" 2>/dev/null; pkill -9 -f "@ista/web" 2>/dev/null; pkill -9 -f "@ista/worker" 2>/dev/null; pkill -9 -f "vite" 2>/dev/null; pkill -9 -f "esbuild" 2>/dev/null; pkill -9 -f "tsx" 2>/dev/null; pkill -9 -f "worker-service.cjs --daemon" 2>/dev/null; lsof -ti:3000 | xargs kill -9 2>/dev/null; echo "All services killed"
```

### Step 2: Clear caches

```bash
rm -rf /Volumes/Coding/Code/platform/apps/web/node_modules/.vite && echo "Caches cleared"
```

### Step 3: Start all services in background

```bash
nohup pnpm --dir /Volumes/Coding/Code/platform run dev:full > /tmp/platform-dev.log 2>&1 &
```

Then confirm: "Dev server starting at http://localhost:3000 (web + worker). Log at `/tmp/platform-dev.log`."
