# Environments

The pipeline supports multiple environments with automatic deployment based on branch patterns.

## How it works

Each environment in `project.json` has a `when` pattern that matches against `GITHUB_REF`:

```json
{
  "environments": {
    "staging": {
      "when": "^refs/heads/develop$",
      "baseurl": "https://staging.example.com"
    },
    "production": {
      "when": "^refs/heads/main$",
      "baseurl": "https://www.example.com"
    }
  }
}
```

When you push to `develop`, the pipeline:

1. Matches `refs/heads/develop` against environment patterns
2. Finds `staging` environment
3. Builds with `baseurl: https://staging.example.com`
4. Tags image as `staging-<sha>`
5. Updates `values-staging.yaml` in GitOps repo

## Environment Options

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `when` | string | Yes | Regex pattern to match `GITHUB_REF` |
| `baseurl` | string | Yes | Hugo `--baseURL` value |
| `robots` | string | No | robots.txt directive |
| `approval_required` | boolean | No | Require manual approval |
| `gitops.repository` | string | Yes | GitOps repo (owner/repo) |
| `gitops.branch` | string | No | Target branch (default: `main`) |
| `gitops.file` | string | Yes | Values file to update |

## Common Patterns

### Two Environments (Simple)

```json
{
  "environments": {
    "staging": {
      "when": "^refs/heads/develop$",
      "baseurl": "https://staging.example.com",
      "gitops": {
        "repository": "myorg/gitops",
        "file": "apps/mysite/values-staging.yaml"
      }
    },
    "production": {
      "when": "^refs/heads/main$",
      "baseurl": "https://www.example.com",
      "gitops": {
        "repository": "myorg/gitops",
        "file": "apps/mysite/values-prod.yaml"
      }
    }
  }
}
```

### Feature Branch Deployments

Deploy every feature branch to a dev environment:

```json
{
  "environments": {
    "dev": {
      "when": "^refs/heads/feature/.*$",
      "baseurl": "https://dev.example.com",
      "robots": "noindex, nofollow",
      "gitops": {
        "repository": "myorg/gitops",
        "file": "apps/mysite/values-dev.yaml"
      }
    }
  }
}
```

### Release Branch to QA

```json
{
  "environments": {
    "qa": {
      "when": "^refs/heads/release/.*$",
      "baseurl": "https://qa.example.com",
      "gitops": {
        "repository": "myorg/gitops",
        "file": "apps/mysite/values-qa.yaml"
      }
    }
  }
}
```

### Full Pipeline

```json
{
  "environments": {
    "dev": {
      "when": "^refs/heads/feature/.*$",
      "baseurl": "https://dev.example.com",
      "robots": "noindex, nofollow",
      "gitops": {
        "repository": "myorg/gitops",
        "file": "apps/mysite/values-dev.yaml"
      }
    },
    "staging": {
      "when": "^refs/heads/develop$",
      "baseurl": "https://staging.example.com",
      "robots": "noindex, nofollow",
      "gitops": {
        "repository": "myorg/gitops",
        "file": "apps/mysite/values-staging.yaml"
      }
    },
    "qa": {
      "when": "^refs/heads/release/.*$",
      "baseurl": "https://qa.example.com",
      "robots": "noindex, nofollow",
      "gitops": {
        "repository": "myorg/gitops",
        "file": "apps/mysite/values-qa.yaml"
      }
    },
    "production": {
      "when": "^refs/heads/main$",
      "baseurl": "https://www.example.com",
      "robots": "index, follow",
      "approval_required": true,
      "gitops": {
        "repository": "myorg/gitops",
        "file": "apps/mysite/values-prod.yaml"
      }
    }
  }
}
```

## Image Tags

Each environment gets its own image tags:

| Environment | Tags |
|-------------|------|
| dev | `dev-abc123f`, `dev-latest` |
| staging | `staging-def456a`, `staging-latest` |
| qa | `qa-ghi789b`, `qa-latest` |
| production | `prod-jkl012c`, `prod-latest` |

The SHA-based tag ensures traceability back to the exact commit.
