# Convex Patterns

## Table of Contents
- [Return Validators](#always-define-return-validators)
- [Project Structure](#project-structure)
- [Helper Functions](#helper-functions-pattern)
- [Schema Definition](#schema-definition)
- [TypeScript Types](#typescript-types)
- [Validator Types Reference](#validator-types-reference)
- [Reusable Validators](#reusable-validators)
- [ESLint Plugin](#eslint-plugin)
- [Discriminated Unions](#discriminated-unions)
- [Additional Validator Types](#additional-validator-types)
- [Optional vs Nullable Fields](#optional-vs-nullable-fields)
- [Convex Components](#convex-components)
- [RBAC Pattern](#rbac-pattern)
- [Audit Trail Pattern](#audit-trail-pattern)

## Always Define Return Validators

Every function should have a `returns` validator:

```typescript
export const getUser = query({
  args: { userId: v.id("users") },
  returns: v.union(
    v.object({
      _id: v.id("users"),
      _creationTime: v.number(),
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

## Project Structure

```
convex/
├── _generated/
├── lib/              # Shared utilities
│   └── auth.ts       # Auth helpers
├── model/            # Business logic
│   ├── users.ts
│   └── tasks.ts
├── schema.ts         # Database schema
├── users.ts          # Public API (thin wrappers)
└── tasks.ts
```

## Helper Functions Pattern

Business logic in `model/`, thin wrappers in public API:

```typescript
// convex/model/users.ts
import { QueryCtx, MutationCtx } from '../_generated/server';
import { Doc, Id } from '../_generated/dataModel';
import { ConvexError } from "convex/values";

export async function getCurrentUser(ctx: QueryCtx | MutationCtx): Promise<Doc<"users">> {
  const identity = await ctx.auth.getUserIdentity();
  if (!identity) {
    throw new ConvexError({ code: "UNAUTHENTICATED", message: "Not logged in" });
  }

  const user = await ctx.db
    .query("users")
    .withIndex("by_tokenIdentifier", (q) => q.eq("tokenIdentifier", identity.tokenIdentifier))
    .unique();

  if (!user) {
    throw new ConvexError({ code: "NOT_FOUND", message: "User not found" });
  }
  return user;
}

export async function getById(ctx: QueryCtx, userId: Id<"users">): Promise<Doc<"users"> | null> {
  return await ctx.db.get("users", userId);
}
```

```typescript
// convex/users.ts (thin wrapper)
import { query } from "./_generated/server";
import { v } from "convex/values";
import * as Users from "./model/users";

export const get = query({
  args: { userId: v.id("users") },
  returns: v.union(v.object({ _id: v.id("users"), name: v.string() }), v.null()),
  handler: async (ctx, args) => Users.getById(ctx, args.userId),
});
```

## Schema Definition

```typescript
// convex/schema.ts
import { defineSchema, defineTable } from "convex/server";
import { v } from "convex/values";

export default defineSchema({
  users: defineTable({
    tokenIdentifier: v.string(),
    name: v.string(),
    email: v.string(),
    role: v.union(v.literal("user"), v.literal("admin")),
  })
    .index("by_tokenIdentifier", ["tokenIdentifier"])
    .index("by_email", ["email"]),

  tasks: defineTable({
    title: v.string(),
    description: v.optional(v.string()),
    userId: v.id("users"),
    status: v.union(v.literal("pending"), v.literal("done")),
    priority: v.union(v.literal("low"), v.literal("medium"), v.literal("high")),
  })
    .index("by_user", ["userId"])
    .index("by_user_and_status", ["userId", "status"]),
});
```

## TypeScript Types

```typescript
// Context types
import { QueryCtx, MutationCtx, ActionCtx } from "./_generated/server";

// Document and ID types
import { Doc, Id } from "./_generated/dataModel";
type User = Doc<"users">;
type UserId = Id<"users">;

// Infer types from validators
import { Infer, v } from "convex/values";
const priorityValidator = v.union(v.literal("low"), v.literal("medium"), v.literal("high"));
type Priority = Infer<typeof priorityValidator>;  // "low" | "medium" | "high"

// Without system fields (for inserts)
import { WithoutSystemFields } from "convex/server";
type NewUser = WithoutSystemFields<Doc<"users">>;

// Client-side return types
import { FunctionReturnType } from "convex/server";
type UserData = FunctionReturnType<typeof api.users.get>;
```

## Validator Types Reference

| Validator | TypeScript | Example |
|-----------|-----------|---------|
| `v.string()` | `string` | `"hello"` |
| `v.number()` | `number` | `42` |
| `v.boolean()` | `boolean` | `true` |
| `v.null()` | `null` | `null` |
| `v.id("table")` | `Id<"table">` | Document reference |
| `v.array(v)` | `T[]` | `[1, 2, 3]` |
| `v.object({})` | `{ ... }` | `{ name: "..." }` |
| `v.optional(v)` | `T \| undefined` | Optional field |
| `v.union(...)` | `T1 \| T2` | Multiple types |
| `v.literal(x)` | `"x"` | Exact value |

## Reusable Validators

```typescript
// convex/validators.ts
import { v } from "convex/values";

export const userValidator = v.object({
  _id: v.id("users"),
  _creationTime: v.number(),
  name: v.string(),
  email: v.string(),
  role: v.union(v.literal("user"), v.literal("admin")),
});

export const taskValidator = v.object({
  _id: v.id("tasks"),
  _creationTime: v.number(),
  title: v.string(),
  userId: v.id("users"),
  status: v.union(v.literal("pending"), v.literal("done")),
});
```

## ESLint Plugin

Install `@convex-dev/eslint-plugin` for build-time validation:

```bash
npm i @convex-dev/eslint-plugin --save-dev
```

```js
// eslint.config.js
import { defineConfig } from "eslint/config";
import convexPlugin from "@convex-dev/eslint-plugin";

export default defineConfig([
  ...convexPlugin.configs.recommended,
]);
```

| Rule | Enforces |
|------|----------|
| `no-old-registered-function-syntax` | Object syntax with `handler` |
| `require-argument-validators` | `args: {}` on all functions |
| `explicit-table-ids` | Table name in db operations |
| `import-wrong-runtime` | No Node imports in Convex runtime |

## Discriminated Unions

Use for polymorphic data in schemas:

```typescript
export default defineSchema({
  events: defineTable(
    v.union(
      v.object({
        type: v.literal("user_signup"),
        userId: v.id("users"),
        email: v.string(),
      }),
      v.object({
        type: v.literal("purchase"),
        userId: v.id("users"),
        orderId: v.id("orders"),
        amount: v.number(),
      }),
      v.object({
        type: v.literal("page_view"),
        sessionId: v.string(),
        path: v.string(),
      })
    )
  ).index("by_type", ["type"]),
});
```

## Additional Validator Types

| Validator | TypeScript | Example |
|-----------|-----------|---------|
| `v.int64()` | `bigint` | `9007199254740993n` |
| `v.bytes()` | `ArrayBuffer` | Binary data |
| `v.any()` | `any` | Any value (avoid for sensitive data) |
| `v.record(k, v)` | `Record<K, V>` | Dynamic keys |

## Optional vs Nullable Fields

```typescript
items: defineTable({
  description: v.optional(v.string()),          // May not exist
  deletedAt: v.union(v.number(), v.null()),     // Exists but can be null
  notes: v.optional(v.union(v.string(), v.null())), // Both
}),
```

## Convex Components

Self-contained, reusable packages with isolated tables and functions.

### Component Configuration

```typescript
// convex/convex.config.ts
import { defineApp } from "convex/server";
import myComponent from "my-convex-component";

const app = defineApp();
app.use(myComponent, { name: "myComponent" });
// Multiple instances supported:
app.use(myComponent, { name: "instance2" });

export default app;
```

### Authoring a Component

```typescript
// convex.config.ts
import { defineComponent } from "convex/server";
export default defineComponent("myComponent");
```

Component tables are isolated from the main app. Export clear TypeScript types for consumers. See https://docs.convex.dev/components/authoring for full guide.

## RBAC Pattern

```typescript
// convex/lib/auth.ts
type UserRole = "user" | "moderator" | "admin" | "superadmin";

const roleHierarchy: Record<UserRole, number> = {
  user: 0, moderator: 1, admin: 2, superadmin: 3,
};

export async function requireRole(
  ctx: QueryCtx | MutationCtx,
  minRole: UserRole
): Promise<Doc<"users">> {
  const user = await getUser(ctx);
  if (!user) throw new ConvexError({ code: "UNAUTHENTICATED", message: "Auth required" });

  const userLevel = roleHierarchy[user.role as UserRole] ?? 0;
  if (userLevel < roleHierarchy[minRole]) {
    throw new ConvexError({ code: "FORBIDDEN", message: `Role '${minRole}' required` });
  }
  return user;
}
```

## Audit Trail Pattern

Log sensitive operations for security review:

```typescript
// convex/audit.ts
export const logEvent = internalMutation({
  args: {
    action: v.string(),
    userId: v.optional(v.string()),
    resourceType: v.string(),
    resourceId: v.string(),
    details: v.optional(v.any()),
  },
  returns: v.id("auditLogs"),
  handler: async (ctx, args) => {
    return await ctx.db.insert("auditLogs", { ...args, timestamp: Date.now() });
  },
});
```

## Testing with a Local Backend

Test Convex functions against a local backend instance. See: https://stack.convex.dev/testing-with-local-oss-backend

### Setup

1. Download a pre-built binary from https://github.com/get-convex/convex-backend/releases
2. Create a `clearAll` function to reset data between tests
3. Use `ConvexTestingHelper` from `convex-helpers/testing`
4. Run one test at a time for isolation

```typescript
// convex/testingFunctions.ts
import { internalMutation } from "./_generated/server";

export const clearAll = internalMutation({
  args: {},
  handler: async (ctx) => {
    // Only allow in test environment
    if (!process.env.IS_TEST) throw new Error("Not in test");
    for (const table of ["users", "messages", "posts"]) {
      const docs = await ctx.db.query(table).collect();
      for (const doc of docs) await ctx.db.delete(doc._id);
    }
  },
});
```

### Test Harness Pattern

```typescript
// convex/example.test.ts
import { ConvexTestingHelper } from "convex-helpers/testing";
import { api } from "./_generated/api";

describe("my functions", () => {
  let t: ConvexTestingHelper;

  beforeEach(async () => {
    t = new ConvexTestingHelper();
    await t.mutation(api.testingFunctions.clearAll, {});
  });

  afterAll(async () => {
    await t.close();
  });

  test("creates a user", async () => {
    const id = await t.mutation(api.users.create, { name: "Alice" });
    const user = await t.query(api.users.get, { id });
    expect(user?.name).toBe("Alice");
  });
});
```

### ESLint: Enforce Wrapped Mutations

When using custom functions (triggers, RLS, auth), forbid raw imports to prevent bypassing middleware:

```json
// .eslintrc
{
  "rules": {
    "no-restricted-imports": ["error", {
      "paths": [{
        "name": "./_generated/server",
        "importNames": ["mutation", "query"],
        "message": "Use custom wrappers from ./functions.ts instead."
      }]
    }]
  }
}
```

## References

- TypeScript: https://docs.convex.dev/understanding/best-practices/typescript
- Schemas: https://docs.convex.dev/database/schemas
- ESLint: https://docs.convex.dev/eslint
- Components: https://docs.convex.dev/components
- Testing: https://stack.convex.dev/testing-with-local-oss-backend
