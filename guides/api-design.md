# API Design

# 1. General API Design (HTTP, Libraries, tscircuit)

## 1.1 Truthy flags to enable unusual, non-default behavior

## 1.2 Avoid nesting unless very necessary

# 2. HTTP API Design

## 2.1 RPC Routes

## 2.2 AWS Top-Level Key Convention

## 2.3 Always accept POST, accept GET where possible

## 2.4 Common Verbs

- `get`
- `list`
- `create`
- `delete`
- `update`

## 2.5 No [id] in routes (use GET parameters)


## 2.6 Common Error Object

`{ error: { error_code, message } }`

## 2.7 `display_status` enum must always accompany boolean flags

Boolean flags are migration-safe, `status` enums are not. To hint at this,
we always write `status` as `display_status` and force users to use
booleans for flow control. Logic should not be built on `status` enums.

## 2.8 `snake_case`

## 2.9 Error as Early as Possible



