# Button Box: Hardware Spec

Power sequencing and control panel for the sim rig. Mounted on the right-hand side.

---

## 1. Context

**What it does**
- Physically powers the rig on/off in a safe, enforced order
- Actuates the wheelbase power button mechanically (it is momentary, not latching — see §3)
- Displays live system + sensor status
- Controls two wind-feel fans, from live telemetry or manual override

**What it does NOT do** (deliberately out of scope)
- Software shortcuts: handled by the existing Stream Deck (left side)
- In-game functions (BB / DIFF / TC / ABS / MAP): separate future project
- FFB strength / seat intensity: stays in software
- **USB bus-powered peripherals**: pedals, Universal Hub, handbrake, sequential and H-pattern
  shifters all draw power from USB and follow the PC automatically. No relay, no toggle, no LED,
  no sequencing logic. Out of scope entirely.

**Architecture**
- Gatekeeper ESP32: state machine, reads inputs, drives LEDs, drives the fan PWM and the wheelbase
  servo, talks to Shelly devices over local network, listens for telemetry over UDP
- CrowPanel HMI: semi-independent display client, polls gatekeeper + LibreHardwareMonitor + BME280
- All device-to-device comms are local network (mDNS / local IP), no cloud
- Sequencing logic lives on the gatekeeper, **not** in Shelly scripts: the ESP32 drives the plugs
  over Shelly's local HTTP RPC. No hub, no Home Assistant, no cloud account.

**Only four loads need sequencing logic**

| Load | Control | Gating |
|---|---|---|
| PC | Ignition button → Shelly plug, confirmed by wattage | Full sequence |
| Wheelbase | Shelly plug **+ servo button press** (§3) | Off only when idle / game closed |
| Screen | Shelly plug | None — always safe to cut |
| Seat | Shelly plug | Off only when idle / game closed |

---

## 2. Front Panel Components

| # | Component | Qty | Function |
|---|---|---|---|
| 1 | Ignition push button | 1 | PC power only (momentary) |
| 2 | Toggle switch, RGB LED ring, 16mm | 3 | Wheelbase / Screen / Seat requests |
| 3 | Selector switch, 22mm, 3-position, maintained | 1 | Fan **manual override**. L = LOW, centre = AUTO, R = HIGH |
| 4 | Selector switch, 22mm, 3-position, maintained | 1 | Fan **telemetry gain**. Only active when #3 = AUTO |
| 5 | CrowPanel ESP32 HMI | 1 | Status display, tilted 15-20 deg toward driver. **Size unresolved, see §12** |
| 6 | Emergency stop, mushroom head | 1 | Seat circuit, hard-wired, bypasses all logic |
| 7 | Spare push buttons | 0-4 | Drawn in the Fusion model, **function unassigned — deliberately not ordered**. §1 puts in-game functions out of scope, so there is nothing in scope for them to do yet. The candidate parts are symbol-printed (horn / headlight / etc.), and choosing the icon *is* choosing the function permanently. Decide the functions first, then buy (§12) |

**Superseded — the fan rotary encoder is removed**

The EC11 encoder + Ø40 alu knob + 24-LED WS2812 ring are **no longer part of this build**. Fan
control moved to the two fixed-position selectors above, whose physical position is self-indicating
even with the rig fully powered off — no LED or display needed to read the setting. The already-
ordered encoder, knob and ring become spares. This frees the Ø70 mm keep-out zone that previously
dominated the panel layout, and frees 2 native ESP32 GPIO (see §5).

**Not on this panel**
- Wheelbase servo actuator: mounts at the wheelbase itself, not here (§3)
- Wheelbase E-stop: Moza's own unit, already owned, must stay **spatially separate** from the seat E-stop
- BME280 sensor: must sit in open air, NOT sealed inside the enclosure (it reads ambient temp/humidity)

---

## 3. Off-Panel Actuators

### 3.1 Wheelbase servo press (required — the plug alone cannot do it)

The Moza wheelbase's rear power button is a **momentary pushbutton, not a latching switch**:

- Power ON = short press
- Power OFF = long press (press and hold)

