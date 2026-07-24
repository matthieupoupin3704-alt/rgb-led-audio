# Fabrication files

Ready-to-upload files for JLCPCB. Generated from the KiCad project in the parent directory.

## Contents

| File | Purpose |
|------|---------|
| `jlc_bom.csv` | BOM in JLCPCB format (`Comment, Designator, Footprint, LCSC Part #`), 24 lines |
| `jlc_cpl.csv` | Pick-and-place, 35 SMD components, **rotations already corrected** |
| `rgb-led-audio-PTH.drl` | Plated holes (Excellon) |
| `rgb-led-audio-NPTH.drl` | Non-plated holes (mounting holes) |

Gerbers live in the parent directory (`rgb-led-audio-*.gbr` + `.gbrjob`).

## Rotations already applied

`jlc_cpl.csv` includes the +180° correction JLCPCB requires on:

- **Q1, Q2, Q3** (SOT-23)
- **U2** (SOT-223)
- **D1** (SMA)

Do not apply it a second time. If you regenerate the CPL from KiCad, you must reapply it.

## Order settings

| Parameter | Value |
|-----------|-------|
| Layers | 2 |
| Thickness | 1.6 mm |
| Copper | 1 oz |
| Via covering | Tented |
| PCBA type | **Standard** (the WROOM-1 is not on the Economic line) |
| Assembly side | Top only |
| Confirm Parts Placement | Recommended |

## Expected warnings

- **J2, J3, J4, J5 missing from the CPL** : intentional, they are through-hole and hand-soldered. Click Continue.
- **J3 is 0.23 mm from the board edge** : accepted, see the DRC section in the main README.

## Not included

Merged drill file. KiCad exported PTH and NPTH separately; JLCPCB prefers a single file. Either re-export with *Merge PTH and NPTH holes into one file*, or upload both.
