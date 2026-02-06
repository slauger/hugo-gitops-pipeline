# Environments

The pipeline supports multiple environments with automatic deployment based on branch patterns.

## How it works

Each environment in `project.json` has a `when` pattern that matches against `GITHUB_REF`:

```json
{
  "environments": {
    "staging": {
      "when": "^refs/heads/develop$",
      "environment": "staging"
    },
    "production": {
      "when": "^refs/heads/main$",
      "environment": "production"
    }
  }
}
```

When you push to `develop`, the pipeline:

1. Matches `refs/heads/develop` against environment patterns
2. Finds `staging` environment
3. Builds with `hugo --environment staging`
4. Tags image as `staging-<sha>`
5. Updates `values-staging.yaml` in GitOps repo

## Environment Options

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `when` | string | Yes | Regex pattern to match `GITHUB_REF` |
| `environment` | string | No | Hugo `--environment` value (defaults to environment key) |
| `baseurl` | string | No | Hugo `--baseURL` override (if not set, uses Hugo config) |
| `gitops.repository` | string | Yes | GitOps repo (owner/repo) |
| `gitops.branch` | string | No | Target branch (default: `main`) |
| `gitops.file` | string | Yes | Values file to update |

## Hugo Configuration Directory

Hugo natively supports [Configuration Directory](https://gohugo.io/getting-started/configuration/#configuration-directory) for environment-specific settings:

```
config/
  _default/           # Base config (always loaded)
    hugo.toml
    params.toml
  development/        # For dev/feature branches
    params.toml
  staging/
    params.toml
  production/
    params.toml
```

Example `config/staging/params.toml`:
```toml
showEnvironmentBanner = true
environmentBannerText = "Staging Environment"
environmentBannerColor = "#ff9800"
```

Example `config/production/params.toml`:
```toml
showEnvironmentBanner = false
```

In your Hugo template:
```html
{{ if site.Params.showEnvironmentBanner }}
<div class="env-banner" style="background: {{ site.Params.environmentBannerColor }}">
  {{ site.Params.environmentBannerText }}
</div>
{{ end }}
```

## Common Patterns

### Two Environments (Simple)

```json
{
  "environments": {
    "staging": {
      "when": "^refs/heads/develop$",
      "environment": "staging",
      "gitops": {
        "repository": "myorg/gitops",
        "file": "apps/mysite/values-staging.yaml"
      }
    },
    "production": {
      "when": "^refs/heads/main$",
      "environment": "production",
      "gitops": {
        "repository": "myorg/gitops",
        "file": "apps/mysite/values-prod.yaml"
      }
    }
  }
}
```

### Feature Branch Deployments

Deploy every feature branch to a dev environment. All feature branches share the same Hugo `development` environment config:

```json
{
  "environments": {
    "dev": {
      "when": "^refs/heads/feature/.*$",
      "environment": "development",
      "baseurl": "https://dev.example.com",
      "gitops": {
        "repository": "myorg/gitops",
        "file": "apps/mysite/values-dev.yaml"
      }
    }
  }
}
```

### Dynamic URLs per Branch

Use `${BRANCH_SLUG}` for unique URLs per feature branch:

```json
{
  "environments": {
    "feature": {
      "when": "^refs/heads/feature/.*$",
      "environment": "development",
      "baseurl": "https://${BRANCH_SLUG}.dev.example.com",
      "gitops": {
        "repository": "myorg/gitops",
        "file": "apps/mysite/values-${BRANCH_SLUG}.yaml"
      }
    }
  }
}
```

Example: `feature/login-page` becomes:
- **URL:** `https://login-page.dev.example.com`
- **Values file:** `values-login-page.yaml`

This requires wildcard DNS (`*.dev.example.com`) and dynamic ArgoCD Applications.

### Full Pipeline

```json
{
  "environments": {
    "dev": {
      "when": "^refs/heads/feature/.*$",
      "environment": "development",
      "baseurl": "https://dev.example.com",
      "gitops": {
        "repository": "myorg/gitops",
        "file": "apps/mysite/values-dev.yaml"
      }
    },
    "staging": {
      "when": "^refs/heads/develop$",
      "environment": "staging",
      "gitops": {
        "repository": "myorg/gitops",
        "file": "apps/mysite/values-staging.yaml"
      }
    },
    "qa": {
      "when": "^refs/heads/release/.*$",
      "environment": "staging",
      "gitops": {
        "repository": "myorg/gitops",
        "file": "apps/mysite/values-qa.yaml"
      }
    },
    "production": {
      "when": "^refs/heads/main$",
      "environment": "production",
      "gitops": {
        "repository": "myorg/gitops",
        "file": "apps/mysite/values-prod.yaml"
      }
    }
  }
}
```

Note: `qa` uses the `staging` Hugo environment but deploys to a separate GitOps values file.

## Image Tags

Each environment gets its own image tags:

| Environment | Tags |
|-------------|------|
| dev | `dev-abc123f`, `dev-latest` |
| staging | `staging-def456a`, `staging-latest` |
| qa | `qa-ghi789b`, `qa-latest` |
| production | `production-jkl012c`, `production-latest` |

The SHA-based tag ensures traceability back to the exact commit.
