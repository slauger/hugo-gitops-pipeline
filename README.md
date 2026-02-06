# Hugo GitOps Pipeline

A complete, reusable CI/CD pipeline for Hugo sites with GitOps deployment.

## Features

- **Zero-config builds**: Just add a `project.json` and go
- **Multi-environment support**: dev, staging, qa, production - as many as you need
- **GitOps deployment**: Automatic image updates via [gitops-image-replacer](https://github.com/slauger/gitops-image-replacer)
- **Automated updates**: Renovate keeps Hugo, nginx, and all dependencies current
- **Semantic versioning**: Automatic releases with semantic-release

## Quick Start

### 1. Create `project.json` in your Hugo project

```json
{
  "$schema": "https://raw.githubusercontent.com/slauger/hugo-gitops-pipeline/main/schemas/project.schema.json",

  "hugo": {
    "source": "hugo"
  },

  "copy": [
    {"from": "node_modules/photoswipe/dist/", "to": "hugo/static/libs/photoswipe/"}
  ],

  "environments": {
    "staging": {
      "when": "^refs/heads/develop$",
      "baseurl": "https://staging.example.com",
      "gitops": {
        "repository": "myorg/gitops",
        "branch": "main",
        "file": "apps/mysite/values-staging.yaml"
      }
    },
    "production": {
      "when": "^refs/heads/main$",
      "baseurl": "https://www.example.com",
      "gitops": {
        "repository": "myorg/gitops",
        "branch": "main",
        "file": "apps/mysite/values-prod.yaml"
      }
    }
  }
}
```

### 2. Create `.github/workflows/ci-cd.yml`

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  pipeline:
    uses: slauger/hugo-gitops-pipeline/.github/workflows/hugo-gitops.yml@v1
    with:
      registry: registry.example.com
      image_name: my-hugo-site
    secrets: inherit
```

### 3. Configure secrets

Add these secrets to your repository:

| Secret | Description |
|--------|-------------|
| `REGISTRY_USERNAME` | Container registry username |
| `REGISTRY_PASSWORD` | Container registry password |
| `GITOPS_APP_ID` | GitHub App ID for GitOps repo access |
| `GITOPS_APP_PRIVATE_KEY` | GitHub App private key |

### Passing secrets to the workflow

GitHub Actions provides two ways to pass secrets to reusable workflows:

**Option A: `secrets: inherit` (simple)**

Passes all secrets from the calling repository to the reusable workflow automatically.

```yaml
jobs:
  pipeline:
    uses: slauger/hugo-gitops-pipeline/.github/workflows/hugo-gitops.yml@v1
    with:
      registry: registry.example.com
      image_name: my-hugo-site
    secrets: inherit
```

**Option B: Explicit secrets (more control)**

Explicitly pass only the required secrets. More verbose but clearer about what is shared.

```yaml
jobs:
  pipeline:
    uses: slauger/hugo-gitops-pipeline/.github/workflows/hugo-gitops.yml@v1
    with:
      registry: registry.example.com
      image_name: my-hugo-site
    secrets:
      REGISTRY_USERNAME: ${{ secrets.REGISTRY_USERNAME }}
      REGISTRY_PASSWORD: ${{ secrets.REGISTRY_PASSWORD }}
      GITOPS_APP_ID: ${{ secrets.GITOPS_APP_ID }}
      GITOPS_APP_PRIVATE_KEY: ${{ secrets.GITOPS_APP_PRIVATE_KEY }}
```

See [GitHub Docs: Using secrets in a reusable workflow](https://docs.github.com/en/actions/sharing-automations/reusing-workflows#using-inputs-and-secrets-in-a-reusable-workflow) for more details.

## Configuration

### project.json

| Section | Description |
|---------|-------------|
| `hugo` | Hugo source directory and config file |
| `copy` | Assets to copy before build (npm packages, images, etc.) |
| `lint` | Enable/disable linting tools |
| `nginx` | Custom nginx config snippets directory |
| `environments` | Branch → environment mapping with GitOps config |

### Environments

Each environment supports:

| Field | Description |
|-------|-------------|
| `when` | Regex pattern to match `GITHUB_REF` |
| `baseurl` | Hugo `--baseURL` value |
| `robots` | robots.txt directive |
| `approval_required` | Require manual approval |
| `gitops.repository` | GitOps repo (owner/repo) |
| `gitops.branch` | Target branch |
| `gitops.file` | Values file to update |

### Custom nginx config

Place nginx config snippets in a `nginx/` directory:

```
nginx/
  redirects.conf
  headers.conf
  cache.conf
```

These are automatically included in the runtime image.

## Images

| Image | Description |
|-------|-------------|
| `ghcr.io/slauger/hugo-gitops-pipeline/builder` | Node.js + Hugo extended |
| `ghcr.io/slauger/hugo-gitops-pipeline/runtime` | Hardened nginx |

## License

MIT
