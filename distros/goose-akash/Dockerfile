# goose-akash — Goose distribution for Akash Network deployments
#
# Extends the base goosecode-server image with:
#   - Akash CLI (deploy SDL templates directly)
#   - IPFS CLI (decentralized storage)
#   - SQLite3 (local agent state)
#   - Pre-loaded Akash SDL templates
#   - Goose system prompt tuned for Akash engineering tasks
#
# Build:
#   docker build -t goose-akash -f distros/goose-akash/Dockerfile .
#
# Run (inherits run.sh vars plus the extras below):
#   AKASH_KEY_NAME=mykey
#   AKASH_KEYRING_BACKEND=os
#   AKASH_NODE=https://rpc.akashnet.net:443
#   AKASH_CHAIN_ID=akashnet-2

ARG BASE_IMAGE=goosecode-server:latest
FROM ${BASE_IMAGE}

USER root

# ---------------------------------------------------------------------------
# System packages
# ---------------------------------------------------------------------------
RUN apt-get update && apt-get install -y --no-install-recommends \
        sqlite3 \
        jq \
        unzip \
        ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# ---------------------------------------------------------------------------
# Akash CLI
# ---------------------------------------------------------------------------
ARG AKASH_VERSION=0.10.7
RUN set -eux; \
    ARCH=$(uname -m); \
    case "$ARCH" in \
      x86_64)  GOARCH=amd64 ;; \
      aarch64) GOARCH=arm64 ;; \
      *)       echo "Unsupported arch: $ARCH" && exit 1 ;; \
    esac; \
    curl -fsSL "https://github.com/akash-network/provider/releases/download/v${AKASH_VERSION}/provider-services_${AKASH_VERSION}_linux_${GOARCH}.zip" \
      -o /tmp/akash.zip; \
    unzip -o /tmp/akash.zip provider-services -d /usr/local/bin/; \
    mv /usr/local/bin/provider-services /usr/local/bin/akash; \
    chmod +x /usr/local/bin/akash; \
    rm /tmp/akash.zip; \
    akash version || true

# ---------------------------------------------------------------------------
# IPFS CLI (kubo)
# ---------------------------------------------------------------------------
ARG IPFS_VERSION=0.29.0
RUN set -eux; \
    ARCH=$(uname -m); \
    case "$ARCH" in \
      x86_64)  GOARCH=amd64 ;; \
      aarch64) GOARCH=arm64 ;; \
      *)       echo "Unsupported arch: $ARCH" && exit 1 ;; \
    esac; \
    curl -fsSL "https://dist.ipfs.tech/kubo/v${IPFS_VERSION}/kubo_v${IPFS_VERSION}_linux-${GOARCH}.tar.gz" \
      | tar -xz -C /usr/local/bin/ --strip-components=1 kubo/ipfs; \
    chmod +x /usr/local/bin/ipfs; \
    ipfs version || true

# ---------------------------------------------------------------------------
# SDL templates
# ---------------------------------------------------------------------------
COPY --chown=coder:coder akash/sdls /opt/akash/sdls/

# ---------------------------------------------------------------------------
# System prompt
# ---------------------------------------------------------------------------
COPY --chown=coder:coder distros/goose-akash/system-prompt.md /opt/goose/system-prompt.md

# ---------------------------------------------------------------------------
# Goose config override — inject system prompt
# ---------------------------------------------------------------------------
RUN mkdir -p /home/coder/.config/goose && \
    printf 'GOOSE_PROVIDER: openai\nGOOSE_MODEL: o3-mini-2025-01-31\nsystem_prompt_file: /opt/goose/system-prompt.md\n' \
      > /home/coder/.config/goose/config.yaml && \
    chown -R coder:coder /home/coder/.config/goose

USER coder

# Workspace SQLite path used by agents
ENV SQLITE_PATH=/workspace/.db/agent_state.sqlite

# Akash environment defaults (override at runtime)
ENV AKASH_NODE=https://rpc.akashnet.net:443 \
    AKASH_CHAIN_ID=akashnet-2 \
    AKASH_KEYRING_BACKEND=os

EXPOSE 8080 8000
