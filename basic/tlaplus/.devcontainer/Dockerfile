FROM node:20

ARG TZ
ENV TZ="$TZ"

# TLA+ & Apalache Version Configuration
ENV TLAPLUS_VERSION=1.7.4
ENV APALACHE_VERSION=0.52.1
ENV COMMUNITY_MODULES_VERSION=202505152026

# Install basic development tools and iptables/ipset
RUN apt update && apt install -y less \
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
  jq

# Install Amazon Corretto
RUN wget -O- https://apt.corretto.aws/corretto.key | gpg --dearmor -o /usr/share/keyrings/corretto-keyring.gpg \
    && echo "deb [signed-by=/usr/share/keyrings/corretto-keyring.gpg] https://apt.corretto.aws stable main" > /etc/apt/sources.list.d/corretto.list \
    && apt-get update \
    && apt-get install -y --no-install-recommends java-24-amazon-corretto-jdk

# Install Latex
RUN apt-get install -y texlive-latex-extra

# Clean apt
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

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

RUN ARCH=$(dpkg --print-architecture) && \
  wget "https://github.com/dandavison/delta/releases/download/0.18.2/git-delta_0.18.2_${ARCH}.deb" && \
  sudo dpkg -i "git-delta_0.18.2_${ARCH}.deb" && \
  rm "git-delta_0.18.2_${ARCH}.deb"

# Set up non-root user
USER node

# Install global packages
ENV NPM_CONFIG_PREFIX=/usr/local/share/npm-global
ENV PATH=$PATH:/usr/local/share/npm-global/bin

# Set the default shell to zsh rather than sh
ENV SHELL=/bin/zsh

# Default powerline10k theme
RUN sh -c "$(wget -O- https://github.com/deluan/zsh-in-docker/releases/download/v1.2.0/zsh-in-docker.sh)" -- \
  -p git \
  -p fzf \
  -a "source /usr/share/doc/fzf/examples/key-bindings.zsh" \
  -a "source /usr/share/doc/fzf/examples/completion.zsh" \
  -a "export PROMPT_COMMAND='history -a' && export HISTFILE=/commandhistory/.bash_history" \
  -x

# Install Claude
RUN npm install -g @anthropic-ai/claude-code

# Copy and set up firewall script
COPY init-firewall.sh /usr/local/bin/
USER root
RUN chmod +x /usr/local/bin/init-firewall.sh && \
  echo "node ALL=(root) NOPASSWD: /usr/local/bin/init-firewall.sh" > /etc/sudoers.d/node-firewall && \
  chmod 0440 /etc/sudoers.d/node-firewall
USER node

# Java Config
ENV LANG=C.UTF-8
ENV JAVA_HOME=/usr/lib/jvm/java-24-amazon-corretto

# Install TLA+ Toolbox CLI tools
USER root
RUN mkdir -p /opt/tlaplus && \
    cd /opt/tlaplus && \
    # TLA+ tools main jar
    wget https://github.com/tlaplus/tlaplus/releases/download/v${TLAPLUS_VERSION}/tla2tools.jar && \
    # apalache-mc
    wget https://github.com/apalache-mc/apalache/releases/download/v${APALACHE_VERSION}/apalache-${APALACHE_VERSION}.tgz && \
    tar -xzvf apalache-${APALACHE_VERSION}.tgz && \
    ln -s /opt/tlaplus/apalache-${APALACHE_VERSION}/bin/apalache-mc /usr/local/bin/apalache-mc && \
    # Community Modules
    wget https://github.com/tlaplus/CommunityModules/releases/download/${COMMUNITY_MODULES_VERSION}/CommunityModules-deps.jar && \
    # Create executable scripts
    echo '#!/bin/bash\njava -XX:+UseParallelGC -jar /opt/tlaplus/tla2tools.jar "$@"' > /usr/local/bin/tlc && \
    echo '#!/bin/bash\njava -cp /opt/tlaplus/tla2tools.jar pcal.trans "$@"' > /usr/local/bin/pcal && \
    echo '#!/bin/bash\njava -cp /opt/tlaplus/tla2tools.jar tla2sany.SANY "$@"' > /usr/local/bin/sany && \
    echo '#!/bin/bash\njava -cp /opt/tlaplus/tla2tools.jar tla2tex.TLA "$@"' > /usr/local/bin/tla2tex && \
    chmod +x /usr/local/bin/tlc /usr/local/bin/apalache-mc /usr/local/bin/pcal /usr/local/bin/sany /usr/local/bin/tla2tex
USER node
