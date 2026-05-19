# CLAUDE.md

Guidance for Claude when working on this repo (KiCad reusable design blocks).

## What this repo is

Self-contained KiCad design blocks (schematic + optional pre-routed PCB)
that can be imported into any KiCad 10 project via the Design Blocks panel.
Each subdirectory is one block.

## Critical conventions

### Block directory naming

- Block directory **MUST** end in `.kicad_block` (e.g., `RP2354B_core.kicad_block/`).
  Without this suffix, KiCad 10 won't list the block in the Design Blocks panel.
  Verified in KiCad source `design_block_io.cpp` (~line 170).

### Library registration

The user's `design-block-lib-table` points to **this repo's root**, not to a
specific block. KiCad scans the root for `*.kicad_block` directories.

Path: `~/Library/Preferences/kicad/10.0/design-block-lib-table` (macOS).

### Schematic ↔ PCB sync

Every block must keep its schematic and PCB in perfect refdes sync.

- **Refdes sequential, no gaps** (C1..Cn, R1..Rn). Gaps cause `Re-link by refdes`
  to fail silently when parents import the block, producing duplicate footprints.
- When parts are added/removed, immediately re-annotate and F8 to push refdes
  to the PCB so the two sides stay in lockstep.
- Inside the block project, `F8 (Update PCB from Schematic)` should report
  **"No changes"**. If it wants to add/remove anything, the block is drifted
  and not safe to import elsewhere — fix before pushing.

### Block-internal ERC

Running ERC on a block standalone produces two expected error classes:

1. `Hierarchical label in root sheet cannot be connected to non-existent parent sheet`
2. `Input Power pin not driven by any Output Power pins`

Both are artifacts of running ERC without a parent context (the parent is what
drives power and connects the hierarchical labels). **Suppress them** at the
block project level:

- `File → Schematic Setup → Electrical Rules → Violation Severity`
- Set both to `Ignore` (or `Warning`)
- Settings persist in `.kicad_pro` and travel with the block

After suppressing, ERC should be clean. Any remaining violations are genuine.

## How blocks get imported into parent projects

This is the workflow Claude (and the user) follows in parent projects that
consume blocks from this repo.

### Schematic-side placement

Parent schematic editor → Design Blocks panel → drag block in.

- **Sheet drop** (preferred): ☑ Place as sheet, ☑ Keep annotations.
  Block appears as a child sheet; hierarchical labels become sheet pins.
  Refdes scoped per sheet path; multi-instance support is clean.
- **Flat drop**: ☐ Place as sheet, ☑ Keep annotations.
  Block's symbols land directly in parent root sheet. Simpler for single
  instance, but loses encapsulation.

**Always ☑ Keep annotations** if you plan to import the block's pre-routed PCB.

### PCB-side import (the hard part)

The schematic placement does NOT bring the PCB layout. Routed PCB requires a
separate paste-and-relink dance:

1. Save parent schematic after placing the block.
2. Open parent `.kicad_pcb` in PCB editor.
3. Open block's `.kicad_pcb` in a second PCB editor window.
4. Block PCB: Select All (Ctrl+A) → Copy (Ctrl+C).
5. Parent PCB: Right-click → **Paste Special** →
   ☑ **"Keep existing reference designators, even if they are duplicated"** →
   click to place.
6. Press **F8** (Update PCB from Schematic) →
   ☑ **"Re-link footprints to schematic symbols based on their reference designators"** →
   Update PCB.

The re-link checkbox is the critical piece. Without it, F8 matches by UUID,
the pasted footprints' UUIDs don't match any schematic symbol, and KiCad adds
duplicates instead of binding the pasted ones.

### Troubleshooting imports

If duplicates appear after the steps above, one of these is true:

- Block has refdes gaps or schematic↔PCB drift → fix the block first.
- Re-link by refdes checkbox wasn't actually checked.
- Flat-drop and parent root already has refdes that collide with block refdes.

### Multi-instance (same block placed twice in one project)

- Second sheet drop → KiCad auto-renumbers to keep refdes globally unique
  (e.g., first instance C1..C17, second instance C18..C34).
- For the second instance's PCB layout, use **Tools → Replicate Layout** in
  the PCB editor (KiCad 8+ built-in). It uses sheet-path UUIDs to duplicate
  footprint positions + routing onto the second instance.
- Don't try paste-special for second instance — refdes collisions make it
  painful.

## Authoring a new block

1. Create `<name>.kicad_block/` directory at repo root.
2. Inside, create the KiCad project (`.kicad_pro`, `.kicad_sch`, `.kicad_pcb`).
3. Design the circuit. Expose external pins as hierarchical labels at root.
4. Annotate from 1 with no gaps.
5. Build the PCB layout. Aim for compactness — parent projects place the
   block as one unit.
6. F8 in block project → confirm "No changes."
7. Configure ERC severities to ignore the two expected block-standalone errors.
8. Write `README.md` inside the block dir: pins, BOM, gotchas.
9. Commit.

When you edit an existing block, always:

- Re-annotate to close any new gaps from deletions
- Run F8 in the block project to verify sch↔PCB sync
- Confirm ERC is still clean (or only shows the suppressed expected errors)

## Why this repo exists

Reusing a working RP2354B core circuit (MCU + decoupling + crystal + USB +
boot) across multiple projects (NeoGeo cart Picos, other RP2354B boards)
without re-wiring it every time. One block, audited once, dropped many
times.
