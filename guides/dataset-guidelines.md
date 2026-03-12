# Dataset Guidelines

## Dataset Naming Guidelines

- When numbering, always increase from the last known dataset.
- When numberings always use two digits e.g. 01, 02

### Prefixes

- Z: Zero-Obstacle Datasets
- SRJ: Simple Route JSON Dataset

## Dataset Library Structure

- `index.ts` or `index.js` file with package.json pointing to it
- NO TRANSPILATION! (won't work with github installation if you transpile)
- DO NOT PUBLISH TO NPM (install datasets using github URL: `bun add https://github.com/tscircuit/dataset-srj05`)

```tsx
// index.ts

export { default as sample001 } from "./samples/sample001.json"
export { default as sample002 } from "./samples/sample002.json"
export { default as sample003 } from "./samples/sample003.json"
// ...
```

## Creating Simple Route JSON from Circuit JSON

You can create the standard dataset Simple Route Json via this import:

```tsx
import { getSimpleRouteJsonFromCircuitJson } from "@tscircuit/core"
```
