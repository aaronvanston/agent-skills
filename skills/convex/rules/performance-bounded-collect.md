---
title: Only use .collect() with bounded result sets
impact: HIGH
tags: [convex, performance, queries]
---

# Only Use .collect() with Bounded Result Sets

`.collect()` loads all matching documents into memory. For tables with 1000+ documents, use `.take()`, pagination, or indexes that naturally bound results.

## Incorrect

```typescript
// Loads entire table into memory
const allMovies = await ctx.db.query("movies").collect();
```

## Correct

```typescript
// Bounded by index + take
const movies = await ctx.db
  .query("movies")
  .withIndex("by_user", (q) => q.eq("userId", userId))
  .take(100);

// Or paginated
const movies = await ctx.db
  .query("movies")
  .withIndex("by_user", (q) => q.eq("userId", userId))
  .order("desc")
  .paginate(paginationOptions);
```
