ARG BUILD_FROM=ghcr.io/hassio-addons/base:16.1.2
# hadolint ignore=DL3006
FROM ${BUILD_FROM}

# Set shell
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Add env
ENV TERM="xterm-256color"

# Copy Python requirements file
COPY requirements.txt /tmp/

# Setup base
ARG BUILD_ARCH=amd64
ARG HA_CLI_VERSION="4.35.0"
ARG TTYD_VERSION="1.7.7"
# hadolint ignore=DL3003,DL3042
RUN \
    apk add --no-cache --virtual .build-dependencies \
        bsd-compat-headers=0.7.2-r6 \
        build-base=0.5-r3 \
        cmake=3.29.3-r0 \
        docker=26.1.5-r0 \
        json-c-dev=0.17-r0 \
        libffi-dev=3.4.6-r0 \
        libuv-dev=1.48.0-r0 \
        openssl-dev=3.3.1-r3 \
        python3-dev=3.12.3-r1 \
        zlib-dev=1.3.1-r1 \
    \
    && apk add --no-cache \
        ack=3.7.0-r1 \
        alsa-plugins-pulse=1.2.7.1-r3 \
        alsa-utils=1.2.11-r1 \
        awake=1.0-r10 \
        bash-completion=2.12.0-r0 \
        bind-tools=9.18.27-r0 \
        bluez=5.76-r0 \
        colordiff=1.0.21-r0 \
        docker-bash-completion=26.1.5-r0 \
        docker-zsh-completion=26.1.5-r0 \
        gcompat=1.1.0-r4 \
        git=2.45.2-r0 \
        htop=3.3.0-r0 \
        json-c=0.17-r0 \
        libltdl=2.4.7-r3 \
        libstdc++=13.2.1_git20240309-r0 \
        libuv=1.48.0-r0 \
        libxml2-utils=2.12.7-r0 \
        mariadb-client=10.11.8-r0 \
        mosh=1.4.0-r8 \
        mosquitto-clients=2.0.18-r0 \
        nano-syntax=8.0-r0 \
        nano=8.0-r0 \
        ncurses=6.4_p20240420-r0 \
        net-tools=2.10-r3 \
        nmap=7.95-r0 \
        openssh=9.7_p1-r4 \
        openssl=3.3.1-r3 \
        procps-ng=4.0.4-r0 \
        pwgen=2.08-r3 \
        pulseaudio-utils=17.0-r0 \
        py3-pip=24.0-r2 \
        python3=3.12.3-r1 \
        rsync=3.3.0-r0 \
        sqlite=3.45.3-r1 \
        sudo=1.9.15_p5-r0 \
        tmux=3.4-r1 \
        vim=9.1.0414-r0 \
        wget=1.24.5-r0 \
        zip=3.0-r12 \
        zsh-autosuggestions=0.7.0-r0 \
        zsh-syntax-highlighting=0.8.0-r0 \
        zsh=5.9-r3 \
    \
    && git clone --depth 1 \
        https://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh \
    \
    && curl -L -s -o /usr/bin/ha \
        "https://github.com/home-assistant/cli/releases/download/${HA_CLI_VERSION}/ha_${BUILD_ARCH}" \
    \
    && chmod a+x /usr/bin/ha \
    && ha completion bash > /usr/share/bash-completion/completions/ha \
    \
    && sed -i -e "s#bin/sh#bin/zsh#" /etc/passwd \
    \
    && git clone --branch "v4.3.3" --depth=1 \
        https://github.com/warmcat/libwebsockets.git /tmp/libwebsockets \
    \
    && mkdir -p /tmp/libwebsockets/build \
    && cd /tmp/libwebsockets/build \
    && cmake .. \
        -DCMAKE_BUILD_TYPE=MinSizeRel \
        -DCMAKE_INSTALL_PREFIX=/usr \
        -DCMAKE_VERBOSE_MAKEFILE=TRUE \
        -DLWS_IPV6=ON \
        -DLWS_STATIC_PIC=ON \
        -DLWS_UNIX_SOCK=OFF \
        -DLWS_WITH_LIBUV=ON \
        -DLWS_WITH_SHARED=ON \
        -DLWS_WITHOUT_TESTAPPS=ON \
    && make \
    && make install \
    \
    && git clone --branch main --single-branch \
        https://github.com/tsl0922/ttyd.git /tmp/ttyd \
    && git -C /tmp/ttyd checkout "${TTYD_VERSION}" \
    \
    && mkdir -p /tmp/ttyd/build \
    && cd /tmp/ttyd/build \
    && cmake .. \
        -DCMAKE_BUILD_TYPE=MinSizeRel \
        -DCMAKE_INSTALL_PREFIX=/usr \
        -DCMAKE_VERBOSE_MAKEFILE=TRUE \
    && make \
    && make install \
    \
    && cp /usr/bin/docker /usr/local/bin/.undocked \
    \
    && pip3 install -r /tmp/requirements.txt \
    \
    && apk del --no-cache --purge .build-dependencies \
    \
    && find /usr \
        \( -type d -a -name test -o -name tests -o -name '__pycache__' \) \
        -o \( -type f -a -name '*.pyc' -o -name '*.pyo' \) \
        -exec rm -rf '{}' + \
    \
    && rm -f -r \
        /root/.cache \
        /root/.cmake \
        /tmp/*

# Copy root filesystem
COPY rootfs /

# Ensure right permissions
RUN \
    chmod 0750 /etc/sudo.conf \
    && chmod 0750 -R /etc/sudoers.d \
    && chmod 0640 /etc/sudoers.d

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