Energising or de-energising the smart plug therefore does **not** turn the wheelbase on or off. The
physical button must be pressed each time, regardless of mains state. A smart plug alone is not
sufficient for this load.

**Solution:** a hobby servo (SG90 / MG90S class) on a 3D-printed bracket, with a printed finger on
the horn positioned to depress the rear button. Non-invasive — no soldering into the wheelbase, no
warranty risk. Driven from one native PWM-capable GPIO on the gatekeeper.

| Sequence | Steps |
|---|---|
| **Start** | Shelly plug ON → wait ~1 s for the wheelbase to settle → servo short press-and-release |
| **Stop** | Servo press-and-**hold** for the long-press duration → release → then Shelly plug OFF |

- Long-press hold duration is **unmeasured** — start at 3-5 s and tune on real hardware (§12)
- All press/hold/settle durations are firmware constants, not inline literals

**Mechanical and electrical rules**

- The parked arm must sit with **visible clearance** off the button. If the park position rests on
  the button, a brownout or servo jitter holds the wheelbase button down indefinitely.
- **Stop driving the PWM signal after each move** (detach). A continuously driven servo hums, heats,
  and fights its own gearbox against the button.
- Firmware watchdog: never drive the arm beyond a hard maximum duration, whatever the state machine
  thinks it is doing.
- A servo stall pulls ~0.7 A (SG90) to ~1.5 A (MG90S) at 5 V. Fit a **470-1000 µF electrolytic**,
  or the inrush at the start of each press browns out the ESP32 mid-sequence. **Mount the cap at the
  servo end of the cable, not at the LM2596** — a metre of thin extension wire has enough resistance
  that a cap at the supply end cannot respond to the inrush.
- **Give the servo its own LM2596.** The 3-pack covers it: one module for logic, one for the servo,
  so a stall current cannot drag the logic rail down at all. Common ground between them is required.

**Mounting.** The bracket attaches with VHB tape — the wheelbase must not be drilled. Design a flat
pad of at least ~15 x 40 mm against the casing for the tape to grip; a small contact patch creeps
under repeated servo torque, and a bracket that shifts 2 mm stops pressing the button.

**Close the loop with the Shelly, do not run this open-loop.** The Shelly Plug M Gen3 measures power.
Read the wattage back after the press: the wheelbase draws measurably more when on than when idle on
mains. If the expected change does not appear, retry the press once, then flag a fault on the display.
This is the same wattage-confirmation trick already used for the PC ignition, and it is the only way
the box can know whether a blind mechanical press actually worked.

### 3.2 Wind fans

Two fans, driven by PWM from the gatekeeper. See §6 for the control logic.

- **Fan type is unspecified and it gates the PSU sizing.** A pair of real wind-feel fans can pull
  several amps at 12 V — far beyond the 2 A supply currently planned in §4. Choose the fans, then
  size the PSU. See §12.
- 4-pin PC fans: PWM the control wire at ~25 kHz, fan stays powered from 12 V continuously. The
  Intel 4-wire spec expects an **open-drain** drive — use a small N-channel MOSFET (2N7000 class)
  pulling the PWM line low, not a direct 3.3 V push-pull from the ESP32.
- 2/3-pin fans: low-side MOSFET switching the ground. Loses the tach signal and can whine at low duty.

---

## 4. Bill of Materials & Order Status

> Audited against the AliExpress and Amazon.fr carts on 2026-08-25.

### Ordered / in cart

