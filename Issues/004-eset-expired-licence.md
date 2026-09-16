# #004: ESET still running with an expired licence

**Status:** RESOLVED
**Opened:** 2026-08-26
**Resolved:** 2026-09-16
**Area:** Software / security

**Symptom:** ESET Security fully installed and running despite the licence expiring in
September 2025. Services `ekrn`, `efwd`, `ekrnEpfw` active. Filter driver `eamonm` stacked above
Defender's `WdFilter` (altitude 328700 vs 328010). Autostart via HKLM Run: `egui -> ecmds.exe
/run /hide /proxy`.

**Measured impact:** none on latency. LatencyMon shows `eamonm.sys` at 0 ISR / 0 DPC and only 90
hard pagefaults over the sample (see [#001](001-f1-23-micro-stutter.md)). It is pure overhead,
not a stutter cause.

**Why it is still there:** the uninstall is password-protected and requires Safe Mode. The 49"
Neo G9 cannot display WinRE / Safe Mode over DisplayPort (basic display driver can't negotiate DSC),
so Safe Mode needs the HDMI input.

**Action:** keep the HDMI cable on the monitor's second input, boot to Safe Mode, run the ESET
uninstaller, then let Defender take over. Do this in the same session as the DisplayPort switch
planned in #001 step 4.

## Resolution (2026-09-16)

ESET uninstalled. Defender is now the only AV/filter driver on the machine. Done in the same
session as the HDMI to DisplayPort switch ([#001](001-f1-23-micro-stutter.md) step 4).

Left to verify on the rig PC, low priority:

- `sc query ekrn` returns "service does not exist"
- `fltmc filters` no longer lists `eamonm`
- HKLM Run key no longer contains the `egui` entry
- Windows Security shows Defender real-time protection On (ESET used to keep it disabled)
