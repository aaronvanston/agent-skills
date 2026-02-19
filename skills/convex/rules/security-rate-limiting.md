---
title: Implement rate limiting for public mutations
impact: HIGH
tags: [convex, security, rate-limiting]
---

# Implement Rate Limiting for Public Mutations

Public mutations that create resources (messages, uploads, signups) should be rate-limited to prevent abuse.

## Incorrect

```typescript
export const sendMessage = mutation({
  args: { channelId: v.id("channels"), content: v.string() },
  returns: v.id("messages"),
  handler: async (ctx, args) => {
    const user = await requireAuth(ctx);
    // No rate limiting - user can spam unlimited messages
    return await ctx.db.insert("messages", {
      channelId: args.channelId,
      content: args.content,
      authorId: user._id,
      createdAt: Date.now(),
    });
  },
});
```

## Correct

```typescript
export const sendMessage = mutation({
  args: { channelId: v.id("channels"), content: v.string() },
  returns: v.id("messages"),
  handler: async (ctx, args) => {
    const user = await requireAuth(ctx);

    // Check rate limit
    const windowStart = Date.now() - 60000;
    const recentMessages = await ctx.db
      .query("messages")
      .withIndex("by_author", (q) => q.eq("authorId", user._id))
      .filter((q) => q.gt(q.field("createdAt"), windowStart))
      .collect();

    if (recentMessages.length >= 10) {
      throw new ConvexError({ code: "RATE_LIMITED", message: "Too many messages" });
    }

    return await ctx.db.insert("messages", {
      channelId: args.channelId,
      content: args.content,
      authorId: user._id,
      createdAt: Date.now(),
    });
  },
});
```
