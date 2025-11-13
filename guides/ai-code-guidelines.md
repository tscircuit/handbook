## 1. Avoid polluting entrypoint files

When there's a high level file, the "main function export" etc. don't add a bunch of
specific logic to it or functions, keep it small and full of high-level function calls


## 2. Avoid Named Closures

When defining a large function, avoid having named functions inside of it. It's rarely
appropriate where a higher-scoped function is possible.


## 3. Avoid `as any` or `as unknown`
