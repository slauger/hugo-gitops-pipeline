# Build Steps

The pipeline provides flexible, customizable build steps. Default steps are defined in the builder image, but projects can override or extend them.

## Phases

The pipeline runs three phases by default:

| Phase | Default Steps | Description |
|-------|---------------|-------------|
| `build` | `npm ci`, `copy-assets`, `hugo --minify` | Install dependencies and build the site |
| `lint` | `htmlhint` | Validate HTML output |
| `test` | (none) | Run tests |

## Customization

### Override Phase Steps

Replace the default steps for a phase:

```json
{
  "build": {
    "steps": [
      "npm ci",
      "curl https://api.example.com/data.json -o hugo/data/api.json",
      "copy-assets project.json",
      "cd hugo && hugo --minify"
    ]
  }
}
```

### Skip a Phase

Remove a phase from the `steps` array:

```json
{
  "steps": ["build", "test"]
}
```

This skips the `lint` phase entirely.

### Add a Custom Phase

Add new phases to the pipeline:

```json
{
  "steps": ["build", "optimize", "lint", "test"],
  "optimize": {
    "steps": [
      "imagemin hugo/public/images/* -o hugo/public/images/"
    ]
  }
}
```

## Environment Variables

Steps can use environment variables that are set by the pipeline:

| Variable | Description | Example |
|----------|-------------|---------|
| `${ENVIRONMENT}` | Hugo environment name | `production`, `staging` |
| `${OUTPUT}` | Build output directory | `public` |
| `${HUGO_SOURCE}` | Hugo source directory | `hugo` |
| `${BASEURL}` | Base URL (if set) | `https://example.com` |
| `${CONFIG}` | Project config file path | `project.json` |

### Variable Syntax

Use shell parameter expansion for defaults:

```json
{
  "build": {
    "steps": [
      "hugo --environment ${ENVIRONMENT:-production}"
    ]
  }
}
```

## Examples

### Standard Hugo Site

Uses all defaults, no customization needed:

```json
{
  "hugo": {
    "source": "hugo"
  },
  "environments": { ... }
}
```

### Non-Hugo Site (Custom Build)

For sites that don't use Hugo (e.g., a custom Node.js renderer):

```json
{
  "steps": ["build"],
  "build": {
    "steps": [
      "npm ci",
      "node scripts/render.js",
      "cp -r dist/* public/"
    ]
  },
  "output": "public",
  "environments": { ... }
}
```

### Hugo with Image Optimization

Add an optimization phase after build:

```json
{
  "steps": ["build", "optimize", "lint"],
  "optimize": {
    "steps": [
      "npx imagemin-cli hugo/public/images/* --out-dir=hugo/public/images/"
    ]
  }
}
```

### Fetch External Data Before Build

Fetch API data and store it in Hugo's data directory:

```json
{
  "build": {
    "steps": [
      "npm ci",
      "curl -s https://api.example.com/products.json -o hugo/data/products.json",
      "copy-assets project.json",
      "cd ${HUGO_SOURCE} && hugo --minify --environment ${ENVIRONMENT:-production}"
    ]
  }
}
```

## Default Steps Reference

The builder image contains these defaults in `/builder/defaults.json`:

```json
{
  "steps": ["build", "lint", "test"],
  "build": {
    "steps": [
      "npm ci",
      "copy-assets ${CONFIG:-project.json}",
      "cd ${HUGO_SOURCE:-.} && hugo --minify --environment ${ENVIRONMENT:-production} ${BASEURL:+--baseURL $BASEURL}"
    ]
  },
  "lint": {
    "steps": [
      "htmlhint ${HUGO_SOURCE:-.}/${OUTPUT:-public}/**/*.html || true"
    ]
  },
  "test": {
    "steps": []
  },
  "output": "public"
}
```

Projects override defaults by defining the same keys in `project.json`. Only the keys you specify are overridden.
