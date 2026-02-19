# Relationships, CRUD, Triggers, Filter & Pagination

> **Package**: `convex-helpers` — install with `npm install convex-helpers`

## Relationship Helpers

Traverse database relationships without boilerplate.

```typescript
import {
  getAll, getOneFrom, getOneFromOrThrow,
  getManyFrom, getManyVia, getManyViaOrThrow,
} from "convex-helpers/server/relationships";
import { asyncMap } from "convex-helpers";
```

### One-to-One

```typescript
const profile = await getOneFrom(ctx.db, "profiles", "userId", user._id);
// Or throw if missing:
const profile = await getOneFromOrThrow(ctx.db, "profiles", "userId", user._id);
```

### One-to-Many (by IDs)

```typescript
const users = await getAll(ctx.db, userIds);
// Returns array in same order (null for missing)
```

### One-to-Many (via index)

```typescript
const posts = await getManyFrom(ctx.db, "posts", "by_authorId", author._id);
```

### Many-to-Many (via join table)

```typescript
// Schema: postCategories with index "by_post" on [postId] and "by_category" on [categoryId]
const categories = await getManyVia(ctx.db, "postCategories", "categoryId", "by_post", post._id);
const posts = await getManyVia(ctx.db, "postCategories", "postId", "by_category", category._id);
```

### Nested Traversal with asyncMap

```typescript
const posts = await asyncMap(
  await getManyFrom(ctx.db, "posts", "authorId", author._id),
  async (post) => {
    const comments = await getManyFrom(ctx.db, "comments", "postId", post._id);
    const categories = await getManyViaOrThrow(
      ctx.db, "postCategories", "categoryId", "postId", post._id
    );
    return { ...post, comments, categories };
  },
);
```

### Complete Relationship Example

```typescript
export const getPostWithDetails = query({
  args: { postId: v.id("posts") },
  handler: async (ctx, args) => {
    const post = await ctx.db.get(args.postId);
    if (!post) return null;
    const [author, comments, categories] = await Promise.all([
      ctx.db.get(post.authorId),
      getManyFrom(ctx.db, "comments", "by_post", post._id),
      getManyVia(ctx.db, "postCategories", "categoryId", "by_post", post._id),
    ]);
    return {
      post, author,
      comments: comments.map((c) => ({ _id: c._id, body: c.body })),
      categories: categories.filter((c): c is NonNullable<typeof c> => c !== null),
    };
  },
});
```

## Triggers

Run code automatically when documents change. Executes atomically within the same transaction.

```typescript
import { Triggers } from "convex-helpers/server/triggers";
import { customCtx, customMutation } from "convex-helpers/server/customFunctions";
import { mutation as rawMutation } from "./_generated/server";
import { DataModel } from "./_generated/dataModel";

const triggers = new Triggers<DataModel>();

// Computed field
triggers.register("users", async (ctx, change) => {
  if (change.newDoc) {
    const fullName = `${change.newDoc.firstName} ${change.newDoc.lastName}`;
    if (change.newDoc.fullName !== fullName) {
      await ctx.db.patch(change.id, { fullName });
    }
  }
});

// Denormalized count (careful: single doc = write contention)
triggers.register("users", async (ctx, change) => {
  const countDoc = (await ctx.db.query("userCount").unique())!;
  if (change.operation === "insert") await ctx.db.patch(countDoc._id, { count: countDoc.count + 1 });
  else if (change.operation === "delete") await ctx.db.patch(countDoc._id, { count: countDoc.count - 1 });
});

// Cascading deletes
triggers.register("users", async (ctx, change) => {
  if (change.operation === "delete") {
    const messages = await ctx.db.query("messages")
      .withIndex("by_author", (q) => q.eq("authorId", change.id)).collect();
    for (const msg of messages) await ctx.db.delete(msg._id);
  }
});

// Schedule post-mutation work (deduped per mutation)
const scheduled: Record<string, Id<"_scheduled_functions">> = {};
triggers.register("users", async (ctx, change) => {
  if (scheduled[change.id]) await ctx.scheduler.cancel(scheduled[change.id]);
  scheduled[change.id] = await ctx.scheduler.runAfter(0, internal.users.updateClerkUser, { user: change.newDoc });
});

export const mutation = customMutation(rawMutation, customCtx(triggers.wrapDB));
```

