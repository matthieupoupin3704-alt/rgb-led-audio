# RGB LED Audio HAT

Raspberry Pi HAT driving a 5 V analog RGB LED strip (common anode) from an ESP32-S3, with ambilight over UART and an optional I2S audio input.

Built to replace the cheap IR controller shipped with a generic 4-pin RGB strip.

> **Status:** schematic and layout complete, fabrication files ready. **Not yet ordered, never physically built** (the PCB costs 2 €, the assembly costs 131 €, and I have 2 €). Nothing here has been validated on real hardware.

---

## Overview

```
Raspberry Pi 3B
  ├── Hyperion (ambilight)
  └── UART (GPIO14/15) ──► ESP32-S3
                              ├── 3× PWM ──► AO3400 MOSFETs ──► R/G/B (low-side)
                              └── I2S header (unpopulated)

RGB strip: 5 V, common anode. VCC to +5 V, each channel pulled to GND by a MOSFET.
Power: single external 5 V / 4 A supply, distributed by the HAT.
```

| Item | Value |
|------|-------|
| Board | 65 × 56 mm, Raspberry Pi HAT form factor |
| Layers | 2, 1.6 mm, 1 oz copper |
| MCU | ESP32-S3-WROOM-1-N16R8 (16 MB flash, 8 MB PSRAM) |
| Drivers | 3× AO3400 logic-level N-MOSFET, low-side |
| Firmware | Stock WLED + Hyperion protocol (no custom usermod) |

## Pinout

| Signal | GPIO | Notes |
|--------|------|-------|
| PWM_R / PWM_G / PWM_B | IO4 / IO5 / IO6 | Grouped, short routes to MOSFET gates |
| I2S_WS / SCK / SD | IO9 / IO10 / IO11 | Header present, not populated |
| UART TX → Pi | IO39 | UART1, **not** TXD0 |
| UART RX ← Pi | IO40 | UART1, **not** RXD0 |
| Status LED | IO41 | |
| USB D− / D+ | IO19 / IO20 | Hardwired to the native USB PHY |
| BOOT | IO0 | Button + 10 k pull-up |

Avoided throughout: GPIO26-37 (flash + octal PSRAM), strapping pins (0/3/45/46), and UART0 (43/44, boot log).

## Design notes

**Soldered module, not a dev board.** The goal of this board was to learn ESP32 routing: RF keep-out, native USB-C, regulator, boot/reset. A socketed DevKit would have skipped exactly the parts worth learning. JLC assembly removes the reflow risk on the module's thermal pad, leaving the layout as the only variable.

**Audio processing lives on the Pi.** Originally planned as a WLED FFT usermod in C++; moved to Python on the Pi instead. Sampling a digital source beats re-recording it through the air, and if the Pi is already running for ambilight, an onboard mic is redundant. The I2S header stays as a fallback (accepts an INMP441 or an S/PDIF receiver).

**Single 5 V supply.** The HAT feeds the Pi through header pins 2/4. Powering the strip from the Pi's own supply doesn't work: the official 2.5 A brick is already marginal, and the GPIO header isn't sized for another 1 A. USB-C PD was ruled out too: 3 A ceiling with no headroom against a ~2.55 A budget.

⚠️ **Never plug the Pi's own USB power in while the barrel jack is connected.** Two sources on one rail.

**J3 placement comes from a physical measurement of a Pi 3B**, not from a spec sheet. If a board ever gets built and doesn't seat correctly, check this first.

## Repository layout

| Path | Contents |
|------|----------|
| `rgb-led-audio.kicad_sch` | Schematic |
| `rgb-led-audio.kicad_pcb` | Layout |
| `rgb-led-audio.kicad_pro` | Project settings + netclasses |

Netclasses: `Power_In` 1.5 mm (jack → fuse, 2.5 A), `Power_5V` 0.8 mm, `Power_3V3` 0.4 mm, `LED` 0.5 mm, `Default` 0.2 mm. Sized for 1 oz copper, ΔT 10 °C.

## DRC status

Clean apart from two accepted warnings:

| Warning | Rationale |
|---------|-----------|
| **J3 is 0.23 mm from the board edge** (JLC recommends 0.5 mm) | Moving it would break the alignment measured on a real Pi. A board that won't seat is worse than a tight milling tolerance, and JLC routinely fabricates at this clearance. |
| **No fiducials** (35 SMD components) | JLC assembles boards of this class without them. Adding three would cost nothing and is worth doing on a revision. |

EMC risk score is 34/100. Twelve nets are flagged for missing stitching vias at layer transitions (RP-001). That rule becomes critical above ~100 MHz; the fastest signal here is USB 2.0 Full Speed at 12 Mbps. The only true HF source is the 2.4 GHz radio, which is internal to the module and behind a respected keep-out.

## Manufacturing

Targeted at JLCPCB, 2-layer, Standard assembly (the WROOM-1 is not available on the Economic line).

KiCad's BOM and CPL exports are **not** directly usable:

- BOM comes out semicolon-separated with no LCSC column → needs exactly `Comment, Designator, Footprint, LCSC Part #`
- CPL uses `Ref/Val/PosX/PosY/Rot/Side` and negative Y → needs `Designator, Mid X, Mid Y, Layer, Rotation` with Y flipped
- **Rotation +180° required** on SOT-23 (Q1-Q3), SOT-223 (U2) and SMA (D1). Skipping this mounts the MOSFETs and regulator backwards, and neither DRC nor ERC will catch it.
- Uncheck *Use drill/place file origin* when exporting the CPL
- Value casing matters: `1k` and `1K` produce two separate BOM lines in JLC's matcher
- J2/J3/J4/J5 are through-hole and excluded from the CPL on purpose (JLC will warn, that's expected)

Silkscreen artwork must stay above **0.15 mm** feature size or JLC silently drops it. Halftone-style images generally don't survive; solid shapes do.

## Bill of materials

| Ref | Part | LCSC |
|-----|------|------|
| U1 | ESP32-S3-WROOM-1-N16R8 | C2913202 |
| U2 | SGM2212-3.3 (LDO, tab = VOUT) | C3294699 |
| Q1-Q3 | AO3400 (SOT-23, G/S/D) | C181090 |
| D1 | SS14 Schottky | C2480 |
| F1 | PPTC 3 A | C883162 |
| J1 | USB-C 16P | C165948 |
| J5 | Barrel jack 5.5×2.1, 3 A | C84007 |
| SW1/SW2 | Tactile 6×6 mm | C55213946 |

Through-hole connectors (J2 1×4, J3 2×20 socket, J4 1×6, J5) are meant to be hand-soldered.

Sourcing notes: on PPTC fuses the number in the part code is in **centiamps** (BSMD1812-**300** = 3 A). Most LCSC barrel jacks are rated 500 mA, which is insufficient here.

## Cost

| Item | Price |
|------|-------|
| 5 bare PCBs | 2 € |
| 2 assembled + 3 bare | ~131 € |

Fabrication is negligible; the BOM is the entire cost. ~28 $ is the two ESP32 modules and ~21 $ is JLC's feeder fee for seven Extended Parts.

## Not done

- Never fabricated or tested
- No firmware written (stock WLED assumed, unverified)
- Pi-side FFT is a plan, not code
- Ambilight needs an HDMI splitter with HDCP strip + USB capture dongle, unless the Pi itself plays the content

## License

MIT
