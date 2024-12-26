# Using yalc

## Installation

```bash
# Install yalc globally on your machine
npm install -g yalc
```

## Usage

## Publishing Local Package

```bash
# In your local package directory (where package.json is)
# Example: /your-local-package
yalc publish
```

## To watch for changes and publish automatically
```bash
# Still in your package directory
yalc publish --watch
```
## Adding Package to Your Project

```bash
# In your project directory (where you want to use the package)
# Example: /your-project
yalc add my-package
```

## Update Package


```bash
# In your project directory
# After the package source has changed
yalc update
```
## Update Specific Package

```bash
# In your project directory
yalc update my-package
```

## Remove Package

```bash
# In your project directory
# Remove specific package
yalc remove my-package
```

## Remove all yalc links and restore package.json

```bash
# In your project directory
yalc remove --all
```

# Development Workflow

```bash
# 1. In your package directory (/your-local-package)
yalc publish

# 2. In your project directory (/your-project)
yalc add my-package

# 3. After making changes to your package
# In your package directory
yalc publish --push

# 4. When finished developing
# In your project directory
yalc remove --all
npm install
```
