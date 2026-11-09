# Webcam Manual — ustreamer + cncjs

> **Purpose**: expose a USB webcam on the print server as an MJPEG stream that the
> cncjs webcam widget consumes, so you can watch the laser while a job runs.
> **Date**: 2026-11-09 · **ustreamer 5.4** (Debian 13 Trixie) · **cncjs 1.11.x**
> **Scope**: Debian mini PC and BTT Pi v1.2 / CB1 (Armbian). Defaults below were
> verified on a BTT Pi v1.2 with a `1908:2310` "USB2.0 PC CAMERA".

---

## 1. Architecture

```text
[USB webcam] ──UVC──> [ustreamer :8080] ──MJPEG──> [cncjs :8000] ──> [browser]
                       server                         server
```

ustreamer reads the camera through `uvcvideo` and re-serves it as MJPEG. cncjs
only points an `<img>` at that URL — it does not talk to the camera itself.

| Component | Port | Role |
|---|---|---|
| ustreamer | 8080 | `/stream` (MJPEG), `/snapshot` (single JPEG), `/state` (JSON) |
| cncjs | 8000 | Web UI, loads the stream in the Webcam widget |

---

## 2. Requirement: USB **high-speed** (480 Mbps) — not full-speed

This is the single most common cause of a camera that "works on another server"
but shows only a low resolution here.

A USB port negotiates **high-speed** (480 Mbps) or falls back to **full-speed**
(12 Mbps, USB 1.1). At 12 Mbps the camera cannot carry large frames, so its
firmware exposes a reduced set of modes — and the driver will only show those.

| Mode (YUYV) | Bandwidth | Fits 12 Mbps? | Fits 480 Mbps? |
|---|---|---|---|
| 640x480 @ 30 | ~147 Mbps | no | yes |
| 640x480 @ 15 | ~74 Mbps | no | yes |
| 320x240 @ 30 | ~37 Mbps | no | yes |
| 320x240 @ 4 | ~4.9 Mbps | yes | yes |

**Check the negotiated speed** — this is the first thing to look at:

```bash
# 480 = high-speed (correct) | 12 = full-speed (the problem)
cat /sys/bus/usb/devices/*/speed

# must say "new high-speed USB device"
sudo dmesg | grep -iE "new (high|full)-speed"
```

If it says `12`, the camera (or its cable) is not reaching high-speed. What it
looks like on the BTT Pi v1.2 — the four USB-A ports map to EHCI (480M) with an
OHCI companion (12M), and the same camera lands on either one depending on the
port and cable:

```text
Bus 001..004  ehci-platform/1p  480M   <- high-speed attach lands here
Bus 005..008  ohci-platform/1p   12M   <- full-speed fallback lands here
```

Proven on the reference setup: the identical camera exposed
`320x240@4 / 160x120@15` on an OHCI port (`speed=12`, isoc endpoint
`wMaxPacketSize 768 B/ms`) and `640x480@30 / 352x288 / 320x240 / 176x144 /
160x120` on an EHCI port (`speed=480`).

Actions, in order, until `speed` reads `480`:

1. **Change the USB cable.** A cable with the high-speed pairs damaged or missing
   always falls back to full-speed. Typical kernel signature:
   `device descriptor read/64, error -62` followed by
   `not running at top speed; connect to a high speed hub`.
2. **Change the USB-A port.** Move the camera to a different port on the board.
3. Re-read `speed`. Only then continue to §3.

> A `USBDEVFS_RESET` or `unbind`/`bind` does **not** fix this — it re-enumerates
> the device on the *same* link and comes back at the same speed. Velocity is
> negotiated electrically, so the fix is physical.

---

## 3. Install

```bash
sudo apt install -y ustreamer v4l-utils
ustreamer --version    # 5.4
```

---

## 4. Identify the camera node

**`/dev/video0` is usually wrong on the BTT Pi / CB1.** Video nodes are numbered
by probe order, and the H616's hardware video decoder (`cedrus`) often takes
video0:

```bash
for d in /sys/class/video4linux/*; do echo "$d -> $(cat $d/name)"; done
# /sys/class/video4linux/video0 -> cedrus                      <- decoder, NOT the camera
# /sys/class/video4linux/video1 -> USB2.0 PC CAMERA: ...       <- the camera
```

Use `v4l2-ctl` to map nodes to drivers and check what each one really is:

```bash
v4l2-ctl --list-devices
# cedrus (platform:cedrus):
#         /dev/video0
#         /dev/media0
# USB2.0 PC CAMERA: USB2.0 PC CAM (usb-5311000.usb-1):
#         /dev/video1
#         /dev/video2
#         /dev/media1
```

