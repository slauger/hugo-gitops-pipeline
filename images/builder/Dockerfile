# Hugo Builder Image
# Contains Node.js, Hugo extended, and copy-assets CLI

# renovate: datasource=docker depName=node
FROM node:24-alpine@sha256:d32cdf619f63fe0471182d08996dd516c6275bb5fd31ae06e55a570bd9e1ad43

ARG HUGO_VERSION=0.163.2

# Install dependencies
RUN apk add --no-cache \
    git \
    libc6-compat \
    libstdc++ \
    curl \
    bash \
    jq \
    tzdata

# Install Hugo extended (architecture-aware)
ARG TARGETARCH
RUN HUGO_ARCH=$([ "$TARGETARCH" = "arm64" ] && echo "arm64" || echo "amd64") && \
    curl -L "https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-${HUGO_ARCH}.tar.gz" \
    | tar -xz -C /usr/local/bin && \
    hugo version

# Install linting tools
COPY package.json package-lock.json /builder/
RUN cd /builder && npm ci && npm link

# Install pipeline defaults and scripts
COPY defaults.json .htmlhintrc /builder/
COPY bin/copy-assets /usr/local/bin/copy-assets
COPY bin/run-phase /usr/local/bin/run-phase
RUN chmod +x /usr/local/bin/copy-assets /usr/local/bin/run-phase

WORKDIR /build

ENTRYPOINT ["/bin/sh", "-c"]
