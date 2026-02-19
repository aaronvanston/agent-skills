# Convex Security

> **Base Convex** — these patterns use core Convex features. For convex-helpers RLS and rate limiting, see [helpers-security.md](helpers-security.md).

## Error Handling with ConvexError

Use `ConvexError` for user-facing errors with structured error codes:

```typescript
import { ConvexError } from "convex/values";

export const updateTask = mutation({
  args: { taskId: v.id("tasks"), title: v.string() },
  returns: v.null(),
  handler: async (ctx, args) => {
    const task = await ctx.db.get("tasks", args.taskId);
    if (!task) {
      throw new ConvexError({ code: "NOT_FOUND", message: "Task not found" });
    }
    await ctx.db.patch("tasks", args.taskId, { title: args.title });
    return null;
  },
});
```

**Standard error codes**: `UNAUTHENTICATED`, `FORBIDDEN`, `NOT_FOUND`, `RATE_LIMITED`, `VALIDATION_ERROR`

## Argument AND Return Validators

Always define both `args` and `returns` validators on every public function:

```typescript
export const createTask = mutation({
  args: {
    title: v.string(),
    priority: v.union(v.literal("low"), v.literal("medium"), v.literal("high")),
  },
  returns: v.id("tasks"),
  handler: async (ctx, args) => {
    return await ctx.db.insert("tasks", {
      title: args.title, priority: args.priority, completed: false,
    });
  },
});
```

**ESLint**: Use `@convex-dev/eslint-plugin` which enforces `require-argument-validators` and more.

## Authentication Helpers

Create reusable auth helpers in `convex/lib/auth.ts`:

```typescript
import { QueryCtx, MutationCtx } from "./_generated/server";
import { ConvexError } from "convex/values";
import { Doc } from "./_generated/dataModel";

export async function getUser(ctx: QueryCtx | MutationCtx): Promise<Doc<"users"> | null> {
  const identity = await ctx.auth.getUserIdentity();
  if (!identity) return null;
  return await ctx.db
    .query("users")
    .withIndex("by_tokenIdentifier", (q) => q.eq("tokenIdentifier", identity.tokenIdentifier))
    .unique();
}

export async function requireAuth(ctx: QueryCtx | MutationCtx): Promise<Doc<"users">> {
  const user = await getUser(ctx);
  if (!user) throw new ConvexError({ code: "UNAUTHENTICATED", message: "Authentication required" });
  return user;
}

type UserRole = "user" | "moderator" | "admin" | "superadmin";
const roleHierarchy: Record<UserRole, number> = { user: 0, moderator: 1, admin: 2, superadmin: 3 };

export async function requireRole(ctx: QueryCtx | MutationCtx, minRole: UserRole): Promise<Doc<"users">> {
  const user = await requireAuth(ctx);
  const userLevel = roleHierarchy[user.role as UserRole] ?? 0;
  if (userLevel < roleHierarchy[minRole]) {
    throw new ConvexError({ code: "FORBIDDEN", message: `Role '${minRole}' or higher required` });
  }
  return user;
}
```

### Usage

```typescript
export const adminOnly = mutation({
  args: {},
  returns: v.null(),
  handler: async (ctx) => {
    await requireRole(ctx, "admin");
    // ... admin logic
    return null;
  },
});
```

## Row-Level Access Control

Verify ownership before operations:

```typescript
export const updateTask = mutation({
  args: { taskId: v.id("tasks"), title: v.string() },
  returns: v.null(),
  handler: async (ctx, args) => {
    const user = await requireAuth(ctx);
    const task = await ctx.db.get("tasks", args.taskId);
    if (!task || task.userId !== user._id) {
      throw new ConvexError({ code: "FORBIDDEN", message: "Not authorized" });
    }
    await ctx.db.patch("tasks", args.taskId, { title: args.title });
    return null;
  },
});
```

## Internal Functions for Scheduling

Always use `internal.*` for `ctx.run*` and scheduling — never `api.*`:

```typescript
// ❌ Bad - exposes public function
await ctx.scheduler.runAfter(0, api.tasks.process, { id });

// ✅ Good - uses internal function
await ctx.scheduler.runAfter(0, internal.tasks.process, { id });

export const process = internalMutation({
  args: { id: v.id("tasks") },
  returns: v.null(),
  handler: async (ctx, args) => { /* ... */ return null; },
});
```

**Tip**: Never import `api` from `_generated/api.ts` in Convex server functions.

## Explicit Table Names in DB Calls

```typescript
// ❌ Bad
await ctx.db.get(movieId);
await ctx.db.patch(movieId, { title: "Whiplash" });

// ✅ Good
await ctx.db.get("movies", movieId);
await ctx.db.patch("movies", movieId, { title: "Whiplash" });
```

**ESLint**: `@convex-dev/explicit-table-ids` rule with autofix.

## Environment Variables

- Store API keys in environment variables, not code
- Access environment variables only in actions (`process.env.MY_KEY`)
- Use different keys for dev/prod environments
- Never expose secrets in query/mutation return values

## ESLint Plugin

```bash
npm i @convex-dev/eslint-plugin --save-dev
```

```js
// eslint.config.js
import convexPlugin from "@convex-dev/eslint-plugin";
export default [...convexPlugin.configs.recommended];
```

Four rules enforced:
| Rule | What it enforces |
|------|-----------------|
| `no-old-registered-function-syntax` | Object syntax with `handler` |
| `require-argument-validators` | `args: {}` on all functions |
| `explicit-table-ids` | Table name in db operations |
| `import-wrong-runtime` | No Node imports in Convex runtime |

## References

- Error Handling: https://docs.convex.dev/functions/error-handling
- Authentication: https://docs.convex.dev/auth/functions-auth
- Production Security: https://docs.convex.dev/production
- ESLint Plugin: https://docs.convex.dev/eslint
