# syntax=docker/dockerfile:1
FROM mcr.microsoft.com/devcontainers/base:ubuntu-20.04
SHELL ["/bin/bash","-lc"]
ENV DEBIAN_FRONTEND=noninteractive

# --- Desktop + VNC/noVNC stack ---
RUN apt-get update && apt-get install -y --no-install-recommends \
    xfce4 xfce4-terminal dbus-x11 tigervnc-standalone-server \
    novnc websockify supervisor x11-xserver-utils \
    # handy tools
    ca-certificates curl vim less htop git \
 && rm -rf /var/lib/apt/lists/*

# --- Minimal deps for OpenTimer (glog + gflags are REQUIRED) ---
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential autoconf automake libtool m4 perl \
    libgoogle-glog-dev libgflags-dev \
 && rm -rf /var/lib/apt/lists/*

# --- noVNC startup script (reads VNC_PASSWORD env at runtime) ---
RUN install -d -m 0755 /usr/local/bin && cat >/usr/local/bin/start-desktop.sh <<'EOS'
#!/usr/bin/env bash
set -euo pipefail

: "${VNC_DISPLAY:=:1}"
: "${VNC_RESOLUTION:=1280x800}"
: "${VNC_DEPTH:=24}"
: "${VNC_PASSWORD:=vsdiat}"

# Start dbus (optional)
pgrep -x dbus-daemon >/dev/null 2>&1 || (dbus-launch >/dev/null 2>&1 || true)

# Set VNC password for 'vscode'
sudo -u vscode bash -lc "mkdir -p ~/.vnc && echo -n '${VNC_PASSWORD}' | vncpasswd -f > ~/.vnc/passwd && chmod 600 ~/.vnc/passwd"

# Start VNC (XFCE)
sudo -u vscode vncserver ${VNC_DISPLAY} -geometry ${VNC_RESOLUTION} -depth ${VNC_DEPTH} -localhost no -xstartup /usr/bin/startxfce4

# Expose via noVNC on 6080
websockify --web=/usr/share/novnc/ 6080 localhost:5901
EOS
RUN chmod +x /usr/local/bin/start-desktop.sh

# --- Supervisor keeps the desktop running ---
RUN install -d -m 0755 /etc/supervisor/conf.d && cat >/etc/supervisor/conf.d/desktop.conf <<'EOS'
[program:desktop]
command=/usr/local/bin/start-desktop.sh
autorestart=true
priority=10
stdout_logfile=/var/log/desktop.log
stderr_logfile=/var/log/desktop.err
user=root
EOS

# --- Build OpenTimer (skip buggy dist programs) as the 'vscode' user ---
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential git autoconf automake libtool m4 perl \
    libgoogle-glog-dev libgflags-dev \
 && rm -rf /var/lib/apt/lists/*

USER vscode
WORKDIR /workspaces

# Build OpenTimer WITHOUT any error fallbacks that rewrite Makefiles
RUN set -euo pipefail; \
    git clone https://github.com/skyformat99/OpenTimer-1.0.5.git && \
    cd OpenTimer-1.0.5 && \
    autoreconf -fi && \
    CXXFLAGS="-O3 -std=gnu++20 -pthread" ./configure && \
    # strip buggy dist programs so 'make' doesn't build them
    sed -i -E 's/(^|[[:space:]])bin\/OpenTimer-agent([[:space:]]|$)//g' Makefile || true && \
    sed -i -E 's/(^|[[:space:]])ot_dist_master([[:space:]]|$)//g; s/(^|[[:space:]])ot_dist_agent([[:space:]]|$)//g' Makefile main/Makefile 2>/dev/null || true && \
    sed -i '/OpenTimer-agent/d;/ot_dist_master/d;/ot_dist_agent/d' Makefile main/Makefile 2>/dev/null || true && \
    make -j"$(nproc)" && \
    sudo install -m 0755 ./bin/OpenTimer /usr/local/bin/OpenTimer && \
    echo "OpenTimer installed"

# --- Run desktop by default; connect via forwarded port 6080 ---
USER root
CMD ["/usr/bin/supervisord","-n","-c","/etc/supervisor/supervisord.conf"]
