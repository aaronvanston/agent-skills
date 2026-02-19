---
title: Always await async operations
impact: MEDIUM
tags: [convex, code-quality, promises]
---

# Always Await Async Operations

Unawaited promises in Convex functions silently fail. Always `await` calls to `ctx.db.*`, `ctx.scheduler.*`, and other async operations.

## Incorrect

```typescript
// Missing await - these operations may silently fail
ctx.scheduler.runAfter(0, internal.tasks.process, { id });
ctx.db.patch("tasks", docId, { status: "done" });
```

## Correct

```typescript
await ctx.scheduler.runAfter(0, internal.tasks.process, { id });
await ctx.db.patch("tasks", docId, { status: "done" });
```

**ESLint:** Use `no-floating-promises` rule to catch these automatically.
