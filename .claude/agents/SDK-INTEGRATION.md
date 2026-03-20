---
name: sdk-integration
description: Guide for integrating Anthropic's Agent SDK with TanStack Start platform agents, covering core concepts and workflow patterns.
---

# Agent SDK Integration Guide

**Purpose**: Comprehensive guide for integrating Anthropic's Agent SDK with the TanStack Start platform agents.

**Reference**: `/Agent SDK reference - TypeScript.md`

**Task Management**: All agents use **macro-task/micro-task system**. See `.claude/agents/TASK-MANAGEMENT.md` for task patterns.

---

## 📚 Agent SDK Overview

The Agent SDK provides programmatic control over Claude agent workflows with powerful capabilities:

### Core Concepts

**`query()`**: Execute agent workflows with custom configuration
- Primary entry point for agent execution
- Configure prompts, tools, models, and constraints
- Handle streaming responses and tool approvals

**`tool()`**: Create custom tools for platform-specific operations
- Define tool schemas with Zod validation
- Implement tool handlers with platform logic
- Register tools with agents via MCP servers

**`createSdkMcpServer()`**: Package tools into MCP servers
- Group related tools by domain (database, auth, RBAC)
- Version and namespace your tools
- Distribute reusable tool collections

### Key Features

1. **Tool Creation**: Define custom tools with typed inputs/outputs
2. **MCP Server Management**: Package and distribute tool collections
3. **Hook System**: Inject logic at specific execution points
4. **Permission Control**: Fine-grained tool access control
5. **Settings Management**: Project and global configuration
6. **Streaming Support**: Real-time response processing

---

## 🔧 Tool Creation Patterns

### Basic Tool Structure

```typescript
import { tool } from '@anthropic-ai/claude-agent-sdk';
import { z } from 'zod';

const myTool = tool(
  'toolName',           // Unique identifier
  'Tool description',   // What this tool does
  {                     // Input schema (Zod)
    param1: z.string(),
    param2: z.number().optional(),
  },
  async (args, extra) => {  // Handler function
    // Tool implementation
    return {
      content: [{
        type: 'text',
        text: JSON.stringify(result)
      }]
    };
  }
);
```

### Platform-Specific Tool Examples

#### RBAC Permission Checker

```typescript
import { tool } from '@anthropic-ai/claude-agent-sdk';
import { z } from 'zod';
import { db } from '@/db';
import { resolveUserAccess } from '@/db/utils/access-control';

const checkRBACTool = tool(
  'checkUserPermission',
  'Check if a user has specific RBAC permission',
  {
    userId: z.string().uuid(),
    permission: z.string(),
  },
  async (args, extra) => {
    const access = await resolveUserAccess(args.userId);
    const hasPermission = access.permissions.includes(args.permission);

    return {
      content: [{
        type: 'text',
        text: JSON.stringify({
          userId: args.userId,
          permission: args.permission,
          hasPermission,
          userRole: access.role,
          allPermissions: access.permissions,
        })
      }]
    };
  }
);
```

#### Database Migration Runner

```typescript
import { tool } from '@anthropic-ai/claude-agent-sdk';
import { z } from 'zod';
import { execSync } from 'child_process';

const runMigrationTool = tool(
  'runDatabaseMigration',
  'Execute pending Drizzle migrations on Neon database',
  {
    environment: z.enum(['local', 'production']).default('local'),
    dryRun: z.boolean().default(true),
  },
  async (args, extra) => {
    try {
      const command = args.environment === 'production'
        ? 'NODE_ENV=production pnpm db:migrate'
        : 'pnpm db:migrate';

      const output = args.dryRun
        ? 'DRY RUN: Would execute: ' + command
        : execSync(command, { encoding: 'utf-8' });

      return {
        content: [{
          type: 'text',
          text: JSON.stringify({
            success: true,
            environment: args.environment,
            dryRun: args.dryRun,
            output,
          })
        }]
      };
    } catch (error) {
      return {
        content: [{
          type: 'text',
          text: JSON.stringify({
            success: false,
            error: error.message,
            environment: args.environment,
          })
        }]
      };
    }
  }
);
```

#### WorkOS Auth Flow Tester

