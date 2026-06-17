FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y \
    ca-certificates curl git gnupg lsb-release sudo openssh-client \
    ripgrep jq unzip less vim nano build-essential \
    python3 python3-pip nodejs npm \
    zsh-autosuggestions zsh-syntax-highlighting zsh \
    locales \
  && locale-gen en_US.UTF-8 \
  && rm -rf /var/lib/apt/lists/*

ENV LANG=en_US.UTF-8
ENV LC_ALL=en_US.UTF-8

RUN echo "ubuntu ALL=(root) NOPASSWD:ALL" > /etc/sudoers.d/ubuntu \
  && chmod 0440 /etc/sudoers.d/ubuntu \
  && mkdir -p /home/ubuntu/.ssh && chmod 700 /home/ubuntu/.ssh \
  && printf 'Host *\n  StrictHostKeyChecking accept-new\n' > /home/ubuntu/.ssh/config \
  && chmod 600 /home/ubuntu/.ssh/config \
  && chown -R ubuntu:ubuntu /home/ubuntu/.ssh


# AI CLI tools — pinned to exact versions to prevent silent supply-chain updates
RUN npm install -g \
  @anthropic-ai/claude-code@2.1.177 \
  @openai/codex@0.139.0 \
  opencode-ai@1.17.4

USER ubuntu
ENV PATH="/home/ubuntu/.local/bin:$PATH"

# Install oh-my-zsh for the ubuntu user
RUN sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" "" --unattended

# Wire apt-installed zsh plugins into oh-my-zsh and configure
# oh-my-zsh requires <plugin>/<plugin>.plugin.zsh; apt ships <plugin>.zsh so we link with the right name
RUN mkdir -p /home/ubuntu/.oh-my-zsh/custom/plugins/zsh-autosuggestions \
  && ln -s /usr/share/zsh-autosuggestions/zsh-autosuggestions.zsh \
     /home/ubuntu/.oh-my-zsh/custom/plugins/zsh-autosuggestions/zsh-autosuggestions.plugin.zsh \
  && mkdir -p /home/ubuntu/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting \
  && ln -s /usr/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh \
     /home/ubuntu/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.plugin.zsh \
  && sed -i 's/^plugins=.*/plugins=(git zsh-autosuggestions zsh-syntax-highlighting)/' /home/ubuntu/.zshrc \
  && sed -i 's/^ZSH_THEME=.*/ZSH_THEME="robbyrussell"/' /home/ubuntu/.zshrc

# rtk pinned to a specific release tag
RUN curl -fsSL https://raw.githubusercontent.com/rtk-ai/rtk/v0.42.4/install.sh | sh

# Wire rtk into Claude Code for ubuntu
RUN mkdir -p /home/ubuntu/.claude \
  && rtk init -g --auto-patch \
  && rtk --version \
  && rtk gain

WORKDIR /workspace
CMD ["zsh"]