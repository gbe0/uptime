## Base on Ubuntu Groovy
FROM ubuntu:groovy

## Set maintainer label
LABEL maintainer="Chris <git@gbe0.com>"

## Configure the default apt mirror for package installations
ARG APT_MIRROR="http://archive.ubuntu.com/ubuntu/"

## Configure the Node repository name
## This will be appended to the URL like https://deb.nodesource.com/node_${NODE_VERSION}.x
## As an example for Node 15.x the URL will look like this: https://deb.nodesource.com/node_15.x/
ARG NODE_VERSION="15"

## Configure default user ID and group ID that will be used for VS Code
ARG UID=1000
ARG GID=$UID

## Configure LANG, LC_ALL and LANGUAGE environment vars to prevent various errors
ENV LANG=C LC_ALL=C LANGUAGE=C

## Update sources list and run system upgrade
RUN echo \
      ### Update current sources list and install lsb_release utility
      && apt-get update \
      && DEBIAN_FRONTEND="noninteractive" apt-get -q -y install \
        lsb-release \
      ### Configure package sources list
      && echo "deb ${APT_MIRROR} $(lsb_release -sc) main restricted" > /etc/apt/sources.list \
      && echo "deb ${APT_MIRROR} $(lsb_release -sc)-updates main restricted" >> /etc/apt/sources.list \
      && echo "deb ${APT_MIRROR} $(lsb_release -sc) universe" >> /etc/apt/sources.list \
      && echo "deb ${APT_MIRROR} $(lsb_release -sc)-updates universe" >> /etc/apt/sources.list \
      && echo "deb ${APT_MIRROR} $(lsb_release -sc) multiverse" >> /etc/apt/sources.list \
      && echo "deb ${APT_MIRROR} $(lsb_release -sc)-updates multiverse" >> /etc/apt/sources.list \
      && echo "deb ${APT_MIRROR} $(lsb_release -sc)-backports main restricted universe multiverse" >> /etc/apt/sources.list \
      && echo "deb ${APT_MIRROR} $(lsb_release -sc)-proposed main restricted universe multiverse" >> /etc/apt/sources.list \
      && echo "deb http://archive.canonical.com/ubuntu $(lsb_release -sc) partner" >> /etc/apt/sources.list \
      && echo "deb http://security.ubuntu.com/ubuntu/ $(lsb_release -sc)-security main restricted" >> /etc/apt/sources.list \
      && echo "deb http://security.ubuntu.com/ubuntu/ $(lsb_release -sc)-security universe" >> /etc/apt/sources.list \
      && echo "deb http://security.ubuntu.com/ubuntu/ $(lsb_release -sc)-security multiverse" >> /etc/apt/sources.list \
      ### To add source package lists, uncomment the below lines
      # && echo "deb-src ${APT_MIRROR} $(lsb_release -sc) main restricted" >> /etc/apt/sources.list \
      # && echo "deb-src ${APT_MIRROR} $(lsb_release -sc)-updates main restricted" >> /etc/apt/sources.list \
      # && echo "deb-src ${APT_MIRROR} $(lsb_release -sc) universe" >> /etc/apt/sources.list \
      # && echo "deb-src ${APT_MIRROR} $(lsb_release -sc)-updates universe" >> /etc/apt/sources.list \
      # && echo "deb-src ${APT_MIRROR} $(lsb_release -sc) multiverse" >> /etc/apt/sources.list \
      # && echo "deb-src ${APT_MIRROR} $(lsb_release -sc)-updates multiverse" >> /etc/apt/sources.list \
      # && echo "deb-src ${APT_MIRROR} $(lsb_release -sc)-backports main restricted universe multiverse" >> /etc/apt/sources.list \
      # && echo "deb-src ${APT_MIRROR} $(lsb_release -sc)-proposed main restricted universe multiverse" >> /etc/apt/sources.list \
      # && echo "deb-src http://archive.canonical.com/ubuntu $(lsb_release -sc) partner" >> /etc/apt/sources.list \
      # && echo "deb-src http://security.ubuntu.com/ubuntu/ $(lsb_release -sc)-security main restricted" >> /etc/apt/sources.list \
      # && echo "deb-src http://security.ubuntu.com/ubuntu/ $(lsb_release -sc)-security universe" >> /etc/apt/sources.list \
      # && echo "deb-src http://security.ubuntu.com/ubuntu/ $(lsb_release -sc)-security multiverse" >> /etc/apt/sources.list \
      ### Update package cache
      && apt-get update \
      ### Perform full system upgrade
      && DEBIAN_FRONTEND="noninteractive" apt-get -q -y dist-upgrade

## Install development related tools
RUN echo \
      ### Install useful command line utilities
      && DEBIAN_FRONTEND="noninteractive" apt-get -q -y install \
        vim neovim less strace curl wget screen telnet git tmux dnsutils jq net-tools \
        iputils-ping bash-completion sudo tcptraceroute \
      ### Install language servers and other utilities for vscode extensions to work
      && DEBIAN_FRONTEND="noninteractive" apt-get -q -y install \
        #### Java for Sonarlint
        openjdk-11-jdk-headless \
      ### Create nvim config directory
      && mkdir -p /root/.config/nvim

## Add editor configuration files
COPY vimrc /root/.vimrc
COPY neovimrc /root/.config/nvim/init.vim

## Install Node
RUN echo \
      ### Install requirements
      && DEBIAN_FRONTEND="noninteractive" apt-get -q -y install \
        apt-transport-https gnupg \
      ### Trust the node.js GPG key
      && wget -qO- https://deb.nodesource.com/gpgkey/nodesource.gpg.key | apt-key add - \
      ### Add the package repository
      && echo "deb https://deb.nodesource.com/node_${NODE_VERSION}.x $(lsb_release -sc) main" > /etc/apt/sources.list.d/nodesource.list \
      ### Update package sources
      && apt-get update \
      ### Install Node
      && DEBIAN_FRONTEND="noninteractive" apt-get -q -y install \
        nodejs

## Configure the user that VS Code will run as
RUN echo \
      ### Add user and group
      && groupadd --gid ${GID} vscode \
      && useradd --uid ${UID} --gid ${GID} -s /bin/bash -m vscode \
      ### Create directories for extensions that are installed by VS Code
      && mkdir -p /home/vscode/.vscode-server /home/vscode/.vscode-server-insiders \
      ### Copy vim config from root user
      && cp /root/.vimrc /home/vscode/.vimrc \
      ### Copy neovim config from root user
      && cp -R /root/.config /home/vscode/.config \
      ### Fix permissions on home directory
      && chown -R ${UID}:${GID} /home/vscode \
      ### Add sudo configuration to allow no password access for vscode user
      && echo vscode ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/vscode \
      && chmod 0440 /etc/sudoers.d/vscode \
      ### Add work around for error with sudo 1.8.31
      ### https://github.com/sudo-project/sudo/issues/42
      ### Without this fix, every sudo command gives this error:
      ###   sudo: setrlimit(RLIMIT_CORE): Operation not permitted
      && echo "Set disable_coredump false" >> /etc/sudo.conf

## Add the bash aliases file
COPY .bash_aliases /home/vscode/.bash_aliases

## Switch to the VS Code user
USER vscode

## Set the work directory
WORKDIR /uptime