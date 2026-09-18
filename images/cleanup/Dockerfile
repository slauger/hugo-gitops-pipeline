# Registry Cleanup Image
# Contains regctl for registry operations and cleanup script

# renovate: datasource=docker depName=ghcr.io/regclient/regctl
FROM ghcr.io/regclient/regctl:latest@sha256:db2e5f3a5de80b24d53fea5af9edf7021a53f2689f1e166ef2f0e200dad4fb3a AS regctl

# renovate: datasource=docker depName=alpine
FROM alpine:3.24@sha256:5b02b42e375f7426f8d65c3af331ca05d9878f9989230354504e0b9dfd431f60

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
