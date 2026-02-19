---
name: convex
description: Convex backend development - queries, mutations, actions, schemas, indexes, realtime subscriptions, optimistic updates, file storage, HTTP endpoints, webhooks, cron jobs, migrations, AI/LLM agents, RAG, security, authentication, authorization, rate limiting, error handling with ConvexError, code review, convex-helpers, custom functions, triggers, row-level security, relationship helpers, Hono, CORS, Zod validation, CRUD, manual pagination, QueryStreams, query caching, sessions, workpool, components, RBAC, audit trails. Use when writing Convex functions, defining schemas, building real-time features, integrating external APIs, handling file uploads, setting up scheduled jobs, performing database migrations, building AI chat interfaces, reviewing Convex code for production readiness, using convex-helpers patterns, or building reusable Convex components.
---

# Convex Development

## Workflow

1. **Starting a project** - See [patterns](references/patterns.md) for structure, types, validators
2. **Database queries** - See [queries](references/queries.md) for indexes, filtering, performance
3. **Actions and scheduling** - See [actions](references/actions.md) for external APIs, transactions
4. **Realtime features** - See [realtime](references/realtime.md) for subscriptions, optimistic updates
5. **HTTP endpoints** - See [http](references/http.md) for webhooks, REST, CORS
6. **File handling** - See [file-storage](references/file-storage.md) for uploads, serving
7. **Background jobs** - See [cron](references/cron.md) for scheduled functions
8. **Schema changes** - See [migrations](references/migrations.md) for data evolution
9. **AI integration** - See [agents](references/agents.md) for LLM chat, RAG, tool calling
10. **Security** - See [security](references/security.md) for auth, ConvexError, ESLint, access control
11. **Code review** - See [review](references/review.md) for production readiness checklist
12. **Components** - See [components](references/components.md) for reusable Convex packages

### convex-helpers (npm package)

13. **Custom functions & Zod** - See [helpers-functions](references/helpers-functions.md) for middleware, auth wrappers, Zod validation
14. **Data helpers** - See [helpers-data](references/helpers-data.md) for relationships, CRUD, triggers, filter, pagination
15. **Security helpers** - See [helpers-security](references/helpers-security.md) for RLS, rate limiting, sessions, CORS
16. **Async helpers** - See [helpers-async](references/helpers-async.md) for workpool, action retries, migrations
17. **Validation helpers** - See [helpers-validation](references/helpers-validation.md) for validator utils, Standard Schema, codegen
18. **Integration helpers** - See [helpers-integrations](references/helpers-integrations.md) for Hono, query caching, useQuery, QueryStreams

## Key Principles

- Always define `args` AND `returns` validators on every function
- Use `ConvexError` with structured codes, not plain `Error`
- Use `.withIndex()` not `.filter()` for database queries
- Use `internal.*` (never `api.*`) for scheduling and `ctx.run*` calls
- Use `"use node";` only when you need Node.js APIs
- Keep business logic in `convex/model/` helpers, thin public API wrappers
- Always `await` promises (`ctx.db.*`, `ctx.scheduler.*`)
- Install `@convex-dev/eslint-plugin` for build-time validation

## Rule Categories by Priority

| Priority | Category | Impact | Prefix |
|----------|----------|--------|--------|
| 1 | Security | CRITICAL | `security-` |
| 2 | Validation | HIGH | `validation-` |
| 3 | Performance | HIGH | `performance-` |
| 4 | Code Quality | MEDIUM | `code-quality-` |

### 1. Security (CRITICAL)

- `security-auth-check` - Always check authentication in public functions
- `security-internal-functions` - Use internal functions for scheduling and ctx.run calls
- `security-row-level-access` - Verify row-level ownership before mutations
- `security-convex-error` - Use ConvexError instead of plain Error
- `security-rate-limiting` - Implement rate limiting for public mutations
- `security-table-name` - Include table name in ctx.db calls

### 2. Validation (HIGH)

- `validation-return-types` - Always define return validators on functions

### 3. Performance (HIGH)

- `performance-no-filter` - Use .withIndex() instead of .filter() on queries
- `performance-bounded-collect` - Only use .collect() with bounded result sets
- `performance-no-date-now` - Don't use Date.now() in query functions

### 4. Code Quality (MEDIUM)

- `code-quality-await-promises` - Always await async operations

## How to Use

Read individual rule files for detailed explanations and code examples:

```
rules/security-auth-check.md
rules/performance-no-filter.md
```

Each rule file contains:
- Brief explanation of why it matters
- Incorrect code example with explanation
- Correct code example with explanation
