# syntax=docker/dockerfile:1

# Base image tuned for GitHub Codespaces/devcontainers
FROM mcr.microsoft.com/devcontainers/base:ubuntu-24.04

ENV DEBIAN_FRONTEND=noninteractive
ARG USERNAME=vscode
ARG WORKDIR=/workspaces

# --------------------------------------------------------------------
# OS packages (build tools, libs, and the ones you explicitly asked)
# Ubuntu 24.04 already has gcc/g++ 13 (a "higher version").
# --------------------------------------------------------------------
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      # your requested packages
      m4 libx11-dev tcsh tcl tclsh autoconf automake libtool swig tcllib \
      libreoffice \
      # build & dev essentials
      build-essential git curl ca-certificates pkg-config \
      cmake ninja-build bison flex gawk libreadline-dev tcl-dev libffi-dev \
      libboost-all-dev python3 python3-pip \
      # nice to have
      vim tree unzip && \
    rm -rf /var/lib/apt/lists/*

# Make sure gcc/g++ point to the higher default (13 on 24.04)
RUN gcc --version && g++ --version

# --------------------------------------------------------------------
# Build and install OpenTimer 1.0.5
# --------------------------------------------------------------------
WORKDIR /opt
RUN git clone https://github.com/skyformat99/OpenTimer-1.0.5.git && \
    cmake -S OpenTimer-1.0.5 -B OpenTimer-1.0.5/build -DCMAKE_BUILD_TYPE=Release && \
    cmake --build OpenTimer-1.0.5/build -j"$(nproc)" && \
    cmake --install OpenTimer-1.0.5/build || \
    (echo "OpenTimer 'cmake --install' not provided; copying binary instead." && \
     cp OpenTimer-1.0.5/build/ot /usr/local/bin/ot)

# --------------------------------------------------------------------
# Build and install Yosys (latest from main)
# --------------------------------------------------------------------
RUN git clone https://github.com/YosysHQ/yosys.git && \
    make -C yosys -j"$(nproc)" && \
    make -C yosys install

# --------------------------------------------------------------------
# Create a place for your project and set permissions
# --------------------------------------------------------------------
RUN mkdir -p ${WORKDIR} && chown -R ${USERNAME}:${USERNAME} /opt ${WORKDIR}
WORKDIR ${WORKDIR}

# Quick PATH sanity (devcontainers already include /usr/local/bin)
ENV PATH="/usr/local/bin:${PATH}"

# Default shell
SHELL ["/bin/bash", "-lc"]

# Helpful versions on container start
CMD echo -e "\n== Tool versions ==" \
    && ot -h | head -n 1 || true \
    && yosys -V \
    && gcc --version | head -n 1 \
    && libreoffice --version
