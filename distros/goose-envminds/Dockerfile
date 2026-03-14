# goose-envminds — Goose distribution for 371-Minds/envminds workflows
#
# Extends the base goosecode-server image with:
#   - A shallow checkout of 371-Minds/envminds
#   - Goose system prompt tuned for environment orchestration tasks
#
# Build:
#   docker build -t goose-envminds -f distros/goose-envminds/Dockerfile .

ARG BASE_IMAGE=goosecode-server:latest
FROM ${BASE_IMAGE}

USER root

RUN mkdir -p /opt/integrations && \
    git clone --depth 1 https://github.com/371-Minds/envminds.git /opt/integrations/envminds

COPY --chown=coder:coder distros/goose-envminds/system-prompt.md /opt/goose/system-prompt.md

RUN mkdir -p /home/coder/.config/goose && \
    printf 'GOOSE_PROVIDER: openai\nGOOSE_MODEL: o3-mini-2025-01-31\nsystem_prompt_file: /opt/goose/system-prompt.md\n' \
      > /home/coder/.config/goose/config.yaml && \
    chown -R coder:coder /home/coder/.config/goose

USER coder

ENV ENVMINDS_HOME=/opt/integrations/envminds

EXPOSE 8080 8000
