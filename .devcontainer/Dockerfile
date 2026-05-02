FROM mcr.microsoft.com/devcontainers/dotnet:9.0

ENV DEBIAN_FRONTEND=noninteractive

# Base toolchain for F# (.NET) and Haskell development.
USER root

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        ca-certificates \
        curl \
        git \
        build-essential \
        pkg-config \
        xz-utils \
        libffi-dev \
        libgmp-dev \
        libncurses-dev \
        libtinfo-dev \
        zlib1g-dev \
        libbz2-dev \
        liblzma-dev \
    && rm -rf /var/lib/apt/lists/*

USER vscode

ENV BOOTSTRAP_HASKELL_NONINTERACTIVE=1
ENV BOOTSTRAP_HASKELL_ADJUST_BASHRC=0
ENV BOOTSTRAP_HASKELL_INSTALL_STACK=1
ENV BOOTSTRAP_HASKELL_INSTALL_HLS=1

RUN curl --proto '=https' --tlsv1.2 -sSf https://get-ghcup.haskell.org | sh

ENV PATH="/home/vscode/.ghcup/bin:/home/vscode/.cabal/bin:/home/vscode/.local/bin:/home/vscode/.dotnet/tools:${PATH}"
