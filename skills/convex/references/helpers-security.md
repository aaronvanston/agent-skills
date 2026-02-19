# RLS, Rate Limiting, Sessions & CORS

> **Package**: `convex-helpers` — install with `npm install convex-helpers`

## Row-Level Security (RLS)

Declarative access control at the database layer. RLS wraps the database context to enforce rules on every read and write.

```typescript
import {
  Rules, RLSConfig, wrapDatabaseReader, wrapDatabaseWriter,
} from "convex-helpers/server/rowLevelSecurity";
import { customCtx, customQuery, customMutation } from "convex-helpers/server/customFunctions";
import { query, mutation, QueryCtx } from "./_generated/server";
import { DataModel } from "./_generated/dataModel";

async function rlsRules(ctx: QueryCtx) {
  const identity = await ctx.auth.getUserIdentity();
  return {
    users: {
      read: async (_, user) => {
        if (!identity && user.age < 18) return false;
        return true;
      },
      insert: async () => true,
      modify: async (_, user) => {
        if (!identity) throw new Error("Must be authenticated");
        return user.tokenIdentifier === identity.tokenIdentifier;
      },
    },
    messages: {
      read: async (_, message) => {
        const conversation = await ctx.db.get(message.conversationId);
        return conversation?.members.includes(identity?.subject ?? "") ?? false;
      },
      modify: async (_, message) => message.authorId === identity?.subject,
    },
    publicPosts: {
      read: async () => true,
      insert: async () => true,
      modify: async () => true,
    },
  } satisfies Rules<QueryCtx, DataModel>;
}

// defaultPolicy: "deny" means tables without rules are inaccessible
const config: RLSConfig = { defaultPolicy: "deny" };

export const queryWithRLS = customQuery(
  query,
  customCtx(async (ctx) => ({
    db: wrapDatabaseReader(ctx, ctx.db, await rlsRules(ctx), config),
  }))
);

export const mutationWithRLS = customMutation(
  mutation,
  customCtx(async (ctx) => ({
    db: wrapDatabaseWriter(ctx, ctx.db, await rlsRules(ctx), config),
  }))
);
```

### Using RLS-Wrapped Functions

```typescript
// RLS automatically filters out unauthorized messages on read
export const list = queryWithRLS({
  args: { conversationId: v.id("conversations") },
  handler: async (ctx, args) => {
    return await ctx.db.query("messages")
      .withIndex("by_conversation", (q) => q.eq("conversationId", args.conversationId))
      .collect();
  },
});

// RLS checks if user can modify this message
export const update = mutationWithRLS({
  args: { messageId: v.id("messages"), content: v.string() },
  returns: v.null(),
  handler: async (ctx, args) => {
    await ctx.db.patch(args.messageId, { content: args.content });
    return null;
  },
});
```

### ⚠️ Pitfall: Missing Table Rules

With `defaultPolicy: "allow"` (the default), tables without rules are unprotected. Always use `defaultPolicy: "deny"` and define rules for ALL tables, or use `satisfies Rules<QueryCtx, DataModel>` to get type checking.

## Rate Limiting

> Also available as a [`rate-limiter` component](https://www.convex.dev/components/rate-limiter) (`npm i @convex-dev/rate-limiter`).

```typescript
import { RateLimiter } from "convex-helpers/server/rateLimit";
import { components } from "./_generated/api";

export const rateLimiter = new RateLimiter(components.rateLimit, {
  global: { kind: "token bucket", rate: 100, period: 60000 },
  perUser: { kind: "token bucket", rate: 10, period: 60000 },
});
```

### Usage in Mutations

```typescript
export const createPost = mutation({
  args: { title: v.string(), body: v.string() },
  handler: async (ctx, args) => {
    const identity = await ctx.auth.getUserIdentity();
    if (!identity) throw new Error("Unauthorized");

    const { ok, retryAfter } = await rateLimiter.limit(ctx, "perUser", {
      key: identity.subject,
    });
    if (!ok) {
      // Add jitter to prevent thundering herd
      const jitter = Math.random() * 1000;
      throw new ConvexError({
        code: "RATE_LIMITED",
        message: "Too many requests",
        retryAfter: retryAfter + jitter,
      });
    }

    return await ctx.db.insert("posts", { ...args, authorId: identity.subject });
  },
});
```

## Session Tracking

Track users without authentication using client-generated session IDs.

### Client Setup

```tsx
import { SessionProvider, useSessionQuery, useSessionMutation } from "convex-helpers/react/sessions";

<ConvexProvider client={convex}>
  <SessionProvider>
    <App />
  </SessionProvider>
</ConvexProvider>

// In component
const results = useSessionQuery(api.myModule.mySessionQuery, { arg1: 1 });
```

### Server Setup

```typescript
import { customQuery } from "convex-helpers/server/customFunctions";
import { SessionIdArg } from "convex-helpers/server/sessions";

export const queryWithSession = customQuery(query, {
  args: SessionIdArg,
  input: async (ctx, { sessionId }) => {
    const anonymousUser = await getAnonUser(ctx, sessionId);
    return { ctx: { ...ctx, anonymousUser }, args: {} };
  },
});

// Usage
export const mySessionQuery = queryWithSession({
  args: { arg1: v.number() },
  handler: async (ctx, args) => {
    // ctx.anonymousUser available
  },
});
```

## CORS Support

```typescript
import { corsRouter } from "convex-helpers/server/cors";
import { httpRouter } from "convex/server";

const http = httpRouter();
const cors = corsRouter(http, {
  allowedOrigins: ["http://localhost:8080"],  // Default: ["*"]
  // allowedOrigins: (req: Request) => Promise<string[]>  // Also accepts async function
  allowedMethods: ["GET", "POST"],            // Defaults to route spec method
  allowedHeaders: ["Content-Type"],           // Default: ["Content-Type"]
  exposedHeaders: ["Custom-Header"],          // Default: ["Content-Range", "Accept-Ranges"]
  allowCredentials: true,                      // Default: false
  browserCacheMaxAge: 60,                      // Default: 86400 (1 day)
  enforceAllowOrigins: true,                   // Returns 403 for disallowed origins. Default: false
  debug: true,                                 // Default: false
});

cors.route({
  path: "/foo",
  method: "GET",
  handler: httpAction(async () => new Response("ok")),
});

// Per-route overrides
cors.route({
  path: "/foo",
  method: "POST",
  handler: httpAction(async () => new Response("ok")),
  allowedOrigins: ["http://localhost:8080"],
});

// Non-CORS routes still work on different paths
http.route({
  path: "/notcors",
  method: "GET",
  handler: httpAction(async () => new Response("ok")),
});

export default http;
```

## Import Reference

```typescript
import { Rules, RLSConfig, wrapDatabaseReader, wrapDatabaseWriter } from "convex-helpers/server/rowLevelSecurity";
import { RateLimiter } from "convex-helpers/server/rateLimit";
import { SessionProvider, useSessionQuery } from "convex-helpers/react/sessions";
import { SessionIdArg } from "convex-helpers/server/sessions";
import { corsRouter } from "convex-helpers/server/cors";
```