A UVC camera exposes **two** video nodes: one real capture node and one
metadata node. Only the capture node lists formats:

```bash
v4l2-ctl -d /dev/video1 --info | grep -A3 "Device Caps"   # must say Video Capture
v4l2-ctl -d /dev/video1 --list-formats-ext               # must list modes
```

| Node | Driver | `--list-formats-ext` | Use? |
|---|---|---|---|
| `/dev/video0` | `cedrus` | decoder formats (`NV12`, `YU12`, …) | no |
| `/dev/video1` | `uvcvideo` | camera modes (`640x480`, `320x240`, …) | **yes** |
| `/dev/video2` | `uvcvideo` | empty (metadata node) | no |

Point `--device=` at the node that actually lists camera modes.

---

## 5. systemd service

The heredoc delimiter is **unquoted** (`<<EOF`, not `<<'EOF'`) so `$(id -un)`
substitutes your real login when the file is written — no manual editing:

```bash
sudo tee /etc/systemd/system/ustreamer.service > /dev/null <<EOF
[Unit]
Description=uStreamer MJPEG webcam
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=$(id -un)
Group=$(id -un)
ExecStart=/usr/bin/ustreamer --host=0.0.0.0 --port=8080 --device=/dev/video1 -r 640x480 --device-timeout 10
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable --now ustreamer.service
```

Flag by flag:

| Flag | Why |
|---|---|
| `--host=0.0.0.0` | listen on the LAN, not just loopback (default is `127.0.0.1`) |
| `--port=8080` | port the cncjs widget will point at |
| `--device=/dev/video1` | the UVC capture node from §4, not `video0` |
| `-r 640x480` | requested resolution; falls back gracefully — see §7 |
| `--device-timeout 10` | **do not omit** — see below |

> **`--device-timeout 1` is the default and is too short for many cheap
> cameras.** The camera can take longer than one second to deliver its first
> frame; ustreamer then abandons it and reopens in a loop, logging
> `Mainloop select() error: Success`, while `/state` reports `"online": false`
> and clients get the black `NO SIGNAL` placeholder. Raising the timeout to 10 s
> fixes it. This failure mode looks exactly like a broken camera and is not one.

---

## 6. Verify

```bash
systemctl is-active ustreamer        # active
curl -s http://127.0.0.1:8080/state  # see below
```

Read `/state` — `online` and `captured_fps` are the two fields that matter:

```json
{"ok": true, "result": {
  "encoder": {"type": "CPU", "quality": 80},
  "source": {"resolution": {"width": 640, "height": 480},
             "online": true, "desired_fps": 0, "captured_fps": 10},
  "stream": {"queued_fps": 0, "clients": 0, "clients_stat": {} }}}
```

| Field | Expected | Meaning when wrong |
|---|---|---|
| `source.online` | `true` | `false` → device not delivering; check `--device`, permissions, `--device-timeout` |
| `source.captured_fps` | `> 0` | `0` → opens the device but gets no frames |
| `source.resolution` | camera's real mode | a resolution the camera does not list means you are reading the placeholder |

Endpoints:

```bash
curl -s http://127.0.0.1:8080/snapshot -o /tmp/snap.jpg && file /tmp/snap.jpg
curl -sI http://127.0.0.1:8080/stream   # content-type: multipart/x-mixed-replace
```

> **`NO SIGNAL` on a black frame is ustreamer's own placeholder**, not the
> camera's image and not a cncjs bug. It is drawn when the source is offline, and
> it is generated at ustreamer's *placeholder* resolution — which can differ from
> the camera's. Always confirm with `/state` before blaming the hardware. If
> `online` is `false`, nothing you change on the cncjs side will help.

The user running the service must be able to open the device. The
`video` group owns `/dev/video*`; add the user if needed and restart:

```bash
groups | grep video || sudo usermod -aG video "$(id -un)"
```

---

## 7. Resolution negotiation

`-r` is a **request**. ustreamer asks for it and falls back to the nearest mode
the camera advertises, logging both facts:

```text
ERROR [stream] -- Requested resolution=640x480 is unavailable
INFO  [stream] -- Using resolution: 320x240
```

So the resolution you see can be lower than the one you configured. Check what
the camera actually offers before assuming a software bug:

```bash
v4l2-ctl -d /dev/video1 --list-formats-ext
```

