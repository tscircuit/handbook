# Using the debug module in tscircuit

## Overview

The `debug` module is extensively used throughout the tscircuit codebase for logging and debugging purposes. It provides a flexible way to enable/disable debug output based on environment variables, making it perfect for development, testing, and troubleshooting.

## Installation

```bash
bun add -D debug @types/debug
```

## Basic Usage

### Import and Setup

```tsx
import Debug from "debug"

const debug = Debug("module-name:functionName")

function functionName(params: any) {
  debug("anything goes here", { params })
}
```

### Conventional tscircuit Usage Patterns

Based on the tscircuit codebase, here are the established patterns:

#### 1. Module-Level Debuggers

Create debuggers for specific modules or components:

```tsx
import Debug from "debug"

// For core functionality
const debug = Debug("tscircuit:core")

// For renderable components
const debug = Debug("tscircuit:renderable")

// For specific component types
const debug = Debug("tscircuit:resistor")
```

#### 2. Function-Specific Debuggers

Create debuggers for specific functions or operations:

```tsx
import Debug from "debug"

const debug = Debug("Group_doInitialSchematicLayoutMatchpack")

export function doInitialSchematicLayoutMatchPack() {
  debug("Processing child nodes for input problem")
  // ... implementation
}
```

#### 3. Conditional Debugging

Use debug conditionally based on component state:

```tsx
import Debug from "debug"

const debug = Debug("tscircuit:component")

class MyComponent {
  render() {
    debug(`${phase}:${startOrEnd} ${this.getString()}`)
    
    // Conditional debug based on component state
    if (debug.enabled && global.debugGraphics) {
      global.debugGraphics.push({
        title: "component-state",
        graphics: this.getDebugGraphics()
      })
    }
  }
}
```

## Enabling Debug Output

### Development Environment

Set the `DEBUG` environment variable to enable specific debug namespaces:

```bash
# Enable all tscircuit debug output
export DEBUG=tscircuit*

# Enable specific module debug output
export DEBUG=tscircuit:core*

# Enable multiple specific debuggers
export DEBUG=tscircuit:core*,tscircuit:renderable*

# Enable specific function debug output
export DEBUG=Group_doInitialSchematicLayoutMatchpack
```

### Test Patterns

When debugging node modules during testing, use these patterns:

#### 1. Test-Specific Debug Output

```bash
# Run tests with debug output
DEBUG=tscircuit:core* bun test

# Enable debug for specific test file
DEBUG=tscircuit:renderable* bun test renderable.test.ts
```

#### 2. Debug Graphics in Tests

For tests that generate debug graphics:

```tsx
import { writeGlobalDebugGraphics } from "./fixtures/writeGlobalDebugGraphics"

test("component rendering", () => {
  // Your test code here
  
  // Write debug graphics to files
  writeGlobalDebugGraphics()
})
```

#### 3. Conditional Debug in Tests

```tsx
import Debug from "debug"

const debug = Debug("test:component")

test("component behavior", () => {
  if (debug.enabled) {
    console.log("Running component test with debug enabled")
  }
  
  // Test implementation
})
```

### Browser Context

To enable debug output in browser environments, set localStorage:

```javascript
// Enable all tscircuit debug output
localStorage.setItem('debug', 'tscircuit*')

// Enable specific modules
localStorage.setItem('debug', 'tscircuit:core*,tscircuit:renderable*')

// Enable specific functions
localStorage.setItem('debug', 'Group_doInitialSchematicLayoutMatchpack')

// Disable debug output
localStorage.removeItem('debug')
```

## Advanced Usage Patterns

### 1. Debug with Structured Data

```tsx
const debug = Debug("tscircuit:component")

debug("Component state:", {
  name: this.name,
  props: this.props,
  children: this.children.length
})
```

### 2. Debug with Conditional Logic

```tsx
const debug = Debug("tscircuit:layout")

if (debug.enabled) {
  debug("Layout calculation:", {
    width: this.width,
    height: this.height,
    constraints: this.constraints
  })
}
```

### 3. Debug Graphics Integration

For components that generate visual debug output:

```tsx
const debug = Debug("tscircuit:autorouter")

if (debug.enabled && global.debugGraphics) {
  global.debugGraphics.push({
    title: "autorouter-debug",
    graphics: this.solver?.preview() || undefined
  })
}
```

## Best Practices

### 1. Naming Conventions

- Use `tscircuit:` prefix for all tscircuit-related debuggers
- Follow the pattern: `tscircuit:module:function`
- Use descriptive names that indicate the component or operation

### 2. Performance Considerations

- Debug calls are automatically stripped in production builds
- Use conditional checks for expensive operations:

```tsx
const debug = Debug("tscircuit:expensive")

if (debug.enabled) {
  const expensiveData = this.calculateExpensiveData()
  debug("Expensive calculation result:", expensiveData)
}
```

### 3. Debug Output Organization

- Group related debug output under the same namespace
- Use consistent formatting for similar types of data
- Include relevant context in debug messages

### 4. Testing with Debug

- Enable debug output during development and testing
- Use specific debug namespaces to focus on relevant output
- Consider writing debug graphics to files for visual inspection

## Common Debug Namespaces in tscircuit

Based on the codebase analysis:

- `tscircuit:core` - Core functionality
- `tscircuit:renderable` - Renderable components
- `Group_doInitialSchematicLayoutMatchpack` - Group layout operations
- `tscircuit:autorouter` - Auto-routing functionality
- `tscircuit:component` - General component operations

## Troubleshooting

### Debug Output Not Appearing

1. Check that the `DEBUG` environment variable is set correctly
2. Verify the debug namespace matches your code
3. Ensure you're not in a production build where debug is stripped

### Browser Debug Not Working

1. Check that localStorage is set correctly
2. Refresh the page after setting localStorage
3. Verify the debug namespace matches your code

### Test Debug Issues

1. Ensure the `DEBUG` environment variable is set when running tests
2. Check that your test environment supports environment variables
3. Use specific debug namespaces to avoid overwhelming output
