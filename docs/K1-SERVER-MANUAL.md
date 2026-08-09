# K1 Server Manual — Debian + cncjs

> **Architecture**: you design and generate the job in LaserGRBL or ACMER Studio
> (your PC, via Wine), send the G-code file to cncjs running on the mini PC
> (server), and the server runs the K1 directly via USB. Your PC can be turned
> off — the server handles everything, and you monitor via browser (PC or
> phone).
>
> **Date**: 2026-08-08 · **cncjs v1.11.x** (verified: active releases in 2026)

```text
[Your PC]  Wine + LaserGRBL or ACMER Studio — design, power, export G-code
    │
    │  WiFi (local network) — only file upload
    ▼
[mini PC]  Debian + cncjs — receives the file, streams via USB
    │  USB serial 115200
    ▼
[K1]  GRBL — executes the job
```

**Golden rule**: all configuration (power, speed, passes, order) happens in the
design software (LaserGRBL or ACMER Studio), when generating the G-code. cncjs
has no material/power configuration — it only sends the file to the machine,
byte by byte. Nothing is repeated on the server.

---

## 1. Installing Debian on the mini PC

1. Download **Debian netinst amd64**: <https://www.debian.org/distrib/>
2. Flash to a USB stick (e.g.: `dd if=debian.iso of=/dev/sdX bs=4M status=progress`)
3. Boot from the USB and install:
   - **Hostname**: `printbox`
   - **User**: `galvani` + password
   - **Network**: during installation, connect to **WiFi** (the installer configures it)
   - **Partitioning**: guided, entire disk (the disk has nothing important)
   - **Software selection**: check **SSH server** + **standard system utilities**
     — **no desktop, no GNOME/KDE** (console only; nobody will use a monitor)
4. Reboot. Test with `ip a` — the WiFi IP should appear.

---

## 2. Post-installation (once)

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Basic tools
sudo apt install -y curl iw git

# User in dialout group (access to K1 serial port)
sudo usermod -aG dialout $USER
# (log out/in after — or reboot)
```

### 2.1. Disable WiFi Power Save (mandatory)

Power save causes **latency spikes** on WiFi — and that's exactly what can
freeze a job mid-way (the machine stops with the laser on). Turn it off:

```bash
# Check the interface (ip a — usually wlp1s0 or wlan0)
ip a

# Turn off now:
sudo iw dev wlp1s0 set power_save off

# Confirm:
iw dev wlp1s0 get power_save
```

To persist after reboot, create the service:

```bash
sudo tee /etc/systemd/system/wifi-powersave-off.service > /dev/null <<'EOF'
[Unit]
Description=Disable WiFi power save
After=network-online.target
Wants=network-online.target

[Service]
Type=oneshot
ExecStart=/usr/sbin/iw dev wlp1s0 set power_save off

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl enable --now wifi-powersave-off.service
```

### 2.2. Static IP

Configure directly on Debian (without relying on router DHCP reservation).
Edit `/etc/network/interfaces`:

```bash
sudo nano /etc/network/interfaces
```

Replace contents with:

```text
auto lo
iface lo inet loopback

auto wlp1s0
iface wlp1s0 inet static
    address 10.10.10.190/24
    gateway 10.10.10.1
    wpa-ssid NETWORK-NAME
    wpa-psk NETWORK-PASSWORD
```

> Replace `NETWORK-NAME` and `NETWORK-PASSWORD` with your WiFi credentials.
> If the WiFi password has special characters, use quotes: `wpa-psk "password"`.

Apply:

```bash
sudo systemctl restart networking
# or reboot
```

Confirm:

```bash
ip a show wlp1s0   # should show 10.10.10.190
ping 10.10.10.1   # should respond
```

Note down `10.10.10.190` — this IP goes in the browser, always.

### 2.3. SSH Test (optional, but recommended)

The mini PC has no monitor — manage it via SSH from your PC:

```bash
ssh galvani@10.10.10.190
```

---

## 3. Installing cncjs

```bash
# Node.js + npm (Debian 13 ships Node 20.x — cncjs needs ≥12)
sudo apt install -y nodejs npm

# cncjs (global install; --unsafe-perm is required with sudo)
sudo npm install --unsafe-perm -g cncjs@latest

# Confirm version (should show 1.11.x)
cncjs --version
```

### 3.1. Minimal Configuration (`~/.cncrc`)

```bash
mkdir -p ~/gcode
cat > ~/.cncrc <<'EOF'
{
  "controller": "Grbl",
  "baudrates": [115200, 250000],
  "watchDirectory": "/home/galvani/gcode"
}
EOF
```

- `controller: Grbl` — the K1 is GRBL
- `baudrates: 115200` — K1 baud rate
- `watchDirectory` — monitored folder: files dropped there appear in the web UI
- cncjs automatically adds `state` and `secret` when saving preferences via the web UI

### 3.2. systemd Service (starts automatically on boot)

```bash
sudo tee /etc/systemd/system/cncjs.service > /dev/null <<'EOF'
[Unit]
Description=cncjs web controller
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=galvani
ExecStart=/usr/local/bin/cncjs -H 0.0.0.0 -p 8000 --controller Grbl
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl enable --now cncjs.service

