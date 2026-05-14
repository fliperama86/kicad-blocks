# RP2354B_core

Reusable RP2354B (or RP2350B) MCU core block, derived from the official
Raspberry Pi RP2350B Minimal Board reference (RP-006442-DD-2).

The schematic is QFN-80 footprint compatible with both:

- **RP2354B** (preferred) — 2 MB on-die stacked flash, no external QSPI flash needed.
- **RP2350B** — wire an external QSPI flash to the `QSPI_*` sheet pins.

## Status

Schematic v0.1 — pruned from the reference design via the KiCad MCP
server. Still requires a UI pass to:

1. Run ERC and prune dangling wires left behind by the deleted parts
   (USB, breakout headers, mounting holes, flash, buttons).
2. Add hierarchical labels at the boundary (replace power flags and
   former net-to-connector exits with `Hierarchical Label`s matching
   the sheet pin names below).
3. Reannotate (refdes numbers are sparse after deletions).
4. Optionally relocate parts so the remaining circuit fits a tighter
   sheet area before exporting as a Design Block.

## Files

- `RP2354B_core.kicad_pro` / `.kicad_sch` / `.kicad_pcb`
- `MCU_RaspberryPi_RP2350.kicad_sym` — project-local symbol library
  (RP2350_80QFN symbol; same footprint accepts RP2354B).
- `RP2350_80QFN_minimal.pretty/` — project-local footprint library
  (RP2350 QFN-80, crystal, regulator, passives).

The unmodified RPi reference archive used as the starting point lives
in `~/Downloads/RP-008295-DS-1-Minimal-KiCAD rp2350/` (Raspberry Pi
RP-006442-DD-2). Re-import from there if a clean baseline is needed.

## What's inside (post-prune, 46 placed symbols)

- **U1** — RP2350_80QFN (assemble as RP2354B; see U1 properties).
- **U2** — NCP1117-3.3 LDO, VBUS → 3V3 (DNP if consumer feeds 3V3 directly).
- **Y1** — 12 MHz crystal (`ABM8-272-T3`) with `C3`/`C4` 15 pF load caps,
  `R2` biasing, GND-pinned package.
- **L1 / C6 / C7** — 3.3 µH + 4.7 µF LC filter for VREG_VIN.
- **C9 / C10** — 4.7 µF 1V1 bulk.
- **C19** — 10 µF 3V3 bulk.
- **C1 / C5** — 10 µF caps around the regulator.
- **C8, C11–C18, C20, C21, C13** — 100 nF per-supply-pin decoupling.
- **R3** — 33 Ω (VREG bias).
- **R6** — 1 kΩ (RUN pull-up).
- Power flags: `+3V3`, `+1V1`, `VBUS`, `GND`.

## Proposed sheet pin interface

To convert this into a Design Block / hierarchical sheet, the consumer
sees these pins only:

| Group | Sheet Pins | Direction (sub-sheet POV) |
| --- | --- | --- |
| Power in | `VBUS` (optional, into reg), `3V3`, `GND` | input |
| Reset / boot | `RUN`, `BOOTSEL`, `USB_BOOT` | bidirectional |
| Debug | `SWCLK`, `SWDIO` | bidirectional |
| USB | `USB_DP`, `USB_DM` | bidirectional |
| QSPI (PSRAM on RP2354B; flash+PSRAM on RP2350B) | `QSPI_SCK`, `QSPI_CSn`, `QSPI_SD0`–`QSPI_SD3` | bidirectional |
| GPIO | `GPIO0`..`GPIO47` | bidirectional |
| ADC | `ADC_AVDD`, `ADC_GND` | input |

Lock the final list when adding the hierarchical labels in KiCad.

## Revision

- **v0.1 (2026-05-14)** — pruned reference design: deleted USB connector,
  2× 26-pin breakouts, 4× mounting holes, BOOTSEL + RUN push buttons,
  USB 27 Ω series resistors, primary + alt W25Q128JVS flash and their
  decoupling/pull-ups; tagged U1 with MPN/Description properties. ERC
  and hierarchical labels still pending.
- v0 — scaffolding only.