```typescript
import { tool } from '@anthropic-ai/claude-agent-sdk';
import { z } from 'zod';
import { getAuth, getSessionFromCookie } from '@/workosAuth/ssr/session';

const testAuthFlowTool = tool(
  'testWorkOSAuthFlow',
  'Test WorkOS authentication flow and session state',
  {
    userId: z.string().uuid().optional(),
  },
  async (args, extra) => {
    try {
      const auth = await getAuth();
      const session = await getSessionFromCookie();

      return {
        content: [{
          type: 'text',
          text: JSON.stringify({
            authenticated: !!auth,
            user: auth?.user || null,
            sessionExists: !!session,
            sessionMaxAge: session?.maxAge || null,
            cookieSet: !!session,
          })
        }]
      };
    } catch (error) {
      return {
        content: [{
          type: 'text',
          text: JSON.stringify({
            authenticated: false,
            error: error.message,
          })
        }]
      };
    }
  }
);
```

---

## 🌐 MCP Server Integration

### Creating Platform MCP Server

```typescript
import { createSdkMcpServer } from '@anthropic-ai/claude-agent-sdk';
import { checkRBACTool, runMigrationTool, testAuthFlowTool } from './tools';

const tanstackPlatformServer = createSdkMcpServer({
  name: 'tanstack-platform',
  version: '1.0.0',
  tools: [
    checkRBACTool,
    runMigrationTool,
    testAuthFlowTool,
  ],
});
```

### Using MCP Server in Agent Workflows

```typescript
import { query } from '@anthropic-ai/claude-agent-sdk';
import { tanstackPlatformServer } from './mcpServer';

const result = await query({
  prompt: "Check if user has admin access and run pending migrations",
  options: {
    mcpServers: {
      platform: tanstackPlatformServer,
    },
  },
});
```

### Organizing Tools by Domain

```typescript
// Database Tools MCP Server
const databaseServer = createSdkMcpServer({
  name: 'tanstack-database',
  version: '1.0.0',
  tools: [
    runMigrationTool,
    verifySchemaToolz,
    queryDatabaseTool,
    rollbackMigrationTool,
  ],
});

// Authentication Tools MCP Server
const authServer = createSdkMcpServer({
  name: 'tanstack-auth',
  version: '1.0.0',
  tools: [
    testAuthFlowTool,
    verifySessionTool,
    checkCookieTool,
  ],
});

// RBAC Tools MCP Server
const rbacServer = createSdkMcpServer({
  name: 'tanstack-rbac',
  version: '1.0.0',
  tools: [
    checkRBACTool,
    assignRoleTool,
    invalidateCacheTool,
    verifyAccessEntityTool,
  ],
});

// Use multiple servers together
const result = await query({
  prompt: "Complete platform health check",
  options: {
    mcpServers: {
      database: databaseServer,
      auth: authServer,
      rbac: rbacServer,
    },
  },
});
```

---

## 🪝 Hook System Integration

### PreToolUse Hook - Schema Validation

```typescript
import { query } from '@anthropic-ai/claude-agent-sdk';

const result = await query({
  prompt: "Create new user profile table",
  options: {
    hooks: {
      PreToolUse: [{
        hooks: [async (input, toolUseID, options) => {
          // Validate schema changes before writing
          if (input.tool_name === 'Write' &&
              input.tool_input.file_path.includes('/db/schema/')) {

            // Remind agent about migration requirements
            return {
              hookSpecificOutput: {
                hookEventName: 'PreToolUse',
                permissionDecision: 'allow',
                permissionDecisionReason: `
                  ⚠️ REMINDER: After modifying schema files:
                  1. Run: pnpm db:generate
                  2. Review SQL in /src/db/migrations/
                  3. Run: pnpm db:migrate

                  All tables MUST have: id, created_at, updated_at
                  All foreign keys MUST have indexes
                `,
              }
            };
          }

          return { continue: true };
        }]
      }]
    }
  }
});
```

### PostToolUse Hook - Auto Cache Invalidation

```typescript
import { query } from '@anthropic-ai/claude-agent-sdk';
import { clearAccessCache } from '@/db/utils/access-control';

const result = await query({
  prompt: "Update user roles and permissions",
  options: {
    hooks: {
      PostToolUse: [{
        hooks: [async (toolUse, toolResult, options) => {
          // Auto-invalidate RBAC cache after role changes
          if (toolUse.input.tool_name === 'runServerFunction' &&
              toolUse.input.tool_input.functionName.includes('assignUserRole')) {

            // Invalidate access cache
            await clearAccessCache();

            return {
              hookSpecificOutput: {
                hookEventName: 'PostToolUse',
                additionalContext: `
                  ✅ RBAC cache automatically invalidated after role assignment.
                  All users will get updated permissions on next request.
                `,
              }
            };
          }

          return { continue: true };
        }]
      }]
    }
  }
});
```

