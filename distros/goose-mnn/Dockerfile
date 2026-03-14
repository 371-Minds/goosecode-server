# goose-mnn — Goose distribution for Alibaba MNN workflows
#
# Extends the base goosecode-server image with:
#   - A shallow checkout of alibaba/MNN
#   - Goose system prompt tuned for edge/mobile inference tasks
#
# Build:
#   docker build -t goose-mnn -f distros/goose-mnn/Dockerfile .

ARG BASE_IMAGE=goosecode-server:latest
FROM ${BASE_IMAGE}

USER root

RUN mkdir -p /opt/integrations && \
    git clone --depth 1 https://github.com/alibaba/MNN.git /opt/integrations/mnn

COPY --chown=coder:coder distros/goose-mnn/system-prompt.md /opt/goose/system-prompt.md

RUN mkdir -p /home/coder/.config/goose && \
    printf 'GOOSE_PROVIDER: openai\nGOOSE_MODEL: o3-mini-2025-01-31\nsystem_prompt_file: /opt/goose/system-prompt.md\n' \
      > /home/coder/.config/goose/config.yaml && \
    chown -R coder:coder /home/coder/.config/goose

USER coder

ENV MNN_HOME=/opt/integrations/mnn

EXPOSE 8080 8000
