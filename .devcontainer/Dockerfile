# syntax=docker/dockerfile:1
FROM mcr.microsoft.com/devcontainers/base:ubuntu-22.04

SHELL ["/bin/bash", "-lc"]
ENV DEBIAN_FRONTEND=noninteractive
ENV DISPLAY=:1

# -----------------------------
# System & build dependencies
# -----------------------------
RUN apt-get update && apt-get install -y --no-install-recommends \
    git build-essential autoconf automake libtool m4 \
    tcl tcl-dev tk tk-dev libcairo2-dev \
    libx11-dev libxext-dev libxrender-dev libxpm-dev libxaw7-dev \
    libreadline-dev libncurses-dev flex bison gawk tcsh \
    python3 python3-pip pkg-config curl ca-certificates \
    xfce4 xfce4-terminal x11vnc xvfb novnc websockify supervisor dbus-x11 \
    libgoogle-glog-dev libgflags-dev \
    clang lld libffi-dev libboost-system-dev libboost-filesystem-dev \
    libboost-python-dev libfl-dev zlib1g-dev graphviz xdot \
 && apt-get clean && rm -rf /var/lib/apt/lists/*

# -----------------------------
# Copy precompiled OpenTimer
# -----------------------------
WORKDIR /workspaces/vsd-tcl

COPY OpenTimer /usr/bin/OpenTimer
COPY libOpenTimer.so.0 /usr/lib/libOpenTimer.so.0

RUN chmod 755 /usr/bin/OpenTimer /usr/lib/libOpenTimer.so.0 && \
    ldconfig

ENV LD_LIBRARY_PATH=/usr/lib:$LD_LIBRARY_PATH

# -----------------------------
# Build & install Yosys
# -----------------------------
WORKDIR /opt

RUN git clone https://github.com/YosysHQ/yosys.git && \
    cd yosys && \
    make config-gcc && \
    make -j"$(nproc)" && \
    make install && \
    echo "✅ Yosys installed successfully"

# -----------------------------
# Restore workspace
# -----------------------------
WORKDIR /workspaces/vsd-tcl

# -----------------------------
# noVNC desktop environment
# -----------------------------
COPY .devcontainer/supervisord.conf /etc/supervisor/conf.d/vnc.conf

EXPOSE 6080

CMD ["/usr/bin/supervisord", "-n"]