### PreAgentMessage Hook - Platform Context

```typescript
import { query } from '@anthropic-ai/claude-agent-sdk';

const result = await query({
  prompt: "Debug authentication issue",
  options: {
    hooks: {
      PreAgentMessage: [{
        hooks: [async (messages, options) => {
          // Inject platform context before agent responds
          return {
            hookSpecificOutput: {
              hookEventName: 'PreAgentMessage',
              additionalSystemPrompt: `
                Platform Context:
                - Framework: TanStack Start (SSR)
                - Database: Neon Postgres + Drizzle ORM
                - Auth: WorkOS AuthKit + iron-session
                - RBAC: Role-based access control

                Critical Rules:
                - Server functions MUST wrap data: { data: { ... } }
                - All schemas in /apps/web/src/db/schema/
                - Permissions use constants (no magic strings)
                - Migrations: generate → review → apply
              `,
            }
          };
        }]
      }]
    }
  }
});
```

---

## 🔐 Permission System Integration

### Custom Permission Function

```typescript
import { query, CanUseTool } from '@anthropic-ai/claude-agent-sdk';

const platformPermissions: CanUseTool = async (toolName, input, options) => {
  // Platform-specific permission logic

  // Block database migrations in production without confirmation
  if (toolName === 'Bash' && input.command.includes('db:migrate')) {
    const isProduction = input.command.includes('NODE_ENV=production');

    if (isProduction && !options.context?.migrationReviewed) {
      return {
        behavior: 'deny',
        message: `
          ⚠️ Production migration blocked!

          You must:
          1. Review migration SQL in /src/db/migrations/
          2. Verify schema changes are safe
          3. Set migrationReviewed: true in context
          4. Re-run with confirmation
        `,
      };
    }
  }

  // Block RBAC changes without cache invalidation plan
  if (toolName === 'runServerFunction' &&
      input.functionName.includes('assignUserRole')) {

    if (!options.context?.cacheInvalidationPlanned) {
      return {
        behavior: 'deny',
        message: `
          ⚠️ RBAC change blocked!

          You must:
          1. Plan cache invalidation strategy
          2. Set cacheInvalidationPlanned: true
          3. Ensure clearAccessCache() will be called
        `,
      };
    }
  }

  // Block schema modifications without migration
  if (toolName === 'Write' && input.file_path.includes('/db/schema/')) {
    if (!options.context?.migrationPlanned) {
      return {
        behavior: 'allow',
        updatedInput: input,
        message: `
          ℹ️ Schema modification detected.

          Remember to:
          1. Run: pnpm db:generate
          2. Review migration SQL
          3. Run: pnpm db:migrate
        `,
      };
    }
  }

  return {
    behavior: 'allow',
    updatedInput: input,
  };
};

// Use in query
const result = await query({
  prompt: "Update database schema",
  options: {
    canUseTool: platformPermissions,
    context: {
      migrationPlanned: true,
      migrationReviewed: false,
    }
  }
});
```

---

## 🤖 Agent Definition Patterns

### Programmatic Agent Definitions

