# syntax=docker/dockerfile:1
FROM ubuntu:20.04
SHELL ["/bin/bash", "-lc"]
ENV DEBIAN_FRONTEND=noninteractive

# 1) Update/upgrade like your working sequence
RUN apt-get update && apt-get upgrade -y

# 2) Dev deps you used + a few essentials
RUN apt-get install -y --no-install-recommends \
    build-essential clang bison flex libreadline-dev gawk tcl-dev libffi-dev \
    git graphviz xdot pkg-config python3 \
    libboost-system-dev libboost-python-dev libboost-filesystem-dev zlib1g-dev \
    autoconf automake libtool m4 perl libgoogle-glog-dev \
    ca-certificates curl unzip tree vim && \
    rm -rf /var/lib/apt/lists/*