If only small modes are listed, go back to §2 — it is the USB link, not
ustreamer. Leaving `-r` at `640x480` is safe: once the link is high-speed it
picks the full mode with no config change.

`-f` / `--desired-fps` is likewise a request; the log shows the negotiated value
as `Using HW FPS: 0 -> 30 (coerced)`.

---

## 8. cncjs widget

The widget does **not** use the stream by default. Its factory settings are:

```js
webcam: { disabled: true, mediaSource: "local", deviceId: "", url: "" }
```

`mediaSource: "local"` makes it grab the **browser's** own camera through
`getUserMedia`, not ustreamer. With no device selected it renders a black
`<video>` — indistinguishable from a broken stream, but it is not one.
`disabled: true` means the widget is not even shown until you enable it.

Three steps in the cncjs UI (`http://<server-ip>:8000`):

1. **Enable the widget** — Settings → **Widgets** → tick **Webcam Widget**.
2. **Select the media source** — in the Webcam widget, open its settings:
   - "Use a built-in camera or a connected webcam" → **wrong** (the default)
   - **"Connect to an IP camera"** → the one you want
3. **Set the URL** and **Save Changes**. Use the native endpoint:

```text
http://10.10.10.127:8080/stream
```

| Endpoint | Response |
|---|---|
| `/stream` | native ustreamer MJPEG (`multipart/x-mixed-replace`) — **use this** |
| `/?action=stream` | mjpg-streamer compatibility alias; answers, but not needed |
| `/snapshot` or `/?action=snapshot` | single JPEG (`image/jpeg`) |

Replace `10.10.10.127` with the server's address. Use the **same** address you
used to open cncjs — if the page is at `10.10.10.127:8000`, the widget must use
`10.10.10.127:8080`. Never `localhost` or `0.0.0.0`: those resolve to the machine
running the browser, not the server. The `http://0.0.0.0:8080/?action=stream`
shown as a placeholder in the dialog is only an example.

**Verify without the UI** — open the URL in a browser tab, or:

```bash
curl -sI http://<server-ip>:8080/stream      # 200, multipart/x-mixed-replace
curl -s  http://<server-ip>:8080/snapshot -o /tmp/snap.jpg
```

Server-side confirmation that a browser is actually pulling frames:

```bash
curl -s http://127.0.0.1:8080/state | grep -o '"clients": [0-9]*'
```

`"clients": 0` means no browser is consuming the stream — the problem is in the
widget URL, not in ustreamer. With the widget working it reports `1` or more,
plus a per-client `fps`.

> Restarting cncjs changes none of this: `disabled`, `mediaSource` and `url` are
> widget settings, not service settings.

---

## 9. Troubleshooting

| Symptom | Cause | Solution |
|---|---|---|
| Widget black, `/state` `"clients": 0` | widget still on `mediaSource: "local"` (browser camera) or URL wrong | select **"Connect to an IP camera"** and use a URL from §8; `localhost` / `0.0.0.0` never work |
| Widget black, `/state` `"online": false` | device not delivering frames | check `--device`; raise `--device-timeout` (default 1 s is too short) |
| Widget shows `NO SIGNAL` | ustreamer placeholder — source offline | fix `source.online` first; it is not a cncjs problem |
| Widget black, `/state` `online: true` but `captured_fps: 0` | device opens, no data | wrong node (§4) or full-speed link (§2) |
| Only low resolutions available (`320x240`, `160x120`) | USB negotiated full-speed | `cat /sys/bus/usb/devices/*/speed` → must be `480`; change cable, then port |
| `Can't set input channel` in the log | camera was left in a bad state by a previous reader | stop the other reader / restart the service |
| `Mainloop select() error: Success` repeating every ~1 s | `--device-timeout` too short | set `--device-timeout 10` |
| Stream works, image out of focus | lens not adjusted | turn the lens ring on the camera barrel; the sensor is fixed-focus only on some models |
| Works locally, not from the phone | AP isolation on the router, or wrong IP in the URL | disable client isolation; use the server's real LAN IP |
| Widget loads nothing after a reboot | service not enabled | `systemctl is-enabled ustreamer` |

---

## 10. Notes

- Do **not** expose port 8080 to the internet (same rule as cncjs port 8000 —
  see `K1-SERVER-MANUAL.md` §10).
- ustreamer runs fine as the normal user; no root needed once the user is in the
  `video` group.
- Update: `sudo apt update && sudo apt install --only-upgrade ustreamer`, then
  `sudo systemctl restart ustreamer`.
- Logs: `journalctl -u ustreamer -f`.
