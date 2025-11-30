# Guidelines for Typescript Parser/Serializer Libraries

Often, when converting between formats, you need an intermediate representation
in Typescript of whatever format you're parsing. The libraries in the `tscircuit`
ecosystem for this ([`kicadts`](https://github.com/tscircuit/kicadts), [`dsnts`](https://github.com/tscircuit/dsnts), [`stepts`](https://github.com/tscircuit/stepts), [`lbrnts`](https://github.com/tscircuit/lbrnts)) all follow a similar pattern:

- Every “token” or entity in the file format is modeled as a class
- There is a small set of well-known root classes
- There are top-level parse functions and instance methods to serialize back to text
- The library is designed to round-trip existing files *without* losing information

These guidelines are meant to help you design new parser/serializer libraries
that feel consistent with those existing packages.

---

## 1. Goals & Non‑Goals

### 1.1 Goals

- **Type-safe intermediate representation**  
  Every construct in the target format should correspond to a TypeScript class
  with strongly typed properties, not untyped `any` blobs.

- **Round-trippable**  
  You should be able to:
  1. Parse an existing file into your class model
  2. Mutate it in TypeScript
  3. Serialize back to text
  4. Get functionally equivalent output (ideally byte‑identical)

- **Ergonomic to author by hand**  
  Higher-level code should be able to *build* new files using constructors,
  helper methods, and small utility functions, without manually dealing with
  tokens or low-level syntax.

- **Predictable formatting**  
  Serialization should be deterministic: the same model produces the same string
  every time. This is important for snapshot tests and diff‑friendly workflows.

### 1.2 Non‑Goals

- **Not a full semantic validator**  
  The library doesn’t have to enforce every domain-specific rule. Stick to
  structural correctness; let other tools (or calling code) do heavy validation.

- **Not a generic parser framework**  
  You’re designing a *format-specific* library with a nice API, not a generic
  parsing framework. It’s fine to have some shared utilities, but the main API
  should be tailored to the specific file format.

---

## 2. Overall Library Shape

New libraries in this family should have a similar top‑level feel:

- A root **TypeScript module** (published on npm)
- A `lib/` directory containing the implementation
- A `tests/` directory with round-trip and unit tests
- A small `index.ts` (or similarly named file) that re‑exports the public API

### 2.1 Public API surface

Try to provide:

- One or more **root document classes**, e.g.:

  - `KicadSch`, `KicadPcb`, `Footprint` in `kicadts`
  - `SpectraDsn`, `SpectraSes` in `dsnts`
  - `Repository` for STEP files in `stepts`
  - `LightBurnProject` in `lbrnts`

- Top‑level **parse functions** that return root classes:

  - `parseKicadSch(text) → KicadSch`
  - `parseKicadPcb(text) → KicadPcb`
  - `parseKicadMod(text) → Footprint`
  - `parseSpectraDsn(text) → SpectraDsn`
  - `parseSpectraSes(text) → SpectraSes`
  - `parseRepository(text) → Repository`
  - Or a static method when that feels nicer, e.g.  
    `LightBurnProject.parse(text) → LightBurnProject`

- One canonical **serialization method** per root class:

  - `root.getString()` (for S-expression / text-based formats)
  - `repo.toPartFile(options?)` (for STEP)

Pick one naming convention per library and stick to it for all root types
(e.g. always `getString()` for “give me the text of this file”).

---

## 3. Class-Based Model

### 3.1 One class per token/entity

For text-based structured formats (S-expressions, STEP, etc.), follow this pattern:

- **Every syntactic token or entity becomes a class**, e.g.:

  - KiCad: `KicadSch`, `Sheet`, `SchematicSymbol`, `Wire`, `Footprint`, `PcbNet`, …
  - Specctra DSN: `SpectraDsn`, `Structure`, `Boundary`, `Network`, `Net`, `Library`, `Padstack`, …
  - STEP: `CartesianPoint`, `Direction`, `Plane`, `VertexPoint`, `ManifoldSolidBrep`, …

- Classes should be **small, focused, and mostly data + helpers**, not giant
  god-objects. Behavior is mostly:

  - Converting to/from the underlying syntax/AST
  - Managing child entities and references
  - Providing convenience getters/setters

### 3.2 Common base class

Introduce one or more base classes that all nodes/entities extend. Examples:

- For S-expression-like formats (`kicadts`, `dsnts`), a base `SxClass` that
  knows how to render itself as an S-expression and expose children.
- For STEP (`stepts`), a base entity type with a `type` tag and the
  ability to serialize to a STEP entity line.

Your base class should typically provide:

- A **discriminant**:

  ```ts
  abstract class BaseNode {
    abstract readonly type: string; // or token/kind
  }
  ```

* A **children accessor** for tree walking:

  ```ts
  abstract getChildren(): BaseNode[];
  ```

* A **serialization hook** (often used by the root serializer):

  ```ts
  abstract toSource(): string | unknown; // depends on format
  ```

The exact shape can vary per library, but try to keep it consistent within the
ecosystem (e.g. `getChildren()` is the standard name for “give me child nodes”).

---

## 4. Constructors & Ergonomics

### 4.1 “Init object” constructors

Prefer **object-shaped constructors** where all properties are passed as a single
`init` object, often with optional fields:

```ts
class LightBurnProject {
  constructor(
    init?: {
      appVersion?: string;
      formatVersion?: string;
      materialHeight?: number;
      mirrorX?: boolean;
      mirrorY?: boolean;
      children?: LightBurnBaseElement[];
    } = {}
  ) {
    // assign defaults + init
  }
}
```

Benefits:

* Call sites remain readable as the number of fields grows
* It’s easy to add optional properties without breaking callers
* TypeScript can give great autocomplete on field names

### 4.2 Flexible value inputs

Where the underlying syntax supports multiple ways to express the same thing
(e.g. positions, vectors, matrices), make your constructors **accept ergonomic
variants**, not just “raw token objects”. For example:

* Allow a position to be:

  * A dedicated class (`new At(...)`)
  * A tuple (`[x, y]` or `[x, y, angle]`)
  * An object (`{ x, y, angle? }`)

…and normalize internally. This pattern is used extensively in the KiCad
library for positions, sizes, and coordinate lists.

General pattern:

```ts
type PositionInput =
  | At
  | [number, number]
  | { x: number; y: number; angle?: number };

class Footprint {
  position: At;

  constructor(init: { position: PositionInput; /* ... */ }) {
    this.position = normalizeAt(init.position);
  }
}
```

Do this wherever it significantly improves ergonomics (coordinates, colors,
simple 2D/3D vectors, etc).

---

## 5. Root Documents & Parse Functions

Design a clear path from **raw text → root class instance**.

### 5.1 Specialized parse functions

For each top-level file type, expose a dedicated parse function:

* `parseFooMain(text: string): FooMain`
* `parseFooAux(text: string): FooAux`

These should:

* Validate that the root node is of the expected type
* Throw a helpful error otherwise
* Return a fully typed root instance (not a generic AST node)

Example shape:

```ts
export function parseFooMain(source: string): FooMain {
  const [root] = parseFooSexpr(source); // or other low-level parser
  if (!(root instanceof FooMain)) {
    throw new Error(`Expected FooMain root, got ${root.type}`);
  }
  return root;
}
```

### 5.2 Generic parse function (optional but recommended)

If your format is “tree of tokens” based (e.g. S-expressions), also expose a
low-level parse function that returns an array of generic nodes:

* `parseFooSexpr(text: string): BaseNode[]`

This is useful when:

* You want to inspect or manipulate **unknown** or not-yet-modeled tokens
* You need a generic inspector or transformer for the entire tree

Every node type should extend a shared base (`BaseNode` / `SxClass`) so
callers can do:

```ts
for (const node of parseFooSexpr(text)) {
  walk(node); // uses getChildren()
}
```

---

## 6. Serialization

### 6.1 Root serialization method

Each root document class should have a **single obvious way** to get the
complete text:

* `getString(): string`
* or, for special formats, something like `toPartFile(options?): string`

This method should:

* Always produce deterministic output for the same instance
* Include any required header/footer sections for the format
* Defer to child nodes’ `toSource()` or equivalent where possible

### 6.2 Deterministic formatting and round‑trip tests

Follow these principles:

* **Property order is fixed**, not dependent on insertion order in user code
* Arrays are emitted in a stable order
* Whitespace, quoting, and numeric formatting are consistent

Then, add tests that:

1. Load one or more canonical fixture files from the upstream tool
2. Parse them into your model
3. Immediately serialize back out
4. Compare against the original text (snapshot or exact equality)

This is the main guardrail that:

* Your parser is robust enough to handle real-world files
* Your serializer won’t drift in subtle ways as the code evolves

---

## 7. Tree Navigation & Children

### 7.1 Strongly-typed properties *and* generic `getChildren()`

For each node that aggregates other nodes:

* Expose **strongly-typed fields**:

  ```ts
  class KicadSch extends SxClass {
    sheets: Sheet[];
    symbols: SchematicSymbol[];
    wires: Wire[];
    // ...
  }
  ```

* Implement **`getChildren()`** returning all direct child nodes as
  a `BaseNode[]`:

  ```ts
  getChildren(): BaseNode[] {
    return [
      ...this.sheets,
      ...this.symbols,
      ...this.wires,
      // ...
    ];
  }
  ```

Callers then have two ways to work:

* **Structured**: `schematic.symbols.push(new SchematicSymbol(...))`
* **Generic**: `walk(schematic)` with a tree-walker that just uses `getChildren()`

This pattern is shared across the S-expression-based libraries and makes it easy
to write generic tooling.

### 7.2 Optional child helper methods

Consider adding tiny helpers on complex root types:

* `addSheet(sheet: Sheet): void`
* `addWire(wire: Wire): void`
* `getChildren()` as described above

These are *nice-to-haves*, not mandatory. Don’t overdo it: if direct array
mutation is ergonomic enough, prefer that.

---

## 8. Identity, References & Repositories

Some formats (notably STEP) are **graph-like** rather than pure trees: entities
reference each other by IDs. In those cases:

### 8.1 Repository-style root

Use a “repository” object to own all entities:

* A `Repository` manages:

  * A mapping from IDs → entity instances
  * Allocation of new IDs when adding entities
  * Serialization order

Example responsibilities:

```ts
class Repository {
  add<T extends BaseEntity>(entity: T): Ref<T> { /* ... */ }
  get<T extends BaseEntity>(id: EntityId): T | undefined { /* ... */ }
  entries(): Iterable<[EntityId, BaseEntity]> { /* ... */ }
  toPartFile(opts?: PartFileOptions): string { /* ... */ }
}
```

### 8.2 Type-safe references

Define a small **reference wrapper** type rather than passing raw IDs around:

```ts
interface Ref<T> {
  readonly id: number;
  resolve(repo: Repository): T;
}
```

Then your entity classes can express their dependencies with type-safe refs:

```ts
class VertexPoint extends BaseEntity {
  constructor(
    public name: string,
    public point: Ref<CartesianPoint>
  ) { /* ... */ }
}
```

This gives:

* Type-safe links between entities
* Ergonomic resolution (`ref.resolve(repo)`)
* The ability to keep raw IDs in the serialized form while still being nice
  to work with in TypeScript

---

## 9. Unknown / Future Tokens

To keep your library future-proof:

* Provide a generic **“unknown token” class** or mechanism
* When the parser encounters an unrecognized token:

  * Wrap it into an `UnknownSomething` node
  * Preserve its raw representation (so it can be serialized back unchanged)
  * Attach it into the tree at the right place

This ensures:

* New tool versions (with new fields/tokens) won’t break your parser
* Round-tripping doesn’t silently drop user data

---

## 10. File & Module Layout

You don’t have to copy this exactly, but a typical layout looks like:

```txt
lib/
  index.ts             # re-exports public API
  ast/                 # base AST / S-expression classes (optional)
  parser/              # low-level tokenizer / parser
  format-root.ts       # root classes (FooMain, FooBoard, etc.)
  entities/            # individual entity classes
tests/
  fixtures/            # sample upstream files
  roundtrip/           # parse → serialize → compare tests
  unit/                # targeted tests for tricky entities
```

Guidelines:

* Keep **format-agnostic utilities** (generic S-expression utilities, etc.)
  in separate modules so they can be reused across libraries if helpful.
* Keep **public API** small and explicit: export root types, entity types,
  parse functions, and a handful of helpers. Avoid leaking internal helpers.

---

## 11. Suggested Minimum API for a New Library

When in doubt, aim for something like:

```ts
// Public types
export class FooMain extends BaseNode {
  // format-specific fields...
  getChildren(): BaseNode[];
  getString(): string;
}

export abstract class FooEntity extends BaseNode {
  // common behavior for non-root entities
}

// Parsing
export function parseFooMain(source: string): FooMain;
export function parseFooSexpr(source: string): BaseNode[]; // optional

// Re-exports of important entity classes
export { FooSubEntity, FooOtherEntity } from "./entities";
```

And if the format is ID/reference heavy:

```ts
export class Repository {
  add<T extends FooEntity>(entity: T): Ref<T>;
  get<T extends FooEntity>(id: EntityId): T | undefined;
  entries(): Iterable<[EntityId, FooEntity]>;
  toPartFile(opts?: PartFileOptions): string;
}

export interface Ref<T extends FooEntity> {
  readonly id: number;
  resolve(repo: Repository): T;
}

export function parseRepository(source: string): Repository;
```

If you match these shapes, your library will feel “at home” alongside
`kicadts`, `dsnts`, `stepts`, and `lbrnts`, and higher-level tools can work
with it more or less uniformly.

