---
title: Use internal functions for scheduling and ctx.run calls
impact: CRITICAL
tags: [convex, security, internal]
---

# Use Internal Functions for Scheduling and ctx.run Calls

Always use `internal.*` (never `api.*`) for `ctx.scheduler.runAfter`, `ctx.runQuery`, `ctx.runMutation`, `ctx.runAction`, and cron job targets. Public `api.*` functions can be called by anyone.

## Incorrect

```typescript
// Exposes function to unauthenticated callers
await ctx.scheduler.runAfter(0, api.tasks.process, { id });
await ctx.runMutation(api.users.updateRole, { userId, role: "admin" });

// In crons.ts
crons.interval("cleanup", { hours: 1 }, api.cleanup.run, {});
```

## Correct

```typescript
// Internal functions are server-only
await ctx.scheduler.runAfter(0, internal.tasks.process, { id });
await ctx.runMutation(internal.users.updateRole, { userId, role: "admin" });

// In crons.ts
crons.interval("cleanup", { hours: 1 }, internal.cleanup.run, {});
```
