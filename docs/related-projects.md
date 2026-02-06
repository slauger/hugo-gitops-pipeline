# Related Projects

These projects work together to create a complete, self-hosted Hugo platform.

## Core Components

| Project | Description |
|---------|-------------|
| [hugo-gitops-pipeline](https://github.com/slauger/hugo-gitops-pipeline) | This project - CI/CD pipeline for Hugo sites |
| [gitops-image-replacer](https://github.com/slauger/gitops-image-replacer) | Updates image references in GitOps repositories |

## Infrastructure

| Project | Description |
|---------|-------------|
| [flatcar-hcloud](https://github.com/slauger/flatcar-hcloud) | Flatcar Linux + K3S on Hetzner Cloud |

## Helm Charts

Available at [slauger/helm-charts](https://github.com/slauger/helm-charts):

| Chart | Description |
|-------|-------------|
| [hugo-nginx](https://github.com/slauger/helm-charts/tree/main/charts/hugo-nginx) | Helm chart for deploying Hugo sites |
| [hcloud-registry](https://github.com/slauger/helm-charts/tree/main/charts/hcloud-registry) | Docker Registry with Hetzner S3 backend |

## Usage

```bash
# Add Helm repository
helm repo add slauger https://slauger.github.io/helm-charts

# Install hugo-nginx
helm install my-site slauger/hugo-nginx \
  --set image.repository=registry.example.com/my-site \
  --set image.tag=prod-latest

# Install registry
helm install registry slauger/hcloud-registry \
  --set s3.accessKey=xxx \
  --set s3.secretKey=xxx
```