```typescript
import { query } from '@anthropic-ai/claude-agent-sdk';

const platformAgents = {
  'database-specialist': {
    description: 'Handles all database schema and migration operations',
    tools: ['Read', 'Write', 'Edit', 'Bash', 'Grep', 'Glob'],
    prompt: `
      You are a database specialist for a TanStack Start platform.

      Your expertise:
      - Neon Postgres database management
      - Drizzle ORM schema definitions
      - Migration generation and execution
      - Schema standards enforcement

      Critical rules:
      - ALL schemas in /apps/web/src/db/schema/
      - Every table needs: id, created_at, updated_at
      - Index foreign keys + WHERE columns
      - Always: generate → review → apply migrations
    `,
    model: 'sonnet',
  },

  'auth-specialist': {
    description: 'Handles WorkOS authentication and session management',
    tools: ['Read', 'Bash', 'Grep'],
    prompt: `
      You are an authentication specialist for a TanStack Start platform.

      Your expertise:
      - WorkOS AuthKit integration
      - iron-session cookie management
      - OAuth flow debugging
      - Session persistence

      Critical rules:
      - Use getAuth() in route loaders
      - Use withAuth() in server functions
      - Session cookies encrypted with iron-session
      - Verify WORKOS_REDIRECT_URI matches exactly
    `,
    model: 'sonnet',
  },

  'rbac-specialist': {
    description: 'Handles role-based access control and permissions',
    tools: ['Read', 'Bash', 'Grep', 'Edit'],
    prompt: `
      You are an RBAC specialist for a TanStack Start platform.

      Your expertise:
      - Role-based access control configuration
      - Permission constant management
      - Access entity registration
      - Cache invalidation strategies

      Critical rules:
      - Permissions use constants from /src/db/utils/roles.ts
      - NEVER use magic strings for permissions
      - Register all features in access_entities table
      - Invalidate cache after role/permission changes
      - Resolution: (Role Defaults + Additional) - Denied
    `,
    model: 'sonnet',
  },
};

// Use specialized agent
const result = await query({
  prompt: "Fix RBAC access denied issue",
  options: {
    agentName: 'rbac-specialist',
    systemPrompt: {
      type: 'custom',
      custom: platformAgents['rbac-specialist'].prompt,
    },
  }
});
```

---

## ⚙️ Settings Source Configuration

### Project Settings

```typescript
import { query } from '@anthropic-ai/claude-agent-sdk';

const result = await query({
  prompt: "Build new feature",
  options: {
    // Load project-level settings from .claude/settings.json
    settingSources: ['project'],

    // Load platform instructions from CLAUDE.md
    systemPrompt: {
      type: 'preset',
      preset: 'claude_code',  // Loads .claude/CLAUDE.md
    },
  }
});
```

### Combined Settings

```typescript
import { query } from '@anthropic-ai/claude-agent-sdk';

const result = await query({
  prompt: "Complete platform feature implementation",
  options: {
    // Load both global and project settings
    settingSources: ['global', 'project'],

    // Combine with custom system prompt
    systemPrompt: {
      type: 'preset',
      preset: 'claude_code',
      additionalContext: `
        Current Task: Implementing new admin feature
        Platform: TanStack Start + Neon + WorkOS + RBAC

        Requirements:
        - Database schema with migrations
        - Protected route with auth checks
        - RBAC access control
        - Server functions with validation
        - Comprehensive testing
      `,
    },
  }
});
```

---

## 🎯 Complete Integration Example

### Full Platform Workflow

```typescript
import { query, createSdkMcpServer, tool } from '@anthropic-ai/claude-agent-sdk';
import { z } from 'zod';
import { tanstackPlatformServer } from './mcpServer';
import { platformPermissions } from './permissions';

// Execute complete feature implementation
const result = await query({
  prompt: `
    Implement a new "user preferences" feature:
    1. Create database schema with proper standards
    2. Generate and apply migration
    3. Create server functions with auth
    4. Create protected route with RBAC
    5. Register RBAC access entity
    6. Test complete flow with Playwright
  `,
  options: {
    // Load platform configuration
    settingSources: ['project'],
    systemPrompt: {
      type: 'preset',
      preset: 'claude_code',
    },

    // Enable platform tools
    mcpServers: {
      platform: tanstackPlatformServer,
    },

    // Apply platform permissions
    canUseTool: platformPermissions,

    // Platform-specific hooks
    hooks: {
      PreToolUse: [{
        hooks: [async (input, toolUseID, options) => {
          // Validate operations before execution
          if (input.tool_name === 'Write' &&
              input.tool_input.file_path.includes('/db/schema/')) {
            return {
              hookSpecificOutput: {
                hookEventName: 'PreToolUse',
                permissionDecision: 'allow',
                permissionDecisionReason: 'Remember migration workflow after schema changes',
              }
            };
          }
          return { continue: true };
        }]
      }],

      PostToolUse: [{
        hooks: [async (toolUse, toolResult, options) => {
          // Auto-invalidate cache after RBAC changes
          if (toolUse.input.tool_name.includes('assignUserRole')) {
            await clearAccessCache();
            return {
              hookSpecificOutput: {
                hookEventName: 'PostToolUse',
                additionalContext: 'RBAC cache invalidated automatically',
              }
            };
          }
          return { continue: true };
        }]
      }],
    },

    // Context for permission system
    context: {
      migrationPlanned: true,
      cacheInvalidationPlanned: true,
    },
  },
});

console.log(result.text);
```

