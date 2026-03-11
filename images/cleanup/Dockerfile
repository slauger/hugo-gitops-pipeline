# Registry Cleanup Image
# Contains regctl for registry operations and cleanup script

# renovate: datasource=docker depName=ghcr.io/regclient/regctl
FROM ghcr.io/regclient/regctl:latest@sha256:c9bbb961a475964c5c4541eddc48cccde0b8d9f941d234b76b3b5d94153538b2 AS regctl

# renovate: datasource=docker depName=alpine
FROM alpine:3.23@sha256:25109184c71bdad752c8312a8623239686a9a2071e8825f20acb8f2198c3f659

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
