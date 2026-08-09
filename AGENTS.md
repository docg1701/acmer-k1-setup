# AGENTS.md

## Project Context

Documentation repository for the **ACMER K1 7W** laser engraver and the
**printbox** print server (Debian + cncjs). Language: **English**. No code —
only hardware specifications, firmware parameters, material table, and server
configuration.

## Stack (facts — do not question)

- **Machine**: ACMER K1 7W, GRBL 1.1h fork ACMER, USB serial `/dev/ttyACM0` 115200, no WiFi
- **Server**: Debian 13 (console) + cncjs 1.11.2 (Node 20), port 8000, hostname `printbox`
- **Webcam**: ustreamer 5.4, port 8080, MJPEG stream consumed by the cncjs widget
- **Network**: server static IP `10.10.10.190/24`, gateway `10.10.10.1`, hostname `printbox` / `printbox.local`
- **User**: `galvani` (server)

## Source of Truth

1. `docs/ACMER-K1-User-Manual-EN.pdf` — power/speed tables
2. Machine firmware (`$$`) — hardware parameters
3. `docs/SPECIFICATIONS-K1.md` — consolidates 1–2

## Absolute Rules

- Firmware `$x=` **never changes**. Factory is correct.
- Power/speed **always** from the 7W table (official ACMER manual).
- **Never** propose hardware swap (WiFi module, board swap) as primary solution.

## Conventions

- Material table: sole source is the official ACMER manual
- Markdown: pipe tables, no HTML
- PDF: `ACMER-K1-User-Manual-EN.pdf` (ASCII)

## Commands

```bash
# Extract text from manual
pdftotext -layout docs/ACMER-K1-User-Manual-EN.pdf /tmp/k1_manual.txt

# cncjs version on server
cncjs --version
```

## Limits

- No build/test/run — documentation repo
- Do not recommend laser control software without verifying recent maintenance
- ACMER Studio runs on Linux via **Bottles + GE-Proton11-3** — follow `docs/ACMER-STUDIO-BOTTLES-MANUAL.md`
- **No plain Wine, LaserGRBL, or Rayforge** — removed and not coming back
