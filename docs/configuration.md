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

  "hugo": {
    "source": "hugo",
    "config": "hugo.toml"
  },

  "copy": [
    {"from": "node_modules/photoswipe/dist/photoswipe.esm.js", "to": "hugo/static/libs/photoswipe/"},
    {"from": "node_modules/photoswipe/dist/photoswipe.css", "to": "hugo/static/libs/photoswipe/"},
    {"from": "assets/brand/", "to": "hugo/static/images/brand/"}
  ],

  "lint": {
    "htmlhint": true,
    "links": false,
    "a11y": false
  },

  "nginx": {
    "config_dir": "nginx/"
  },

  "environments": {
    "staging": {
      "when": "^refs/heads/develop$",
      "baseurl": "https://staging.example.com",
      "robots": "noindex, nofollow",
      "gitops": {
        "repository": "myorg/gitops",
        "branch": "main",
        "file": "apps/mysite/values-staging.yaml"
      }
    },
    "production": {
      "when": "^refs/heads/main$",
      "baseurl": "https://www.example.com",
      "robots": "index, follow",
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

### lint

Enable or disable linting tools.

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| `htmlhint` | boolean | `true` | HTML validation |
| `links` | boolean | `false` | Broken link checking |
| `a11y` | boolean | `false` | Accessibility checks |

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
