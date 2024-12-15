# tscircuit Code

tscircuit code has a consistent pragmatic style with an emphasis on readability.

Where possible, we always prefer linting rules for enforcement of style.

## Two-Parameter Rule

We never use more than two parameters in a function, whenever there are more than two parameters we either...

1. Convert to "named parameters" and take a single parameter as an object
2. Switch to the context passing pattern.

- [core#422](https://github.com/tscircuit/core/pull/422#discussion_r1885804180)

## Context-Passing Pattern

The context-passing pattern means using a two-parameter function where...
- the first parameter is a function-specific named object
- the second parameter is a common named object ("the context")

```ts
export function renderSymbol(params: { symbolName: string, shape: any }, ctx: AppContext) {
  // ...
}
```


When using the context-passing pattern, the most important thing is to make sure you're not inventing too many contexts. Generally
there will be one or two obvious contexts for an app. It's common to create a named interface like `AuthContext` or `ComponentContext`

- [core#422](https://github.com/tscircuit/core/pull/422#discussion_r1885804180)

## Banned Words

There are some words that are so bad, that a more domain-specific word is always better.

- `data`, `info`, `value`, `param` - Just "say the thing", e.g. `cartData` -> `currentOrderDetails`
