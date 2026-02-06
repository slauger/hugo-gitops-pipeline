# Hugo GitOps Pipeline

A complete, reusable CI/CD pipeline for Hugo sites with GitOps deployment.

## Features

- **Zero-config builds** - Just add a `project.json` and go
- **Multi-environment support** - dev, staging, qa, production - as many as you need
- **GitOps deployment** - Automatic image updates via [gitops-image-replacer](https://github.com/slauger/gitops-image-replacer)
- **Automated updates** - Renovate keeps Hugo, nginx, and all dependencies current
- **Semantic versioning** - Automatic releases with semantic-release
- **GDPR-compliant** - Self-hosted in Germany, no external CDNs

## How it works

```mermaid
flowchart LR
    subgraph Branches
        feature["feature/*"]
        develop["develop"]
        main["main"]
    end

    subgraph Environments
        dev["Dev"]
        staging["Staging"]
        prod["Production"]
    end

    subgraph Images
        devImg["mysite:dev-abc123"]
        stagingImg["mysite:staging-def456"]
        prodImg["mysite:prod-ghi789"]
    end

    feature -->|"PR"| develop
    develop -->|"merge"| main

    feature -.->|"optional"| dev
    develop -->|"auto deploy"| staging
    main -->|"auto deploy"| prod

    dev --- devImg
    staging --- stagingImg
    prod --- prodImg

    classDef devStyle fill:#1e88e5,stroke:#1565c0,color:#fff
    classDef stagingStyle fill:#fb8c00,stroke:#ef6c00,color:#fff
    classDef prodStyle fill:#43a047,stroke:#2e7d32,color:#fff

    class dev devStyle
    class staging stagingStyle
    class prod prodStyle
```

## Quick Start

1. Create a [`project.json`](configuration.md) in your Hugo repository
2. Add the [workflow](getting-started.md#create-workflow) to `.github/workflows/`
3. Configure [secrets](getting-started.md#configure-secrets) and push

That's it! Your Hugo site will automatically build and deploy on every push.

## Next Steps

- [Getting Started](getting-started.md) - Step-by-step setup guide
- [Configuration](configuration.md) - All `project.json` options
- [Architecture](architecture.md) - Reference architecture for self-hosted setup
