# RP2354B_core

Reusable RP2354B MCU core block.

RP2354B = RP2350B with 2 MB of stacked on-die flash, so this block
intentionally omits the external QSPI flash chip and its support
components.

## Status

**Scaffolding only.** The schematic itself still needs to be drawn in
KiCad from the Raspberry Pi RP2350B minimal reference design
(`~/Downloads/RP-008295-DS-1-Minimal-KiCAD rp2350/RP-006442-DD-2-RP2350B Minimal Board Kicad archive/`),
swapping QFN-80 RP2350B for RP2354B and removing the flash section.

## What's inside (target scope)

- RP2354B (QFN-80) MCU
- Bulk + per-rail decoupling on IOVDD / DVDD / USB / ADC rails
- 1V1 LDO (or buck) feeding DVDD from VDD_IO
- 12 MHz crystal with load caps on XIN/XOUT
- RUN pull-up + reset cap
- BOOTSEL pull-up (or test-point) — keep BOOTSEL exposed via sheet pin
  so each consuming board decides whether it's a button, header, or
  hard-tied
- USB D+/D− pulled out to sheet pins (no ESD/series resistors inside
  the block — keep flexibility per project)

## What's NOT inside

- External QSPI flash (RP2354B has 2 MB internal)
- External PSRAM (project-specific routing; pull these out as sheet pins)
- LDO input filtering beyond what the chip needs (project-level
  power tree decides)
- Crystal frequency choice if non-12 MHz (re-spin block if needed)

## Sheet pin interface (proposed)

| Group | Pins |
| --- | --- |
| Power in | `VBUS` (5 V), `3V3`, `GND` |
| USB | `USB_DP`, `USB_DM` |
| Control | `RUN`, `BOOTSEL`, `SWDIO`, `SWCLK` |
| GPIO | `GPIO0..GPIO47` (RP2350B/RP2354B has 48 GPIOs in QFN-80) |
| QSPI (PSRAM-only on RP2354B) | `QSPI_SCK`, `QSPI_CSn`, `QSPI_SD0..SD3` |
| ADC reference | `ADC_AVDD`, `ADC_GND` |

Final pin list to be locked when the schematic is drawn.

## Revision

- v0 — scaffolding, no schematic yet (2026-05-14).
