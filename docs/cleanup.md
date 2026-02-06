# Registry Cleanup

The cleanup image helps manage old container images in your registry using [regctl](https://github.com/regclient/regclient).

## Why Cleanup?

Each deployment creates new image tags (`staging-abc123`, `prod-def456`). Over time, these accumulate and consume storage. The cleanup tool automatically removes old images while keeping recent ones.

## Configuration

Create a `config.yaml` with retention rules:

```yaml
registry: registry.example.com

retention:
  # Default settings for all images
  default:
    keep_latest: 10
    keep_days: 30

  # Per-image rules
  rules:
    - image: my-hugo-site
      keep_latest: 5
      keep_tags:
        - "prod-latest"
        - "staging-latest"

    - image: another-site
      keep_latest: 3
      keep_tags:
        - "prod-latest"
```

### Options

| Field | Description |
|-------|-------------|
| `registry` | Registry URL |
| `retention.default.keep_latest` | Default number of tags to keep |
| `retention.default.keep_days` | Default age limit in days |
| `retention.rules[].image` | Image name |
| `retention.rules[].keep_latest` | Tags to keep for this image |
| `retention.rules[].keep_tags` | Tags to never delete |

## Usage

### Run Manually

```bash
# Dry run (default) - shows what would be deleted
docker run -v $(pwd)/config.yaml:/config/config.yaml \
  -e REGISTRY_USERNAME=user \
  -e REGISTRY_PASSWORD=pass \
  ghcr.io/slauger/hugo-gitops-pipeline/cleanup:latest

# Actually delete
docker run -v $(pwd)/config.yaml:/config/config.yaml \
  -e REGISTRY_USERNAME=user \
  -e REGISTRY_PASSWORD=pass \
  -e DRY_RUN=false \
  ghcr.io/slauger/hugo-gitops-pipeline/cleanup:latest
```

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `CONFIG_FILE` | `/config/config.yaml` | Path to config file |
| `DRY_RUN` | `true` | Set to `false` to actually delete |
| `REGISTRY_USERNAME` | - | Registry username |
| `REGISTRY_PASSWORD` | - | Registry password |

## Kubernetes CronJob

Deploy as a scheduled job via ArgoCD:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: registry-cleanup-config
data:
  config.yaml: |
    registry: registry.example.com
    retention:
      default:
        keep_latest: 10
      rules:
        - image: my-hugo-site
          keep_latest: 5
          keep_tags:
            - "prod-latest"
            - "staging-latest"
---
apiVersion: batch/v1
kind: CronJob
metadata:
  name: registry-cleanup
spec:
  schedule: "0 3 * * 0"  # Every Sunday at 3:00 AM
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
            - name: cleanup
              image: ghcr.io/slauger/hugo-gitops-pipeline/cleanup:latest
              env:
                - name: DRY_RUN
                  value: "false"
              envFrom:
                - secretRef:
                    name: registry-credentials
              volumeMounts:
                - name: config
                  mountPath: /config
          volumes:
            - name: config
              configMap:
                name: registry-cleanup-config
```

See [examples/cleanup-cronjob.yaml](https://github.com/slauger/hugo-gitops-pipeline/blob/main/examples/cleanup-cronjob.yaml) for a complete example.

## Registry Requirements

Your registry must have deletion enabled:

- **Docker Registry**: Set `REGISTRY_STORAGE_DELETE_ENABLED=true`
- **Harbor**: Enabled by default
- **GHCR**: Deletion via API supported

After cleanup, run garbage collection on your registry to reclaim disk space.
