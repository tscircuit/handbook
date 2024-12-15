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

## Casing

We use the "google-style" convention for determining words. Here's a basic algorithm you can follow:

1. Convert your name into `snake_case`
2. Capitalize each word (except the first for `camelCase`)
3. 

### Database, Circuit JSON

Some objects use snake casing because they're "inheriting the API convention" which specifies underscores. This
is common and OK. Just keep in mind you'll want to use `snake_case` for these objects, even as you pass them
around.


## Variable Transparency (DO NOT RENAME UNLESS DISAMBIGUATING)

Variable transparency means a variable has the same name as it traverses throughout the codebase.

To a developer, this means if they see the word `db`, they immediately know it's type because it
never changes meaning.

Similarly, if they see `cartItemList` they know that it is the same type as everywhere else in the
code. It has never been renamed, and there is no alternate variable name for that same type.

There is an exception to this, say you have `userProfile` but you have two `userProfile`s in the same
function. In this case, you should **rename both of the variables to disambiguate (differentiate) them**

```ts
const currentUserProfile = userProfile
const friendUserProfile = await getFriend(userProfile)
```

### Never Rename a Transparent Variable

This is a common mistake when working between `snake_case` and `camelCase` code. Do not rename variables,
keep them the same unless you are disambiguating

```ts
// BAD: DO NOT DO THIS
const userProfile = ctx.user_profile
```
