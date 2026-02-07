# Configuration

The `project.json` file in your repository root configures the build pipeline.

## Schema

Use the JSON schema for validation and autocompletion in your editor:

```json
{
  "$schema": "https://raw.githubusercontent.com/slauger/hugo-gitops-pipeline/main/schemas/project.schema.json"
}
```

## Full Example

```json
{
  "$schema": "https://raw.githubusercontent.com/slauger/hugo-gitops-pipeline/main/schemas/project.schema.json",

  "output": "public",

  "hugo": {
    "source": "hugo",
    "config": "hugo.toml"
  },

  "copy": [
    {"from": "node_modules/photoswipe/dist/photoswipe.esm.js", "to": "hugo/static/libs/photoswipe/"},
    {"from": "node_modules/photoswipe/dist/photoswipe.css", "to": "hugo/static/libs/photoswipe/"},
    {"from": "assets/brand/", "to": "hugo/static/images/brand/"}
  ],

  "nginx": {
    "config_dir": "nginx/"
  },

  "environments": {
    "feature": {
      "when": "^refs/heads/feature/.*$",
      "environment": "development",
      "baseurl": "https://${BRANCH_SLUG}.dev.example.com",
      "gitops": {
        "repository": "myorg/gitops",
        "branch": "main",
        "file": "apps/mysite/values-${BRANCH_SLUG}.yaml"
      }
    },
    "staging": {
      "when": "^refs/heads/develop$",
      "environment": "staging",
      "gitops": {
        "repository": "myorg/gitops",
        "branch": "main",
        "file": "apps/mysite/values-staging.yaml"
      }
    },
    "production": {
      "when": "^refs/heads/main$",
      "environment": "production",
      "gitops": {
        "repository": "myorg/gitops",
        "branch": "main",
        "file": "apps/mysite/values-prod.yaml"
      }
    }
  }
}
```

## Sections

### steps

Build phases to run. Defaults to `["build", "lint", "test"]`.

```json
{
  "steps": ["build", "lint", "test"]
}
```

See [Build Steps](build-steps.md) for customizing build phases.

### output

Build output directory. Defaults to `public`.

```json
{
  "output": "public"
}
```

### hugo

Hugo-specific configuration.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `source` | string | `hugo` | Directory containing Hugo site |
| `config` | string | `hugo.toml` | Hugo config file name |

### copy

Assets to copy before build. Useful for npm packages or external assets.

```json
{
  "copy": [
    {"from": "node_modules/lib/dist/file.js", "to": "hugo/static/libs/lib/"},
    {"from": "assets/images/", "to": "hugo/static/images/"}
  ]
}
```

| Field | Type | Description |
|-------|------|-------------|
| `from` | string | Source path (file or directory) |
| `to` | string | Destination directory |

### lint (deprecated)

> **Note:** The `lint.htmlhint`, `lint.links`, and `lint.a11y` options are deprecated. Use [Build Steps](build-steps.md) to customize linting.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `steps` | array | (see defaults) | Custom lint commands |
| `htmlhint` | boolean | `true` | HTML validation (deprecated) |
| `links` | boolean | `false` | Broken link checking (deprecated) |
| `a11y` | boolean | `false` | Accessibility checks (deprecated) |

### nginx

Custom nginx configuration.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `config_dir` | string | `nginx/` | Directory with nginx config snippets |

Place your nginx configs in this directory:

```
nginx/
  redirects.conf
  headers.conf
  cache.conf
```

These are automatically included in the runtime image.

### environments

See [Environments](environments.md) for detailed documentation.
