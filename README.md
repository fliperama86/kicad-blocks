# kicad-blocks

Reusable KiCad design blocks shared across projects.

Each subdirectory is a single block: a self-contained schematic snippet
(plus optional layout, symbols, and footprints) that can be dropped into
any KiCad project as a hierarchical sheet or, in KiCad 9+, registered as
a Design Block library.

## Layout

```
kicad-blocks/
├── README.md
├── <block-name>/
│   ├── README.md                  # what the block is, sheet pins, BOM, gotchas
│   ├── <block-name>.kicad_sch     # the reusable schematic
│   ├── <block-name>.kicad_pcb     # optional preplaced layout
│   ├── symbols/                   # block-specific symbol library (.kicad_sym)
│   └── footprints.pretty/         # block-specific footprints (.kicad_mod)
└── ...
```

A block should expose its interface only via hierarchical sheet pins.
Anything internal (decoupling caps, crystal, regulator) stays inside
the sheet so projects can reuse without re-wiring.

## Two ways to consume a block

### KiCad 9+ Design Block library (preferred)

1. Open KiCad → Preferences → Manage Design Block Libraries.
2. Add a global library: Path = `~/Projects/kicad-blocks`, Nickname = `kicad-blocks`.
3. In the schematic editor, the Design Block panel will list each
   subdirectory as a block. Drop one onto a sheet.

### Hierarchical sheet (KiCad 7/8 fallback)

1. In the schematic editor, Place → Add Hierarchical Sheet.
2. Point the sheet file at `~/Projects/kicad-blocks/<block>/<block>.kicad_sch`.
3. Annotate; per-instance refdes are tracked by KiCad automatically.

## Editing a block

Open the block's own `.kicad_pro` (or the `.kicad_sch` directly) — never
edit the file from inside a consuming project, or KiCad will save a
project-local copy and break the upstream link.

After editing, every project that consumes the block needs an ERC/DRC
re-run and a reannotate; bump the block's README "Revision" line so
downstream consumers know to pull.

## Blocks

- [`RP2354B_core/`](RP2354B_core/README.md) — RP2354B MCU core with
  XOSC, decoupling, RUN/boot circuitry, 1V1 regulator. No external
  flash (RP2354B has 2 MB on-die).
