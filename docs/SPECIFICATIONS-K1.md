# ACMER K1 7W — Hardware, Firmware, and Material Specifications

> **Hardware**: official ACMER manual + community docs.
> **Parameters `$$`**: firmware dump from the machine (2026-08-08), validated field by field.
> **Materials**: `ACMER K1-User Manual（English）.pdf`, section 7 — **7W Compressed Spot**.

---

## 1. Hardware

| Specification | Value |
|---|---|
| Laser type | Blue diode 455 nm (±5 nm), beam-combining |
| Power | 7W |
| Work area | 150 × 150 mm |
| Spot size | 0.08 × 0.08 mm (advertised); 0.06 mm measured |
| Repeatability | 0.01 mm |
| Max engraving speed | 10,000 mm/min |
| Dimensions | 286 × 273 × 190 mm |
| Weight | 2.4 kg |
| Safety class | Class 1 (enclosed — no goggles needed with lid closed) |
| Connectivity | USB / GRBL compatible |
| Baud rate | 115200 |
| Serial device | `/dev/ttyACM0` |
| Noise | ~59–61 dB |
| Firmware | Grbl 1.1h (ACMER fork) |

### Safety

- **Lid sensor**: opening pauses the job in ~1s; closing resumes
- **Tilt sensor**: if tipped over — shuts off before causing harm
- **Built-in exhaust fan**: reduces smoke/odor
- **Air assist**: not factory-installed; side hole in the housing for a hose

### Focusing

1. Place the measuring sheet over the material
2. Release the side button
3. Slide the laser module down until the nozzle touches the sheet
4. Tighten the button

Magnetic protective nozzle (easy to remove for cleaning).

---

## 2. GRBL Parameters (dump `$$`)

> Factory values — **do not change**. Reference for configuring any software
> that communicates with the K1 firmware.

| Parameter | Value | Description |
|---|---|---|
| `$20` | 0 | Soft limits OFF |
| `$21` | 1 | Hard limits ON |
| `$22` | 1 | Homing cycle enable |
| `$23` | 3 | Homing direction invert (origin: bottom-left corner) |
| `$30` | 1000 | S-value max (PWM) |
| `$31` | 0 | S-value min (PWM) |
| `$32` | 1 | Laser mode ON |
| `$100` | 80 | Steps/mm — X axis (1 step = 0.0125 mm) |
| `$101` | 80 | Steps/mm — Y axis (1 step = 0.0125 mm) |
| `$110` | 10000 | Max rate — X axis (mm/min) |
| `$111` | 10000 | Max rate — Y axis (mm/min) |
| `$120` | 300 | Acceleration — X axis (mm/s²) |
| `$121` | 300 | Acceleration — Y axis (mm/s²) |
| `$130` | 150 | Max travel — X axis (mm) |
| `$131` | 150 | Max travel — Y axis (mm) |

### Firmware Notes

- **Grbl 1.1h (ACMER fork)**: supports G5 bezier and extra parameters `$41–$48`
- **Recommended G-code precision**: 3 decimal places (12× finer than machine step)
- **Arcs**: G2/G3 natively supported
- **Limits**: hard limits ON protects against crash at end of travel
- **Homing**: on power-up, the machine homes automatically; if something is in the path, it may crash — perform manual homing with the lid open and area clear (`$H` in console)
- **`$3` already handles axis direction inversion** — do not use Reverse in software to compensate

---

## 3. Material Parameter Table (7W Compressed Spot)

### Cutting (M3)

| Material | Power | Speed (mm/min) | Passes |
|---|---|---|---|
| Kraft paper 0.5mm | 100% | 1500 | 1 |
| Kraft paper 1.0mm | 100% | 1000 | 1 |
| Kraft paper 2.0mm | 100% | 300 | 1 |
| Plywood 2.0mm | 100% | 200 | 1 |
| Solid wood 2.0mm | 95% | 200 | 2 |
| Bamboo 2.0mm | 95% | 150 | 1 |
| Red Acrylic 1.0mm | 100% | 150 | 1 |
| Red Acrylic 2.0mm | 100% | 100 | 1 |
| Black Acrylic 1mm | 100% | 150 | 1 |
| Black Acrylic 2mm | 100% | 100 | 1 |
| Light-colored Felt 1mm | 80% | 1500 | 1 |

### Engraving (M4, 10 lines/mm)

| Material | Power | Speed (mm/min) | Passes |
|---|---|---|---|
| Kraft paper | 50% | 8000 | 1 |
| Plywood | 50% | 6000 | 1 |
| Solid wood | 80% | 3500 | 1 |
| Bamboo | 80% | 4500 | 1 |
| Cork | 60% | 5000 | 1 |
| Leather | 50% | 8000 | 1 |
| Light-colored Felt | 50% | 8000 | 1 |
| Dark Felt | 50% | 8000 | 1 |
| Transparent Acrylic* | 70% | 6000 | 1 |
| Glass* | 100% | 2000 | 1 |
| Silica gel | 50% | 2000 | 1 |
| Cobblestone | 90% | 80 | 1 |
| Ceramics | 100% | 2000 | 1 |
| Black alumina | 80% | 7000 | 1 |
| Tin plate | 90% | 3000 | 1 |
| Stainless steel (matte) | 90% | 200 | 1 |
| Stainless steel (smooth) | 90% | 150 | 2 |

\* Requires prior black paint on the surface.

---

## 4. Corrugated Cardboard (box) — 4mm

