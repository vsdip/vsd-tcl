---

## VSD-TCL Environment (OpenTimer + Yosys)

This Codespace provides a complete synthesis and timing analysis setup on Ubuntu 22.04 with both terminal and GUI access.

---

### Option 1 – Use the Terminal

Open the **TERMINAL** tab and run:

```bash
OpenTimer
```

or

```bash
yosys
```

Exit anytime using `exit`.

![Terminal view](images/1.jpg)

---

### Option 2 – Use the GUI (noVNC Desktop)

1. Open the **PORTS** tab and find the forwarded port named **noVNC Desktop (6080)**.
2. Click the forwarded URL.
3. In the browser, select **`vnc_lite.html`** to open the XFCE desktop.
4. Open a terminal inside the desktop and run:

   ```bash
   OpenTimer
   yosys
   ```

![Ports tab showing noVNC](images/2.jpg)
![VNC page showing vnc\_lite.html](images/3.jpg)

---

### Installed Tools

| Tool      | Command     | Description            |
| --------- | ----------- | ---------------------- |
| OpenTimer | `OpenTimer` | Static timing analyzer |
| Yosys     | `yosys`     | RTL synthesis tool     |

---

### Verify Installation

```bash
OpenTimer -h
yosys -V
```

Both commands should display version information.

---

That’s it — participants can now choose between the terminal or GUI and start using OpenTimer and Yosys immediately.