---

## 📚 Best Practices

### Tool Design
- ✅ Use descriptive tool names and descriptions
- ✅ Validate inputs with Zod schemas
- ✅ Return structured JSON responses
- ✅ Handle errors gracefully with clear messages
- ✅ Include platform context in responses

### MCP Server Organization
- ✅ Group related tools by domain (DB, Auth, RBAC)
- ✅ Version your MCP servers
- ✅ Document tool capabilities and requirements
- ✅ Test tools independently before integration

### Hook Usage
- ✅ Use PreToolUse for validation and reminders
- ✅ Use PostToolUse for cleanup and follow-up
- ✅ Keep hooks focused and non-blocking
- ✅ Provide clear context in hook outputs

### Permission Control
- ✅ Implement platform-specific permission logic
- ✅ Block dangerous operations with clear explanations
- ✅ Allow operations with helpful reminders
- ✅ Use context to track approval state

### Settings Management
- ✅ Use project settings for platform configuration
- ✅ Leverage claude_code preset for platform instructions
- ✅ Combine settings sources as needed
- ✅ Provide additional context in system prompts

---

## 📋 Task Management Integration

All agents (orchestrator, coder, tester, stuck) follow a standardized **macro-task/micro-task system** when working with Agent SDK integrations.

### Macro-Task Example: "Implement RBAC Tool with Agent SDK"

**Orchestrator creates macro-task**:
```typescript
TodoWrite([{
  content: "Implement RBAC checking tool using Agent SDK",
  status: "pending",
  activeForm: "Implementing RBAC tool"
}]);

// Delegate to coder with SDK context
Task({
  subagent_type: "coder",
  description: "Implement RBAC tool",
  prompt: `
    Create an RBAC checking tool using Agent SDK.
    Reference SDK-INTEGRATION.md section 2 for tool() pattern.
    Break into micro-tasks and track with TodoWrite.
  `
});
```

### Micro-Task Breakdown (Coder)

**Coder breaks macro-task into micro-tasks**:
```typescript
TodoWrite([
  {
    content: "Create RBAC tool with tool() function and Zod schema",
    status: "in_progress",
    activeForm: "Creating RBAC tool"
  },
  {
    content: "Implement handler with resolveUserAccess() integration",
    status: "pending",
    activeForm: "Implementing handler"
  },
  {
    content: "Package into MCP server with createSdkMcpServer()",
    status: "pending",
    activeForm: "Packaging MCP server"
  },
  {
    content: "Add PreToolUse hook for permission validation",
    status: "pending",
    activeForm: "Adding validation hook"
  },
  {
    content: "Test tool integration with sample query",
    status: "pending",
    activeForm: "Testing tool"
  }
]);

// Execute each micro-task, mark completed immediately after
```

### Testing Micro-Tasks (Tester)

**Tester creates test micro-tasks**:
```typescript
TodoWrite([
  {
    content: "Verify RBAC tool registered in MCP server",
    status: "in_progress",
    activeForm: "Verifying registration"
  },
  {
    content: "Test tool with admin user permissions",
    status: "pending",
    activeForm: "Testing admin access"
  },
  {
    content: "Test tool with non-admin user permissions",
    status: "pending",
    activeForm: "Testing non-admin access"
  },
  {
    content: "Verify PreToolUse hook validates correctly",
    status: "pending",
    activeForm: "Testing hook validation"
  }
]);
```

**Reference**: `.claude/agents/TASK-MANAGEMENT.md` for complete task patterns

---

## 🔗 References

- **Agent SDK Docs**: `/Agent SDK reference - TypeScript.md`
- **Task Management**: `/.claude/agents/TASK-MANAGEMENT.md` (Macro/micro-task system)
- **Platform Architecture**: `/apps/web/docs/structure.md`
- **Platform Instructions**: `/.claude/CLAUDE.md`
- **Orchestrator Agent**: `/.claude/CLAUDE.md`
- **Coder Agent**: `/.claude/agents/coder.md`
- **Tester Agent**: `/.claude/agents/tester.md`
- **Stuck Agent**: `/.claude/agents/stuck.md`
