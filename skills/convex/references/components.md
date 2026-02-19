# Convex Components

> **Base Convex** — components are a core Convex feature for reusable, isolated packages.

## What Are Components?

Self-contained packages with isolated database tables, functions, types, and optional frontend hooks. Tables are isolated from the main app.

## Structure

```
my-convex-component/
├── package.json
├── tsconfig.json
├── convex.config.ts       # Component configuration
└── src/
    ├── index.ts           # Main exports
    ├── component.ts       # Component definition
    ├── schema.ts          # Component schema (isolated tables)
    └── functions/
        ├── queries.ts
        ├── mutations.ts
        └── actions.ts
```

## Creating a Component

### 1. Configuration

```typescript
// convex.config.ts
import { defineComponent } from "convex/server";
export default defineComponent("myComponent");
```

### 2. Schema (Isolated Tables)

```typescript
// src/schema.ts
import { defineSchema, defineTable } from "convex/server";
import { v } from "convex/values";

export default defineSchema({
  items: defineTable({
    name: v.string(),
    data: v.any(),
    createdAt: v.number(),
  }).index("by_name", ["name"]),
});
```

### 3. Functions

```typescript
// src/functions/queries.ts
export const list = query({
  args: { limit: v.optional(v.number()) },
  returns: v.array(v.object({ _id: v.id("items"), name: v.string(), data: v.any(), createdAt: v.number() })),
  handler: async (ctx, args) => {
    return await ctx.db.query("items").order("desc").take(args.limit ?? 10);
  },
});

// src/functions/mutations.ts
export const create = mutation({
  args: { name: v.string(), data: v.any() },
  returns: v.id("items"),
  handler: async (ctx, args) => {
    return await ctx.db.insert("items", { ...args, createdAt: Date.now() });
  },
});
```

### 4. Exports

```typescript
// src/index.ts
export { default as component } from "./component";
export * from "./functions/queries";
export * from "./functions/mutations";
export type { Id } from "./_generated/dataModel";
```

## Using a Component

### App Configuration

```typescript
// convex/convex.config.ts
import { defineApp } from "convex/server";
import myComponent from "my-convex-component";

const app = defineApp();
app.use(myComponent, { name: "myComponent" });
// Multiple instances supported:
// app.use(myComponent, { name: "instance2" });
export default app;
```

### In Client Code

```typescript
import { useQuery, useMutation } from "convex/react";
import { api } from "../convex/_generated/api";

const items = useQuery(api.myComponent.list, { limit: 10 });
const createItem = useMutation(api.myComponent.create);
```

## Known Components

Several convex-helpers features are available as standalone components:

| Component | Package | Use Case |
|-----------|---------|----------|
| Rate Limiter | `@convex-dev/rate-limiter` | Application-level rate limiting |
| Action Retrier | `@convex-dev/action-retrier` | Retry idempotent actions |
| Migrations | `@convex-dev/migrations` | Stateful data migrations |
| Workpool | `@convex-dev/workpool` | Concurrency control |
| Aggregate | `@convex-dev/aggregate` | Efficient aggregations |
| Agent | `@convex-dev/agent` | AI agent with threads/tools |

## Publishing

```json
{
  "name": "my-convex-component",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "files": ["dist", "convex.config.ts"],
  "peerDependencies": { "convex": "^1.0.0" }
}
```

## Best Practices

- Keep component tables isolated — don't reference main app tables
- Export clear TypeScript types for consumers
- Use semantic versioning
- Document all public functions and arguments
- Test components in isolation before publishing

## References

- Components: https://docs.convex.dev/components
- Component Authoring: https://docs.convex.dev/components/authoring
