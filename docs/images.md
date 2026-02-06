# Container Images

The pipeline provides three container images, all versioned together with semantic versioning.

## Available Images

| Image | Description |
|-------|-------------|
| `ghcr.io/slauger/hugo-gitops-pipeline/builder` | Build environment with Node.js and Hugo |
| `ghcr.io/slauger/hugo-gitops-pipeline/runtime` | Hardened nginx for serving static sites |
| `ghcr.io/slauger/hugo-gitops-pipeline/cleanup` | Registry cleanup tool |

## Builder Image

Contains everything needed to build a Hugo site:

- Node.js 20 (Alpine)
- Hugo extended
- `copy-assets` CLI tool

Used internally by the pipeline. You don't need to reference it directly.

## Runtime Image

A hardened nginx image for serving static Hugo sites:

- nginx (Alpine)
- Security headers pre-configured
- Gzip compression enabled
- Cache headers for static assets
- Health check endpoint

### Security Features

- Runs as non-root user
- Read-only root filesystem compatible
- Security headers:
  - `X-Frame-Options: SAMEORIGIN`
  - `X-Content-Type-Options: nosniff`
  - `X-XSS-Protection: 1; mode=block`
  - `Referrer-Policy: strict-origin-when-cross-origin`

## Cleanup Image

See [Registry Cleanup](cleanup.md) for details.

## Versioning

All images share the same version, managed by semantic-release:

```
ghcr.io/slauger/hugo-gitops-pipeline/builder:v1.0.0
ghcr.io/slauger/hugo-gitops-pipeline/runtime:v1.0.0
ghcr.io/slauger/hugo-gitops-pipeline/cleanup:v1.0.0
```

The `latest` tag always points to the most recent release.

## Release Assets

Each release includes an `images.json` asset with pinned image references including digests:

```json
{
  "version": "1.0.0",
  "builder": {
    "image": "ghcr.io/slauger/hugo-gitops-pipeline/builder",
    "tag": "v1.0.0",
    "digest": "sha256:abc123...",
    "full": "ghcr.io/slauger/hugo-gitops-pipeline/builder:v1.0.0@sha256:abc123..."
  },
  "runtime": { ... },
  "cleanup": { ... }
}
```

The pipeline automatically resolves image references from the latest release, ensuring reproducible builds with pinned digests.

## Updates

Images are automatically updated via Renovate when:

- New Hugo version is released
- New nginx version is released
- Dependencies are updated

A new semantic release is created, which triggers the image build and uploads the `images.json` asset.
