# ACMER K1 7W — Documentation

Documentation repository for the **ACMER K1 7W** laser engraver: hardware
specifications, firmware parameters (`$$`), material table, and **printbox**
print server configuration (Debian + cncjs + webcam).

## Documents

| File | Content |
|---|---|
| `docs/SPECIFICATIONS-K1.md` | Hardware, `$$` parameters, 7W material table |
| `docs/K1-SERVER-MANUAL.md` | Printbox server (Debian + cncjs + webcam) |
| `docs/ACMER-STUDIO-BOTTLES-MANUAL.md` | ACMER Studio on Linux (Bottles + GE-Proton) |
| `docs/ACMER-K1-User-Manual-EN.pdf` | Official ACMER manual — primary source |

## Screenshots

![ACMER Studio running via Bottles](assets/acmer-studio.jpg)

![Bottles — ACMER Studio bottle configuration](assets/bottles.jpg)

![cncjs web interface](assets/cncjs.jpg)

## Immutable Rules

- K1 firmware (`$x=`) **must not be changed** — factory values are correct
- Power/speed **always from the official 7W table** (ACMER manual)
- **Never** propose hardware swaps (WiFi module, board swap) as a primary solution

## License

Documentation — free use. Official ACMER PDF: copyright ACMER.
