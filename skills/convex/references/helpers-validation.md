# Validator Utilities, Standard Schema & Code Generation

> **Package**: `convex-helpers` — install with `npm install convex-helpers`

## Validator Utilities

```typescript
import { literals, nullable, deprecated, brandedString, partial } from "convex-helpers/validators";
import { doc, typedV, validate } from "convex-helpers/validators";
import { pick, omit } from "convex-helpers";
```

### Shorthand Validators

```typescript
// Union of literals
const status = literals("active", "inactive");  // v.union(v.literal("active"), v.literal("inactive"))

// Nullable field
const deletedAt = nullable(v.number());  // v.union(v.number(), v.null())

// Deprecated field (accepts any value, signals removal)
const oldField = deprecated;

// Branded string type - type-safe string wrappers
const emailValidator = brandedString("email");
type Email = Infer<typeof emailValidator>;  // branded string type
```

### Schema Usage

```typescript
import { literals, nullable, deprecated, brandedString } from "convex-helpers/validators";

export default defineSchema({
  accounts: defineTable({
    balance: nullable(v.bigint()),
    status: literals("active", "inactive"),
    email: brandedString("email"),
    oldField: deprecated,
  }).index("status", ["status"]),
});
```

### typedV — Schema-Aware `v` Replacement

```typescript
import { typedV } from "convex-helpers/validators";
import schema from "./schema";

const vv = typedV(schema);
vv.id("accounts");   // typed to tables in your schema (autocomplete!)
vv.doc("accounts");   // full document validator with system fields (_id, _creationTime)
```

### partial, pick, omit

```typescript
import { partial } from "convex-helpers/validators";
import { pick, omit } from "convex-helpers";

const subset = pick(vv.doc("accounts").fields, ["balance", "email"]);
const without = omit(vv.doc("accounts").fields, ["balance"]);

// partial for optional fields
const partialAccount = partial(vv.doc("accounts").fields);
```

### doc Helper

```typescript
import { doc } from "convex-helpers/validators";

export const replaceUser = internalMutation({
  args: {
    id: vv.id("accounts"),
    replace: vv.object({
      ...schema.tables.accounts.validator.fields,
      ...partial(systemFields("accounts")),
    }),
  },
  returns: doc(schema, "accounts"),  // validates document with system fields
  handler: async (ctx, args) => {
    await ctx.db.replace(args.id, args.replace);
    return await ctx.db.get(args.id);
  },
});
```

### Table Utility

Define a table and keep references to fields to avoid re-defining validators:

```typescript
import { Table } from "convex-helpers/server";

const Users = Table("users", {
  name: v.string(),
  email: v.string(),
  role: literals("user", "admin"),
});

// Reuse validators in args:
export const create = mutation({
  args: Users.withoutSystemFields,
  handler: async (ctx, args) => { /* ... */ },
});
```

### Runtime Validation

```typescript
import { validate } from "convex-helpers/validators";

// Returns boolean
validate(subset, value);

// Throws ValidationError
validate(subset, value, { throw: true });

// Validates ID exists in database
validate(vv.id("accounts"), accountId, { db: ctx.db });
```

## Standard Schema

Convert Convex validators to [Standard Schema](https://github.com/standard-schema/standard-schema) format for interop.

```typescript
import { toStandardSchema } from "convex-helpers/standardSchema";

const standardValidator = toStandardSchema(
  v.object({ name: v.string(), age: v.number() })
);

standardValidator["~standard"].validate({ name: "John", age: 30 });
```

## TypeScript and OpenAPI Generation

Generate typed API objects for external repos or OpenAPI specs.

```bash
# TypeScript API spec (connects to dev deployment by default)
npx convex-helpers ts-api-spec
npx convex-helpers ts-api-spec --prod

# OpenAPI spec
npx convex-helpers open-api-spec
npx convex-helpers open-api-spec --prod
```

The TS spec generates a `convexApi{timestamp}.ts` file with full type safety for use in external repositories. The OpenAPI spec generates a YAML file for creating clients in unsupported languages or connecting tools like Retool.

## Import Reference

```typescript
import { literals, nullable, deprecated, brandedString, partial } from "convex-helpers/validators";
import { doc, typedV, validate } from "convex-helpers/validators";
import { pick, omit } from "convex-helpers";
import { toStandardSchema } from "convex-helpers/standardSchema";
```
