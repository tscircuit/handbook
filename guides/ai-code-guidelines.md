# AI Code Guidelines

Guidelines for code written by or with AI assistants. These keep AI-generated code
readable and consistent with the rest of the codebase. See also [`code.md`](./code.md)
for general style and [`api-design.md`](./api-design.md) for API conventions.

## 1. Avoid polluting entrypoint files

When there's a high-level file — the "main function export", a CLI entrypoint, a
top-level `index.ts` — don't add specific logic to it. Keep it small and full of
high-level function calls so it reads like a table of contents.

```ts
// BAD: entrypoint stuffed with logic
export async function build(opts: BuildOptions) {
  const files = await readdir(opts.dir)
  const circuitJson = []
  for (const file of files) {
    if (!file.endsWith(".tsx")) continue
    const source = await readFile(file, "utf-8")
    // ...50 more lines of parsing, transforming, writing...
  }
}

// GOOD: entrypoint reads like a summary
export async function build(opts: BuildOptions) {
  const files = await getCircuitFiles(opts.dir)
  const circuitJson = await renderFilesToCircuitJson(files)
  await writeBuildOutput(circuitJson, opts)
}
```

AI assistants tend to inline everything into one function. Push the detail down
into named, separately-testable functions.

## 2. Avoid Named Closures

When defining a large function, avoid declaring named functions inside of it. It's
rarely appropriate when a higher-scoped function is possible — a hoisted function is
easier to test, reuse, and read.

```ts
// BAD: named closure trapped inside the function
function processNets(nets: Net[]) {
  function normalizeNet(net: Net) {
    // ...
  }
  return nets.map(normalizeNet)
}

// GOOD: hoist it to module scope
function normalizeNet(net: Net) {
  // ...
}

function processNets(nets: Net[]) {
  return nets.map(normalizeNet)
}
```

Anonymous inline callbacks (`nets.map((net) => ...)`) are fine. This rule targets
*named* `function foo()` / `const foo = () => ...` declarations nested inside a
larger function body.

## 3. Avoid `as any` or `as unknown`

`as any` and `as unknown` disable type checking exactly where bugs hide. AI
assistants reach for them to silence a type error instead of fixing the type.

```ts
// BAD: casts away the error, loses all type safety
const pkg = JSON.parse(body) as any
ship(pkg.pakage_id) // typo silently compiles, fails at runtime

// GOOD: type the value, let the compiler catch mistakes
const pkg: Package = JSON.parse(body)
ship(pkg.pakage_id) // compile error: did you mean package_id?
```

If you genuinely need an escape hatch:

- Prefer a type guard (`if (isPort(x)) { ... }`) over a cast.
- Prefer a precise cast (`as Port`) over `as any`.
- If you must write `as unknown as X`, leave a comment explaining why the types
  can't line up, so the next reader doesn't assume it's an accident.

The same applies to `// @ts-ignore` and `// @ts-expect-error` — fix the type, or
explain in a comment why you can't.
