ARG BUILD_FROM=ghcr.io/hassio-addons/debian-base:7.8.1
# hadolint ignore=DL3006
FROM ${BUILD_FROM}

# Confiure locale
ENV \
    LANG=en_US.UTF-8 \
    LANGUAGE=en_US:en \
    LC_ALL=en_US.UTF-8 \
    PIP_BREAK_SYSTEM_PACKAGES=1

# Set shell
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

WORKDIR /tmp
# Copy Python requirements file
COPY ./requirements.txt /tmp/requirements.txt

# Copy in extensions list
WORKDIR /root
COPY ./vscode.extensions /root/vscode.extensions

# Setup base system
ARG BUILD_ARCH=amd64
ARG CODE_SERVER_VERSION="4.98.2"
ARG HA_CLI_VERSION="4.37.0"

# Install packages
RUN apt-get update \
    && apt-get full-upgrade -y \
    && apt-get install -y --no-install-recommends \
    ack=2.24-3 \
    libarchive-tools=3.4.3-2 \
    build-essential=12.9 \
    colordiff=1.0.20-1 \
    git=1:2.30.2-1 \
    iputils-ping=3:20210202-1 \
    locales=2.31-13+deb11u6 \
    mariadb-client=1:10.5.19-0+deb11u2 \
    mosquitto-clients=2.0.11-1+deb11u1 \
    net-tools=1.60+git20181103.0eebece-1 \
    nmap=7.91+dfsg1+really7.80+dfsg1-2+deb11u1 \
    openssh-client=1:8.4p1-5+deb11u1 \
    openssl=1.1.1n-0+deb11u5 \
    python3-dev=3.9.2-3 \
    python3=3.9.2-3 \
    unzip=6.0-26+deb11u1 \
    wget=1.21-1+deb11u1 \
    zip=3.0-12 \
    zsh=5.8-6+deb11u1 \
    less=551-2 \
    htop=3.0.5-7 \
    &&  rm -rf /var/lib/apt/lists/*

#System configuration
RUN sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen && locale-gen && \
    update-locale LANG=en_US.UTF-8 && \
    curl https://bootstrap.pypa.io/get-pip.py | python3

# Install code-server
WORKDIR /tmp/
RUN curl -fsSL https://code-server.dev/install.sh | sh  -s -- --version ${CODE_SERVER_VERSION}

# Install code-server extensions
CMD ["/bin/bash", "-c", "while read -r extention; do code-server --install-extension \"$extention\"; done < /root/vscode.extensions"]

# Install Home Assistant CLI
WORKDIR /tmp/
RUN if [ "${BUILD_ARCH}" = "aarch64" ]; then HA_CLI_URL="https://github.com/home-assistant/cli/releases/download/${HA_CLI_VERSION}/ha_arm64";  \
elif  [ "${BUILD_ARCH}" = "amd64" ]; then  HA_CLI_URL="https://github.com/home-assistant/cli/releases/download/${HA_CLI_VERSION}/ha_amd64";  fi \
&& curl  -J -L  -o ha  ${HA_CLI_URL} \
&& mv ./ha "/usr/bin/ha" \
&& chmod +x "/usr/bin/ha" \
&& ha completion zsh > /usr/share/zsh/site-functions/_ha

#Copy root file system
COPY rootfs /

# Install Oh My ZSH
RUN git clone --branch master --single-branch --depth 1 "https://github.com/robbyrussell/oh-my-zsh.git" ~/.oh-my-zsh  \
    && git clone --branch master --single-branch --depth 1 "https://github.com/zsh-users/zsh-autosuggestions"  ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions \
    && git clone --branch master --single-branch --depth 1  "https://github.com/zsh-users/zsh-syntax-highlighting.git" \~/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting \
    &&sed -i -e "s#bin/bash#bin/zsh#" /etc/passwd 

# Install Python packages
RUN  update-alternatives --install /usr/bin/python python /usr/bin/python3 10  && \
    pip3 install --no-cache-dir -r /tmp/requirements.txt

# Cleanup apt cache
RUN apt-get purge -y --auto-remove \
    build-essential \
    libarchive-tools \
    python3-dev \
    uuid-runtime

# Cleanup files
RUN find /usr/local \
    \( -type d -a -name test -o -name tests -o -name '__pycache__' \) \
    -o \( -type f -a -name '*.pyc' -o -name '*.pyo' \) \
    -exec rm -rf '{}' + && \
    rm -fr /root/.cache /tmp/* /var/{cache,log}/* /var/lib/apt/lists/*

# Health check
HEALTHCHECK \
    CMD curl --fail http://127.0.0.1:1337/healthz || exit 1

# Build arguments
ARG BUILD_ARCH
ARG BUILD_DATE
ARG BUILD_DESCRIPTION
ARG BUILD_NAME
ARG BUILD_REF
ARG BUILD_REPOSITORY
ARG BUILD_VERSION

# Labels
LABEL \
    io.hass.name="${BUILD_NAME}" \
    io.hass.description="${BUILD_DESCRIPTION}" \
    io.hass.arch="${BUILD_ARCH}" \
    io.hass.type="addon" \
    io.hass.version=${BUILD_VERSION} \
    maintainer="Franck Nijhof <frenck@addons.community>" \
    org.opencontainers.image.title="${BUILD_NAME}" \
    org.opencontainers.image.description="${BUILD_DESCRIPTION}" \
    org.opencontainers.image.vendor="Home Assistant Community Add-ons" \
    org.opencontainers.image.authors="Franck Nijhof <frenck@addons.community>" \
    org.opencontainers.image.licenses="MIT" \
    org.opencontainers.image.url="https://addons.community" \
    org.opencontainers.image.source="https://github.com/${BUILD_REPOSITORY}" \
    org.opencontainers.image.documentation="https://github.com/${BUILD_REPOSITORY}/blob/main/README.md" \
    org.opencontainers.image.created=${BUILD_DATE} \
    org.opencontainers.image.revision=${BUILD_REF} \
    org.opencontainers.image.version=${BUILD_VERSION}
