# syntax=docker/dockerfile:1
FROM mcr.microsoft.com/devcontainers/base:ubuntu-20.04
SHELL ["/bin/bash", "-lc"]
ENV DEBIAN_FRONTEND=noninteractive

# Minimal deps for OpenTimer
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential git autoconf automake libtool m4 perl \
    libgoogle-glog-dev && \
    rm -rf /var/lib/apt/lists/*

