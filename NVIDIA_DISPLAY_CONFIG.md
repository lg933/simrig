# NVIDIA & Display Configuration

Full capture of the NVIDIA App configuration for the sim rig.

- **Captured:** 2026-08-26
- **Source:** NVIDIA App 11.0.8.299, all tabs (System / Graphics / Settings), cross-checked against
  Windows WMI where possible
- **Purpose:** baseline reference for the F1 23 micro-stutter investigation, and a restore point if
  settings drift

> ⚠️ Several values below are **misconfigured** and are the current leading explanation for the F1 23
> stutter. See [§9 Findings](#9-findings--recommended-changes) before treating this as a good baseline.

---

## 1. Display Hardware & Link

| Property | Value |
|---|---|
| Model | Samsung **LS49AG95** (Odyssey Neo G9, 49") |
| Manufacture year | 2023 |
| Serial | HNTW700773 |
| **Connector** | **HDMI — classified as `HDTV`** ⚠️ |
| Native resolution | 5120 x 1440 |
| Active resolution | 5120 x 1440 (native) |
| Refresh rate | **144 Hz** per NVIDIA App / **143 Hz** per Windows ⚠️ |
| VRR range (Windows) | 50 – 143 Hz |
| HDCP | Supported |
| Orientation | Landscape |
| Displays attached | 1 (primary) |
| Surround | Not available |

**Scaling**

| Setting | Value |
|---|---|
| Mode | Aspect Ratio / No Scaling / Full-screen — all greyed out |
| Scaling device | GPU / Display — greyed out |
| Override scaling set by games | Unchecked |

The scaling controls being greyed out is a consequence of the `HDTV` connector classification.

---

## 2. G-SYNC / VRR

| Scope | Setting | Value |
|---|---|---|
| Global | G-SYNC | At capture `On, Full screen` → **changed to `On, Full screen and windowed` 2026-08-26** (required: F1 23 runs borderless) |
| **Per-display** | **G-SYNC (Allow VRR for selected display)** | Off at capture → **switched ON 2026-08-26** |
| Per-display | Validation status | ⚠️ *"Selected display is not validated as G-SYNC compatible"* — because the link is HDMI. The LS49AG95 **is** certified G-SYNC Compatible over **DisplayPort** |
| Driver profile | Monitor Technology (global) | G-SYNC Compatible |
| Driver profile | Monitor Technology (F1 23) | Global — G-SYNC Compatible |

**Two independent reasons VRR is not active:**

1. **The per-display toggle is Off**, and it overrides the global setting.
2. **The global mode is `Full screen` only**, which excludes borderless windowed — and **F1 23 is
   run in borderless windowed**. Even with the toggle on, G-SYNC would never engage for it.

The *My Rig* tab lists "Variable Refresh Rate" as a display *capability*, not as an enabled state.

**Fix:** per-display toggle → On (**done**), global mode → **Full screen and windowed** (required —
F1 23 runs borderless).

Borderless is fine to keep: Windows' `SwapEffectUpgradeEnable=1` is set globally with no per-app
override for F1 23, so it gets independent flip — proper VRR and near-fullscreen latency. Exclusive
fullscreen is not necessary.

**V-Sync placement** (a common trap): in-game V-Sync must stay **off**, or F1 23 greys out its own
FPS limiter. Force V-Sync **on** in the NVIDIA driver profile instead, and cap with NVIDIA's
**Max Frame Rate = 138** rather than the in-game limiter. With Reflex On+Boost active, Reflex also
auto-caps below refresh once G-SYNC and V-Sync are both on.

**Flicker watch:** unvalidated VRR over HDMI can cause brightness flicker in menus and loading
screens where framerate swings hard. If that appears, DisplayPort is genuinely required.

---

## 3. Colour Output

| Setting | Value |
|---|---|
| Output Color Settings | Default |
| Output Color Format | RGB |
| Output Color Depth | 10 bpc |
| Desktop Color Depth | Highest (32-bit) |
| Output Dynamic Range | **Full** |
| Color Accuracy Mode | Accurate |
| Override to reference mode | Unchecked |
| Content Type | Auto-select (recommended) |

**Adjustments (All Channels)**

| Control | Value |
|---|---|
| Brightness | 100 |
| Contrast | 100 |
| Gamma | 1 |
| Digital Vibrance | 50 % |
| Hue | 0° |

Colour is correct — RGB Full range at 10 bpc, no adjustments applied. Despite the `HDTV`
classification the driver has *not* forced limited range or chroma subsampling.

---

## 4. GPU State & Limits

**Idle sample (2026-08-26)**

| Metric | Value |
|---|---|
| GPU Clock | 210 MHz |
| GPU Power | 24 W |
| GPU Temperature | 41 °C |
| GPU Voltage | 0.935 V |
| VRAM Clock | 405 MHz |
| GPU Utilization | 4 % |
| CPU Utilization | 9 % |

**Load sample (F1 23 running, via `nvidia-smi`)**

| Metric | Value |
|---|---|
| GPU Utilization | **98 %** |
| VRAM used | 10 913 / 16 376 MiB (5 149 MiB free) |
| Power draw | 266 W |
| Temperature | 64 °C |

**Tuning & limits**

| Setting | Value |
|---|---|
| Automatic Tuning | Off |
| GPU Tuning | +0 MHz |
| VRAM Tuning | +0 MHz |
| Voltage maximum | 0 % |
| Power maximum | 100 % |
| Temperature target | 84 °C |
| Fan speed target | Automatic |
| Debug Mode | Off |

No overclock applied. Card is stock.

---

## 5. System & Driver

| Property | Value |
|---|---|
| OS | Windows 11 Pro, 10.0.26200 |
| Driver | **Game Ready 591.86 — 27 Jan 2026** ⚠️ stale |
| GPU | NVIDIA GeForce RTX 4070 Ti SUPER (16 GB) |
| CPU | Intel Core i7-14700KF |
| RAM | 64.0 GB |
| Storage | SSD — 3.6 TB |
| GPU Performance Counters | Admin users only (recommended) |

---

## 6. Driver Settings — Global

| Setting | Value |
|---|---|
| DLSS Override — Model Presets | Use 3D app setting |
| DLSS Override — Frame Generation Mode | Use 3D app setting |
| DLSS Override — Super Resolution Mode | Use 3D app setting |
| Smooth Motion | Off |
| Low Latency Mode | Off |
| CUDA — Sysmem Fallback Policy | Driver Default |
| DSR — Factors | Off |
| GPU App Assignment | RTX 4070 Ti SUPER |
| **Image Scaling** | **Sharpen 50, Render resolution 85 % (4352 x 1224)** ⚠️ non-default |
| **Max Frame Rate** | **Off** ⚠️ |
| Monitor Technology | G-SYNC Compatible |
| OpenGL GDI Compatibility | Auto |
| **Power Management Mode** | **Normal** ⚠️ |
| RTX Dynamic Vibrance | Off |
| RTX HDR | Off |
| Shader Cache | Default |
| **Vertical Sync** | **Use 3D app setting** ⚠️ |
| VR — Variable Rate Super Sampling | Off |
| Vulkan/OpenGL Present Method | Auto |
| Anisotropic Filtering | Application-controlled |
| Antialiasing — FXAA | Off |
| Antialiasing — Transparency | Off |
| Background Application Max Frame Rate | Off |
| Multi-Frame Sampled AA (MFAA) | Off |
| PhysX | Auto |
| Texture Filtering — Anisotropic Sample Optimization | Off |
| Texture Filtering — Negative LOD Bias | Allow |
| Texture Filtering — Quality | Quality |
| Texture Filtering — Trilinear Optimization | On |

---

## 7. Program Settings — F1 23

Status: **"Game is optimized"** by NVIDIA. Quality slider sits at *Recommended*.

**In-game settings NVIDIA reports**

| Setting | Value |
|---|---|
| Ambient Occlusion | HBAO+ |
| Anisotropic Filtering | 16x |
| **Anti-aliasing** | **NVIDIA DLSS** (mode not reported) |

**Driver settings for this program** — all inherit global unless noted

| Setting | Value |
|---|---|
| DLSS Override — Model Presets | Unsupported |
| DLSS Override — Frame Generation | Unsupported |
| DLSS Override — Super Resolution | Unsupported |
| Smooth Motion | Global — Off |
| Low Latency Mode | Global — Off |
| CUDA — Sysmem Fallback Policy | Global — Driver Default |
| DSR — Factors | Global — Off |
| GPU App Assignment | Global — RTX 4070 Ti SUPER |
| Image Scaling | Off |
| **Max Frame Rate** | **Global — Off** ⚠️ |
| Monitor Technology | Global — G-SYNC Compatible |
| OpenGL GDI Compatibility | Global — Auto |
| Power Management Mode | Global — Normal |
| RTX Dynamic Vibrance | Global — Off |
| RTX HDR | Global — Off |
| **Vertical Sync** | **Global — Use 3D app setting** ⚠️ |
| VR — Variable Rate Supersampling | Off |
| Vulkan/OpenGL Present Method | Global — Auto |
| Anisotropic Filtering | Global — Application-controlled |
| Antialiasing — FXAA | Global — Off |
| Antialiasing — Transparency | Global — Off |
| Background Application Max Frame Rate | Global — Off |
| Multi-Frame Sampled AA (MFAA) | Global — Off |
| Texture Filtering — Anisotropic Sample Optimization | Global — Off |
| Texture Filtering — Negative LOD Bias | Global — Allow |
| Texture Filtering — Quality | Global — Quality |
| Texture Filtering — Trilinear Optimization | Global — On |

**Other programs with NVIDIA profiles (10 total):** F1 23, Assetto Corsa, Assetto Corsa Competizione,
Assetto Corsa Rally, Assetto Corsa EVO, DiRT Rally 2.0, Euro Truck Simulator 2, SOLIDWORKS, OBS
Studio, VLC.

---

## 7b. F1 23 In-Game Graphics Settings

Captured 2026-08-26. UI is French.

**Display**

| Setting | Value |
|---|---|
| Résolution | 5120 x 1440 |
| Mode d'affichage | **Fenêtré (plein écran)** — borderless |
| Format de l'image | Auto |
| Synchronisation verticale | Non ✅ (driver handles it) |
| Intervalle de synchronisation verticale | Auto |
| Fréquence de rafraîchissement | 143 Hz |
| Limite d'IPS | **Oui** |
| IPS maximales | **138** |
| Périphérique d'affichage | 1 |
| Filtrage anisotrope | 16x |
| Anticrénelage | NVIDIA DLSS |
| Mode anticrénelage | **Équilibré** (Balanced) |
| Netteté anticrénelage | 50 |
| Résolution dynamique | Non ✅ |
| **HDR** | **Activé (scRGB)** ⚠️ see §9 |

**Image / post**

| Setting | Value |
|---|---|
| Correction gamma | 100 |
| Intensité du flou cinétique | 20 |
| Animation de la direction | Oui |
| Ajustement de la netteté optimale | 92 |
| Aberration chromatique | Non |

**Ray tracing — all enabled** ⚠️

| Setting | Value |
|---|---|
| Qualité du lancer de rayon | **Élevée** |
| Lancer de rayon : Ombres | Oui |
| Lancer de rayon : Reflets | Oui |
| Lancer de rayon : Occlusion ambiante | Oui |
| Reflets transparents en lancer de rayon | Oui |
| **Lancer de rayon DDGI** | **Oui** — most expensive single option |

**Quality**

| Setting | Value |
|---|---|
| Qualité des détails | Personnalisée |
| Qualité de l'éclairage | Ultra-élevée |
| Qualité du post-traitement | Élevée |
| Qualité des ombres | Ultra-élevée |
| Qualité des particules | Élevée |
| Qualité de la foule | Ultra-élevée |
| Qualité des rétroviseurs | Ultra-élevée |
| Qualité des reflets (voitures et casques) | Ultra-élevée |
| Qualité des effets de la météo | Ultra-élevée |
| Qualité des surfaces | Ultra-élevée |
| Qualité des arbres | Ultra-élevée |
| Présence de traces de dérapage | Élevée |
| Transparence des traces de dérapage | Oui |
| Occlusion ambiante | HBAO+ ⚠️ redundant with RT AO |
| Qualité des reflets de surface (SSR) | Ultra-élevée ⚠️ redundant with RT Reflets |
| Calcul asynchrone | Oui |
| Qualité des textures | Ultra-élevée |
| Ombrage à taux variable (VRS) | Non |
| Cheveux de haute qualité | Oui |
| **NVIDIA Reflex** | **Oui + boost** ✅ |

---

## 8. NVIDIA App Settings

| Setting | Value |
|---|---|
| NVIDIA Overlay | **On** (Alt+Z) |
| Game Filters & Photo Mode | Off |
| Auto-download drivers | Off |
| Auto-optimize newly added games and apps | On — see §9, preference not a fault |
| Scan locations | 3 |
| Games and apps detected | 10 |
| Language | English (US) |
| Early Access / Beta | Opted out |
| Telemetry — required data | On (cannot disable) |
| Telemetry — configuration/performance/usage | On |
| Telemetry — error and crash data | Off |

**Video (RTX Video Enhancements)**

| Setting | Value |
|---|---|
| Super Resolution | Off |
| HDR | Off |

---

## 9. Findings & Recommended Changes

Ordered by expected impact on the F1 23 stutter.

### Critical

**1. G-SYNC is not actually active.** The per-display toggle is OFF while the global setting reads
"On, Full screen". With the GPU pinned at 98 % utilization the framerate fluctuates constantly, but
the panel refreshes on a rigid 144 Hz clock — frames land out of step, some shown twice, some
dropped. That is the stutter.

**2. The link is HDMI, classified `HDTV`.** The Neo G9 is a DisplayPort 1.4 panel. Over HDMI the
driver treats it as a television, which greys out scaling control and is the most likely reason the
VRR toggle will not engage. **Move to DisplayPort, then enable the per-display G-SYNC toggle.**

> Trade-off: this ultrawide cannot display Windows Recovery / Safe Mode over DisplayPort because the
> basic display driver cannot negotiate DSC. If HDMI was chosen deliberately for that reason, keep
> the HDMI cable connected to the second input and switch inputs when recovery is needed — Safe Mode
> access is still required to remove ESET.

**3. No frame cap.** `Max Frame Rate` is Off both globally and for F1 23. Once G-SYNC works, cap
around **138 fps** so the framerate stays inside the VRR window instead of bouncing off the ceiling.

**4. Vertical Sync is `Use 3D app setting`.** Combined with G-SYNC and a cap below refresh, forcing
V-Sync **On** at driver level acts as a backstop without adding latency. This trio is the standard
VRR configuration.

### Worth changing

**5. Power Management Mode is `Normal`.** This lets the GPU drop clocks during momentary load dips,
which produces frametime variance. Set to **Prefer Maximum Performance** for F1 23.

**6. Global Image Scaling is enabled** — Sharpen 50 at 85 % render resolution (4352 x 1224). This is
a non-default upscaler running globally, while F1 23 separately uses **DLSS**. The per-program entry
shows Image Scaling Off, so they should not be stacking — but two upscalers configured at once is
worth resolving. Turn global Image Scaling **Off** and let DLSS do the work.

**7. Driver 591.86 dates from 27 Jan 2026** — seven months stale. It is also the top DPC consumer
measured by LatencyMon (1 284 µs, the only driver over 1 ms).

### Added 2026-08-26 after seeing the in-game settings

**9. HDR is enabled in F1 23 as `Activé (scRGB)`** while Windows HDR is off (no `AdvancedColorEnabled`
value exists). scRGB is FP16 — double the framebuffer bandwidth of SDR — and in borderless windowed
it pushes DWM into HDR composition, breaking independent flip. Leading candidate for the measured
**52.4 ms** PC latency. **Turn it off and retest.**

**10. Ray tracing is fully on at `Élevée`** — shadows, reflections, AO, transparent reflections and
DDGI, on top of near-universal Ultra settings at 5120×1440. Explains 96 % GPU at 95 FPS even with
DLSS Balanced, and RT is a known cause of random hitching.

**11. Two redundant pairs are both enabled:** HBAO+ alongside RT Ambient Occlusion, and SSR Ultra
alongside RT Reflections. In each case the RT option supersedes the raster one — the raster pass is
pure waste. Also `Ombrage à taux variable` (VRS) is Off; enabling it recovers performance cheaply.

### Preference, not a fault

**8. "Auto-optimize newly added games and apps" is On.** This does **not** affect the stutter — it
only touches *newly added* titles, and F1 23 is already in the list. Noted because NVIDIA optimizes
for visual quality at a playable framerate, whereas this rig is tuned for frametime consistency and
input latency; the two targets diverge. One concrete risk: if F1 23 is ever reinstalled or its Steam
library moves, it counts as newly added and any tuning here gets overwritten. Leaving it on is a
reasonable choice for unfamiliar new sims.

### Confirmed healthy — do not chase these

- Colour output: RGB **Full** range, 10 bpc, no adjustments
- VRAM: 5.1 GB free under load — not a constraint
- Temps: 64 °C under load against an 84 °C target
- No overclock; power limit at stock 100 %
- Refresh mismatch (144 vs 143 Hz) is a symptom of the HDMI timing path, not a fault in itself

---

## Related

- [`SETUP_AUDIT.md`](SETUP_AUDIT.md) — full rig hardware and software audit
- [`Hardware/Button_Box/README.md`](Hardware/Button_Box/README.md) — power sequencing panel project
