# syntax=docker/dockerfile:1
FROM mcr.microsoft.com/devcontainers/base:ubuntu-20.04
SHELL ["/bin/bash","-lc"]
ENV DEBIAN_FRONTEND=noninteractive

# --- Desktop & VNC ---
RUN apt-get update && apt-get install -y --no-install-recommends \
    xfce4 xfce4-terminal dbus-x11 tigervnc-standalone-server \
    novnc websockify supervisor x11-xserver-utils \
    # handy tools
    curl ca-certificates vim less htop \
 && rm -rf /var/lib/apt/lists/*

# --- (Optional) OpenTimer minimal deps (kept small) ---
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential git autoconf automake libtool m4 perl libgoogle-glog-dev \
 && rm -rf /var/lib/apt/lists/*

# Create directories for noVNC + Supervisor
RUN sudo -u vscode mkdir -p /home/vscode/.vnc /home/vscode/Desktop
RUN mkdir -p /etc/supervisor/conf.d

# Startup script: sets VNC password, launches XFCE VNC + noVNC
RUN cat >/usr/local/bin/start-desktop.sh <<'EOS'
#!/usr/bin/env bash
set -euo pipefail

# Defaults
: "${VNC_DISPLAY:=:1}"
: "${VNC_RESOLUTION:=1280x800}"
: "${VNC_DEPTH:=24}"
: "${VNC_PASSWORD:=codespace}"

# Ensure DBus is running for XFCE niceties (not strictly required)
if ! pgrep -x dbus-daemon >/dev/null 2>&1; then
  (dbus-launch >/dev/null 2>&1 || true)
fi

# Create VNC password non-interactively for user 'vscode'
sudo -u vscode bash -lc "mkdir -p ~/.vnc && echo -n '${VNC_PASSWORD}' | vncpasswd -f > ~/.vnc/passwd && chmod 600 ~/.vnc/passwd"

# Start VNC server (as vscode)
sudo -u vscode bash -lc "
  vncserver ${VNC_DISPLAY} -geometry ${VNC_RESOLUTION} -depth ${VNC_DEPTH} -localhost no -xstartup /usr/bin/startxfce4
"

# Map VNC :1 → TCP 5901 for noVNC
VNC_PORT=5901
WEB_PORT=6080

# Start noVNC
websockify --web=/usr/share/novnc/ ${WEB_PORT} localhost:${VNC_PORT}
EOS
RUN chmod +x /usr/local/bin/start-desktop.sh

# Supervisor config to keep services running
RUN cat >/etc/supervisor/conf.d/desktop.conf <<'EOS'
[program:desktop]
command=/usr/local/bin/start-desktop.sh
autorestart=true
priority=10
stdout_logfile=/var/log/desktop.log
stderr_logfile=/var/log/desktop.err
user=root
EOS

# Set a sensible default password (override with CODESPACES secret!)
ENV VNC_PASSWORD=codespace

# --- (Optional) Build OpenTimer here (comment out if you want GUI only) ---
RUN git clone https://github.com/skyformat99/OpenTimer-1.0.5.git /workspaces/OpenTimer-1.0.5 && \
    cd /workspaces/OpenTimer-1.0.5 && \
    autoreconf -fi && \
    CXXFLAGS="-O3 -std=gnu++20 -pthread" ./configure && \
    # prevent dist programs from building (those break on modern compilers)
    sed -i -E 's/(^|[[:space:]])bin\/OpenTimer-agent([[:space:]]|$)//g' Makefile || true && \
    sed -i -E 's/(^|[[:space:]])ot_dist_master([[:space:]]|$)//g; s/(^|[[:space:]])ot_dist_agent([[:space:]]|$)//g' Makefile main/Makefile 2>/dev/null || true && \
    sed -i '/OpenTimer-agent/d;/ot_dist_master/d;/ot_dist_agent/d' Makefile main/Makefile 2>/dev/null || true && \
    make -j"$(nproc)" && \
    install -m 0755 ./bin/OpenTimer /usr/local/bin/OpenTimer && \
    echo "OpenTimer installed"

# Ensure supervisor is PID 1
CMD ["/usr/bin/supervisord","-n","-c","/etc/supervisor/supervisord.conf"]
