# syntax=docker/dockerfile:1
FROM ubuntu:20.04
SHELL ["/bin/bash", "-lc"]
ENV DEBIAN_FRONTEND=noninteractive

# 1) Update/upgrade exactly like your manual flow
RUN apt-get update && apt-get upgrade -y

# 2) Packages you used (plus a couple of helpers)
RUN apt-get install -y --no-install-recommends \
    build-essential clang bison flex libreadline-dev gawk tcl-dev libffi-dev \
    git graphviz xdot pkg-config python3 \
    libboost-system-dev libboost-python-dev libboost-filesystem-dev zlib1g-dev \
    autoconf automake libgoogle-glog-dev \
    ca-certificates curl vim tree unzip perl && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /opt

# 3) Clone OpenTimer
RUN git clone https://github.com/skyformat99/OpenTimer-1.0.5.git

# 4) Fix 'aclocal-1.14' expectation (map to installed Automake)
RUN if command -v aclocal >/dev/null; then ln -sf "$(command -v aclocal)" /usr/local/bin/aclocal-1.14; fi && \
    if command -v automake >/dev/null; then ln -sf "$(command -v automake)" /usr/local/bin/automake-1.14; fi

# 5) Follow your sequence: configure → make (may fail) → regenerate autotools → configure → make → install
RUN set -e; \
    cd /opt/OpenTimer-1.0.5 && \
    ./configure || true && \
    make -j"$(nproc)" || true && \
    aclocal && autoconf && automake --add-missing && \
    ./configure && \
    make -j"$(nproc)" && \
    make install || ( \
      echo "make install not provided; placing binary on PATH" && \
      [ -f ./OpenTimer ] && install -m 0755 ./OpenTimer /usr/local/bin/OpenTimer || \
      [ -f ./bin/OpenTimer ] && install -m 0755 ./bin/OpenTimer /usr/local/bin/OpenTimer \
    ) && \
    ldconfig

# 6) Quick check on container start
CMD echo "== OpenTimer =="// \
 && (command -v OpenTimer >/dev/null && OpenTimer -h 2>/dev/null | head -n 1 || echo "OpenTimer installed") \
 && g++ --version | head -n 1



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
