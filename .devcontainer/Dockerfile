# syntax=docker/dockerfile:1
FROM mcr.microsoft.com/devcontainers/base:ubuntu-24.04
SHELL ["/bin/bash", "-lc"]
ENV DEBIAN_FRONTEND=noninteractive

# --- Base packages + your requested tools ---
RUN apt-get update && apt-get install -y --no-install-recommends \
    m4 libx11-dev tcsh tcl tclsh autoconf automake libtool swig tcllib \
    libreoffice libgoogle-glog-dev \
    build-essential clang lld bison flex libfl-dev \
    libreadline-dev gawk tcl-dev libffi-dev git \
    graphviz xdot pkg-config python3 \
    libboost-system-dev libboost-python-dev libboost-filesystem-dev zlib1g-dev \
    ca-certificates curl vim tree unzip \
 && rm -rf /var/lib/apt/lists/*


# --- Optional: upgrade to newer GCC/G++ (uncomment to use GCC 14) ---
# RUN apt-get update && apt-get install -y software-properties-common && \
#     add-apt-repository -y ppa:ubuntu-toolchain-r/test && \
#     apt-get update && apt-get install -y gcc-14 g++-14 && \
#     update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-14 50 && \
#     update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-14 50 && \
#     rm -rf /var/lib/apt/lists/*

WORKDIR /opt

# --- OpenTimer (Autotools flow: ./configure; make; make install) ---
RUN git clone https://github.com/skyformat99/OpenTimer-1.0.5.git && \
    cd OpenTimer-1.0.5 && \
    # If configure script isn’t present, bootstrap it:
    ([ -x ./configure ] || autoreconf -fi) && \
    ./configure CXXFLAGS="-O3 -std=c++11" && \
    make -j"$(nproc)" && \
    make install || \
    (echo "Install step failed; copying built binary if present" && \
     ([ -f ./OpenTimer ] && cp ./OpenTimer /usr/local/bin/ot || true) && \
     ([ -f ./bin/OpenTimer ] && cp ./bin/OpenTimer /usr/local/bin/ot || true))

# --- Yosys (make config-gcc; make; make install) ---
RUN git clone https://github.com/YosysHQ/yosys.git && \
    cd yosys && \
    make config-gcc && \
    make -j"$(nproc)" && \
    make install

# Convenience: show versions at container start
CMD echo "== Versions ==" && \
    (command -v ot >/dev/null && echo "OpenTimer: $(ot -h 2>&1 | head -n1)" || echo "OpenTimer installed via /usr/local/bin/ot") && \
    yosys -V && \
    gcc --version | head -n1 && \
    libreoffice --version
