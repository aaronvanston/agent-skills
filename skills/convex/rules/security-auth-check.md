---
title: Always check authentication in public functions
impact: CRITICAL
tags: [convex, security, authentication]
---

# Always Check Authentication in Public Functions

All public queries, mutations, and actions should verify the user's identity. Missing auth checks expose data and operations to unauthenticated users.

## Incorrect

```typescript
export const getUserData = query({
  args: { userId: v.id("users") },
  returns: v.object({ name: v.string() }),
  handler: async (ctx, args) => {
    // No auth check - anyone can read any user's data
    const user = await ctx.db.get("users", args.userId);
    return { name: user!.name };
  },
});
```

## Correct

```typescript
export const getUserData = query({
  args: { userId: v.id("users") },
  returns: v.object({ name: v.string() }),
  handler: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) {
      throw new ConvexError({ code: "UNAUTHENTICATED", message: "Not logged in" });
    }

    const user = await ctx.db.get("users", args.userId);
    if (!user) {
      throw new ConvexError({ code: "NOT_FOUND", message: "User not found" });
    }
    return { name: user.name };
  },
});
```
