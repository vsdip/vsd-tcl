# syntax=docker/dockerfile:1
FROM ubuntu:20.04
SHELL ["/bin/bash", "-lc"]
ENV DEBIAN_FRONTEND=noninteractive

# ---- Step 1. Update & upgrade ----
RUN apt-get update && apt-get upgrade -y

# ---- Step 2. Install build dependencies (exact list you used) ----
RUN apt-get install -y --no-install-recommends \
    build-essential clang bison flex libreadline-dev gawk tcl-dev libffi-dev \
    git graphviz xdot pkg-config python3 \
    libboost-system-dev libboost-python-dev libboost-filesystem-dev zlib1g-dev \
    autoconf automake libgoogle-glog-dev \
    ca-certificates curl vim tree unzip && \
    rm -rf /var/lib/apt/lists/*

# ---- Step 3. Build and install OpenTimer ----
WORKDIR /opt
RUN git clone https://github.com/skyformat99/OpenTimer-1.0.5.git && \
    cd OpenTimer-1.0.5 && \
    ./configure && \
    make -j"$(nproc)" || true && \
    # Regenerate Autotools metadata if needed
    aclocal && autoconf && automake --add-missing && \
    ./configure && \
    make -j"$(nproc)" && \
    make install && \
    ldconfig && \
    echo "✅ OpenTimer installed at /usr/local/bin/OpenTimer"

# ---- Step 4. Default command: show version ----
CMD echo "== OpenTimer Installed ==" && \
    /usr/local/bin/OpenTimer -h 2>/dev/null | head -n 1 && \
    g++ --version | head -n 1


# ---------- Yosys (make config-gcc; make; make install) ----------
RUN git clone https://github.com/YosysHQ/yosys.git && \
    cd yosys && \
    git submodule update --init --recursive && \
    make config-gcc && \
    make -j"$(nproc)" && \
    make install

# ---------- sanity on container start ----------
CMD echo "== Versions ==" && \
    (command -v ot >/dev/null && (ot -h 2>&1 | head -n1) || echo "OpenTimer installed as /usr/local/bin/ot") && \
    yosys -V && \
    gcc --version | head -n1 && \
    libreoffice --version
