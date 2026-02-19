# Convex Code Review Checklist

> **Base Convex** — production readiness audit checklist.

## Security Checklist

### 1. Argument AND Return Validators
- [ ] All public `query`, `mutation`, `action` have `args` validators
- [ ] All functions have `returns` validators
- [ ] No `v.any()` for sensitive data
- [ ] HTTP actions validate request body (Zod recommended)

### 2. Error Handling
- [ ] Uses `ConvexError` for user-facing errors (not plain `Error`)
- [ ] Error codes are structured: `{ code: "NOT_FOUND", message: "..." }`
- [ ] No sensitive info leaked in error messages

### 3. Access Control
- [ ] All public functions check `ctx.auth.getUserIdentity()` where needed
- [ ] Uses auth helpers (`requireAuth`, `requireRole`)
- [ ] No client-provided email/username for authorization
- [ ] Row-level access verified (ownership checks)

### 4. Internal Functions
- [ ] `ctx.runQuery`, `ctx.runMutation`, `ctx.runAction` use `internal.*` not `api.*`
- [ ] `ctx.scheduler.runAfter` uses `internal.*` not `api.*`
- [ ] Crons in `crons.ts` use `internal.*` not `api.*`

### 5. Table Names in DB Calls
- [ ] All `ctx.db.get`, `patch`, `replace`, `delete` include table name as first arg

### 6. Authentication
- [ ] Auth provider configured (Clerk, Auth0, etc.)
- [ ] Unauthenticated access explicitly allowed where intended
- [ ] Session tokens properly validated
- [ ] HTTP actions validate origin/authentication

### 7. Environment Variables
- [ ] API keys stored in environment variables
- [ ] No secrets in code or schema
- [ ] Different keys for dev/prod
- [ ] Environment variables accessed only in actions

## Performance Checklist

### 8. Database Queries
- [ ] No `.filter()` on queries (use `.withIndex()` or filter in code)
- [ ] `.collect()` only with bounded results (<1000 docs)
- [ ] Pagination for large result sets

### 9. Indexes
- [ ] No redundant indexes (`by_foo` + `by_foo_and_bar`)
- [ ] All filtered queries use `.withIndex()`
- [ ] Index names include all fields

### 10. Date.now() in Queries
- [ ] No `Date.now()` in query functions (breaks caching/determinism)
- [ ] Time filtering uses boolean fields or client-passed timestamps

### 11. Promise Handling
- [ ] All promises awaited (`ctx.scheduler`, `ctx.db.*`)

## Architecture Checklist

### 12. Action Usage
- [ ] Actions have `"use node";` if using Node.js APIs
- [ ] `ctx.runAction` only when switching runtimes
- [ ] No sequential `ctx.runMutation`/`ctx.runQuery` in actions (combine for consistency)

### 13. Code Organization
- [ ] Business logic in helper functions (`convex/model/`)
- [ ] Public API handlers are thin wrappers
- [ ] Auth helpers in `convex/lib/auth.ts`

### 14. Transaction Consistency
- [ ] Related reads in same query/mutation
- [ ] Batch operations in single mutation
- [ ] Mutations are idempotent where possible

## Quick Regex Searches

| Issue | Regex | Fix |
|-------|-------|-----|
| `.filter()` | `\.filter\(\(?q` | Use `.withIndex()` |
| Missing returns | `handler:.*async` without `returns:` | Add `returns:` |
| Plain Error | `throw new Error\(` | Use `ConvexError` |
| Missing table name | `db\.(get\|patch)\([^"']` | Add table name |
| `Date.now()` in query | `Date\.now\(\)` | Remove from queries |
| `api.*` scheduling | `api\.[a-z]` | Use `internal.*` |

## Production Readiness Summary

1. **Security**: Validators + ConvexError + Auth checks + Internal functions + ESLint plugin
2. **Performance**: Indexes + Bounded queries + No Date.now() + Awaited promises
3. **Architecture**: Helper functions + Proper action usage + `"use node"` directive
4. **Code Quality**: Table names in DB calls + Return validators + No `v.any()`
