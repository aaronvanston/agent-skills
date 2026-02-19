---
title: Use ConvexError instead of plain Error
impact: HIGH
tags: [convex, security, error-handling]
---

# Use ConvexError Instead of Plain Error

Use `ConvexError` with structured error codes for user-facing errors. Plain `Error` messages may leak sensitive implementation details to clients.

## Incorrect

```typescript
export const getTask = query({
  args: { taskId: v.id("tasks") },
  returns: v.object({ title: v.string() }),
  handler: async (ctx, args) => {
    const task = await ctx.db.get("tasks", args.taskId);
    if (!task) {
      throw new Error("Task not found in database table 'tasks'");
    }
    return { title: task.title };
  },
});
```

## Correct

```typescript
import { ConvexError } from "convex/values";

export const getTask = query({
  args: { taskId: v.id("tasks") },
  returns: v.object({ title: v.string() }),
  handler: async (ctx, args) => {
    const task = await ctx.db.get("tasks", args.taskId);
    if (!task) {
      throw new ConvexError({ code: "NOT_FOUND", message: "Task not found" });
    }
    return { title: task.title };
  },
});
```
