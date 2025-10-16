## VSD-TCL Dev Environment (OpenTimer + Yosys + GUI)

This Codespace provides a ready-to-use synthesis and timing analysis environment built on Ubuntu 22.04.
It includes:

* OpenTimer for static timing analysis
* Yosys for RTL synthesis
* XFCE desktop access through noVNC for GUI use

---

### Option 1 – Using the Terminal

1. Open the **TERMINAL** tab in your Codespace (as shown in image 1).

2. Run OpenTimer:

   ```bash
   OpenTimer
   ```

   This opens the OpenTimer shell. Type `exit` to quit.

3. Run Yosys:

   ```bash
   yosys
   ```

   The Yosys prompt will appear. Type `exit` when done.

This mode is lightweight and recommended for command-line work or quick testing.

---

### Option 2 – Using the GUI (noVNC Desktop)

1. Go to the **PORTS** tab in your Codespace (as shown in image 2).
   You will find a forwarded port labeled **noVNC Desktop (6080)**.
2. Click the forwarded URL to open it in your browser.
3. You will see a directory listing similar to image 3.
   Click **`vnc_lite.html`** to launch the XFCE desktop.
4. Inside the desktop, open a terminal and run:

   ```bash
   OpenTimer
   yosys
   ```

This mode is useful when you need a graphical interface or multiple terminal windows.

---

### Installed Tools

| Tool                 | Command     | Location               | Description            |
| -------------------- | ----------- | ---------------------- | ---------------------- |
| OpenTimer            | `OpenTimer` | `/usr/bin/OpenTimer`   | Static timing analyzer |
| Yosys                | `yosys`     | `/usr/local/bin/yosys` | RTL synthesis tool     |
| TCL/Tk               | `tclsh`     | System default         | For GUI and scripting  |
| XFCE Desktop + noVNC | Port 6080   | Browser UI             | Remote desktop access  |

---

### Environment Details

* Base OS: Ubuntu 22.04 (devcontainers/base)
* Library path: `/usr/lib/libOpenTimer.so.0`
* LD_LIBRARY_PATH is preconfigured
* VNC Server: Xvfb + x11vnc + websockify + supervisor
* Desktop: XFCE4

---

### Verification

Run the following commands to confirm that the tools are installed correctly:

```bash
OpenTimer -h
yosys -V
```

Both commands should display their respective version information.

---

### Notes

* The environment starts automatically when the Codespace is launched.
* The forwarded VNC link remains active as long as the Codespace is running.
* You can switch between terminal and GUI modes at any time.

---

Would you like me to add Markdown image placeholders (for the three screenshots) so participants can visually follow the same flow as your setup?
