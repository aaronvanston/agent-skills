# Workpool, Action Retries & Migrations

> **Separate packages** — install individually as needed.

## Workpool (Concurrency Control)

> **Package**: `npm install @convex-dev/workpool`

Manage concurrent execution with parallelism limits. Use for:
- Serializing writes to avoid OCC conflicts
- Fan-out parallel processing with limits
- Rate-limited external API calls

### Setup

```typescript
// convex/convex.config.ts
import { defineApp } from "convex/server";
import workpool from "@convex-dev/workpool/convex.config";

const app = defineApp();
app.use(workpool, { name: "workpool" });
export default app;
```

### Serialized Counter (maxParallelism: 1)

```typescript
import { Workpool } from "@convex-dev/workpool";
import { components, internal } from "./_generated/api";

const counterPool = new Workpool(components.workpool, { maxParallelism: 1 });

export const increment = mutation({
  args: {},
  returns: v.null(),
  handler: async (ctx) => {
    await counterPool.enqueueMutation(ctx, internal.counters.doIncrement, {});
    return null;
  },
});

export const doIncrement = internalMutation({
  args: {},
  returns: v.null(),
  handler: async (ctx) => {
    const counter = await ctx.db.query("counters").unique();
    if (counter) await ctx.db.patch(counter._id, { count: counter.count + 1 });
    return null;
  },
});
```

### Fan-Out with Limits

```typescript
const processingPool = new Workpool(components.workpool, { maxParallelism: 5 });

export const processAll = mutation({
  args: { itemIds: v.array(v.id("items")) },
  returns: v.null(),
  handler: async (ctx, args) => {
    for (const id of args.itemIds) {
      await processingPool.enqueueAction(ctx, internal.items.processOne, { itemId: id });
    }
    return null;
  },
});
```

## Action Retries

> **Package**: `npm install @convex-dev/action-retrier` (component version recommended)

Retry idempotent actions until they succeed.

```typescript
import { makeActionRetrier } from "convex-helpers/server/retries";

export const { runWithRetries, retry } = makeActionRetrier("utils:retry");

// In a mutation or action
export const myMutation = mutation({
  args: { /* ... */ },
  handler: async (ctx, args) => {
    await runWithRetries(ctx, internal.myModule.myAction, { arg1: 123 });
  },
});
```

**Only retry idempotent actions** — actions that are safe to run multiple times without side effects.

## Stateful Migrations

> **Package**: `npm install @convex-dev/migrations` (component version recommended)
> The component has the benefit of not needing to add any tables to your schema.

```typescript
import { migration } from "convex-helpers/server/migrations";

export const myMigration = migration({
  table: "users",
  migrateOne: async (ctx, doc) => {
    await ctx.db.patch(doc._id, { newField: "value" });
  },
});
```

See also: [migrations.md](migrations.md) for full migration patterns.

## Import Reference

```typescript
// Workpool (separate package)
import { Workpool } from "@convex-dev/workpool";

// Retries (convex-helpers or separate component)
import { makeActionRetrier } from "convex-helpers/server/retries";

// Migrations (convex-helpers or separate component)
import { migration } from "convex-helpers/server/migrations";
```
