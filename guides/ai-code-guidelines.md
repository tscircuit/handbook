# AI Code Guidelines

Guidelines for code written by or with AI assistants. These keep AI-generated code
readable and consistent with the rest of the codebase. See also [`code.md`](./code.md)
for general style and [`api-design.md`](./api-design.md) for API conventions.

## 1. Avoid polluting entrypoint files

A high-level file — the CLI entrypoint, the main export, a top-level `index.ts` —
should contain high-level calls, not the implementations. Keep the logic in sibling
modules and import it, so the entrypoint reads like a summary.

```ts
// BAD: index.ts defines every helper itself
function getCircuitFiles(dir: string) { /* ...20 lines... */ }
function renderFilesToCircuitJson(files: string[]) { /* ...40 lines... */ }
function writeBuildOutput(json: CircuitJson, opts: BuildOptions) { /* ...30 lines... */ }

export async function build(opts: BuildOptions) {
  const files = getCircuitFiles(opts.dir)
  const circuitJson = renderFilesToCircuitJson(files)
  writeBuildOutput(circuitJson, opts)
}

// GOOD: index.ts imports and orchestrates
import { getCircuitFiles } from "./get-circuit-files"
import { renderFilesToCircuitJson } from "./render-files-to-circuit-json"
import { writeBuildOutput } from "./write-build-output"

export async function build(opts: BuildOptions) {
  const files = await getCircuitFiles(opts.dir)
  const circuitJson = await renderFilesToCircuitJson(files)
  await writeBuildOutput(circuitJson, opts)
}
```

AI assistants tend to stuff entrypoint files, defining every helper inline instead
of splitting them into their own modules.

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
