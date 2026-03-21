# Bootstrapping Repos

Generally we do the following commands to bootstrap a repo. You can copy and paste these
to AI.

- `bun init -y`
- `rm -rf CLAUDE.md .cursor bun.lock index.ts`
- `mkdir lib tests tests/fixtures`
- `plop` (select biome.json, bunfig.toml)
- Edit the `bunfig.toml`, uncommenting the lines that disable the lock file (we generally disable lockfiles)
- `mkdir -p .github/workflows`
- `cd .github/workflows` and `plop` (select bun-typecheck.yml, bun-test.yml, bun-formatcheck.yml)
- `bun add -D @biomejs/biome`
- Add `.cursor` and `bun.lock` to the `.gitignore`

PREFERRED: Use github-style installation

- Edit the package.json to have the `"module": "lib/index.ts"`
- Set the `package.json` `"files": ["lib"]`
- In your `tsconfig.json`, modify the compiler option `"paths": { "lib/*": ["./lib/*"], "tests/*": ["./tests/*"] }`

LEGACY: Use tsup and npm

- `plop` `bun-pver-release.yml` into `.github/workflows`
- `bun add -D tsup`
- In package.json "scripts" add `"build": "tsup lib/index.ts --format esm --dts"`
- Set package.json `"files": ["dist"]`

## React Visualizers/Debuggers

All projects that have visualizations or interactive debuggers should add 

- `bun add -D cosmos vite cosmos-plugin-react-vite`
- Edit the `package.json` to have `build:site: "cosmos-export` and `start: "cosmos"`
- `plop` select `cosmos.decorator.tsx` and `cosmos.config.json`
- `mkdir pages`
- `echo 'export default () => "hello world"' > pages/hello-world.page.tsx`

Create a `vite.config.json` and add an alias for `lib` and `tests` (vite doesn't
read the tsconfig paths)

Confirm that `bun run start` shows hello world in the sidebar.

## Solvers (Algorithm Repos)

In addition to the React Visualizers, `bun add -D @tscircuit/solver-utils`
and create a `page.tsx` file that constructs your solver like so:

```tsx
// solver.page.tsx
import { GenericSolverDebugger } from "@tscircuit/solver-utils/react"
import { MySolver } from "lib/my-solver"

export default <GenericSolverDebugger createSolver={() => new MySolver()} />
```

```tsx
// mysolver.tsx
import { BaseSolver } from "@tscircuit/solver-utils"
import type { GraphicsObject } from "graphics-debug"

export class MySolver extends BaseSolver {
  constructor(params: {}) {}

  _step() {
    this.solved = true
  }

  visualize(): GraphicsObject {
    // Add stuff here
    return {}
  }
}
```
