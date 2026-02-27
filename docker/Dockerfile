FROM node:20

ARG TZ
ENV TZ="$TZ"

ARG CLAUDE_CODE_VERSION=latest

# Install basic development tools and iptables/ipset
RUN apt-get update && apt-get install -y --no-install-recommends \
  less \
  git \
  procps \
  sudo \
  fzf \
  zsh \
  man-db \
  unzip \
  gnupg2 \
  gh \
  iptables \
  ipset \
  iproute2 \
  dnsutils \
  aggregate \
  jq \
  nano \
  vim \
  python3 \
  python3-pip \
  python3-venv \
  python3-full \
  && apt-get clean && rm -rf /var/lib/apt/lists/*

# Remove Python externally-managed-environment restriction
RUN rm -f /usr/lib/python*/EXTERNALLY-MANAGED

# Install Python packages
# Note: asyncio and datetime are built-in modules, no need to install
RUN pip3 install --no-cache-dir mcp requests

# Ensure default node user has access to /usr/local/share
RUN mkdir -p /usr/local/share/npm-global && \
  chown -R node:node /usr/local/share

ARG USERNAME=node

# Persist bash history.
RUN SNIPPET="export PROMPT_COMMAND='history -a' && export HISTFILE=/commandhistory/.bash_history" \
  && mkdir /commandhistory \
  && touch /commandhistory/.bash_history \
  && chown -R $USERNAME /commandhistory

# Set `DEVCONTAINER` environment variable to help with orientation
ENV DEVCONTAINER=true

# Create workspace and config directories and set permissions
RUN mkdir -p /workspace /home/node/.claude && \
  chown -R node:node /workspace /home/node/.claude

WORKDIR /workspace

ARG GIT_DELTA_VERSION=0.18.2
RUN ARCH=$(dpkg --print-architecture) && \
  wget "https://github.com/dandavison/delta/releases/download/${GIT_DELTA_VERSION}/git-delta_${GIT_DELTA_VERSION}_${ARCH}.deb" && \
  sudo dpkg -i "git-delta_${GIT_DELTA_VERSION}_${ARCH}.deb" && \
  rm "git-delta_${GIT_DELTA_VERSION}_${ARCH}.deb"

# Set up non-root user
USER node

# Install global packages
ENV NPM_CONFIG_PREFIX=/usr/local/share/npm-global
ENV PATH=$PATH:/usr/local/share/npm-global/bin

# Set the default shell to zsh rather than sh
ENV SHELL=/bin/zsh

# Set the default editor and visual
ENV EDITOR=nano
ENV VISUAL=nano

# Default powerline10k theme
ARG ZSH_IN_DOCKER_VERSION=1.2.0
RUN sh -c "$(wget -O- https://github.com/deluan/zsh-in-docker/releases/download/v${ZSH_IN_DOCKER_VERSION}/zsh-in-docker.sh)" -- \
  -p git \
  -p fzf \
  -a "source /usr/share/doc/fzf/examples/key-bindings.zsh" \
  -a "source /usr/share/doc/fzf/examples/completion.zsh" \
  -a "export PROMPT_COMMAND='history -a' && export HISTFILE=/commandhistory/.bash_history" \
  -x

# Install Claude
RUN npm install -g @anthropic-ai/claude-code@${CLAUDE_CODE_VERSION}

# Install cc-switch-cli
# Note: For macOS installation, use the following commands:
# curl -LO https://github.com/saladday/cc-switch-cli/releases/latest/download/cc-switch-cli-darwin-universal.tar.gz
# tar -xzf cc-switch-cli-darwin-universal.tar.gz
# chmod +x cc-switch
# sudo mv cc-switch /usr/local/bin/
# xattr -cr /usr/local/bin/cc-switch  # If "unable to verify developer" error occurs

USER root
RUN ARCH=$(uname -m) && \
  if [ "$ARCH" = "x86_64" ]; then \
    # Linux x64
    curl -LO https://github.com/saladday/cc-switch-cli/releases/latest/download/cc-switch-cli-linux-x64-musl.tar.gz && \
    tar -xzf cc-switch-cli-linux-x64-musl.tar.gz && \
    chmod +x cc-switch && \
    mv cc-switch /usr/local/bin/ && \
    rm cc-switch-cli-linux-x64-musl.tar.gz; \
  elif [ "$ARCH" = "aarch64" ] || [ "$ARCH" = "arm64" ]; then \
    # Linux ARM64 (Raspberry Pi or ARM servers)
    curl -LO https://github.com/saladday/cc-switch-cli/releases/latest/download/cc-switch-cli-linux-arm64-musl.tar.gz && \
    tar -xzf cc-switch-cli-linux-arm64-musl.tar.gz && \
    chmod +x cc-switch && \
    mv cc-switch /usr/local/bin/ && \
    rm cc-switch-cli-linux-arm64-musl.tar.gz; \
  else \
    echo "Unsupported architecture: $ARCH"; \
    exit 1; \
  fi

# Set up sudo permissions for node user
USER root
RUN echo "node ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/node-nopasswd && \
  chmod 0440 /etc/sudoers.d/node-nopasswd

# Configure Claude settings
USER node
RUN echo '{"hasCompletedOnboarding": true}' > /home/node/.claude.json && \
  chmod 644 /home/node/.claude.json

# Verify Python packages installation
# RUN python3 -c "import mcp, requests; print('✅ Python packages installed: mcp, requests')" || true
