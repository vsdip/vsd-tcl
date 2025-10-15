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

WORKDIR /opt

# 3) Clone OpenTimer
RUN git clone https://github.com/skyformat99/OpenTimer-1.0.5.git

# 4) Map legacy aclocal-1.14/automake-1.14 to what we have (Automake 1.16 on Ubuntu 20.04)
RUN if command -v aclocal >/dev/null;  then ln -sf "$(command -v aclocal)"  /usr/local/bin/aclocal-1.14;  fi && \
    if command -v automake >/dev/null; then ln -sf "$(command -v automake)" /usr/local/bin/automake-1.14; fi

# 5) Configure, regenerate autotools, then build ONLY the OpenTimer binary (skip dist targets)
ENV OT_CXXFLAGS="-O3 -std=gnu++20 -pthread"
RUN set -e; \
    cd /opt/OpenTimer-1.0.5 && \
    ./configure CXXFLAGS="${OT_CXXFLAGS}" || true && \
    autoreconf -fi && \
    ./configure CXXFLAGS="${OT_CXXFLAGS}" && \
    # build just the core binary; do NOT build 'all' (which drags in dist targets)
    make -j"$(nproc)" OpenTimer || make OpenTimer && \
    # install the binary; some snapshots may not ship install rules
    install -m 0755 ./OpenTimer /usr/local/bin/OpenTimer || \
      install -m 0755 ./bin/OpenTimer /usr/local/bin/OpenTimer && \
    ldconfig

# 6) Yosys (with submodules)
RUN git clone https://github.com/YosysHQ/yosys.git && \
    cd yosys && \
    git submodule update --init --recursive && \
    make config-gcc && \
    make -j"$(nproc)" && \
    make install

# 7) Simple version check on container start
CMD echo "== OpenTimer ==" && \
    (OpenTimer -h 2>/dev/null | head -n1 || true) && \
    echo "== Yosys ==" && yosys -V && \
    echo "== GCC ==" && g++ --version | head -n1
