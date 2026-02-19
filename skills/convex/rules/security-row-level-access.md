---
title: Verify row-level ownership before mutations
impact: CRITICAL
tags: [convex, security, authorisation]
---

# Verify Row-Level Ownership Before Mutations

Always check that the authenticated user owns or has permission to modify a document before patching or deleting it.

## Incorrect

```typescript
export const deleteTask = mutation({
  args: { taskId: v.id("tasks") },
  returns: v.null(),
  handler: async (ctx, args) => {
    // No ownership check - any user can delete any task
    await ctx.db.delete(args.taskId);
    return null;
  },
});
```

## Correct

```typescript
export const deleteTask = mutation({
  args: { taskId: v.id("tasks") },
  returns: v.null(),
  handler: async (ctx, args) => {
    const user = await requireAuth(ctx);
    const task = await ctx.db.get("tasks", args.taskId);

    if (!task || task.userId !== user._id) {
      throw new ConvexError({ code: "FORBIDDEN", message: "Not authorised" });
    }

    await ctx.db.delete(args.taskId);
    return null;
  },
});
```
