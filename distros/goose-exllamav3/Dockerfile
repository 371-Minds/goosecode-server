# goose-exllamav3 — Goose distribution for exllamav3 workflows
#
# Extends the base goosecode-server image with:
#   - A shallow checkout of turboderp-org/exllamav3
#   - Goose system prompt tuned for local LLM inference tasks
#
# Build:
#   docker build -t goose-exllamav3 -f distros/goose-exllamav3/Dockerfile .

ARG BASE_IMAGE=goosecode-server:latest
FROM ${BASE_IMAGE}

USER root

RUN mkdir -p /opt/integrations && \
    git clone --depth 1 https://github.com/turboderp-org/exllamav3.git /opt/integrations/exllamav3

COPY --chown=coder:coder distros/goose-exllamav3/system-prompt.md /opt/goose/system-prompt.md

RUN mkdir -p /home/coder/.config/goose && \
    printf 'GOOSE_PROVIDER: openai\nGOOSE_MODEL: o3-mini-2025-01-31\nsystem_prompt_file: /opt/goose/system-prompt.md\n' \
      > /home/coder/.config/goose/config.yaml && \
    chown -R coder:coder /home/coder/.config/goose

USER coder

ENV EXLLAMAV3_HOME=/opt/integrations/exllamav3

EXPOSE 8080 8000