# Verify it's running:
ss -ltn | grep 8000
```

> If `which cncjs` shows a different path, adjust the `ExecStart`.

---

## 4. USB Connection with the K1

1. **Plug the K1 USB cable into the mini PC** and turn on the machine.
2. Find the device:

   ```bash
   ls /dev/ttyACM* /dev/ttyUSB*
   ```

   → in practice, it will be `/dev/ttyACM0` (same device your PC uses today).
3. Confirm it's the K1:

   ```bash
   lsusb
   ```

   (look for the board — CH340 appears as `1a86:7523`, WCH/ACM as CDC)

**If the device changes** (K1 re-plugged may become `ttyACM1`): restart cncjs:

```bash
sudo systemctl restart cncjs
```

The device is `/dev/ttyACM0` — use that name in cncjs.

---

## 5. cncjs Configuration (web UI)

1. In your PC browser: **`http://10.10.10.190:8000`**
2. **Connect** (top corner):
   - **Controller**: `Grbl`
   - **Serial port**: `/dev/ttyACM0`
   - **Baud rate**: `115200`
   - **Connect**
3. The GRBL status should appear (Idle, position, firmware version).
4. **Sanity test**: use the jog panel (arrows) — the K1 should move.
5. **DO NOT touch** the override sliders (feedrate/laser) — the job uses the
   file values; override only if you want to change on the fly, consciously.

---

## 6. Day-to-Day Workflow

### 6.1. In LaserGRBL / ACMER Studio (your PC, via Wine) — generate the job

1. Design and configure everything (power, speed, passes)
2. **Export G-code** → save e.g.: `placard.nc`
   - The file contains EVERYTHING: power, speed, passes, order

### 6.2. Send to the server

**Method 1 — browser**: cncjs → G-code tab → **Upload** → select `placard.nc`

**Method 2 — monitored folder**: copy the file to `~/gcode` on the mini PC
(`scp placard.nc galvani@10.10.10.190:/home/galvani/gcode/`) — it appears in the web UI
by itself.

### 6.3. Run

1. Select the file in the list → **Load**
2. Check the visualizer (toolpath) and machine position
3. **Run** → the server streams the job via USB
4. **Turn off the PC** — the job continues; monitor from your phone at
   `http://10.10.10.190:8000`
5. Buttons available during the job: **Pause**, **Resume**, **Cancel**

---

## 7. Recommended Tests in the First Week

1. **Connection test**: connect, read status, jog in all directions
2. **Small job**: a paper placard (already validated) via cncjs
3. **"PC off" test**: start a job, turn off the laptop, confirm the job finishes
   and the server shows 100%
4. **Pause/cancel test**: pause midway, resume, then cancel — the K1 should
   respond correctly

---

## 8. Maintenance

```bash
# Update cncjs
sudo npm install --unsafe-perm -g cncjs@latest
sudo systemctl restart cncjs

# View logs
journalctl -u cncjs -f

# Reboot everything (mini PC)
sudo reboot
```

After reboot, WiFi power save off and cncjs start automatically (systemd).
Verify with `iw dev wlp1s0 get power_save` and `ss -ltn | grep 8000`.

---

## 9. Troubleshooting

| Symptom | Cause | Solution |
|---|---|---|
| Web UI won't open | cncjs not running / wrong IP | `ss -ltn \| grep 8000`; `ip a`; restart the service |
| "No serial port found" | user not in dialout / USB loose | `sudo usermod -aG dialout $USER` + re-login; check `ls /dev/ttyACM*` |
| Port changed after re-plugging K1 | USB re-enumeration | `sudo systemctl restart cncjs` (or udev rule, section 4) |
| Job freezes mid-way / machine stops with laser on | WiFi power save active | `iw dev wlp1s0 get power_save` → should say `off` |
| "Port in use" when connecting | another program opened the serial | nothing else can use the port; restart cncjs |
| Phone can't access | AP isolation on router | disable "client isolation" on WiFi |
| Job starts but runs slow/with gaps | Congested 2.4GHz WiFi | prefer 5GHz on the mini PC (if supported) |
| K1 stalls at `<Home>` after jog | homing stall after manual movement | **Unlock** button in cncjs, or `$X` in MDI console, or power-cycle |

---

## 10. Security (one rule only)

**Do not open port 8000 to the internet** (no port forwarding on the router).
cncjs is for use on your local network — and on the LAN, without a password,
is the normal usage pattern for this tool. If you ever need external access,
set up a VPN — do not expose the port.

---

## 11. Webcam (optional)

cncjs has a webcam widget that consumes an external MJPEG stream.
Use `ustreamer` to expose the server's USB webcam.

```bash
# Install
sudo apt install -y ustreamer

# systemd service
sudo tee /etc/systemd/system/ustreamer.service > /dev/null <<'EOF'
[Unit]
Description=uStreamer MJPEG webcam
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=galvani
ExecStart=/usr/bin/ustreamer --host=0.0.0.0 --port=8080 --device=/dev/video0 -r 1280x720 -f 15
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable --now ustreamer.service
```

In cncjs, enable the **Webcam Widget** and configure the URL:
`http://10.10.10.190:8080/stream`.

---

## Architecture Summary (what runs where)

| Component | Where | Role |
|---|---|---|
| LaserGRBL / ACMER Studio | Your PC (Wine) | Design, power, **export G-code** |
| cncjs | mini PC (Debian) | Web UI, receives file, **streams via USB** |
| K1 | — | Executes (GRBL) |
| ustreamer | mini PC (Debian) | Webcam MJPEG stream (port 8080), consumed by cncjs widget |
| WiFi | between PC and mini PC | Only file upload (job runs locally on server) |
