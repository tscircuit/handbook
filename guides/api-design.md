# API Design

Conventions for designing APIs across HTTP services, libraries, and tscircuit
itself. See also [`code.md`](./code.md) for naming/casing rules these build on.

# 1. General API Design (HTTP, Libraries, tscircuit)

## 1.1 Truthy flags to enable unusual, non-default behavior

Boolean options should default to `false`, where presence of the flag opts into
the unusual behavior. The "normal" path needs no flags; you only pass a flag to
deviate from it.

```ts
// GOOD: defaults are the common case, flags opt into the unusual
listPackages({ include_archived: true })
build({ ignore_errors: true })

// BAD: requires a flag to get normal behavior
listPackages({ only_active: false })
```

This keeps call sites readable — a bare call does the expected thing, and any flag
you see is a signal that something non-standard is happening.

## 1.2 Avoid nesting unless very necessary

Prefer flat objects. Deep nesting is hard to type, hard to migrate, and hard to
query. Add a level of nesting only when the grouping is real and stable.

```ts
// BAD: gratuitous nesting
{ package: { meta: { naming: { name: "foo" } } } }

// GOOD: flat
{ package_name: "foo" }
```

# 2. HTTP API Design

## 2.1 RPC Routes

Routes are verb-based RPC, not REST resource trees. The path names an action on a
resource: `/<resource>/<verb>`.

```
# GOOD: verb-based, flat
POST /package/create
POST /package/get
POST /package/list
POST /package_release/create

# BAD: REST resource trees, nested ids
GET    /packages/123
POST   /packages/123/releases
DELETE /packages/123/releases/456
```

Don't nest resources in the path (`/package/123/releases/456`). Pass identifiers
as parameters instead (see 2.5).

## 2.2 AWS Top-Level Key Convention

The only thing allowed in a top-level key is an object, e.g.
`{ order: { order_id, ... } }`. Name the key after the resource (singular for one
object, plural for a list), like AWS APIs.

```jsonc
// GOOD: object under a named top-level key
{ "package": { "package_id": "pkg_123", "name": "..." } }
{ "packages": [ { "package_id": "pkg_123" } ] }

// BAD: bare scalar / array / fields at the top level
{ "package_id": "pkg_123", "name": "..." }
[ { "package_id": "pkg_123" } ]
```

## 2.3 Always accept POST, accept GET where possible

Every route must accept `POST` (body params). Read-only routes should *also* accept
`GET` (query params) so they're easy to hit from a browser, `curl`, or a link.
Routes that mutate state MUST have `POST`, and can also have REST convention `PUT` etc.

```
# GOOD: read-only route reachable both ways
POST /package/get      { "package_id": "pkg_123" }
GET  /package/get?package_id=pkg_123

# BAD: GET that mutates state
GET  /package/delete?package_id=pkg_123
```

## 2.4 Common Verbs

Use these verbs. Don't invent synonyms — a synonym makes readers wonder how it
differs from the standard verb.

- `get`
- `list`
- `create`
- `delete`
- `update`

```
# GOOD
POST /package/get
POST /package/list

# BAD: synonyms for the standard verbs
POST /package/fetch
POST /package/getAll
POST /package/remove
```

## 2.5 No [id] in routes (use GET parameters)

Never put an id in the path. Pass it as a parameter — query param for `GET`, body
field for `POST`. This keeps routes static and verb-based (2.1).

```
# BAD: id in the path
GET  /package/pkg_123

# GOOD: id as a parameter
GET  /package/get?package_id=pkg_123
POST /package/get   { "package_id": "pkg_123" }
```

## 2.6 Common Error Object

Errors always come back as `{ error: { error_code, message } }` — never a bare
string or a top-level error field.

```jsonc
// GOOD
{ "error": { "error_code": "package_not_found", "message": "No package pkg_123" } }

// BAD: bare string, no code to branch on
{ "error": "No package pkg_123" }

// BAD: error fields at top level
{ "error_code": "package_not_found", "message": "..." }
```

## 2.7 `display_status` enum must always accompany boolean flags

Boolean flags are migration-safe, `status` enums are not. To hint at this,
we always write `status` as `display_status` and force users to use
booleans for flow control. Logic should not be built on `status` enums.

```ts
// GOOD: booleans drive logic, display_status is for humans only
{ is_complete: true, has_errors: false, display_status: "ready" }
if (build.is_complete && !build.has_errors) ship()

// BAD: status enum drives logic (breaks the moment a new status is added)
{ status: "ready" }
if (build.status === "ready") ship()
```

## 2.8 `snake_case`

All keys in request bodies, response bodies, and query parameters are
`snake_case`. This matches the "inheriting the API convention" rule in
[`code.md`](./code.md) §4.1: objects that cross the API boundary keep snake casing
even as you pass them around in TypeScript.

```jsonc
// GOOD
{ "package_id": "pkg_123", "created_at": "...", "is_private": false }

// BAD: camelCase across the API boundary
{ "packageId": "pkg_123", "createdAt": "...", "isPrivate": false }
```

## 2.9 Error as Early as Possible

Validate inputs and fail at the top of the handler, before doing any work. Return
a clear error (2.6) the moment you know the request can't succeed — don't get
halfway through a mutation and then bail.

```ts
// GOOD: bail early with throws, keep the happy path flat
async function handler(req) {
  const pkg = await getPackage(req.package_id)
  if (!pkg) throw new Error("Package not found")
  if (!req.user.can_edit) throw new Error("Forbidden")
  await mutate(pkg)
}

// BAD: nesting the happy path instead of bailing early
async function handler(req) {
  const pkg = await getPackage(req.package_id)
  if (pkg) {
    if (req.user.can_edit) {
      await mutate(pkg)
    } else {
      throw new Error("Forbidden")
    }
  } else {
    throw new Error("Package not found")
  }
}
```
