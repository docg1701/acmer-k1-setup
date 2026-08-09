# ACMER K1 7W — Documentation

Documentation repository for the ACMER K1 7W laser engraver: hardware specs, firmware parameters, material table, and a complete guide to running ACMER Studio on Linux + cncjs print server.

## Why This Setup

A laser engraver is a hazardous machine: it produces toxic fumes, poses a fire risk, and the laser beam itself is a danger to eyesight. Running it on the same desk where you do your design work is a bad idea.

This repo documents how to:

- Keep the **laser in a separate, ventilated area** (garage, workshop, enclosed space)
- Run a **dedicated print server** next to the machine (mini PC + cncjs)
- Do all your **design work on your main PC** (ACMER Studio via Bottles on Linux)
- **Send jobs over the network** — no need to be near the machine while it runs
- **Monitor remotely** from any browser (PC or phone)

The result: the laser stays in a safe environment, and you control it from your desk without health risks.

## Screenshots

![ACMER Studio running via Bottles](assets/acmer-studio.jpg)
*ACMER Studio running on Linux via Bottles + GE-Proton*

![Bottles — ACMER Studio bottle configuration](assets/bottles.jpg)
*Bottle configuration with GE-Proton11-3 runner*

![cncjs web interface](assets/cncjs.jpg)
*cncjs web interface — job running with live toolpath*

## Quick Start

**1. Use ACMER Studio on Linux**

Follow [ACMER-STUDIO-BOTTLES-MANUAL.md](docs/ACMER-STUDIO-BOTTLES-MANUAL.md) to install Bottles, set up the GE-Proton11-3 runner, fix keyboard dialogs, and get the tool DLLs loading.

**2. Set Up the Print Server**

Follow [K1-SERVER-MANUAL.md](docs/K1-SERVER-MANUAL.md) to install Debian on a mini PC, configure cncjs, set a static IP, and stream jobs via USB.

**3. Material Settings**

Check [SPECIFICATIONS-K1.md](docs/SPECIFICATIONS-K1.md) for the official 7W power/speed table, plus community-tested values for PLA, ABS, PETG, and corrugated cardboard.

## Architecture Overview

```
[Your PC]       ACMER Studio on Linux — design, power settings, export G-code
     │
     │  WiFi (local network) — file upload only
     ▼
[Print Server]  Debian + cncjs — receives file, streams via USB
     │  USB serial 115200
     ▼
[K1 Laser]      GRBL — executes the job (in a separate, ventilated area)
```

You design at your desk, upload the job, and walk away. The server runs the machine headless. Monitor the job from your phone or any browser — you never need to be in the same room as the laser.

## Webcam

The print server supports a live webcam feed via **ustreamer** (MJPEG stream on port 8080). The cncjs web interface has a webcam widget that consumes this stream — so you can watch the machine running from anywhere on your network.

Setup is covered in the [K1-SERVER-MANUAL.md](docs/K1-SERVER-MANUAL.md) section 11.

## Documents

| File | Content |
|---|---|
| [SPECIFICATIONS-K1.md](docs/SPECIFICATIONS-K1.md) | Hardware, `$$` parameters, 7W material table |
| [K1-SERVER-MANUAL.md](docs/K1-SERVER-MANUAL.md) | Printbox server (Debian + cncjs + webcam) |
| [ACMER-STUDIO-BOTTLES-MANUAL.md](docs/ACMER-STUDIO-BOTTLES-MANUAL.md) | ACMER Studio on Linux (Bottles + GE-Proton) |
| [ACMER-K1-User-Manual-EN.pdf](docs/ACMER-K1-User-Manual-EN.pdf) | Official ACMER manual (primary source) |

## License

[MIT](LICENSE)