| Part | Qty | Price | Source | Note |
|---|---|---|---|---|
| **ESP32-S3-N8R2 dev board, 44-pin, USB-C** (gatekeeper) | 2 | 6,19 € ea | AliExpress [1005006418608267](https://fr.aliexpress.com/item/1005006418608267.html), Top Tech Store (4,9★, 10 000+ sold). **N8R2, not N16R8** — see §5. Two units clears the seller's 10 € free-shipping threshold, and the spare matters: this board is the single point of failure for the whole box |
| CrowPanel ESP32 3.5" HMI 480x320 | 1 | 23,59 € | AliExpress, Intelligent IOT Store | **Does not match the 5" model in Fusion — resolve before ordering, §12** |
| DaierTek 16mm RGB toggle, 3-pack | 1 | 15,29 € | AliExpress, Daier Store | Matches TS16-11EL-A-RGB in §7 |
| Ignition push button, 12V | 1 | 5,89 € | AliExpress, Car Engineer | Cut-out unconfirmed, see §7 |
| Metal LED push button, 22mm | 1 | 3,29 € | AliExpress, Hate Mondays | Backup unit if the ignition button cut-out fails |
| BME280, 3.3V | 1 | 5,69 € | AliExpress, EXinko | **Verify it is not a BMP280 on arrival**, see §10 |
| MCP23017 I2C I/O expander | 1 | 2,28 € | AliExpress, SZHJW | No PWM — see §5 and §12 |
| ULN2803 Darlington array, 10-pack | 1 | 1,89 € | AliExpress, Topchip | Low-side driver for the 12V toggle LEDs |
| LM2596 buck converter, **3-pack** | 1 | ~2,80 € | AliExpress, Shenzhen Yuxinxin | 12V → 5V. **Check the pack size — the 10-pack (8,29 €) has been selected by mistake before.** Adjustable output: set to 5,00 V with a meter and verify **before** connecting the ESP32, see §10 |
| Shelly Plug M Gen3, 4-pack, 13A/3000W | 1 | 38,00 € | **Amazon.fr, Shelly Europe** | Official reseller per §13. Exactly 4 loads: PC / wheelbase / screen / seat |

**Now spare, not part of the build** (ordered before the encoder was dropped, §2):
EC11 encoder 5-pack 3,79 € · Alu knob Ø40 2,26 € · WS2812B ring 24-LED 5,19 €

The Ø40 alu knob **cannot be reused on the selectors** — the 22 mm selector heads come with their own
moulded lever and take no aftermarket knob at all.

### Not yet ordered

| Part | Qty | Est. | Blocking |
|---|---|---|---|
| **E-stop, XB2-542 mushroom head, 22mm, 2NC** | 1 | 3,69 € | §2 item 6. Panel layout cannot be finalised without it |
| **XB2-BJ33 selector, 22mm, 3-position, 2NO, self-locking, long handle** | 3 (2 + spare) | 1,22 € ea | §2 items 3-4. AliExpress [1005010214322970](https://fr.aliexpress.com/item/1005010214322970.html), ABILKEEN. **BJ = long lever, BD = short knob.** See the variant trap below |
| **Servo, MG90S (metal gear)** | 2 | ~3 € ea | §3.1. Metal gear, **not SG90** — plastic gears strip on a stiff button. One spare |
| **Servo extension cable, 3-wire JR/Futaba, 50-100 cm** | 1-2 | ~2 € | §3.1. Servos ship with ~20 cm; the wheelbase is 50-100 cm from the panel |
| **3M VHB double-sided tape** | 1 roll | ~3 € | §3.1. Mounts the servo bracket without drilling the wheelbase |
| **12V PSU, always-on** | 1 | ~8 €+ | Nothing in the BOM generates 12V. **Size after the fans are chosen**, §3.2 |
| **Wind fans** | 2 | ? | §3.2. Gates PSU sizing |
| 2N7000 MOSFETs (fan PWM drive) | 2 | ~1 € | §3.2 |
| 4.7 kΩ resistors (selector pull-ups) | 4 | ~1 € | §5 — wetting current. Do not rely on the internal pull-ups |
| 470-1000 µF electrolytic, 10V+ | 1 | ~1 € | §3.1 servo inrush |
| Dupont / JST leads, protoboard | — | ~3 € | Assembly |

**Selector variant trap.** The XB2 grid puts thirteen near-identical swatches side by side. Only
**BJ33** works. Confirm the swatch text reads *3 position / 2NO / self locking* — the code alone is
not enough at thumbnail resolution.

| Variant | Why it fails |
|---|---|
| **BJ35** | 3-position but 1NO1NC — the NC contact is closed at rest, breaking the §5 decode table |
| **BJ51 / BJ53** | Self-reset (spring return) — springs back to centre, cannot hold LOW or HIGH |
| BJ41 / BJ42 | 2-position, spring return |
| BJ21 - BJ25 | 2-position |
| BD-prefix | Correct electrically, but short knob instead of the long lever |
| Contactor for the seat circuit | 0-1 | ~15 € | **Conditional** — only if the ForceSeat exceeds ~600 W, see §12 and §13 |

---

## 5. I/O Architecture

### Power rails

The toggle switches and the ignition button are 12V parts; the ESP32 and MCP23017 are 3.3V.
The LM2596 steps 12V down to 5V, but **nothing in the BOM generates the 12V itself** — a dedicated
supply is required.

That supply must be **always on and independent of the PC**. It cannot come off a Molex or SATA rail,
because the gatekeeper has to be alive while the PC is off in order to switch the PC on. That is the
whole purpose of the box.

### Pin placement rules

| Signal | Where it must live | Why |
|---|---|---|
| Fan selector contacts (4) | **Native ESP32 GPIO**, digital | 2 lines per 3-position switch; only 3 MCP pins are free, so keep these native |
| Fan PWM out (2) | **Native ESP32 GPIO** (LEDC) | Needs hardware PWM at ~25 kHz |
| Servo signal (1) | **Native ESP32 GPIO** (LEDC) | Needs hardware PWM at 50 Hz |
| Toggle switch inputs (3) | MCP23017 | Slow, human-speed |
| Toggle RGB LED channels (9) | MCP23017 → ULN2803 | Slow, human-speed |
| Ignition button input | MCP23017 | Slow, human-speed |
| BME280 | I2C, shared bus | |

### Reading the 3-position selectors

Each selector carries **two contact blocks**: left closes one, right closes the other, centre closes
neither. Two digital lines per switch, four in total, decoded in firmware:

| Line A | Line B | State |
|---|---|---|
| closed | open | Left |
| open | open | **Centre** |
| open | closed | Right |
| closed | closed | Fault — impossible, flag it |

The MCP23017 has only 3 free pins (13 of 16 used), so put all four on **native ESP32 GPIO**. There
is plenty spare now that the encoder and WS2812 ring are gone. An earlier revision of this spec used
a resistor ladder into the ADC; that was only necessary for 6-position switches and no longer applies.

**Wetting current — do not skip this.** These are industrial contact blocks with silver-alloy contacts
designed to switch AC at amps. At 3.3 V and microamps they never self-clean, an oxide film builds up,
and the switch starts reading intermittently after months of use. The ESP32's internal pull-up sources
only ~50 µA, nowhere near enough.

- Fit an **external 4.7 kΩ pull-up** on each of the four lines (~0.7 mA) instead of relying on the
  internal one
- Debounce ~20 ms in firmware; the lever passes through centre during every throw
- Treat the both-closed case as a fault, not as a valid position

### Proposed pin map (ESP32-S3-N8R2, 44-pin)

| Signal | GPIO | Note |
|---|---|---|
| Selector 1 — contacts A / B | 4, 5 | External 4.7 kΩ pull-ups regardless (wetting current, above) |
| Selector 2 — contacts A / B | 6, 7 | Same |
| I2C SDA / SCL | 8, 9 | MCP23017 + BME280 share the bus |
| Fan PWM 1 / 2 | 15, 16 | LEDC, ~25 kHz, via 2N7000 open-drain |
| Servo | 17 | LEDC, 50 Hz |

**Pins to keep clear on the S3:** GPIO 26-32 are wired to the SPI flash. GPIO 43/44 are UART0.
GPIO 0, 45 and 46 are boot-strapping pins. GPIO 19/20 are USB D-/D+ — leave them free, see below.

**Why N8R2 and not N16R8:** the N16R8's *octal* PSRAM consumes GPIO 33-37. The N8R2's *quad* PSRAM
leaves them free. This project needs GPIO, not memory.

**Native USB is the reason for choosing the S3.** §1 defers in-game functions to a later project;
the way to build that is a USB HID game controller, which the S3 does natively and the classic
ESP32-WROOM-32 cannot. Keep GPIO 19/20 unused so that path stays open.

**Power the board from the LM2596's 5 V into `VIN`/`5V`**, never into `3V3` — that bypasses the
onboard regulator and destroys the module. If USB is also connected for HID, two 5 V sources are
present at once: check the board's USB input diode before running both, and never let the 5 V rail
back-feed the PC's USB port.

### The 12V / 3.3V boundary

12V must never reach an ESP32 or MCP23017 pin — 3.3V is the absolute maximum and 12V will destroy it.

- Sense every switch as a **dry contact to GND** (external pull-up on the selectors, see above)
- The selector contacts must be pulled up to **3.3V**, never to the 12V rail
- Keep 12V strictly on the **output** side of the ULN2803, driving the LEDs only
- The switch's 20A contact rating is irrelevant here; it is being used as a logic input

---

## 6. Fan Control Logic

Fans are driven by live telemetry over UDP (SimHub-style: speed / G-force / RPM, updating many times
per second), with a hard manual override.

```
if Switch1 != AUTO:
    fan_output = MANUAL_STEPS[Switch1_position]        # telemetry fully ignored
else:
    fan_output = live_telemetry_value * GAIN[Switch2_position]
```

- **The manual switch takes full priority.** Off the AUTO position, telemetry processing for fan
  output is bypassed entirely — not blended, not dampened, not rate-limited against it.
- **Telemetry output stays continuous.** The discreteness applies only to the two selectors' own
  settings (mode and gain), never to the telemetry-driven fan curve itself. On AUTO the fan tracks
  the telemetry smoothly and in real time.
- Suggested starting maps, both tunable in firmware without rewiring:
  - `MANUAL_STEPS` = LOW 35% / **AUTO** / HIGH 100%
  - `GAIN` = 60% / 100% / 140%

> The original spec called for 6 manual steps and 5 gain steps. The 22mm industrial selector family
> (§2) only exists in 2- and 3-position variants, so both dropped to 3. The priority logic below is
> unchanged — only the number of steps differs.
- Clamp the final output after gain is applied — 120% gain on a high telemetry value must not
  overflow the duty cycle.

**Decide: what do the fans do when the PC is off?** The fans run from the always-on 12V rail, so a
manual override position keeps them spinning with the rig otherwise dead. That is either a feature
(cooling without racing) or an irritation (fans running all night because the knob was left at 80%).
Options: honour it, or force output to 0 unless the PC is on. Also define the behaviour when UDP
telemetry stops arriving mid-session — fail to 0, or hold the last value briefly then decay. See §12.

---

## 7. Panel Cut-Out Dimensions

> **Status legend:** `CONFIRMED` = industry standard or manufacturer spec. `VERIFY` = measure the actual part with calipers before cutting.

| Component | Cut-out Ø | Body depth behind panel | Status |
|---|---|---|---|
| Toggle 16mm (DaierTek TS16-11EL-A-RGB) | 16.0 mm | ~35 mm incl. wire loom | CONFIRMED |
| E-stop XB2-542 | 22.0 mm | **~60 mm** (contact block is long) | CONFIRMED |
| Push button 22mm (backup unit) | 22.0 mm | ~30 mm incl. connector | CONFIRMED |
| Ignition button (ENGINE START STOP) | ~22 mm assumed | ~30 mm | **VERIFY** |
| Selector XB2-BJ33, 3-position | 22.0 mm **+ anti-rotation notch**; bushing measures 21.7 mm | **~42 mm** incl. contact blocks | CONFIRMED (seller drawing) |
| CrowPanel | rectangular, see §9 | ~20 mm + connectors | **VERIFY** |

**The E-stop (~60 mm) is the deepest part and sets the minimum enclosure depth on its own.** The
XB2-BJ33 selectors measure ~42 mm including their contact blocks — an earlier revision of this spec
estimated 60-70 mm from the Schneider family datasheet, which was wrong for this part.

The panel is now four 22 mm cut-outs (E-stop, ignition, 2 selectors) plus three 16 mm toggles.
All 22 mm neighbours need **32 mm centre-to-centre** and ≥16 mm to any edge (§8) — this is a
material change from the 10 mm selector holes assumed in the previous revision, and the Fusion
layout must be redrawn around it.

**Component outer dimensions (not cut-outs)**

| Component | Size |
|---|---|
| E-stop mushroom head | Ø40 mm |
| CrowPanel 3.5" active display area | 48.96 x 73.44 mm (W x H) |
| XB2-BJ33 body behind panel | **29.6 x 29.3 mm, square** |
| XB2-BJ33 lever above panel | ~28 mm tall |

**Selector spacing is set by the knob, not the nut.** The 10 mm hole is misleadingly small — a
Ø20-25 mm pointer knob needs **≥35 mm centre-to-centre** for finger clearance, and clearance to the
E-stop mushroom (Ø40) matters more than either.

---

## 8. Spacing Rules (critical)

The cut-out diameter is **not** what limits spacing. The **retaining nut behind the panel** does.

| Cut-out | Nut OD (approx) | Min. centre-to-centre |
|---|---|---|
| 16 mm | ~20 mm | **25 mm** |
| 22 mm | ~28 mm | **32 mm** |

- Mixed 16 + 22 mm neighbours: use **30 mm** minimum
- **The XB2-BJ33 breaks the nut-OD assumption.** Its body behind the panel is a ~29.6 mm *square*,
  wider than the round nut the table above is based on. At 32 mm centres two adjacent selectors clear
  each other by only 2.4 mm. Model both square bodies and check, or space them **35 mm** apart.
- **Edge margin**: a 22 mm part needs its centre ≥16 mm from any panel edge, plus room to get a
  spanner on the nut. Parts placed near the outline are the easiest spacing mistake to make.
- Add clearance for a socket/spanner, not just the nut itself
- Panel thickness: check each component's max panel thickness rating (typically 1 to 5 mm). A thick 3D printed plate can prevent the nut from engaging.

---

## 9. Where to Get the Remaining Dimensions

| Part | Source | Notes |
|---|---|---|
| CrowPanel 3.5" | [Elecrow GitHub repo](https://github.com/Elecrow-RD/CrowPanel-3.5-HMI-ESP32-Display-480x320) | Contains a `3D File` folder with **.stp (STEP)** models. **Import directly into Fusion 360**, do not re-model. STEP already downloaded and verified. Elecrow sells several 3.5" variants — confirm the PCB outline against the STEP file before cutting. |
| CrowPanel 5.0" | Elecrow's repo for the 5" part, if that size is chosen | **The 3.5" STEP does not apply.** A generic 5" LCD placeholder is not accurate enough to cut a panel from — PCB outline, mounting holes and connector positions all differ. |
| CrowPanel electrical spec | [Elecrow Wiki](https://www.elecrow.com/wiki/esp32-display-352727-intelligent-touch-screen-wi-fi26ble-320480-hmi-display.html) | Active area confirmed here, full PCB outline is not published |
| Selectors, 22mm 3-position | [Schneider Harmony XB5 reference](https://www.se.com/us/en/product/XB5AD25/selector-switch-harmony-xb5-plastic-black-22mm-2-positions-stay-put-1no+1nc/) | The clones (XB2-BD33 / LA38-20X/33) copy the Harmony dimensions. Use Schneider's published drawings for the 22 mm collar and notch; the AliExpress listings will not give them |
| Everything else | **Calipers, on arrival** | AliExpress listing dimensions are unreliable |

---

## 10. On Arrival

Checks that must happen when the parcels land, before any cutting or wiring.

- [ ] **Set the LM2596 to 5,00 V and verify with a meter BEFORE connecting anything to it.** It is an
  adjustable module shipping at an unknown setting, and its range goes to 35 V. One unverified
  connection kills the ESP32, the MCP23017 and the BME280 together
- [ ] **BME280 is not a BMP280.** The listing is titled for both and ships either. The BMP280 has no humidity sensor, which §2 requires. Check the chip marking; open a dispute if wrong.
- [ ] **CrowPanel PCB outline matches the STEP file** from the Elecrow repo (§9)
- [ ] **DaierTek LED polarity** (common anode vs common cathode): decides ULN2803 low-side vs high-side driver
- [ ] Measure every part with calipers, do not trust listing specs
- [ ] Confirm the ignition button cut-out (listing gives no dimensions); fall back to the 22mm backup unit if it does not work out
- [ ] Check max panel thickness per component vs. your printed plate thickness
- [ ] **Selectors are maintained, not spring-return.** Turn one and let go — the lever must stay put
- [ ] **Selectors are 2NO, not 1NO1NC.** Meter both contacts in all three lever positions: centre must show *both open*. If one reads closed at centre, a BJ35 shipped instead of a BJ33
- [ ] **Both contact blocks present** on each selector, and they latch onto the head correctly
- [ ] Measure the anti-rotation notch position on the 22 mm collar and model it (§7)
- [ ] **Measure the wheelbase long-press duration** with a stopwatch before writing the servo constant (§3.1)
- [ ] Measure the wheelbase's Shelly wattage in both states, to set the confirmation threshold (§3.1)

---

## 11. Before You Cut

- [ ] Dry-fit all parts in a test print (single strip with all cut-outs) before printing the full plate
- [ ] Confirm nut clearance with parts physically side by side
- [ ] Confirm the E-stop's full ~60 mm depth clears everything behind the panel (§7)
- [ ] Keep the BME280 outside the sealed volume, away from any heat source
- [ ] Confirm no 12V line routes to an ESP32 or MCP23017 pin (§5)

---

## 12. Open Items

| Item | Blocking | Owner |
|---|---|---|
| **CrowPanel 3.5" vs 5.0"** | The Fusion model uses a generic **5"** LCD; the cart contains a **3.5"**. The verified STEP file is for the 3.5". Pick one, then use that part's real STEP — the panel cut-out cannot be drawn from a placeholder | Decision |
| **Choose the wind fans** | Sets the 12V PSU current rating and the drive circuit (4-pin PWM vs low-side MOSFET), §3.2 | Leo |
| **Source a 12V always-on PSU** | Powers the toggle/ignition LEDs, the servo rail and the fans. Must be independent of PC power state (§5). **Size after the fans** | Leo |
| **Order the E-stop XB2-542 (2NC)** | §2 item 6; panel layout cannot be finalised | Leo |
| **Wheelbase long-press duration** | Servo hold constant, §3.1. Measure on real hardware | Leo |
| **Fan behaviour when the PC is off** | §6 — honour manual override, or force to 0 | Decision |
| **Fan behaviour when telemetry stops** | §6 — fail to 0, or hold-then-decay | Decision |
| **MCP23017 vs PCA9685** | The MCP23017 has **no PWM** — the toggle LEDs get 7 fixed colours (R/G/B/yellow/cyan/magenta/white), no dimming or blended shades. If a dimmed idle state or a true amber is wanted, swap to a PCA9685 (16-ch PWM, same I2C, ~2 €). **Decide before cutting the panel.** | Decision |
| Motion platform power rating | Validates E-stop can break load directly (XB2 AC-15 rating is ~3A/240V, well under its 10A thermal rating). Also decides whether the conditional contactor in §4 must be bought | Leo |
| Function of the 4 spare buttons | They are already in the Fusion model with no assigned purpose | Leo |
| Corsair controller model + which fans | iCUE SDK support level for fan override | Leo |
| Available panel area (W x H x D) | Full layout | Leo |
| Maintenance mode control | Recommended as a dedicated physical switch, not a long-press | Decision |

---

## 13. Safety Notes

- The E-stop is wired **in-line on mains**, in series, bypassing all smart logic. This is mains work.
- E-stop is **2NC** (double pole): French outlets are not polarised, breaking a single pole can leave live present.
- If the motion platform exceeds ~600W, the E-stop should break a **contactor coil**, not the load directly.
- Shelly devices: buy from an official reseller, **not** AliExpress. Mains-connected hardware, counterfeits are common. (The CrowPanel is exempt — it is a USB-powered low-voltage display, not mains hardware.)
- The **selector switches are also exempt** for the same reason: their contacts carry 3.3 V dry
  signals to the ESP32, never mains. Clone XB2/LA38 units are fine. The E-stop is not exempt — it
  breaks mains, and that is a shock hazard, not a reliability question.
- Every relay / smart plug must be configured to default **OFF** after power loss.
- 12V must never reach an ESP32 or MCP23017 pin. See §5.
- The servo arm must park **clear** of the wheelbase button, and firmware must hard-limit how long it
  can be driven. A stuck arm holds a power button down indefinitely. See §3.1.
