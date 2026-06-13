FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y \
    ca-certificates curl git gnupg lsb-release sudo zsh openssh-client \
    ripgrep jq unzip less vim nano build-essential \
    python3 python3-pip nodejs npm \
  && rm -rf /var/lib/apt/lists/*

# Non-root user — reduces blast radius if a package or AI tool misbehaves
RUN groupadd --gid 1010 devuser \
  && useradd --uid 1010 --gid 1010 -m -s /bin/zsh devuser \
  && echo "devuser ALL=(root) NOPASSWD:ALL" > /etc/sudoers.d/devuser \
  && chmod 0440 /etc/sudoers.d/devuser

# AI CLI tools — pinned to exact versions to prevent silent supply-chain updates
RUN npm install -g \
  @anthropic-ai/claude-code@2.1.177 \
  @openai/codex@0.139.0 \
  opencode-ai@1.17.4

USER devuser
ENV PATH="/home/devuser/.local/bin:$PATH"

# rtk pinned to a specific release tag
RUN curl -fsSL https://raw.githubusercontent.com/rtk-ai/rtk/v0.42.4/install.sh | sh

# Wire rtk into Claude Code for devuser
RUN mkdir -p /home/devuser/.claude \
  && rtk init -g --auto-patch \
  && rtk --version \
  && rtk gain

WORKDIR /workspace
CMD ["zsh"]