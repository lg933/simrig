# #001: F1 23 random micro-stutter

**Status:** OPEN

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

### Frametime evidence (benchmark run 2026-08-26 05:34)

F1 23 writes a frametime CSV in benchmark mode:
`Documents\My Games\F1 23\benchmark\Benchmark_2026-08-26_at_05-34-19_frametimes.csv`

| Metric | Value |
|---|---|
| Sample | 1 392 frames over 17.3 s |
| Mean | 12.40 ms (80.6 FPS) |
| Median | 11.76 ms (85.1 FPS) |
| Min | 10.63 ms |
| p90 / p95 | 12.95 / **18.50** ms |
| p99 (1 % low) | 19.93 ms (50.2 FPS) |
| p99.9 (0.1 % low) | 25.05 ms |
| **Max** | **94.77 ms** (10.6 FPS) |
| Frames > 2× median | 4 |
| Frames > 33 ms | 2 |

**Reading it:**

- **The bulk of the distribution is tight.** Median 11.76 ms with p90 at 12.95 ms — 90 % of frames sit
  within 1.2 ms of each other. That is *good* frame pacing, not stutter.
- **There is a cliff between p90 and p95** (12.95 → 18.50 ms). Roughly 5-10 % of frames take ~50 %
  longer than median. That is real pacing inconsistency and is consistent with ray-tracing work
  (BVH rebuilds / DDGI probe updates).
- **The worst frames are clustered, not scattered.** The 94.77 ms spike at frame 543 is followed by a
  recovery tail (frames 545-558 all elevated). One event with a tail, not random hitching.

**Important caveat: this is benchmark mode, not racing.** The F1 23 benchmark is a scripted camera
fly-through with cuts, and a large spike at a camera transition is expected. 17 seconds is also a
very short sample. **This run does not reproduce the reported in-race stutter** — only 2 frames
exceeded 33 ms in the whole capture.

**Action:** capture frametimes during actual racing using CapFrameX or PresentMon. The in-game
benchmark is the only thing that writes this CSV automatically, and it is not representative.

### Changes applied 2026-08-26 (no improvement yet)

- ASUS bloat removed: 7 services → Manual, 17 processes killed (see [`SETUP_AUDIT.md`](../Hardware/SETUP_AUDIT.md))
- Xbox Game Bar / GameDVR disabled
- Duplicate `PlatformManager` killed — **broke the motion seat, had to be restarted.** Two instances
  is this app's normal state; do not kill one
- G-SYNC per-display toggle → On
- Global G-SYNC mode → **Full screen and windowed** (required: F1 23 runs borderless)
- Driver profile: Vertical Sync → On, Power Management → Prefer maximum performance
- In-game: FPS limit → On @ 138, in-game V-Sync → Off

### Changes applied 2026-09-16

- **HDMI → DisplayPort** (next step 4 below). ESET removed in the same session, see
  [#004](004-eset-expired-licence.md). Stutter not yet re-tested after the switch.
- To re-verify in NVIDIA App now that the link changed: the driver treats a new connector as a new
  display, so the per-display G-SYNC toggle from 2026-08-26 may have reset. Check that the
  *"not validated as G-SYNC compatible"* warning is gone, scaling controls are no longer greyed out,
  and refresh reads 143 Hz in both NVIDIA App and Windows.

### Next steps, in order

1. **Turn HDR off in F1 23** (`HDR → Désactivé`). Single biggest untested variable, and the likeliest
   explanation for the latency figure.
2. **Turn ray tracing fully off** and retest. If smooth, reintroduce one option at a time — DDGI last,
   it is the most expensive.
3. **Cadence test:** set refresh to 120 Hz and cap FPS at 60. Every frame then shows for exactly two
   refreshes, making clock-mismatch judder mathematically impossible. Smooth ⇒ confirmed sync problem
   ⇒ DisplayPort required. Still stuttering ⇒ not a sync problem at all.
4. ~~**Switch HDMI → DisplayPort.**~~ **Done 2026-09-16.** Resolves the HDTV classification and
   the validation warning together. ⚠️ Trade-off: this ultrawide cannot display WinRE / Safe Mode
   over DisplayPort (basic display driver can't negotiate DSC). Keep the HDMI cable on the second
   input and switch inputs when recovery is needed. Effect on the stutter not yet reported.
5. Update GPU driver 591.86 → 610.88. It was the top DPC consumer at 1 284 µs, the only driver over
   1 ms. Do this **after** the above so variables stay separated, and note that a *clean* install
   resets every NVIDIA setting listed here.

### Redundant settings noticed (fix regardless)

- **HBAO+ and RT Ambient Occlusion are both enabled.** RT AO supersedes HBAO+ — pure waste.
- **SSR at Ultra and RT Reflections both enabled.** Same redundancy.
- `Ombrage à taux variable` (VRS) is **Off** — turning it on would recover performance cheaply.
- Motion blur at 20 — usually turned off entirely for sim racing.
