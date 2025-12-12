FROM debian:stable-slim AS devcontainer

ARG TARGETARCH
ARG TARGETOS

RUN apt-get update
RUN apt-get upgrade -y
RUN apt-get install -y apt-transport-https build-essential buildah ca-certificates curl git gnupg gpg jq less locales-all nano php podman python3 python3-pip python3-venv rust-all skopeo ssh sudo unzip wget zsh

RUN echo "%sudo ALL=(ALL:ALL) NOPASSWD: ALL" | EDITOR="tee -a" visudo --file="/etc/sudoers.d/sudo"

# Go
RUN FILE=$(curl https://go.dev/dl/?mode=json | jq -r "[.[] | select(.stable == true)] | .[0].files[] | select(.arch == \"${TARGETARCH}\" and .os == \"${TARGETOS}\") | .filename") && \
    curl -L -o "/tmp/${FILE}" "https://go.dev/dl/${FILE}" && \
    tar -C /usr/local -xzf /tmp/${FILE} && \
    rm -f /tmp/${FILE} && \
    echo "export PATH=\$PATH:/usr/local/go/bin" > /etc/profile.d/go.sh
RUN 

# AWS CLI
RUN curl -o "/tmp/awscliv2.zip" "https://awscli.amazonaws.com/awscli-exe-linux-$(uname -m).zip" && \
    unzip -d /tmp /tmp/awscliv2.zip && \
    /tmp/aws/install && \
    rm -f /tmp/awscliv2.zip && \
    rm -fr /tmp/aws

# tenv
RUN LATEST_VERSION=$(curl --silent https://api.github.com/repos/tofuutils/tenv/releases/latest | jq -r .tag_name) && \
    curl -L -o "/tmp/tenv_${LATEST_VERSION}_amd64.deb" "https://github.com/tofuutils/tenv/releases/latest/download/tenv_${LATEST_VERSION}_${TARGETARCH}.deb" && \
    dpkg -i "/tmp/tenv_${LATEST_VERSION}_amd64.deb" && \
    rm -f /tmp/tenv_${LATEST_VERSION}_amd64.deb

# Deno
RUN curl -L -o "/tmp/deno.zip" "https://github.com/denoland/deno/releases/latest/download/deno-$(uname -m)-unknown-linux-gnu.zip" && \
    unzip -d /usr/local/bin /tmp/deno.zip && \
    rm -f "/tmp/deno.zip"

# Docker
RUN curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc && \
    { \
    echo "Types: deb"; \
    echo "URIs: https://download.docker.com/linux/debian"; \
    echo "Suites: $(. /etc/os-release && echo $VERSION_CODENAME)"; \
    echo "Components: stable"; \
    echo "Signed-By: /etc/apt/keyrings/docker.asc"; \
    } > /etc/apt/sources.list.d/docker.sources

# Google Cloud CLI
RUN curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | gpg --dearmor -o /etc/apt/keyrings/cloud.google.gpg && \
    { \
    echo "Types: deb"; \
    echo "URIs: https://packages.cloud.google.com/apt"; \
    echo "Suites: cloud-sdk"; \
    echo "Components: main"; \
    echo "Signed-By: /etc/apt/keyrings/cloud.google.gpg"; \
    } > /etc/apt/sources.list.d/google-cloud-sdk.sources

# Node.js
RUN curl -fsSL https://deb.nodesource.com/setup_24.x |bash -

RUN apt-get update
RUN apt-get install -y containerd.io docker-buildx-plugin docker-ce docker-ce-cli docker-compose-plugin google-cloud-cli nodejs

RUN useradd -m -s /bin/bash -G docker,sudo developer

RUN { \
    echo "#!/bin/bash"; \
    echo ""; \
    echo "/sbin/sshd"; \
    echo ""; \
    echo "/bin/dockerd > /var/log/dockerd.log 2>&1"; \
    } > /cmd.sh && \
    chmod +x /cmd.sh

LABEL devcontainer.metadata='[{}]'

FROM devcontainer

ARG UID
ARG GID

RUN usermod -o -u ${UID} developer
RUN groupmod -o -g ${GID} developer
RUN chown -R developer:developer /home/developer
