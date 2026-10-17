# Standard Process for Graphics Debugging

When building and debugging geometric, routing, and conversion algorithms (such as `dsn-converter`, autorouters, or layout solvers), inspecting raw coordinate arrays in terminal logs makes it hard to spot off-by-one, scaling, or orientation bugs. We use [`graphics-debug`](https://github.com/tscircuit/graphics-debug) together with the [`debug`](https://www.npmjs.com/package/debug) package to emit structured graphics stages (`points`, `lines`, `rects`, `circles`) that render directly to Markdown tables and inline SVG visualizations.

## 1. Instrumenting Code with `debug` and Graphics Objects

Create a namespaced logger ending in `:graphics` (for example, `tscircuit:dsn-converter:graphics`) and check `debugGraphics.enabled` before constructing debug geometry so there is zero overhead during normal execution:

```tsx
import Debug from "debug"

const debugGraphics = Debug("tscircuit:dsn-converter:graphics")

if (debugGraphics.enabled) {
  const graphics = dsnSessions.map(simplifyToPoints)
  debugGraphics("graphics section start dsn input")
  debugGraphics({ graphics })
}
```

### Multi-Stage Debugging (`stage1`, `stage2`, ...)

For multi-step pipelines, emit a descriptive stage header and a `GraphicsObject` at each transformation boundary so you can compare stages side-by-side:

- **`stage1-dsn-input.md` sections**: Capture the raw parsed input coordinates, bounds, pads, and wires before coordinate conversion.
- **`stage2-circuit-json-from-dsn.md` sections**: Capture the transformed Circuit JSON PCB elements (`pcb_component`, `pcb_smtpad`, `pcb_trace`, `pcb_via`) in output space.

```tsx
import Debug from "debug"
import type { GraphicsObject } from "graphics-debug"

const debugGraphics = Debug("tscircuit:dsn-converter:graphics")

export function convertDsnSessionToCircuitJson(dsnInput: DsnInput) {
  if (debugGraphics.enabled) {
    const stage1Graphics: GraphicsObject = {
      title: "stage1-dsn-input",
      points: dsnInput.pins.map((pin) => ({
        x: pin.x,
        y: pin.y,
        label: pin.name,
        color: "blue",
      })),
      lines: dsnInput.wires.map((wire) => ({
        points: wire.points,
        strokeColor: "red",
      })),
      rects: dsnInput.Keepouts.map((box) => ({
        center: box.center,
        width: box.width,
        height: box.height,
        fill: "rgba(255, 0, 0, 0.2)",
      })),
    }
    debugGraphics("stage1-dsn-input.md")
    debugGraphics({ graphics: stage1Graphics })
  }

  const circuitJson = transformDsnToCircuitJson(dsnInput)

  if (debugGraphics.enabled) {
    const stage2Graphics: GraphicsObject = {
      title: "stage2-circuit-json-from-dsn",
      points: extractPcbPoints(circuitJson),
      lines: extractPcbTraceLines(circuitJson),
      rects: extractPcbPadRects(circuitJson),
    }
    debugGraphics("stage2-circuit-json-from-dsn.md")
    debugGraphics({ graphics: stage2Graphics })
  }

  return circuitJson
}
```

## 2. Running Tests with `graphics-debug` CLI

Enable all `:graphics` debug namespaces via `DEBUG=tscircuit:*graphics` and pipe your test output into `graphics-debug`:

```bash
export DEBUG=tscircuit:*graphics
bun test ./path/to/test | graphics-debug
# Outputs: graphics.debug.md
```

## 3. Inspecting `graphics.debug.md` Output

The `graphics-debug` CLI parses every logged `{ graphics }` payload and generates `graphics.debug.md` containing:

1. **Stage Sections** (e.g. `stage1-dsn-input.md`, `stage2-circuit-json-from-dsn.md`): Clearly separated sections for each pipeline stage.
2. **Coordinate Tables**: Markdown tables listing all points, lines, and rects with their exact `(x, y)` coordinates, dimensions, layers, and labels so you can verify numerical alignment across stages.
3. **SVG Visualizations**: Rendered SVG diagrams showing all points, lines, and bounding boxes in a unified viewport for immediate visual inspection.
