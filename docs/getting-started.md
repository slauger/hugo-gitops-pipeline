# Getting Started

This guide walks you through setting up the Hugo GitOps Pipeline for your project.

## Prerequisites

- A Hugo site in a GitHub repository
- A container registry (self-hosted or GHCR)
- A GitOps repository for ArgoCD

## Step 1: Create project.json

Create a `project.json` file in your Hugo repository root:

```json
{
  "$schema": "https://raw.githubusercontent.com/slauger/hugo-gitops-pipeline/main/schemas/project.schema.json",

  "hugo": {
    "source": "hugo"
  },

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

See [Configuration](configuration.md) for all available options.

## Step 2: Create Workflow {#create-workflow}

Create `.github/workflows/ci-cd.yml`:

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

## Step 3: Configure Secrets {#configure-secrets}

Add these secrets to your GitHub repository:

| Secret | Description |
|--------|-------------|
| `REGISTRY_USERNAME` | Container registry username |
| `REGISTRY_PASSWORD` | Container registry password |
| `GITOPS_APP_ID` | GitHub App ID for GitOps repo access |
| `GITOPS_APP_PRIVATE_KEY` | GitHub App private key |

### Passing secrets

GitHub Actions provides two ways to pass secrets to reusable workflows:

=== "Simple (inherit)"

    Passes all secrets from the calling repository automatically.

    ```yaml
    jobs:
      pipeline:
        uses: slauger/hugo-gitops-pipeline/.github/workflows/hugo-gitops.yml@v1
        with:
          registry: registry.example.com
          image_name: my-hugo-site
        secrets: inherit
    ```

=== "Explicit (more control)"

    Explicitly pass only the required secrets.

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

See [GitHub Docs](https://docs.github.com/en/actions/sharing-automations/reusing-workflows#using-inputs-and-secrets-in-a-reusable-workflow) for more details.

## Step 4: Push and Deploy

Push your changes to trigger the pipeline:

```bash
git add .
git commit -m "feat: add hugo-gitops-pipeline"
git push
```

The pipeline will:

1. Build your Hugo site
2. Create a Docker image
3. Push to your registry
4. Update your GitOps repository
5. ArgoCD syncs and deploys

## Next Steps

- [Configuration](configuration.md) - Customize your `project.json`
- [Environments](environments.md) - Set up multiple environments
- [Architecture](architecture.md) - Full reference architecture
