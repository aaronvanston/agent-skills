---
title: Always define return validators on functions
impact: HIGH
tags: [convex, validation, types]
---

# Always Define Return Validators on Functions

Every public and internal function should have a `returns` validator. This prevents accidental data leakage and provides type safety.

## Incorrect

```typescript
export const getUser = query({
  args: { userId: v.id("users") },
  // Missing returns validator - leaks all fields including internal ones
  handler: async (ctx, args) => {
    return await ctx.db.get("users", args.userId);
  },
});
```

## Correct

```typescript
export const getUser = query({
  args: { userId: v.id("users") },
  returns: v.union(
    v.object({
      _id: v.id("users"),
      name: v.string(),
      email: v.string(),
    }),
    v.null()
  ),
  handler: async (ctx, args) => {
    return await ctx.db.get("users", args.userId);
  },
});
```
