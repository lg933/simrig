# #002: Intel Raptor Lake mitigations missing

**Status:** OPEN, BIOS flash pending
**Opened:** 2026-08-26
**Area:** CPU / BIOS

**Symptom:** none yet. Tracked for CPU longevity, not for a visible fault.

i7-14700KF running on BIOS **1402 (2023-09-08)**, microcode **0x11D**. Intel's Raptor Lake
degradation fixes are 0x125 / 0x129 / 0x12B, all newer. The chip has run roughly three years
without the elevated-voltage protections. No OS-level microcode load supersedes the BIOS revision
(see `cpu.microcode` in [`specs.json`](../Hardware/PC/specs.json)).

**Not believed to cause** the F1 23 stutter ([#001](001-f1-23-micro-stutter.md)).

**Action:** flash the ASUS TUF GAMING Z790-PRO WIFI BIOS to the latest release. Manual job on the
rig PC. Note that a BIOS update resets XMP (currently disabled anyway, see `memory.anomaly` in
`specs.json`) and may reset boot order.
