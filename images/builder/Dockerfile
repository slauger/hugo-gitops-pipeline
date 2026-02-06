# Hugo Builder Image
# Contains Node.js, Hugo extended, and copy-assets CLI

# renovate: datasource=docker depName=node
FROM node:24-alpine@sha256:cd6fb7efa6490f039f3471a189214d5f548c11df1ff9e5b181aa49e22c14383e

ARG HUGO_VERSION=0.155.2

# Install dependencies and Hugo extended
RUN apk add --no-cache \
    git \
    libc6-compat \
    libstdc++ \
    curl \
    bash \
    && curl -L "https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.tar.gz" \
    | tar -xz -C /usr/local/bin \
    && hugo version

# Install copy-assets CLI
COPY scripts/copy-assets /usr/local/bin/copy-assets
RUN chmod +x /usr/local/bin/copy-assets

WORKDIR /build

ENTRYPOINT ["/bin/sh", "-c"]
