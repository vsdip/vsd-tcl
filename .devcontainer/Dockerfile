# syntax=docker/dockerfile:1
FROM mcr.microsoft.com/devcontainers/base:ubuntu-24.04
SHELL ["/bin/bash", "-lc"]
ENV DEBIAN_FRONTEND=noninteractive

# ---- Base & requested packages ----
RUN apt-get update && apt-get install -y --no-install-recommends \
    # Your requested tools
    m4 libx11-dev tcsh tcl tclsh autoconf automake libtool swig tcllib \
    libreoffice \
    # OpenTimer dependency
    libgoogle-glog-dev \
    # Yosys build deps (as you listed)
    build-essential clang lld bison flex libfl-dev \
    libreadline-dev gawk tcl-dev libffi-dev git \
    graphviz xdot pkg-config python3 \
    libboost-system-dev libboost-python-dev libboost-filesystem-dev zlib1g-dev \
    # helpers
    ca-certificates curl vim tree unzip perl \
 && rm -rf /var/lib/apt/lists/*

# ---- Optional: newer GCC/G++ (uncomment if you want GCC-14) ----
# RUN apt-get update && apt-get install -y software-properties-common && \
#     add-apt-repository -y ppa:ubuntu-toolchain-r/test && \
#     apt-get update && apt-get install -y gcc-14 g++-14 && \
#     update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-14 50 && \
#     update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-14 50 && \
#     rm -rf /var/lib/apt/lists/*

WORKDIR /opt

# ---- OpenTimer (Autotools: ./configure; make; make install) ----
# Some snapshots try to call aclocal-1.14; provide compatibility symlinks to 1.16.
RUN if command -v aclocal-1.16 >/dev/null; then \
      ln -sf "$(command -v aclocal-1.16)" /usr/local/bin/aclocal-1.14; \
      ln -sf "$(command -v automake-1.16)" /usr/local/bin/automake-1.14; \
    fi

RUN git clone https://github.com/skyformat99/OpenTimer-1.0.5.git && \
    cd OpenTimer-1.0.5 && \
    # Use existing generated configure scripts; don't force autoreconf.
    ./configure CXXFLAGS="-O3 -std=c++11" && \
    make -j"$(nproc)" && \
    make install || ( \
      echo "Install failed; copying built binary if present" && \
      ([ -f ./OpenTimer ] && cp ./OpenTimer /usr/local/bin/ot || true) && \
      ([ -f ./bin/OpenTimer ] && cp ./bin/OpenTimer /usr/local/bin/ot || true) )

# ---- Yosys (make config-gcc; make; make install) ----
RUN git clone https://github.com/YosysHQ/yosys.git && \
    cd yosys && \
    git submodule update --init --recursive && \
    make config-gcc && \
    make -j"$(nproc)" && \
    make install

# ---- sanity on container start ----
CMD echo "== Versions ==" && \
    (command -v ot >/dev/null && (ot -h 2>&1 | head -n1) || echo "OpenTimer installed as /usr/local/bin/ot") && \
    yosys -V && \
    gcc --version | head -n1 && \
    libreoffice --version