### Trigger Change Object

```typescript
interface Change<Doc> {
  id: Id<TableName>;
  operation: "insert" | "update" | "delete";
  oldDoc: Doc | null;  // null for inserts
  newDoc: Doc | null;  // null for deletes
}
```

### Trigger Semantics & Warnings

- Triggers run inside the same transaction — writing to hot-spot documents causes OCC conflicts
- Use `ctx.innerDb` to write without triggering more triggers
- Recursive triggers execute breadth-first (queue order)
- If a trigger throws, the error is thrown from the `ctx.db.*` call that caused it
- Triggers only run through wrapped mutations — not from dashboard or `npx convex import`
- Add ESLint `no-restricted-imports` to forbid raw mutation imports
- Use sharding for high-contention counters, not single-doc updates

## CRUD Utilities

Generate basic CRUD functions for a table. **Use only for prototyping or internal functions.**

```typescript
import { crud } from "convex-helpers/server/crud";
import schema from "./schema";

export const { create, read, update, destroy } = crud(schema, "users");

// Also generates: paginate
// Usage in action:
const user = await ctx.runQuery(internal.users.read, { id: userId });
await ctx.runMutation(internal.users.update, { id: userId, patch: { status: "inactive" } });
```

## Filter Helper

Apply arbitrary TypeScript filters to database queries. **Use sparingly** — prefer `.withIndex()` when possible.

```typescript
import { filter } from "convex-helpers/server/filter";

const evens = await filter(ctx.db.query("counter_table"), (c) => c.counter % 2 === 0).collect();

const result = await filter(ctx.db.query("counter_table"), (c) => c.counter > c.name.length)
  .order("desc").first();
```

## Manual Pagination

### getPage

```typescript
import { getPage } from "convex-helpers/server/pagination";
import schema from "./schema";

// First page by creation time
const { page, indexKeys, hasMore } = await getPage(ctx, { table: "messages" });

// Next page
const { page: page2 } = await getPage(ctx, {
  table: "messages",
  startIndexKey: indexKeys[indexKeys.length - 1],
});

// Custom page size, index, and order
const { page } = await getPage(ctx, {
  table: "users", index: "by_name", schema, targetMaxRows: 1000,
});

// Fixed range (stable display as documents change)
const { page } = await getPage(ctx, { table: "messages", startIndexKey, endIndexKey });

// Yesterday's messages, recent first
const { page } = await getPage(ctx, {
  table: "messages",
  startIndexKey: [Date.now() - 86400000],
  startInclusive: true,
  order: "desc",
});
```

### paginator

Drop-in replacement for `.paginate()` supporting multiple calls per query.

```typescript
import { paginator } from "convex-helpers/server/pagination";
import schema from "./schema";

// Replace: ctx.db.query("messages").paginate(opts)
return await paginator(ctx.db, schema).query("messages").paginate(opts);

// With index and ordering
return await paginator(ctx.db, schema)
  .query("messages")
  .withIndex("by_author", (q) => q.eq("author", author))
  .order("desc")
  .paginate(opts);
```

**Note**: `paginator` supports `withIndex` but not `filter`. For gapless pagination with `usePaginatedQuery`, use the hook from `"convex-helpers/react"` or pass `customPagination: true` with cached query helpers.

## Import Reference

```typescript
import { getAll, getOneFrom, getOneFromOrThrow, getManyFrom, getManyVia, getManyViaOrThrow } from "convex-helpers/server/relationships";
import { asyncMap } from "convex-helpers";
import { Triggers } from "convex-helpers/server/triggers";
import { crud } from "convex-helpers/server/crud";
import { filter } from "convex-helpers/server/filter";
import { getPage, paginator } from "convex-helpers/server/pagination";
```
