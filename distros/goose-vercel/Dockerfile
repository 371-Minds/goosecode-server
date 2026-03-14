# goose-vercel — Goose distribution for Vercel Edge + Expo deployments
#
# Extends the base goosecode-server image with:
#   - Node.js LTS + pnpm
#   - Vercel CLI  (deploy Next.js web targets)
#   - Expo CLI + EAS CLI (React Native — iOS/Android)
#   - Goose system prompt tuned for full-stack web/mobile engineering
#
# Build:
#   docker build -t goose-vercel -f distros/goose-vercel/Dockerfile .
#
# Extra environment variables:
#   VERCEL_TOKEN      — Vercel deploy token (required for deploys)
#   EXPO_TOKEN        — Expo EAS token
#   EAS_PROJECT_ID    — Expo project ID
#   NEXT_APP_DIR      — Path to Next.js app inside /workspace (default: web)
#   EXPO_APP_DIR      — Path to Expo app inside /workspace (default: mobile)

ARG BASE_IMAGE=goosecode-server:latest
FROM ${BASE_IMAGE}

USER root

# ---------------------------------------------------------------------------
# System packages
# ---------------------------------------------------------------------------
RUN apt-get update && apt-get install -y --no-install-recommends \
        ca-certificates \
        gnupg \
    && rm -rf /var/lib/apt/lists/*

# ---------------------------------------------------------------------------
# Node.js LTS (via NodeSource)
# ---------------------------------------------------------------------------
ARG NODE_MAJOR=20
RUN curl -fsSL https://deb.nodesource.com/setup_${NODE_MAJOR}.x | bash - && \
    apt-get install -y nodejs && \
    rm -rf /var/lib/apt/lists/*

# ---------------------------------------------------------------------------
# pnpm (global)
# ---------------------------------------------------------------------------
RUN corepack enable && corepack prepare pnpm@latest --activate

# ---------------------------------------------------------------------------
# Vercel CLI
# ---------------------------------------------------------------------------
RUN npm install -g vercel@latest

# ---------------------------------------------------------------------------
# Expo CLI + EAS CLI
# ---------------------------------------------------------------------------
RUN npm install -g expo-cli@latest eas-cli@latest

# ---------------------------------------------------------------------------
# System prompt
# ---------------------------------------------------------------------------
COPY --chown=coder:coder distros/goose-vercel/system-prompt.md /opt/goose/system-prompt.md

# ---------------------------------------------------------------------------
# Goose config override
# ---------------------------------------------------------------------------
RUN mkdir -p /home/coder/.config/goose && \
    printf 'GOOSE_PROVIDER: openai\nGOOSE_MODEL: o3-mini-2025-01-31\nsystem_prompt_file: /opt/goose/system-prompt.md\n' \
      > /home/coder/.config/goose/config.yaml && \
    chown -R coder:coder /home/coder/.config/goose

USER coder

# Defaults — override at runtime
ENV NEXT_APP_DIR=web \
    EXPO_APP_DIR=mobile

EXPOSE 8080 8000 3000