> **Not in the official ACMER table.** Values below = extrapolation from the
> 7W table (kraft paper 2mm solid: 100% @ 300 mm/min) + community data from
> machines with ~5.5–7W blue diode (Ortur). Corrugated has 2 thin liner layers
> + hollow core → **less fiber than solid paper of the same thickness** →
> cuts faster than 2mm kraft.
>
> ⚠️ Validate with a test grid before a real job. If it doesn't cut through:
> **reduce speed or do 2 passes** — never exceed 100% power and beware of fire
> (paper/cellulose ignites easily at low speed).

### ACMER Studio Setup — custom material "Corrugated Cardboard 4mm"

| Operation | Power% | Speed (mm/min) | Passes | Laser |
|---|---|---|---|---|
| **Line** | 35 | 5000 | 1 | M4 |
| **Fill** | 50 | 8000 | 1 | M4, 10 lines/mm |
| **Cut** | 100 | 450 | 1 | M3 |

Cut test range: **350–600 mm/min** (450 = midpoint between solid kraft 2mm @ 300
and Ortur corrugated 3mm @ 500).

### Community References (blue diode, corrugated)

| Source | Laser | Material | Operation | Speed (mm/min) | Power | Passes |
|---|---|---|---|---|---|---|
| Ortur official | 5.5W | Corrugated 3mm | Cut | 500 | 100% | 1 |
| ZapCraft | 5W | 1–3mm | Cut | 330 | 70% | 2 |
| ZapCraft | 10W | 1–3mm | Cut | 600 | 70% | 1 |
| Atomstack A20 | 20W | Cardboard | Cut | 800 | 50% | 1 |

Sources: <https://ortur.net/pages/materials-reference> ·
<https://zapcraft.net/material-settings/cardboard-laser-settings-starting-points/> ·
<https://www.bonnycreations.com/settings/materials/corrugated-cardboard>

---

## 5. Kraft Paper — Outline vs Fill

| Operation | Command | Power | Speed (mm/min) |
|---|---|---|---|
| Cut (outline) | M3 | 100% | 1500 |
| Engrave (fill) | M4 | 50% | 8000 |
| Outline marking (no cut) | M4 | 50% | 8000 |

---

## 6. 3D Printed Thermoplastics (PLA, ABS, PETG)

> **Not in the official ACMER table.** Values = extrapolation from the 7W table
> (black/red acrylic — closest thermoplastic: 1mm @ 150, 2mm @ 100 mm/min) +
> community data (xTool M1/F1, Batch Studio, ComMarker). ⚠️ **Validate with a
> test grid** before a real job.
>
> ⚠️ **Toxic fumes**: ABS releases styrene (and cyanide under thermal
> degradation); PLA and PETG release VOCs. **Exhaust mandatory** — the K1
> enclosure contains smoke but **does not filter it**. ABS: only with strong
> exhaust.

### 6.1. General Rules (why dark color)

- Blue diode 455nm: **only dark colors absorb**. Test xTool F1 (blue diode 10W):
  white PLA = no effect; gray PLA = uneven melting; **black/black/red = clean
  engrave**.
- **PLA**: low melting point (150–180°C) — high power melts and bubbles instead
  of engraving. Always high speed + low power.
- **Printed part**: print with the face to engrave on the bottom and **≥6 solid
  layers** on it (otherwise the laser cuts through the infill). Layer lines make
  the engraving depth uneven — shallow engraving is cleaner.

### 2.2. Dark PLA — ACMER Studio

| Operation | Power% | Speed (mm/min) | Passes | Laser |
|---|---|---|---|---|
| **Line** | 35 | 2000 | 1 | M4 |
| **Fill** | 30 | 4000 | 1 | M4, 10 lines/mm |
| **Cut 1mm** | 100 | 200 | 1 | M3 |
| **Cut 2mm** | 100 | 120 | 1–2 | M3 |
| **Cut 3mm** | 100 | 80 | 2 | M3 |

Fill test: 2000–6000 mm/min (if melted → increase speed). Cut: if it doesn't
go through, reduce speed — never above 100%. **4–5mm: unrealistic to cut with
7W** (too many passes, melted edge) — engraving is fine, cutting is not.

### 6.3. Dark ABS — ACMER Studio

| Operation | Power% | Speed (mm/min) | Passes | Laser |
|---|---|---|---|---|
| **Line** | 30 | 2000 | 1 | M4 |
| **Fill** | 25 | 3000 | 1 | M4, 10 lines/mm |
| **Cut 1mm** | 100 | 150 | 1 | M3 |
| **Cut 2mm** | 100 | 80 | 1–2 | M3 |

> **ABS melts before burning** (community consensus: Bambu Lab forum, Batch
> Studio) — cutting above 2mm turns into goo. Prefer engraving; cut only thin
> pieces. Fumes: styrene — strong exhaust required.

### 6.4. PETG — ACMER Studio

| Operation | Power% | Speed (mm/min) | Passes | Laser |
|---|---|---|---|---|
| **Line** | 30 | 2000 | 1 | M4 |
| **Fill** | 25 | 4000 | 1 | M4, 10 lines/mm |
| **Cut 1mm** | 100 | 150 | 1 | M3 |

> PETG absorbs 455nm poorly and melts forming burrs — cut only thin (1mm).
> Fumes: VOCs.

### 6.5. Sources

- <https://www.batchmade.studio/laser-engrave-3d-prints> (xTool F1: PLA/ABS
  colors)
- <https://blog.commarker.com/archives/57631> (PLA: melting 150–180°C,
  high speed + low power)
- <https://xtool.zendesk.com/hc/en-us/articles/15017421813911> (acrylics,
  thermoplastic reference)
- <https://forum.bambulab.com/t/laser-engraving-3d-printed-parts/74296>
  (ABS melts before burning)
- <https://lahobbyguy.com/bb/viewtopic.php?t=24> (Ortur 7W: avoid 100% on
  engrave)
