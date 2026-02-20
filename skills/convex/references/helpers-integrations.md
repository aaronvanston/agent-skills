# Hono, Query Caching, Richer useQuery & QueryStreams

> **Package**: `convex-helpers` — install with `npm install convex-helpers`

## Hono for HTTP Endpoints

Use [Hono](https://hono.dev/) for advanced HTTP routing instead of the basic `httpRouter`.

```typescript
import { Hono } from "hono";
import { HonoWithConvex, HttpRouterWithHono } from "convex-helpers/server/hono";
import { ActionCtx } from "./_generated/server";

const app: HonoWithConvex<ActionCtx> = new Hono();

app.get("/", async (c) => c.json("Hello world!"));
app.post("/webhook", async (c) => {
  const body = await c.req.json();
  // process webhook...
  return c.json({ ok: true });
});

export default new HttpRouterWithHono(app);
```

See the [Stack guide on Hono with Convex](https://stack.convex.dev/hono-with-convex).

## Query Caching

Persist subscriptions client-side for fast navigation. Keeps subscriptions open after component unmounts for configurable duration.

> **Note**: This optimises UX at the cost of more bandwidth (subscriptions stay open longer).

### Setup

```tsx
import { ConvexQueryCacheProvider } from "convex-helpers/react/cache";
// For Next.js: import from "convex-helpers/react/cache/provider";

<ConvexProvider client={convex}>
  <ConvexQueryCacheProvider
    expiration={300000}      // ms to keep unmounted subscriptions (default: 5 min)
    maxIdleEntries={250}     // max unused subscriptions cached (default: 250)
    debug={false}            // dump console logs every 3s (default: false)
  >
    <App />
  </ConvexQueryCacheProvider>
</ConvexProvider>
```

### Usage

```tsx
import { useQuery, usePaginatedQuery, useQueries } from "convex-helpers/react/cache";
// For Next.js: import from "convex-helpers/react/cache/hooks";

const users = useQuery(api.todos.getAll);
// Drop-in replacement — same API as convex/react hooks
```

## Richer useQuery

Returns status, data, and error instead of just `data | undefined`.

```typescript
import { makeUseQueryWithStatus } from "convex-helpers/react";
import { useQueries } from "convex/react";

export const useQueryWithStatus = makeUseQueryWithStatus(useQueries);

const { status, data, error, isSuccess, isPending, isError } =
  useQueryWithStatus(api.foo.bar, { myArg: 123 });

// Type-safe discriminated union:
// { status: "success", data: T, isSuccess: true, isPending: false, isError: false }
// { status: "pending", data: undefined, isPending: true }
// { status: "error", error: Error, isError: true }
```

## Composable QueryStreams

Merge, filter, and join multiple query streams while preserving order. Supports `.first()`, `.collect()`, `.paginate()`, and `.take(n)`.

```typescript
import { stream, mergedStream, MergedStream } from "convex-helpers/server/stream";
import schema from "./schema";
```

### Merge Streams (UNION ALL)

```typescript
// Paginate all messages by multiple authors
const authorStreams = authors.map((author) =>
  stream(ctx.db, schema)
    .query("messages")
    .withIndex("by_author", (q) => q.eq("author", author))
);
const allMessages = mergedStream(authorStreams, ["author", "_creationTime"]);
return await allMessages.paginate(paginationOpts);
```

### Filter Streams (WHERE)

```typescript
// Pre-filter: applied before page size (may read more data)
const verified = stream(ctx.db, schema)
  .query("messages")
  .order("desc")
  .filterWith(async (message) => {
    const author = await ctx.db.get(message.author);
    return author !== null && author.verified;
  });

return await verified.paginate({
  ...paginationOpts,
  maximumRowsRead: 100,  // safety limit
});
```

### Order by Index Suffix

```typescript
// Index on ["author", "unread"] — get latest 10 regardless of unread status
const read = stream(ctx.db, schema).query("messages")
  .withIndex("by_author", (q) => q.eq("author", author).eq("unread", false))
  .order("desc");
const unread = stream(ctx.db, schema).query("messages")
  .withIndex("by_author", (q) => q.eq("author", author).eq("unread", true))
  .order("desc");

const all = new MergedStream([read, unread], ["_creationTime"]);
return await all.take(10);
```

### Join Tables (flatMap)

```typescript
// Paginate messages across all user's channels, grouped by channel
const channelMemberships = stream(ctx.db, schema)
  .query("channelMemberships")
  .withIndex("userId", (q) => q.eq("userId", await getAuthedUserId(ctx)));

const channels = channelMemberships.map(async (m) => (await ctx.db.get(m.channelId))!);

const messages = channels.flatMap(
  async (channel) =>
    stream(ctx.db, schema)
      .query("messages")
      .withIndex("channelId", (q) => q.eq("channelId", channel._id))
      .map(async (msg) => ({ ...channel, ...msg })),
  ["channelId", "_creationTime"]
);

return await messages.paginate(paginationOpts);
```

### Pagination Note

When using `.paginate()` with streams in reactive queries, use `usePaginatedQuery` from `"convex-helpers/react"` or pass `customPagination: true` with cached query helpers. Pass `endCursor` to prevent holes/overlaps between pages.

## Client-Side Utilities (Copy-Paste Patterns)

These are in the convex-helpers repo for copy-pasting, not exported from the npm package. Copy the source files into your project.

### useSingleFlight

Throttle client-side requests by ensuring only one request is in-flight at a time. See: https://stack.convex.dev/throttling-requests-by-single-flighting

Copy: `src/hooks/useSingleFlight.ts` and `src/hooks/useLatestValue.ts`

### useStableQuery

Return stale results when query parameters change to avoid UI flash. See: https://stack.convex.dev/help-my-app-is-overreacting

Copy: `src/hooks/useStableQuery.ts`

### Presence

Track user presence (online status, typing indicators, facepile). See: https://stack.convex.dev/presence-with-convex

Copy these files and modify for your app:
- `convex/presence.ts` — server-side presence functions
- `src/hooks/usePresence.ts` — client-side React hooks
- `src/hooks/useTypingIndicator.ts` — typing indicator (optional)
- `src/components/Facepile.tsx` — facepile component (optional)

### usePaginatedQuery (convex-helpers)

When using `paginator`, `getPage`, or stream `.paginate()`, use the `usePaginatedQuery` hook from `"convex-helpers/react"` instead of `"convex/react"` for correct cursor handling:

```typescript
import { usePaginatedQuery } from "convex-helpers/react";
// Supports customPagination for cached query helpers
```

## Import Reference

```typescript
// Server
import { HonoWithConvex, HttpRouterWithHono } from "convex-helpers/server/hono";
import { stream, mergedStream, MergedStream } from "convex-helpers/server/stream";

// Client - cached queries (pick ONE usePaginatedQuery based on use case)
import { ConvexQueryCacheProvider } from "convex-helpers/react/cache";
import { useQuery, usePaginatedQuery } from "convex-helpers/react/cache"; // standard cached pagination
import { makeUseQueryWithStatus } from "convex-helpers/react";
// OR for paginator/getPage/stream .paginate() with custom cursors:
// import { usePaginatedQuery } from "convex-helpers/react";
```
