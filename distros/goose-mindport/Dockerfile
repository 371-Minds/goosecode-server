# goose-mindport — Goose distribution for Mindport (Portless) local routing
#
# Extends the base goosecode-server image with:
#   - Node.js LTS
#   - The Portless CLI, exposed as both `portless` and `mindport`
#   - A shallow checkout of vercel-labs/portless under /opt/integrations/mindport
#   - Goose system prompt tuned for Windows-friendly local routing and DNS workflows
#
# Build:
#   docker build -t goose-mindport -f distros/goose-mindport/Dockerfile .

ARG BASE_IMAGE=goosecode-server:latest
FROM ${BASE_IMAGE}

USER root

RUN apt-get update && apt-get install -y --no-install-recommends \
        ca-certificates \
        gnupg \
    && rm -rf /var/lib/apt/lists/*

ARG NODE_MAJOR=20
RUN curl -fsSL https://deb.nodesource.com/setup_${NODE_MAJOR}.x | bash - && \
    apt-get install -y nodejs npm && \
    rm -rf /var/lib/apt/lists/*

RUN npm install -g portless@latest && \
    mkdir -p /opt/integrations && \
    git clone --depth 1 https://github.com/vercel-labs/portless.git /opt/integrations/mindport

RUN cat <<'EOF' > /usr/local/bin/mindport
#!/bin/sh
exec portless "$@"
EOF

RUN chmod +x /usr/local/bin/mindport

COPY --chown=coder:coder distros/goose-mindport/system-prompt.md /opt/goose/system-prompt.md

RUN mkdir -p /home/coder/.config/goose && \
    printf 'GOOSE_PROVIDER: openai\nGOOSE_MODEL: o3-mini-2025-01-31\nsystem_prompt_file: /opt/goose/system-prompt.md\n' \
      > /home/coder/.config/goose/config.yaml && \
    chown -R coder:coder /home/coder/.config/goose

USER coder

ENV MINDPORT_HOME=/opt/integrations/mindport \
    PORTLESS_TLD=localhost \
    PORTLESS_PORT=1355 \
    MINDPORT_TLD=localhost \
    MINDPORT_PROXY_PORT=1355

EXPOSE 8080 8000 1355
