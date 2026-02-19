---
title: Don't use Date.now() in query functions
impact: MEDIUM
tags: [convex, performance, queries, reactivity]
---

# Don't Use Date.now() in Query Functions

Queries are cached and reactive - they re-run when data changes, not when time passes. Using `Date.now()` causes stale results and cache thrashing.

## Incorrect

```typescript
const recentPosts = await ctx.db
  .query("posts")
  .withIndex("by_released_at", (q) => q.lte("releasedAt", Date.now()))
  .take(100);
```

## Correct

```typescript
// Use a boolean field updated by a scheduled function
const recentPosts = await ctx.db
  .query("posts")
  .withIndex("by_is_released", (q) => q.eq("isReleased", true))
  .take(100);
```
