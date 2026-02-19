---
title: Use .withIndex() instead of .filter() on queries
impact: HIGH
tags: [convex, performance, queries]
---

# Use .withIndex() Instead of .filter() on Queries

`.filter()` on database queries scans all documents - it has the same performance as filtering in application code. Use `.withIndex()` for efficient queries.

## Incorrect

```typescript
const tomsMessages = await ctx.db
  .query("messages")
  .filter((q) => q.eq(q.field("author"), "Tom"))
  .collect();
```

## Correct

```typescript
// Define index in schema.ts:
// messages: defineTable({...}).index("by_author", ["author"])

const tomsMessages = await ctx.db
  .query("messages")
  .withIndex("by_author", (q) => q.eq("author", "Tom"))
  .collect();
```

**Exception:** Paginated queries benefit from `.filter()` since they already limit scanned rows.
