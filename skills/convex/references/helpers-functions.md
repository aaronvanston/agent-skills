# Custom Functions, Middleware & Zod Validation

> **Package**: `convex-helpers` — install with `npm install convex-helpers`

## Custom Functions

Build customised versions of `query`, `mutation`, and `action` with shared behaviour (auth, logging, RLS).

```typescript
import { customQuery, customMutation, customCtx, NoOp } from "convex-helpers/server/customFunctions";
import { query, mutation } from "./_generated/server";
```

### Auth Wrapper

```typescript
export const authedQuery = customQuery(query, {
  args: {},
  input: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) throw new Error("Unauthorised");

    const user = await ctx.db
      .query("users")
      .withIndex("by_token", (q) => q.eq("tokenIdentifier", identity.tokenIdentifier))
      .unique();
    if (!user) throw new Error("User not found");

    return { ctx: { ...ctx, user }, args };
  },
});

export const authedMutation = customMutation(mutation, {
  args: {},
  input: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) throw new Error("Unauthorised");
    const user = await ctx.db
      .query("users")
      .withIndex("by_token", (q) => q.eq("tokenIdentifier", identity.tokenIdentifier))
      .unique();
    if (!user) throw new Error("User not found");
    return { ctx: { ...ctx, user }, args };
  },
});
```

### Usage

```typescript
// ctx.user is typed and guaranteed to exist
export const getMyProfile = authedQuery({
  args: {},
  returns: v.object({ _id: v.id("users"), name: v.string(), email: v.string() }),
  handler: async (ctx) => ({
    _id: ctx.user._id, name: ctx.user.name, email: ctx.user.email,
  }),
});
```

### Role-Based Auth Wrapper

```typescript
export const adminQuery = customQuery(query, {
  args: {},
  input: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) throw new Error("Unauthorised");
    const user = await ctx.db
      .query("users")
      .withIndex("by_token", (q) => q.eq("tokenIdentifier", identity.tokenIdentifier))
      .unique();
    if (!user) throw new Error("User not found");
    if (user.role !== "admin") throw new Error("Admin access required");
    return { ctx: { ...ctx, user }, args };
  },
});
```

### Extra Input Arguments

```typescript
const myQueryBuilder = customQuery(query, {
  args: {},
  input: async (ctx, args, { role }: { role: "admin" | "user" }) => {
    const user = await getUser(ctx);
    if (role === "admin" && user.role !== "admin") throw new Error("Not an admin");
    return { ctx: { user }, args: {} };
  },
});

const myAdminQuery = myQueryBuilder({
  role: "admin",
  args: {},
  handler: async (ctx, args) => { /* ctx.user guaranteed admin */ },
});
```

### onSuccess Callback

```typescript
const myQuery = customQuery(query, {
  args: { apiToken: v.id("api_tokens") },
  input: async (ctx, args) => {
    const apiUser = await getApiUser(args.apiToken);
    return {
      ctx: { apiUser },
      args: {},
      onSuccess: ({ args, result }) => {
        console.log(apiUser.name, args, result);
      },
    };
  },
});
```

### Combining Custom Functions + Triggers + RLS

```typescript
import { Triggers } from "convex-helpers/server/triggers";
import { wrapDatabaseWriter } from "convex-helpers/server/rowLevelSecurity";

const triggers = new Triggers<DataModel>();
triggers.register("posts", async (ctx, change) => { /* ... */ });

export const authedMutation = customMutation(rawMutation, {
  args: {},
  input: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) throw new Error("Unauthorized");
    const user = await ctx.db.query("users").withIndex("by_token", (q) =>
      q.eq("tokenIdentifier", identity.tokenIdentifier)).unique();
    if (!user) throw new Error("User not found");
    const wrappedDb = wrapDatabaseWriter(ctx, triggers.wrapDB(ctx).db, await rlsRules(ctx));
    return { ctx: { ...ctx, user, db: wrappedDb }, args };
  },
});
```

## Zod Validation

Use Zod for argument validation instead of Convex validators.

```typescript
import * as z from "zod";
import { zCustomQuery, zid } from "convex-helpers/server/zod4";
import { NoOp } from "convex-helpers/server/customFunctions";

const zodQuery = zCustomQuery(query, NoOp);

export const myQuery = zodQuery({
  args: {
    userId: zid("users"),
    email: z.email(),
    num: z.number().min(0),
    nullableBigint: z.nullable(z.bigint()),
    boolWithDefault: z.boolean().default(true),
    array: z.array(z.string()),
    optionalObject: z.object({ a: z.string(), b: z.number() }).optional(),
    union: z.union([z.string(), z.number()]),
    discriminatedUnion: z.discriminatedUnion("kind", [
      z.object({ kind: z.literal("a"), a: z.string() }),
      z.object({ kind: z.literal("b"), b: z.number() }),
    ]),
    literal: z.literal("hi"),
    enum: z.enum(["a", "b"]),
    readonly: z.object({ a: z.string() }).readonly(),
    pipeline: z.number().pipe(z.coerce.string()),
  },
  handler: async (ctx, args) => {
    // args validated and typed by Zod, including defaults applied
  },
});
```

## Import Reference

```typescript
import { customQuery, customMutation, customAction, customCtx, NoOp } from "convex-helpers/server/customFunctions";
import { zCustomQuery, zCustomMutation, zid } from "convex-helpers/server/zod4";
```
