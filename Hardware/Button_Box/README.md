# Button Box: Hardware Spec

Power sequencing and control panel for the sim rig. Mounted on the right-hand side.

---

## 1. Context

**What it does**
- Physically powers the rig on/off in a safe, enforced order
- Displays live system + sensor status
- Controls wind fan speed

**What it does NOT do** (deliberately out of scope)
- Software shortcuts: handled by the existing Stream Deck (left side)
- In-game functions (BB / DIFF / TC / ABS / MAP): separate future project
- FFB strength / seat intensity: stays in software

**Architecture**
- Gatekeeper ESP32: state machine, reads inputs, drives LEDs, talks to Shelly devices over local network
- CrowPanel 3.5": semi-independent display client, polls gatekeeper + LibreHardwareMonitor + BME280
- All device-to-device comms are local network (mDNS / local IP), no cloud

---

## 2. Front Panel Components

| # | Component | Qty | Function |
|---|---|---|---|
| 1 | Ignition push button | 1 | PC power only (momentary) |
| 2 | Toggle switch, RGB LED ring, 16mm | 3 | Wheelbase / Screen / Seat requests |
| 3 | Rotary encoder EC11 + alu knob | 1 | Fan speed (push = AUTO/manual toggle) |
| 4 | WS2812 ring, 24 LED | 1 | Fan power arc gauge (half-arc visible) |
| 5 | CrowPanel ESP32 3.5" | 1 | Status display, tilted 15-20 deg toward driver |
| 6 | Emergency stop, mushroom head | 1 | Seat circuit, hard-wired, bypasses all logic |

**Not on this panel**
- Wheelbase E-stop: Moza's own unit, already owned, must stay **spatially separate** from the seat E-stop
- BME280 sensor: must sit in open air, NOT sealed inside the enclosure (it reads ambient temp/humidity)

---

## 3. Panel Cut-Out Dimensions

> **Status legend:** `CONFIRMED` = industry standard or manufacturer spec. `VERIFY` = measure the actual part with calipers before cutting.

| Component | Cut-out Ø | Body depth behind panel | Status |
|---|---|---|---|
| Toggle 16mm (DaierTek TS16-11EL-A-RGB) | 16.0 mm | ~35 mm incl. wire loom | CONFIRMED |
| E-stop XB2-542 | 22.0 mm | **~60 mm** (contact block is long) | CONFIRMED |
| Push button 22mm (backup unit) | 22.0 mm | ~30 mm incl. connector | CONFIRMED |
| Ignition button (ENGINE START STOP) | ~22 mm assumed | ~30 mm | **VERIFY** |
| Encoder EC11 | 7.0 mm (M7 nut) | ~25 mm | CONFIRMED |
| CrowPanel 3.5" | rectangular, see §5 | ~20 mm | **VERIFY** |

**Component outer dimensions (not cut-outs)**

| Component | Size |
|---|---|
| Alu knob (fan) | Ø40 x 19 mm, 6mm D-shaft bore |
| WS2812 ring, 24 LED | Ø65 mm outer |
| E-stop mushroom head | Ø40 mm |
| CrowPanel active display area | 48.96 x 73.44 mm (W x H) |

---

## 4. Spacing Rules (critical)

The cut-out diameter is **not** what limits spacing. The **retaining nut behind the panel** does.

| Cut-out | Nut OD (approx) | Min. centre-to-centre |
|---|---|---|
| 16 mm | ~20 mm | **25 mm** |
| 22 mm | ~28 mm | **32 mm** |

- Mixed 16 + 22 mm neighbours: use **30 mm** minimum
- Add clearance for a socket/spanner, not just the nut itself
- Panel thickness: check each component's max panel thickness rating (typically 1 to 5 mm). A thick 3D printed plate can prevent the nut from engaging.

**Fan control cluster**
- Knob Ø40 mm, ring Ø65 mm, concentric
- Leaves 12.5 mm of ring visible per side
- Reserve a Ø70 mm keep-out zone around the encoder centre

---

## 5. Where to Get the Remaining Dimensions

| Part | Source | Notes |
|---|---|---|
| CrowPanel 3.5" | [Elecrow GitHub repo](https://github.com/Elecrow-RD/CrowPanel-3.5-HMI-ESP32-Display-480x320) | Contains a `3D File` folder with **.stp (STEP)** models. **Import directly into Fusion 360**, do not re-model. |
| CrowPanel electrical spec | [Elecrow Wiki](https://www.elecrow.com/wiki/esp32-display-352727-intelligent-touch-screen-wi-fi26ble-320480-hmi-display.html) | Active area confirmed here, full PCB outline is not published |
| EC11 encoder | Generic EC11 datasheet | Check for an anti-rotation lug: it needs an extra small hole next to the 7mm bore |
| Everything else | **Calipers, on arrival** | AliExpress listing dimensions are unreliable |

---

## 6. Before You Cut

- [ ] Measure every part with calipers, do not trust listing specs
- [ ] Confirm the ignition button cut-out (listing gives no dimensions)
- [ ] Check max panel thickness per component vs. your printed plate thickness
- [ ] Verify DaierTek LED polarity (common anode vs common cathode): decides ULN2803 low-side vs high-side driver
- [ ] Dry-fit all parts in a test print (single strip with all cut-outs) before printing the full plate
- [ ] Confirm nut clearance with parts physically side by side
- [ ] Keep the BME280 outside the sealed volume, away from any heat source

---

## 7. Open Items

| Item | Blocking | Owner |
|---|---|---|
| Motion platform power rating | Validates E-stop can break load directly (XB2 AC-15 rating is ~3A/240V, well under its 10A thermal rating) | Leo |
| Corsair controller model + which fans | iCUE SDK support level for fan override | Leo |
| Available panel area (W x H x D) | Full layout | Leo |
| Spare button count and format | Front plate hole count | Leo |
| Maintenance mode control | Recommended as a dedicated physical switch, not a long-press | Decision |

---

## 8. Safety Notes

- The E-stop is wired **in-line on mains**, in series, bypassing all smart logic. This is mains work.
- E-stop is **2NC** (double pole): French outlets are not polarised, breaking a single pole can leave live present.
- If the motion platform exceeds ~600W, the E-stop should break a **contactor coil**, not the load directly.
- Shelly devices: buy from an official reseller, **not** AliExpress. Mains-connected hardware, counterfeits are common.
- Every relay / smart plug must be configured to default **OFF** after power loss.
