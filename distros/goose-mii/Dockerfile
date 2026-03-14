# goose-mii — Goose distribution for DeepSpeed-MII workflows
#
# Extends the base goosecode-server image with:
#   - A shallow checkout of deepspeedai/DeepSpeed-MII
#   - Goose system prompt tuned for distributed inference tasks
#
# Build:
#   docker build -t goose-mii -f distros/goose-mii/Dockerfile .

ARG BASE_IMAGE=goosecode-server:latest
FROM ${BASE_IMAGE}

USER root

RUN mkdir -p /opt/integrations && \
    git clone --depth 1 https://github.com/deepspeedai/DeepSpeed-MII.git /opt/integrations/mii

COPY --chown=coder:coder distros/goose-mii/system-prompt.md /opt/goose/system-prompt.md

RUN mkdir -p /home/coder/.config/goose && \
    printf 'GOOSE_PROVIDER: openai\nGOOSE_MODEL: o3-mini-2025-01-31\nsystem_prompt_file: /opt/goose/system-prompt.md\n' \
      > /home/coder/.config/goose/config.yaml && \
    chown -R coder:coder /home/coder/.config/goose

USER coder

ENV MII_HOME=/opt/integrations/mii

EXPOSE 8080 8000
