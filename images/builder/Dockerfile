# Hugo Builder Image
# Contains Node.js, Hugo extended, and copy-assets CLI

# renovate: datasource=docker depName=node
FROM node:20-alpine@sha256:09e2b3d9726018aecf269bd35325f46bf75046a643a66d28360ec71132750ec8

ARG HUGO_VERSION=0.155.2

# Install dependencies and Hugo extended
RUN apk add --no-cache \
    git \
    libc6-compat \
    libstdc++ \
    curl \
    bash \
    jq \
    && curl -L "https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.tar.gz" \
    | tar -xz -C /usr/local/bin \
    && hugo version

# Install pipeline defaults and scripts
COPY defaults.json /builder/defaults.json
COPY bin/copy-assets /usr/local/bin/copy-assets
COPY bin/run-phase /usr/local/bin/run-phase
RUN chmod +x /usr/local/bin/copy-assets /usr/local/bin/run-phase

WORKDIR /build

ENTRYPOINT ["/bin/sh", "-c"]
