# Using Yalc for Local Development

## Overview

tscircuit is built on a multi-package architecture where several npm packages work together. When contributing, you'll often need to modify multiple packages simultaneously. Rather than waiting for PR approvals to test cross-package changes, you can use yalc for rapid local development.

[yalc](https://github.com/wclr/yalc) creates a local package registry on your machine, letting you:
- Test changes across packages instantly
- Verify fixes before submitting PRs
- Maintain a fast development cycle
- Ensure type safety across package boundaries

## Quick Start

```bash
# Initial setup
bun install -g yalc       # Install yalc globally

# In your package directory (e.g. @tscircuit/builder):
bun run build            # Build TypeScript first!
yalc publish            # Publish to local registry

# In dependent project (e.g. @tscircuit/editor):
yalc add @tscircuit/builder   # Use local version

# Development cycle:
bun run build           # After changes, rebuild
yalc push              # Update all dependent projects

# Before committing:
yalc remove --all      # Remove local links
bun install           # Restore npm versions
```

## Installation

Install yalc globally so you can use it in any project:

```bash
bun install -g yalc
```

## Basic Usage

1. In your package that you want to test locally (e.g., your library), first build the TypeScript files:
```bash
bun run build
```
This crucial first step compiles TypeScript and generates type definition files.

2. Then publish your package locally:
```bash
yalc publish
# or alternatively:
yalc push
```
This command creates a local store of your package, similar to a mini private npm registry on your machine. You can use either `publish` or `push` - they both publish your package, but `push` is a shorthand that also updates all existing installations.

3. In the project where you want to use the local package (e.g., your app):
```bash
yalc add my-package
```
This replaces the npm version with your local version and updates package.json with a special yalc reference.

4. When you make changes to your package:
```bash
bun run build  # Compile TypeScript, generate types, etc.
yalc push      # Push the new build to all projects using this package
```

The build step is crucial because:
- It generates up-to-date TypeScript type definitions
- It compiles any TypeScript/JSX to JavaScript
- It ensures the package contents match what would be published to npm

## Best Practices

### Watch Mode

Watch mode automatically publishes your package whenever files change. This is ideal during active development:

```bash
yalc publish --watch
```

Important notes about watch mode:
- It only handles the publishing step
- You still need to run `bun run build` manually to update compiled files
- Changes won't appear in your app until after building

### Cleaning Up

Before committing code or deploying, you must remove yalc links:

1. Remove all yalc links:
```bash
yalc remove --all
```
This removes the special yalc references from package.json

2. Reinstall the regular npm versions:
```bash
bun install
```
This ensures your project uses the official published versions

### Common Issues and Solutions

- **Changes not showing up?**
  - Run `bun run build` in your package
  - Try deleting node_modules and reinstalling: `rm -rf node_modules && bun install`
  - Check if watch mode is running

- **TypeScript errors?**
  - Make sure you built the package before pushing
  - Check if types are included in your build output

- **Package not found?**
  - Confirm you ran `yalc publish` in the package directory
  - Try removing and re-adding with `yalc remove my-package && yalc add my-package`

## Complete Development Workflow

Here's a step-by-step workflow for local package development:

1. Initial setup:
```bash
# In your package directory
cd my-package
bun run build                # Build the initial version
yalc publish --watch        # Start watching for changes

# In your project directory
cd ../my-project
yalc add my-package         # Link to local version
```

2. During development:
- Make changes to your package code
- Run `bun run build` to compile changes
- Changes will automatically push to your project
- Test the changes in your project

3. When finished:
```bash
# In your package directory
# Stop the watch process (Ctrl+C)

# In your project directory
cd my-project
yalc remove --all           # Remove all yalc links
bun install                 # Restore npm versions
```

4. Finally:
- Commit your changes
- Publish your package to npm if ready
