# Registry Cleanup Image
# Contains regctl for registry operations and cleanup script

# renovate: datasource=docker depName=ghcr.io/regclient/regctl
FROM ghcr.io/regclient/regctl:latest@sha256:45b57933c2b67bb1e280b4baaa220ef1ab212c81268ab19589fa1e83719c7c82 AS regctl

# renovate: datasource=docker depName=alpine
FROM alpine:3.24@sha256:71e9b7bae160ba167fa17414401b94cbcdaf3c32feeab9298bd794a2a0614c35

RUN apk add --no-cache \
    bash \
    jq \
    yq-go \
    curl

# Copy regctl binary from official image
COPY --from=regctl /regctl /usr/local/bin/regctl

# Copy cleanup script
COPY scripts/cleanup /usr/local/bin/cleanup
RUN chmod +x /usr/local/bin/cleanup

# Default config path
ENV CONFIG_FILE=/config/config.yaml
ENV DRY_RUN=true

ENTRYPOINT ["/usr/local/bin/cleanup"]
