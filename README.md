# Simrig

Documentation for the sim racing rig: hardware inventory, driver and display configuration
captures, an issue log, and the spec for the button box / power sequencing project.

## The rig

| | |
|---|---|
| PC | Intel i7-14700KF, 64 GB DDR5, RTX 4070 Ti SUPER 16 GB, WD SN850X 4 TB, Windows 11 Pro |
| Display | Samsung Odyssey Neo G9 49" (LS49AG95), 5120x1440 @ 143 Hz, DisplayPort (HDMI kept on input 2 for Safe Mode) |
| Wheelbase / pedals | MOZA R12, MOZA CRP, MOZA Universal Hub |
| Cockpit | Next Level Racing F-GT Elite 160, bottom mount, NLR-E025, 136 x 84.2 x 94.2 cm listed, 57.1 kg ([assembly manual, PDF](https://cdn.nlr.sh/wp-content/uploads/2023/03/FINAL-FOR-PRINT-F-GT-ELITE-160-BOTTOM-MOUNT-07102022.pdf)) |
| Motion | Motion platform driven by Next Level Racing Platform Manager |
| Controls | Elgato Stream Deck XL (left side), button box planned for the right side |
| Software | SimHub 9.11.1, MOZA Pit House, TrackIR 5 (camera never connected) |

Full detail, down to USB VID/PID and Steam app ids, lives in
[`Hardware/PC/specs.json`](Hardware/PC/specs.json). That file is the source of truth; the
Markdown docs link to it rather than repeating numbers.

## Layout

```
Hardware/
├── SETUP_AUDIT.md            narrative audit of the rig PC, 2026-08-24
├── PC/
│   ├── specs.json            machine-readable inventory, 2026-08-26
│   └── NVIDIA_DISPLAY_CONFIG.md   NVIDIA App capture, all tabs, with findings
├── Button_Box/
│   └── README.md             hardware spec for the power sequencing panel
└── Frame/                    local copy of the cockpit manual (gitignored, link above)
Issues/                       one file per issue, numbered in opening order
CAD/                          CATIA model of the rig, measurements.md holds the tape-measure sheet
Photos/                       rig photos and videos (not committed)
```

## Issues

| # | Title | Status | Opened |
|---|---|---|---|
| [001](Issues/001-f1-23-micro-stutter.md) | F1 23 random micro-stutter | OPEN | 2026-08-26 |
| [002](Issues/002-raptor-lake-microcode.md) | Intel Raptor Lake mitigations missing (BIOS 1402, microcode 0x11D) | OPEN, BIOS flash pending | 2026-08-26 |
| [003](Issues/003-whea-pcie-corrected-errors.md) | WHEA Event 17 corrected PCIe errors, chipset Root Port #5 | TRACKED | 2026-08-26 |
| [004](Issues/004-eset-expired-licence.md) | ESET running with expired licence | RESOLVED 2026-09-16 | 2026-08-26 |

Status values: `OPEN` (being worked), `BLOCKED` (waiting on something named in the file),
`TRACKED` (no action unless it escalates), `RESOLVED` (file stays, fix gets appended).
Each issue file opens with Status, Opened, Area, Symptom. Ruled-out hypotheses stay in the file so
they are not re-tested.

## Conventions

- New issue: next number, `NNN-short-slug.md` in `Issues/`, add a row to the table above.
- Hardware or software facts change: update `specs.json` first, then the docs that cite it.
- Photos, videos and vendor PDFs are gitignored. They exceed GitHub's file size limit and git
  keeps every binary forever. Keep them in cloud storage and link if needed.
