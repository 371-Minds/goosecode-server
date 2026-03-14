# goose-picollm — Goose distribution for Picovoice picoLLM workflows
#
# Extends the base goosecode-server image with:
#   - A shallow checkout of Picovoice/picollm
#   - Goose system prompt tuned for edge LLM inference tasks
#
# Build:
#   docker build -t goose-picollm -f distros/goose-picollm/Dockerfile .

ARG BASE_IMAGE=goosecode-server:latest
FROM ${BASE_IMAGE}

USER root

RUN mkdir -p /opt/integrations && \
    git clone --depth 1 https://github.com/Picovoice/picollm.git /opt/integrations/picollm

COPY --chown=coder:coder distros/goose-picollm/system-prompt.md /opt/goose/system-prompt.md

RUN mkdir -p /home/coder/.config/goose && \
    printf 'GOOSE_PROVIDER: openai\nGOOSE_MODEL: o3-mini-2025-01-31\nsystem_prompt_file: /opt/goose/system-prompt.md\n' \
      > /home/coder/.config/goose/config.yaml && \
    chown -R coder:coder /home/coder/.config/goose

USER coder

ENV PICOLLM_HOME=/opt/integrations/picollm

EXPOSE 8080 8000
