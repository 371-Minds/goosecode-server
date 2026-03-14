# goose-ryzenai — Goose distribution for RyzenAI-SW workflows
#
# Extends the base goosecode-server image with:
#   - A shallow checkout of k-rks/RyzenAI-SW
#   - Goose system prompt tuned for AMD Ryzen AI tasks
#
# Build:
#   docker build -t goose-ryzenai -f distros/goose-ryzenai/Dockerfile .

ARG BASE_IMAGE=goosecode-server:latest
FROM ${BASE_IMAGE}

USER root

RUN mkdir -p /opt/integrations && \
    git clone --depth 1 https://github.com/k-rks/RyzenAI-SW.git /opt/integrations/ryzenai

COPY --chown=coder:coder distros/goose-ryzenai/system-prompt.md /opt/goose/system-prompt.md

RUN mkdir -p /home/coder/.config/goose && \
    printf 'GOOSE_PROVIDER: openai\nGOOSE_MODEL: o3-mini-2025-01-31\nsystem_prompt_file: /opt/goose/system-prompt.md\n' \
      > /home/coder/.config/goose/config.yaml && \
    chown -R coder:coder /home/coder/.config/goose

USER coder

ENV RYZENAI_HOME=/opt/integrations/ryzenai

EXPOSE 8080 8000
