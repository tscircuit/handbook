# Using the debug module

```
bun add -D debug @types/debug
```


```tsx
import Debug from "debug"

const debug = Debug("module-name:functionName")

function functionName(params: any) {
  debug("anything goes here", { params })
}
```


```bash
export DEBUG=module-name*

export DEBUG=module1*,module2*

export DEBUG=module1:functionName
```
