# Known Issues

Running log of unresolved or recurring problems on the rig, so they don't get re-investigated from
scratch. Newest first.

---

## #1 — F1 23 random micro-stutter · **OPEN**

**Opened:** 2026-08-26
**Symptom:** Random micro-freezes / micro-stutter during play. Occurs in **races and free practice
at equal frequency**, with no identifiable trigger. Only title currently played, so no cross-game
comparison is available.

### Measured state at time of writing

| Metric | Value |
|---|---|
| FPS | ~95 |
| GPU utilization | 96–98 % |
| CPU utilization | 23 % |
| PC latency (Reflex overlay) | **52.4 ms** ≈ 5 frames — high |
| VRAM | 10.9 / 16.4 GB (5.1 GB free) |
| Display | 5120×1440 @ 143 Hz, VRR range 50–143 |

### Ruled OUT (do not re-investigate)

| Hypothesis | Evidence |
|---|---|
| Background process / DPC interference | LatencyMon: highest interrupt-to-process latency **138.8 µs** (excellent). Killing Armoury Crate + 6 ASUS services + 17 ASUS processes + RGB SMBus pollers produced **no improvement** |
| ESET file I/O stalling | Hard pagefault count **90** over the sample — very low. `eamonm.sys` shows 0 ISR / 0 DPC |
| VRAM exhaustion | 5.1 GB free under load |
| Thermal throttling | GPU 64 °C against an 84 °C target |
| Storage | 2 583 GB free of 3 725; `storport`/`stornvme` clean in LatencyMon |
| CPU bottleneck | CPU at 23 % — purely GPU-bound |
| Overclock instability | No OC applied; power limit stock 100 % |

### Still open — leading suspects

**A. VRR is probably not actually engaging.** 95 FPS on a 143 Hz fixed-refresh panel is a 1.5×
ratio, so each frame is held for one or two refreshes in an uneven cadence — textbook judder that
reads as random micro-freezing. Evidence VRR isn't working:

- Connector reports `HDMI - **HDTV**` — driver treats the panel as a television
- NVIDIA warns *"Selected display is not validated as G-SYNC compatible"* (it **is** certified over
  DisplayPort, not over HDMI)
- Scaling controls were greyed out, consistent with HDTV classification
- Refresh reported as 144 Hz by NVIDIA vs 143 Hz by Windows

**B. HDR is enabled in-game as `Activé (scRGB)`** while Windows HDR appears to be **off** (no
`AdvancedColorEnabled` value exists in the registry). scRGB is FP16 — double the framebuffer
bandwidth of SDR — and in **borderless windowed** it forces DWM into HDR composition, which breaks
independent flip. Strong candidate for the 52.4 ms latency.

**C. Ray tracing is fully enabled at `Élevée`** — shadows, reflections, ambient occlusion,
transparent reflections **and DDGI**, on top of near-universal Ultra settings at 5120×1440. Explains
the 96 % GPU load, and RT is a known source of random hitching (BVH rebuilds, DDGI probe updates).

### Changes applied 2026-08-26 (no improvement yet)

- ASUS bloat removed: 7 services → Manual, 17 processes killed (see [`SETUP_AUDIT.md`](SETUP_AUDIT.md))
- Xbox Game Bar / GameDVR disabled
- Duplicate `PlatformManager` killed — **broke the motion seat, had to be restarted.** Two instances
  is this app's normal state; do not kill one
- G-SYNC per-display toggle → On
- Global G-SYNC mode → **Full screen and windowed** (required: F1 23 runs borderless)
- Driver profile: Vertical Sync → On, Power Management → Prefer maximum performance
- In-game: FPS limit → On @ 138, in-game V-Sync → Off

### Next steps, in order

1. **Turn HDR off in F1 23** (`HDR → Désactivé`). Single biggest untested variable, and the likeliest
   explanation for the latency figure.
2. **Turn ray tracing fully off** and retest. If smooth, reintroduce one option at a time — DDGI last,
   it is the most expensive.
3. **Cadence test:** set refresh to 120 Hz and cap FPS at 60. Every frame then shows for exactly two
   refreshes, making clock-mismatch judder mathematically impossible. Smooth ⇒ confirmed sync problem
   ⇒ DisplayPort required. Still stuttering ⇒ not a sync problem at all.
4. **Switch HDMI → DisplayPort.** Resolves the HDTV classification and the validation warning
   together. ⚠️ Trade-off: this ultrawide cannot display WinRE / Safe Mode over DisplayPort (basic
   display driver can't negotiate DSC). Keep the HDMI cable on the second input and switch inputs when
   recovery is needed — Safe Mode is still required to remove ESET.
5. Update GPU driver 591.86 → 610.88. It was the top DPC consumer at 1 284 µs, the only driver over
   1 ms. Do this **after** the above so variables stay separated, and note that a *clean* install
   resets every NVIDIA setting listed here.

### Redundant settings noticed (fix regardless)

- **HBAO+ and RT Ambient Occlusion are both enabled.** RT AO supersedes HBAO+ — pure waste.
- **SSR at Ultra and RT Reflections both enabled.** Same redundancy.
- `Ombrage à taux variable` (VRS) is **Off** — turning it on would recover performance cheaply.
- Motion blur at 20 — usually turned off entirely for sim racing.

---

## Background: unrelated but tracked

**Intel Raptor Lake mitigations missing.** i7-14700KF on BIOS **1402 (08-Sep-2023)**, microcode
**0x11D**. Intel's degradation fixes are 0x125 / 0x129 / 0x12B — all newer. The chip has run ~3 years
without the elevated-voltage protections. **Not believed to cause the stutter**, but should be fixed
for CPU longevity. BIOS flash is a manual job.

**WHEA Event 17 corrected PCIe errors**, in bursts, from Root Port #5 (`0:1C.4`, `DEV_7A3C` —
chipset port). Downstream candidates: Intel I226-V Ethernet (disconnected, on Wi-Fi) or a VIA USB 3.0
controller. Corrected, not fatal. Investigate via ASPM if it ever escalates.

**ESET still fully installed and running** despite the licence expiring Sept 2025 — `ekrn`, `efwd`,
`ekrnEpfw` active, `eamonm` filter stacked above Defender's `WdFilter`. Measured as *not* causing
latency, but it is pure overhead. Removal blocked on password-protected uninstall requiring Safe Mode
plus a display that can show it.
