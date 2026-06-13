FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y \
    ca-certificates curl git gnupg lsb-release sudo zsh openssh-client \
    ripgrep jq unzip less vim nano build-essential \
    python3 python3-pip nodejs npm \
  && rm -rf /var/lib/apt/lists/*

RUN install -m 0755 -d /etc/apt/keyrings \
  && curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
    -o /etc/apt/keyrings/docker.asc \
  && chmod a+r /etc/apt/keyrings/docker.asc \
  && echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu noble stable" \
    > /etc/apt/sources.list.d/docker.list \
  && apt-get update \
  && apt-get install -y docker-ce-cli docker-compose-plugin \
  && rm -rf /var/lib/apt/lists/*

# USER ${USERNAME}
WORKDIR /workspace

  # AI CLI tools
RUN npm install -g @anthropic-ai/claude-code @openai/codex
# rtk-ai
ENV PATH="/root/.local/bin:$PATH"
RUN curl -fsSL https://raw.githubusercontent.com/rtk-ai/rtk/master/install.sh | sh
# opencode
RUN npm i -g opencode-ai

# Wire rtk into Claude Code (installs the command-rewrite hook and patches
# ~/.claude/settings.json non-interactively). opencode picks up rtk via the
# "openrtk" plugin declared in ~/.config/opencode/opencode.json.
RUN mkdir -p /root/.claude \
  && rtk init -g --auto-patch \
  && rtk --version \
  && rtk gain

CMD ["zsh"]