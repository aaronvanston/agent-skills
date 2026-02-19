---
title: Include table name in ctx.db calls
impact: MEDIUM
tags: [convex, security, types]
---

# Include Table Name in ctx.db Calls

Always include the table name as the first argument to `ctx.db.get`, `ctx.db.patch`, `ctx.db.replace`, and `ctx.db.delete`. This prevents accidentally operating on the wrong table.

## Incorrect

```typescript
await ctx.db.get(movieId);
await ctx.db.patch(movieId, { title: "Whiplash" });
await ctx.db.delete(movieId);
```

## Correct

```typescript
await ctx.db.get("movies", movieId);
await ctx.db.patch("movies", movieId, { title: "Whiplash" });
await ctx.db.delete("movies", movieId);
```

**ESLint:** Use `@convex-dev/explicit-table-ids` rule with autofix.
