# Research: initial battery-life analysis for the T1 (audiophile listening session)

- **Date:** 2026-10-22
- **Status:** Desk research, **initial estimate**. Nothing here has been built or measured. Every runtime below is a model output, not a promise. It is **landed as a model**, not validated: it becomes a claim only after the prototype measurements in §12.
- **Note (2026-10-22, after this report):** the T1 moved to a two-cell (2S) battery with a quoted pack, [board ADR 0014](https://github.com/opendap-project/board/blob/main/docs/adr/0014-two-cell-battery.md). The Adafruit 328 is no longer a T1 pack; the BQ25895 charger and MAX17055 gauge are replaced by the BQ25798 and BQ28Z610-R1, and a TPS62136 pre-regulator (3.8 V) now feeds the former 1S loads. The figures below are kept as written (1S); the 2S study puts the core session about 9–12 % lower at the same pack energy (est., [2S study](https://github.com/opendap-project/board/blob/main/docs/research/2026-10-battery-2s.md) §5).
- **Requirements served:** T1-009 (battery autonomy is a top priority), T1-012 (≥ 2,000 mAh commercial cell), T1-035 (largest battery that fits), BRD-008 (power down unused blocks), BRD-009 (low-power mode).
- **Maintainers' decision rule this report supports (§11):**
  - After the battery is chosen, the body shrinks toward the ideal **if** the audiophile-session autonomy stays around the **category average**. Slightly below average is acceptable.
  - Closer to credit-card size wins. Thinner is better: toward ≤ 20 mm, and 25 mm only if autonomy needs it.
  - The ideal proportion is the card's height:width ratio (≈ 1.59). A small deviation is allowed when it buys a standard cell or fits the display.
- **Inputs read:**
  - [`board/docs/specs/bom-v0.md`](https://github.com/opendap-project/board/blob/main/docs/specs/bom-v0.md)
  - [`board/docs/research/2026-10-component-selection.md`](https://github.com/opendap-project/board/blob/main/docs/research/2026-10-component-selection.md)
  - [`board/docs/research/2026-10-sourcing-alternatives.md`](https://github.com/opendap-project/board/blob/main/docs/research/2026-10-sourcing-alternatives.md)
  - [`board/docs/research/2026-10-high-speed-io-topology.md`](https://github.com/opendap-project/board/blob/main/docs/research/2026-10-high-speed-io-topology.md)
  - [`board/docs/research/2026-10-battery-selection.md`](https://github.com/opendap-project/board/blob/main/docs/research/2026-10-battery-selection.md) (revised for the 62.5 × 93.75 × 25 mm body: candidates, pocket arithmetic)
  - [`faceplates/touch/docs/adr/0001-focus-lcds-e35gh-display.md`](https://github.com/opendap-project/faceplate-touch/blob/main/docs/adr/0001-focus-lcds-e35gh-display.md) and [`faceplates/touch/docs/research/2026-10-display-selection.md`](https://github.com/opendap-project/faceplate-touch/blob/main/docs/research/2026-10-display-selection.md)
  - [`blackbird-os/docs/research/2026-10-image-composition.md`](https://github.com/opendap-project/blackbird-os/blob/main/docs/research/2026-10-image-composition.md) (§5.6 and §8, power)
  - [`system/docs/adr/0002-system-architecture.md`](https://github.com/opendap-project/system/blob/main/docs/adr/0002-system-architecture.md)
  - [`t1/docs/requirements/README.md`](../requirements/README.md)
  - [`toolkit/verify/docs/research/2026-10-verification-instrumentation.md`](https://github.com/opendap-project/toolkit-verify/blob/main/docs/research/2026-10-verification-instrumentation.md) (P7, battery verification)
- **Evidence convention:**
  - **[DS]** a figure read from the vendor datasheet or product page in this session (link in §14).
  - **[DOC]** a figure taken from one of the OpenDAP reports above.
  - **[EST]** an engineering estimate. Always **UNVERIFIED**.
  - **UNVERIFIED** also marks any figure recalled but not re-read from a primary source in this session.
- **Session limits:**
  - The web-search budget was exhausted before this work started. Datasheets and product pages were fetched directly where the vendor allowed it.
  - These could not be read: the Cirrus CS43198 datasheet (HTTP 403); the ADI MAX17055 and MAX98357A datasheets; the ST LSM6DSV16X datasheet; the Crystek CCHD-957 sheet; the NXP IW612 datasheet (NDA-gated); Shanling's spec pages (rendered as images); Sony's pages (HTTP 403); Astell&Kern's and iBasso's spec pages.
  - **No published RK3566 board power measurement could be fetched.** The SoC row is a bottom-up estimate and the single largest uncertainty in this report.

---

## 0. Summary

### 0.1 Headline runtimes

The core session is scenario 1: screen off, FLAC 16/44.1 bit-perfect, 3.5 mm single-ended (SE) into 32 Ω IEMs, radios off.

- **Modelled battery draw:** ≈ **0.39 W best, 0.66 W typical, 1.5 W pessimistic.**

| Pack | Energy | Runtime, typical | Range (worst – best) |
|---|---|---|---|
| Adafruit 2011 | 7.4 Wh | ≈ 10 h | 4.5–17 h |
| Adafruit 328 (prototype and DIY primary) | 9.4 Wh | ≈ 13 h | 5.7–21.5 h |
| **105060-class, 4.2 V** (commercial target, quote only) | ≈ 12 Wh | **≈ 16 h** | 7.3–27.5 h |
| 105060-class, LiHV (quote only) | ≈ 13.5 Wh | ≈ 18 h | 8.2–31 h |

**Other scenarios, Adafruit 328, typical (range in brackets):**

| Scenario | Typical draw | Runtime |
|---|---|---|
| 2. Balanced 4.4 mm into 300 Ω | 0.78 W | 10.8 h (6.0–18.7) |
| 3. Hi-res 24/192 or DSD128 | 0.69–0.70 W | 12.1–12.3 h |
| 4. Screen on 10 % / 100 % of the time | 0.71 / 1.06 W | 12.0 h / 8.0 h |
| 5. Wi-Fi streaming / LE Audio sink / LE Audio source to earbuds | 0.83 / 0.73 / 0.44 W | 10.2 h / 11.6 h / 19.3 h |

### 0.2 Category benchmark (§10)

- Ten comparable DAPs have makers' claims that were read this session. Their **screen-off SE claims average ≈ 14.5 h** (median 13.5 h, range 11–21 h).
- Their balanced claims average ≈ 11 h.
- The T1's typical draw (0.66 W) matches the two closest architectural neighbours: the **FiiO JM21** (2× CS43198, ≈ 0.66 W implied) and the **HiBy R3Pro II** (2× CS43198 + 4× OPA1622, ≈ 0.65 W implied).

### 0.3 Body sizing against the maintainers' rule (§11)

**Energy needed** (at the typical draw; usable energy is 90 % of label energy):

| Target | Runtime | Label energy needed |
|---|---|---|
| 100 % of the category average | 14.5 h | **≈ 10.7 Wh** |
| 85 % of the category average | 12.3 h | **≈ 9.1 Wh** |

**What each target means for the body:**

- **The width is fixed by the display at ≈ 59.5 mm.** The card ratio then gives **59.5 × 94.3 mm**, which is +21 % in area over a credit card. The shortest body the display allows is 59.5 × 91.5 mm (ratio 1.54).
- **85 % of average:**
  - Reachable at **20 mm** in the card-ratio body (59.5 × 94.3 × 20), but only with a **quote-only LiHV 705055-class pack** (≈ 9–10 Wh, **UNVERIFIED**).
  - With a stocked pack (Adafruit 328) it needs **59.5 × 91.5 × 24 mm**.
- **100 % of average:**
  - Needs a 10 mm-class pack.
  - The best fit is a **105060-class pack in 59.5 × 94.3 × 24 mm** (card ratio, common size code, ≈ 112 % of average at 4.2 V).
  - A 105050 pack fits a shorter 59.5 × 91.5 × 23.5 mm body (ratio 1.54) at ≈ 100 %.
- **Risk case:** if the measured draw is ≈ 0.85 W (§10.3), 100 % needs ≈ 13.7 Wh. That means the LiHV 105060 or 115060 class in a **25 mm** body.

**Recommendation (labelled):**

- Do not shrink below 24 mm until the RK3566 idle floor is measured on a dev kit (§12, step 1). That measurement moves the answer more than any battery choice.
- If it comes in at ≤ 0.6 W, the 20–21 mm card-ratio body is on the table.

### 0.4 What dominates (typical, scenario 1)

| Block | Share |
|---|---|
| SoC + LPDDR4X | ≈ 48 % |
| Headphone amplifiers (OPA1622 quiescent plus load) | ≈ 30 % |
| DACs (with LDO losses) | ≈ 13 % |
| The active master clock | ≈ 8 % |

The display adds ≈ 0.4 W when it is on at 200 nit.

### 0.5 Main findings

1. **The OPA1622 quiescent current is the hidden cost.** Its datasheet gives 2.6 mA per channel, which is 26 mW per enabled channel at ±5 V. EN-pin gating per output and the analog rail voltage are the biggest audio-side levers.
2. **Suspend-to-RAM during playback is not available.** The SoC must keep I2S, DMA and DRAM alive. RP2350 offload would put a helper MCU in the data path, which **BRD-005 forbids**. Runtime-idle quality decides playback life.
3. **The audio power tree is a BOM gap.** No part yet generates the OPA1622 ± rails or the DAC 1.8 V. Their efficiency swings the audio share by ±30 %.

---

## 1. Scope and method

1. Build a **power tree**: battery → charger power path → PMIC bucks or LDOs → loads. Apply a conversion efficiency to every load (§3).
2. For each component, take its **datasheet** current in the state each scenario puts it in (§4). Where no datasheet value was available, use a labelled estimate.
3. Model the **headphone amplifier** from first principles: quiescent current plus load current drawn from the rails (§5).
4. Sum per scenario in three columns:
   - **min (best):** good power management, quiet listening, efficient rails;
   - **typ:** the expected first-firmware outcome;
   - **max (pessimistic):** poor gating, loud listening, inefficient rails.
5. Runtime = usable energy ÷ battery-side power.
   - Usable energy = **90 %** of label Wh. That covers the PCM cut-off and the gauge reserve, as in the battery report.
   - An **aged pack** (80 % state of health) is shown at 72 % of label Wh.
6. Compare with the category (§10), then size the body from the energy needed (§11), using the battery report's pocket arithmetic.

The calculation is a short script that was run locally. Every table can be reproduced from §4, §5 and the battery report's §2.4.

---

## 2. Battery candidates and usable energy

Candidates come from the revised [battery selection report](https://github.com/opendap-project/board/blob/main/docs/research/2026-10-battery-selection.md). BOM line 21 (Jauch LP705176JS) **fits no envelope** according to that report, so it is not used here.

| Pack | Status in the battery report | Label energy | Usable (90 %) | Aged (72 %) |
|---|---|---|---|---|
| Adafruit 2011, 2,000 mAh | Stocked; same certificate status as Adafruit 328 | 7.4 Wh | 6.7 Wh | 5.3 Wh |
| 705050-class, quoted | Largest option at **20 mm**; **UNVERIFIED** stock | ≈ 8.5 Wh (7.4–9.5, the upper end LiHV) | 7.7 Wh | 6.1 Wh |
| **Adafruit 328**, 2,500 mAh | **Prototypes and DIY**; largest stocked pack with certificate evidence (UN 38.3) | 9.4 Wh | 8.5 Wh | 6.8 Wh |
| USE-104060-3K, 3,000 mAh | Largest stocked; no certificates | 11.1 Wh | 10.0 Wh | 8.0 Wh |
| **105060-class, 4.2 V**, quoted | **Commercial target** | ≈ 12.0 Wh (11.1–13.0) | 10.8 Wh | 8.6 Wh |
| 105060-class LiHV, quoted | Commercial upgrade | ≈ 13.5 Wh (12.7–14.6) | 12.2 Wh | 9.7 Wh |

The 115060-class LiHV pack (≈ 14.5–15.5 Wh, the battery report's largest option) is used only in §11's risk case.

---

## 3. Power tree and conversion efficiencies

The BOM defines the charger (BQ25895), the PMIC (RK809), the fuel gauge (MAX17055) and the loads. **It does not define how the analog audio rails are made.** The rows marked "assumed" are design assumptions for this model, not decisions.

| Rail | Source | Efficiency (min-power / typ / max-power case) | Basis |
|---|---|---|---|
| Battery → SYS | BQ25895 BATFET (11 mΩ) | ≈ 100 % (I²R < 1 mW at 0.2 A) | [DS] BQ25895 |
| SoC core, logic, DDR | RK809 BUCK1–4 | Included in the SoC row (≈ 80–85 %) | [DS] RK809: 85 % at 0.3 A (3.8 → 1.0 V), 88 % at 0.3 A (→ 1.5 V), 70 % at 2.5 A. **Light-load efficiency is not given: UNVERIFIED** |
| 3.3 V / 1.8 V IO | RK809 buck plus LDOs | 85 % for the buck; an LDO costs Vout/Vin | [DS] RK809 |
| DAC 1.8 V (VA, VD) | **Assumed:** low-noise LDO | 75 % / 60 % / 46 % | 75 %: LDO fed by a ≈ 2.1 V pre-regulator. 46 %: LDO fed from 3.3 V |
| MCLK 3.3 V | **Assumed:** low-noise LDO | 85 % / 80 % / 75 % | [EST] |
| OPA1622 ± rails (±5 V nominal) | **Assumed:** inverting + boost converter (or charge pump), then LDOs | 80 % / 75 % / 68 % | [EST]. **Not in the BOM** |
| Backlight (10 LEDs, ≈ 15 V) | **Assumed:** boost LED driver with PWM | 90 % / 85 % / 80 % (75 % at very low current) | [DOC] display report |

---

## 4. Component-level budget

States are for **screen-off playback** (scenario 1) unless a row says otherwise. "Battery-side" means after conversion losses.

### 4.1 Always-present and control parts

| Part | State in playback | Datasheet figure | Battery-side power (min / typ / max) | Source |
|---|---|---|---|---|
| TI BQ25895 charger | No VBUS, Hi-Z, BATFET on, ADC off | 32 µA typ / 60 µA max (BATFET on, monitor off). Ship mode 12 / 23 µA | 0.12 / 0.12 / 0.23 mW. A continuous ADC adds an unknown amount (**UNVERIFIED**) | [DS] BQ25895 |
| onsemi FUSB302B | Unattached, toggling | Disabled 0.37 µA; toggle 25 µA typ / 40 µA max; BMC PD standby 40 µA | 0.08 / 0.08 / 0.13 mW | [DS] FUSB302B |
| ADI MAX17055 gauge | Active, or hibernate between updates | 7 µA hibernate / 18 µA active (**recalled, UNVERIFIED**) | 0.03 / 0.05 / 0.07 mW | Datasheet not read |
| RK809 PMIC quiescent | Several converters on, PFM auto | Standby 25 µA (sleep). Active quiescent per converter: **not given** | 0.5 / 1.5 / 4 mW [EST] | [DS] RK809 §1.2 |
| RK809 codec, headphone driver, class-D | Powered down (unused) | 4 mA quiescent when on | 0 (it must be off: a 15 mW trap) | [DS] RK809 |
| TI TPS259470A eFuse ×2 (HAT) | No HAT, EN low (shutdown) | Shutdown 4.4 µA typ; disabled (UVLO) 73 µA; on 428 µA | 0.03 / 0.03 / 0.55 mW (max = left in UVLO, not shut down) | [DS] TPS25947 |
| TI TPS2121 power mux | No input present (input side) | Enabled input 200 µA; disabled input 10 µA | 0 when unplugged | [DS] TPS2121 |
| ST LSM6DSV16X IMU | Power-down, or low-power wake-on-motion | µA range (**UNVERIFIED**) | 0.01 / 0.05 / 0.2 mW [EST] | — |
| Vishay VEML6030 ALS | Shutdown | 0.5 µA shutdown; 2 µA (PSM 4); 45 µA (no PSM) | 0.002 / 0.002 / 0.15 mW | [DS] VEML6030 |
| Infineon IM73D122 mics ×2 | Off: supply gated with the privacy LED (BRD-027) | 0 | 0 | [DOC] BRD-027 |
| ADI MAX98357A speaker amp | SD_MODE low, or rail gated | µA range (**UNVERIFIED**) | ≈ 0 | — |
| Murata Type 2EL (IW612) | Off: PDn low, rfkill | µA range (**UNVERIFIED**, NDA datasheet) | 0.05 / 0.1 / 0.5 mW [EST] | — |
| GT911 touch | Sleep (command 0x05), or rail gated | Mode behaviour [DS]; current **UNVERIFIED** (≈ 70 µA recalled) | 0 / 0.2 / 0.3 mW | [DS] GT911 programming guide |
| Focus E35GH panel and backlight | DPMS off, VCI and backlight gated | — | 0 | [DOC] blackbird-os §8.4 |
| **Subtotal, "misc"** | | | **≈ 1 / 2.5 / 6 mW** | |

### 4.2 Compute, memory and storage

| Part | State | Basis | Battery-side power (min / typ / max) |
|---|---|---|---|
| **RK3566** + **DDR PHY/DMC** + **LPDDR4X 4 GB** | Runtime idle between decode bursts. CPU at the lowest OPP with PSCI cpuidle. GPU, NPU, VOP2 and PIPE power domains off. DDR at its fixed boot frequency (no RK356x DMC devfreq in mainline, [DOC] blackbird-os §8.3) | **[EST], UNVERIFIED.** Rockchip publishes no power data. Bottom-up: vdd_cpu 20–60 mW; vdd_logic 60–150 mW; DDR PHY 30–100 mW; LPDDR4X 5–60 mW (self-refresh or power-down when the DMC idles it); IO and PLLs 10–40 mW; then buck losses | **200 / 320 / 550 mW** |
| Samsung KLMBG2JETD eMMC | Sleep or idle, with a read burst every few minutes (≈ 0.9 Mb/s) | Active reads of ~100–200 mW for well under 1 % of the time (**UNVERIFIED**) | 1 / 2 / 5 mW |
| microSD (if the library is on it) | Same pattern | Idles at 0.3–1 mA at 3.3 V (**UNVERIFIED**) | +1 / +3 / +10 mW over eMMC |

**Cross-checks:** the right comparables are the Anbernic RG353 and the PineNote (both RK3566, battery-powered, mainline). Quartz64 and Radxa ZERO 3 give SoC-only numbers on a bench supply. Their measurements could not be fetched this session. **This row is the first thing to measure** (§12, step 1).

### 4.3 Audio chain

| Part | State | Datasheet figure | Battery-side power (min / typ / max) | Source |
|---|---|---|---|---|
| Cirrus CS43198 ×2 (dual mono) | Playback, both chips | **26 mW per chip, "playback"**, 1.8 V. The per-mode table (PCM rates, DSD, standby) **could not be read: UNVERIFIED** | 2 × 26 mW at the rail (68 mW in the max case, +30 % for high rates), through the §3 LDO efficiency → **69 / 87 / 148 mW** | [DS] Cirrus product page |
| NDK NZ2520SDA 45.1584 MHz, **active** | Running, 3.3 V | **15 mA max** in operation (no typical value given); start-up 4 ms max | 33 mW (≈ 10 mA assumed), 40 mW or 50 mW at the rail | [DS] NDK catalogue |
| NDK NZ2520SDA 49.152 MHz, **unused family** | Standby (OE low) | **35 µA max** | 0.12 mW. **Max case: not gated, +≈ 50 mW** | [DS] NDK |
| Both MCLK oscillators together | — | — | **39 / 50 / 132 mW** | — |
| TI OPA1622 ×4 (8 channels) | Only the active output enabled | **2.6 mA typ / 3.3 mA max per channel; shutdown 5 / 10 µA per channel**; EN off at ≤ 0.78 V, on at ≥ 0.82 V; supply ±2 to ±18 V; output +145 / −130 mA | See §5 | [DS] OPA1622 |

**Topology assumption:** which OPA1622 channels each output uses is not drawn yet. The model brackets it:

- SE: 2 channels enabled (min), 4 (typ), 8 (max: nothing gated).
- Balanced: 4 channels (min), 8 (typ and max).

### 4.4 RP2350 coprocessor

| State | Datasheet figure | Battery-side power |
|---|---|---|
| Power-down states P1.0–P1.7 (Table 1692) | 548 µW down to **188 µW** | min **0.2 mW** |
| SLEEP with clocks gated, waking on GPIO or I2C | Not tabulated (**UNVERIFIED**) | typ **3 mW** [EST] |
| Running at 150 MHz with USB active (`hello_usb`, Table 1693) | **52.7 mW**; USB boot idle 25 mW | max **50 mW** |

The SoC side of the RP2350 USB link (host 0, `usb2phy1`) must autosuspend as well (**UNVERIFIED** cost).

### 4.5 Display and touch (scenario 4 only)

| Item | Figure | Battery-side power |
|---|---|---|
| Backlight, full (800 nit) | **15 V × 60 mA ≈ 0.9 W** [DOC] display report | 1.06 W |
| Backlight, typical indoor (≈ 200 nit) | ≈ 15 mA × 14 V ≈ 0.21 W (luminance taken as proportional to current: **UNVERIFIED**) | 0.23 / 0.25 / 0.26 W |
| Backlight, minimum (≈ 20 nit) | ≈ 1.5 mA × 13 V ≈ 20 mW | ≈ 26 mW |
| Panel logic (ILI9488, VCI 2.8 V) | 5–10 mA assumed (**UNVERIFIED**) | 20 / 28 / 35 mW |
| GT911 active or Green mode (≈ 40 ms scan) | Currents **UNVERIFIED** (≈ 3–9 mA recalled) | 11 / 20 / 33 mW |
| SoC display pipeline (VOP2, DSI, DDR scan-out ≈ 28 MB/s, Slint now-playing updates) | [EST] | 40 / 100 / 250 mW |
| **Screen-on adder, typical brightness** | — | **≈ 300 / 400 / 550 mW** |
| Adder at minimum brightness, static UI | — | ≈ 110 mW |
| Adder outdoors (800 nit, animated) | — | ≈ 1.4 W |

---

## 5. Headphone amplifier model

The OPA1622 is a class-AB amplifier. Its supply power is a quiescent part plus a load-current part:

```text
P_rails = N_ch · I_q · (2 · V_s)                 quiescent, per enabled channel, ±V_s rails
        + k · V_s · I_rms(load) · m              per audio channel
I_rms   = sqrt(P_load / R_load)
k       ≈ 0.8   (mean |i| / I_rms for music-like signals; 0.9 for a sine)
m       = 1 single-ended, 2 balanced (both legs carry the full load current)
P_batt  = P_rails / η(± rails)
```

At **±5 V**, each enabled channel costs **26 mW** quiescent, or 33 mW at the 3.3 mA maximum.

**Load-dependent part, stereo, at the rails (±5 V):**

| Load and output | 0.1 mW/ch | 1 mW/ch | 10 mW/ch |
|---|---|---|---|
| 32 Ω, single-ended | 14 mW | 45 mW | 141 mW |
| 32 Ω, balanced | 28 mW | 89 mW | 283 mW |
| 300 Ω, single-ended | 5 mW | 15 mW | 46 mW |
| 300 Ω, balanced | 9 mW | 29 mW | 92 mW |

**What these listening levels mean** (average music power, not peaks):

- A 105 dB SPL/mW IEM at 1 mW is ≈ 105 dB SPL, which is loud. A moderate 75–85 dB SPL is 0.001–0.1 mW.
- A ≈ 92 dB SPL/mW 300 Ω headphone at 1 mW is ≈ 92 dB SPL.
- The scenarios use **0.1 / 1 / 10 mW per channel** for min / typ / max. The typical column is therefore conservative for IEMs.

**Amplifier stage, battery-side:**

| Case | Enabled channels | Iq/ch | Load | η | Battery-side |
|---|---|---|---|---|---|
| S1 min: SE, 32 Ω | 2 | 2.6 mA | 0.1 mW | 0.80 | **83 mW** |
| S1 typ: SE, 32 Ω | 4 | 2.6 mA | 1 mW | 0.75 | **198 mW** |
| S1 max: SE, 32 Ω | 8 | 3.3 mA | 10 mW | 0.68 | **596 mW** |
| S2 min: balanced, 300 Ω | 4 | 2.6 mA | 0.1 mW | 0.80 | **142 mW** |
| S2 typ: balanced, 300 Ω | 8 | 2.6 mA | 1 mW | 0.75 | **316 mW** |
| S2 max: balanced, 300 Ω | 8 | 3.3 mA | 10 mW | 0.68 | **524 mW** |

**Rail voltage is a first-order lever.** Every row scales almost linearly with V_s.

- ±3.3 V still gives enough headroom at these levels: 10 mW into 32 Ω SE is 0.8 V and 25 mA peak; 10 mW into 300 Ω balanced is 1.22 V peak per leg.
- Lowering the rails to ±3.3 V cuts both the quiescent and the load cost by about 34 %.
- A **low-gain (IEM) mode with lower rails** is worth ≈ 60–70 mW in the typical case.

---

## 6. Scenarios

Each runtime cell reads **worst – typical – best**, in hours: worst = max power, best = min power, usable energy = 90 % of label.

### 6.1 Scenario 1: the core audiophile session

Screen off, local FLAC 16/44.1 bit-perfect through exclusive ALSA, 3.5 mm SE into 32 Ω IEMs, Wi-Fi and BT off.

| Block (battery-side, mW) | min | typ | max |
|---|---|---|---|
| SoC + DDR PHY + LPDDR4X | 200 | 320 | 550 |
| eMMC | 1 | 2 | 5 |
| 2× CS43198 (with LDO loss) | 69 | 87 | 148 |
| MCLK (max = both running) | 39 | 50 | 132 |
| 4× OPA1622 (§5) | 83 | 198 | 596 |
| RP2350 | 0.2 | 3 | 50 |
| Misc (§4.1) | 1 | 2.5 | 6 |
| **Total** | **393** | **662** | **1,487** |

| Pack | Runtime (worst – typ – best) | Aged pack, typ |
|---|---|---|
| Adafruit 2011 (7.4 Wh) | 4.5 – **10.1** – 16.9 | 8.0 |
| 705050-class (≈ 8.5 Wh) | 5.1 – **11.5** – 19.5 | 9.2 |
| **Adafruit 328 (9.4 Wh)** | 5.7 – **12.8** – 21.5 | 10.2 |
| USE-104060-3K (11.1 Wh) | 6.7 – **15.1** – 25.4 | 12.1 |
| **105060-class 4.2 V (≈ 12.0 Wh)** | 7.3 – **16.3** – 27.5 | 13.0 |
| 105060-class LiHV (≈ 13.5 Wh) | 8.2 – **18.3** – 30.9 | 14.7 |

### 6.2 Scenario 2: 4.4 mm balanced into 300 Ω headphones

Same as scenario 1, except the amplifier row (§5): 142 / 316 / 524 mW. The totals are **452 / 780 / 1,415 mW**.

| Pack | Runtime (worst – typ – best) |
|---|---|
| Adafruit 2011 | 4.7 – **8.5** – 14.7 |
| 705050-class | 5.4 – **9.8** – 16.9 |
| **Adafruit 328** | 6.0 – **10.8** – 18.7 |
| USE-104060-3K | 7.1 – **12.8** – 22.1 |
| **105060-class 4.2 V** | 7.6 – **13.8** – 23.9 |
| 105060-class LiHV | 8.6 – **15.6** – 26.9 |

**Notes:**

- Balanced costs **≈ 120 mW more** than SE in the typical case. That is mostly the doubled number of enabled channels.
- This matches the 15–25 % shorter balanced runtimes that makers publish (§10).
- **IEMs on the balanced jack** (32 Ω at 1 mW) are the worst wired case: ≈ 0.86 W typical, or ≈ 9.8 h on the Adafruit 328.

### 6.3 Scenario 3: hi-res PCM 24/192, DSD128 over DoP, native DSD128

**Differences from scenario 1** (adders, battery-side):

| Source of the difference | 24/192 FLAC | DSD128 DoP | DSD128 native | Basis |
|---|---|---|---|---|
| File rate | ≈ 4–5 Mb/s (vs 0.9) | 11.3 Mb/s (DSF) | 11.3 Mb/s | Arithmetic |
| CPU | FLAC decode ≈ 4–5× the 16/44.1 work; still a few % of one A55 | No decode; DoP packing is trivial, but there is more memory traffic | No decode, no packing | [DOC] blackbird-os §5.6; [EST] |
| Storage reads | +1–3 mW (eMMC) | +3–15 mW (microSD libraries) | Same as DoP | [EST] |
| I2S BCLK | 12.288 MHz (vs 2.822 MHz) | 352.8 kHz frames → 22.58 MHz | 176.4 kHz DSD_U32 frames → 11.29 MHz | Arithmetic; blackbird-os ADR 0006 |
| DAC | Higher-rate PCM path (**UNVERIFIED**, +0–10 mW per chip) | DSD path; the interpolator is bypassed (**UNVERIFIED** sign) | Same | Datasheet table not readable |
| MCLK | 49.152 MHz family: same NDK part, same 15 mA max | 45.1584 MHz family | Same | [DS] NDK |
| **Total adder (min / typ / max)** | **+10 / +25 / +60 mW** | **+15 / +35 / +90 mW** | **+10 / +25 / +70 mW** | [EST] |

**Runtimes:**

| Pack | 24/192 (worst – typ – best) | DSD128 DoP | DSD128 native |
|---|---|---|---|
| Adafruit 2011 | 4.3 – 9.7 – 16.5 | 4.2 – 9.5 – 16.3 | 4.3 – 9.7 – 16.5 |
| **Adafruit 328** | 5.5 – **12.3** – 21.0 | 5.4 – **12.1** – 20.7 | 5.4 – **12.3** – 21.0 |
| **105060-class 4.2 V** | 7.0 – **15.7** – 26.8 | 6.8 – **15.5** – 26.5 | 6.9 – **15.7** – 26.8 |
| 105060-class LiHV | 7.9 – 17.7 – 30.1 | 7.7 – 17.4 – 29.8 | 7.8 – 17.7 – 30.1 |

**Conclusion:**

- The format costs **3–6 % of runtime** at most.
- **Where the files live matters more than the format.** DSD libraries on microSD read about 1.4 MB/s continuously.
- **Policy:** read ahead in large chunks (tens of MB per read), so the storage sleeps between bursts.

### 6.4 Scenario 4: screen on at typical brightness, 10 % and 100 % of the time

The adder at typical brightness is 300 / 400 / 550 mW (§4.5). At 10 % duty, each wake (panel power-on, VOP2 and DSI resume, compositor thaw) adds +2 / +5 / +15 mW on average.

| Item | min | typ | max |
|---|---|---|---|
| Total, screen on 10 % of the time (mW) | 425 | 707 | 1,557 |
| Total, screen on 100 % of the time (mW) | 693 | 1,062 | 2,037 |

| Pack | 10 % on (worst – typ – best) | 100 % on |
|---|---|---|
| Adafruit 2011 | 4.3 – 9.4 – 15.7 | 3.3 – 6.3 – 9.6 |
| **Adafruit 328** | 5.4 – **12.0** – 19.9 | 4.2 – **8.0** – 12.2 |
| **105060-class 4.2 V** | 6.9 – **15.3** – 25.4 | 5.3 – **10.2** – 15.6 |
| 105060-class LiHV | 7.8 – 17.2 – 28.6 | 6.0 – 11.4 – 17.5 |

**Sensitivity to brightness:**

- Outdoors at 800 nit with the screen always on, the total is ≈ 2.1 W: ≈ 4 h on the Adafruit 328.
- At minimum brightness, the total is ≈ 0.77 W: ≈ 11 h.
- The ALS brightness curve is a battery feature, not only a comfort feature.

### 6.5 Scenario 5: radios

| Sub-scenario | Adder (min / typ / max, mW) | Basis |
|---|---|---|
| 5a. Wi-Fi associated, idle, power save | +10 / +30 / +70 | [EST]. IW612 figures are under NDA. blackbird-os §8.4 |
| 5b. Wi-Fi streaming (FLAC, buffered in bursts) | +70 / +170 / +400 | [EST]. Radio bursts, networking and TLS on the SoC, SDIO interrupts |
| 5c. LE Audio **receiver** (sink) playing to the wired output | +35 / +70 / +160 | [EST]. Continuous isochronous stream, LC3 decode, HCI UART. Readiness depends on `BT_NXPUART` and BlueZ BAP sink support (**UNVERIFIED**) |
| 5d. LE Audio **source** to earbuds (DACs, MCLK and OPA1622 off) | Replaces the audio chain with +50 / +110 / +250 | [EST]. LC3 encode plus radio transmit |

| Pack | 5a Wi-Fi idle | 5b Wi-Fi streaming | 5c LE sink | 5d LE source |
|---|---|---|---|---|
| Total (mW, min / typ / max) | 403 / 692 / 1,557 | 463 / 832 / 1,887 | 428 / 732 / 1,647 | 252 / 438 / 861 |
| Adafruit 2011 | 4.3 – 9.6 – 16.5 | 3.5 – 8.0 – 14.4 | 4.0 – 9.1 – 15.6 | 7.7 – 15.2 – 26.4 |
| **Adafruit 328** | 5.4 – **12.2** – 21.0 | 4.5 – **10.2** – 18.3 | 5.1 – **11.6** – 19.8 | 9.8 – **19.3** – 33.5 |
| **105060-class 4.2 V** | 6.9 – **15.6** – 26.8 | 5.7 – **13.0** – 23.3 | 6.6 – **14.7** – 25.2 | 12.5 – **24.7** – 42.8 |
| 105060-class LiHV | 7.8 – 17.5 – 30.1 | 6.4 – 14.6 – 26.2 | 7.4 – 16.6 – 28.4 | 14.1 – 27.8 – 48.2 |

**Reading:**

- Leaving Wi-Fi associated "just in case" costs ≈ 5 % of runtime. Streaming costs ≈ 20 %.
- Bluetooth output to earbuds runs longest, because the whole discrete analog chain is off.

---

## 7. What dominates

Typical shares in scenario 1 (662 mW):

| Block | mW | Share | Comment |
|---|---|---|---|
| SoC + DDR (runtime idle) | 320 | 48 % | The biggest and the least certain: idle residency, DDR frequency and self-refresh, power domains |
| OPA1622 ×4 | 198 | 30 % | Quiescent current × rail voltage × enabled channels |
| CS43198 ×2 | 87 | 13 % | 52 mW of silicon; the rest is LDO loss |
| MCLK | 50 | 8 % | One oscillator running is unavoidable; both running is a 50 mW bug |
| RP2350 + misc + eMMC | 7.5 | 1 % | Only if the RP2350 sleeps and its USB link is suspended |

With the screen on, the display becomes the largest single block: ≈ 38 % of 1.06 W at 200 nit.

---

## 8. Levers, ranked by typical saving

| # | Lever | Typical saving (battery-side) | Cost or risk |
|---|---|---|---|
| 1 | **Display fully off during playback.** Backlight and VCI gated; VOP2, DSI and GPU off through genpd; GT911 asleep; UI frozen | 300–550 mW against screen-on | Already the target in blackbird-os §8.5. Verify per power domain |
| 2 | **OPA1622 EN gating per output.** Jack detect drives DAPM, which drives the EN GPIOs. Shutdown is 5 µA per channel [DS] | 100–135 mW | One GPIO per output group. The part handles pop on enable [DS] |
| 3 | **Runtime-idle quality.** 0.5–2 s ALSA buffer; ≤ 2–4 period wakeups per second (against ≈ 20–50 with a small PipeWire quantum); burst decode; lowest OPP; `workqueue.power_efficient=1`; no polling; frozen UI cgroups | 50–200 mW (**UNVERIFIED**) | Engineering only. Guard with xrun counters and bit-perfect captures |
| 4 | **DDR operating point.** Choose a lower fixed DDR frequency in the rkbin configuration; enable the DMC's auto power-down and self-refresh idle timers | 20–80 mW (**UNVERIFIED**) | UI smoothness and HAT bandwidth. Measure it |
| 5 | **Lower analog rails for low-gain (IEM) mode**: ±3.3 V instead of ±5 V | 60–70 mW | A switchable or two-level rail. The gain structure is a design decision |
| 6 | **MCLK OE or standby gating of the unused family** (≤ 4 ms start-up [DS]) | 50 mW (prevents the bug) | Wire as a DAPM clock supply tied to the rate family |
| 7 | **DAC 1.8 V from a pre-regulated rail** (≈ 2.1 V buck + LDO), not an LDO from 3.3 V | 20–60 mW | One more buck. Check its noise on the QA403 |
| 8 | **Radios rfkilled when unused** | 10–70 mW | None |
| 9 | **RP2350 asleep, its USB link autosuspended** | Up to ≈ 50 mW | Firmware discipline |
| 10 | **One DAC for SE** (power down the second CS43198 when only 3.5 mm is used) | ≈ 35–45 mW | Conflicts with dual mono (BRD-022, Draft) unless the SE topology allows it |
| 11 | **Library on eMMC rather than microSD**, with large read-ahead | 1–10 mW | User choice. The OS should default to read-ahead |
| 12 | BQ25895 ADC in one-shot mode on battery | < 1 mW (**UNVERIFIED**) | Check the `bq25890_charger` defaults |

**Levers that are *not* available:**

- **Suspend-to-RAM or s2idle during playback.**
  - The I2S DMA must keep running from DRAM, and s2idle freezes the playback engine.
  - Suspend belongs to **paused** playback, after a policy timeout.
  - Deep "mem" suspend depends on BL31 `SYSTEM_SUSPEND` support (**UNVERIFIED** for mainline TF-A, [DOC] blackbird-os §8.1).
- **RP2350 feeding I2S while the SoC sleeps** (blackbird-os Q10).
  - It conflicts with **BRD-005** (a helper MCU in the data path).
  - Its 520 KB SRAM holds only ≈ 3 s of 16/44.1 stereo, so the SoC would wake every few seconds anyway.

---

## 9. What the architecture must do

These follow from [system ADR 0002](https://github.com/opendap-project/system/blob/main/docs/adr/0002-system-architecture.md) and blackbird-os §8.

1. **Screen-off is a transition owned by the orchestrator.**
   - Freeze every UI activity's cgroup (cgroup freezer).
   - The compositor does KMS DPMS-off. The panel driver runs disable and unprepare. Genpd powers off VOP2 and the GPU.
   - GT911 goes to sleep. The ALS shuts down.
   - No timers run in frozen slices.
2. **The exclusive playback engine is the only thing awake.**
   - It opens `hw:` directly, with no PipeWire graph in the path.
   - It uses a **0.5–2 s ALSA buffer** with a period of ≥ 250 ms, and decodes in bursts.
   - `uclamp.min` is tuned per slice. Real-time priority comes from the unit, not from the `performance` governor.
3. **No polling anywhere.**
   - Battery: MAX17055 alert thresholds, or reads every ≥ 60 s while the screen is off.
   - Storage: uevents. Network: netlink. Buttons: interrupts (HID over I2C).
   - Verify with `/sys/kernel/debug/wakeup_sources` and cpuidle residency.
4. **Audio power is driven by DAPM.**
   - Jack detect → EN group of that output only.
   - Rate family → that MCLK only.
   - Stream close → DAC power-down after a short timeout.
5. **Radio policy:** off by default during wired playback.
6. **Power domains and buses:**
   - `pcie2x1` disabled unless a PCIe HAT is present, so PD_PIPE stays off.
   - USB autosuspend for the RP2350 link.
   - HAT eFuses held in shutdown (4.4 µA), not UVLO (73 µA).
7. **Paused or idle → suspend.** The orchestrator calls logind `Suspend()` after a timeout. Engines hold inhibitor locks only while they play.

---

## 10. Category benchmark

### 10.1 Makers' claims read this session

The claims below come from manufacturers' product pages, fetched on 2026-10-22 (links in §14). They are **makers' claims, not independent tests**. No page states the volume, the load or the file format in text. The screen is off by industry convention.

**Columns:**

- **SE** = 3.5 mm single-ended claim.
- **BAL** = 4.4 mm balanced claim.
- **Implied draw** = mAh × 3.8 V × 0.9 ÷ SE hours. It is a rough figure that assumes a 3.8 V nominal pack.

| Device | OS · DAC · amplifier | Battery | SE claim | BAL claim | Implied draw (SE) |
|---|---|---|---|---|---|
| FiiO M11 | Android · 2× AK4493 | 3,800 mAh | 13 h ("continuous playback", output not stated) | — | ≈ 0.90 W |
| FiiO M11 Plus LTD | Android · 2× AK4497EQ | 6,000 mAh | 11.5 h | 10 h | ≈ 1.78 W |
| FiiO M11S | Android · 2× ES9038Q2M | Not stated | ≈ 14 h | — | — |
| FiiO JM21 (3 + 32 GB) | Android · **2× CS43198**, 2× SGM8262 | 2,400 mAh | 12.5 h | 9.5 h | ≈ 0.66 W |
| FiiO JM21 (4 + 64 GB) | Same | 3,100 mAh | 16 h | 12.5 h | ≈ 0.66 W |
| FiiO M21 | Android · 4× CS43198 | 4,000 mAh | 15 h | 12.5 h | ≈ 0.91 W |
| HiBy R1 | **HiByOS (Linux)** · CS43131 | Not stated | 15 h | — | — |
| HiBy R3 II (2025) | **HiByOS (Linux)**, X1600E · 2× ES9219C | Not stated | ≈ 19 h | — | — |
| **HiBy R3Pro II** | **HiByOS (Linux)**, X1600E · **2× CS43198 + 4× OPA1622** | 4,000 mAh | 21 h (output not stated) | — | ≈ 0.65 W |
| HiBy R4 | Android · 4× ES9018C2M | Not stated | ≈ 11 h | — | — |
| HiBy R6 III (2025) | Android 12, SD665 · 4× CS43198, class AB | Not stated | 13 h | 12 h | — |
| HiBy R5 II | Android · 2× ES9219C, discrete class A | 4,500 mAh | 35 h economy / 10 h class A | 28 h / 7 h | ≈ 0.44 W / 1.54 W |

**Not read, so excluded from the average (UNVERIFIED):**

- Shanling M0 Pro, M3 Plus, M5 Ultra and M6: spec pages are images.
- iBasso DX180 and DX240: spec pages are not served.
- Sony NW-A306 and NW-ZX707: HTTP 403.
- Astell&Kern A&norma SR35: no page found.
- FiiO M15S: no figure on the page. FiiO M3 Pro: the page is gone. FiiO M23: the runtime is in a chart only.

These brands mostly sit in the same band. Sony and the small Shanlings are the known low-power exceptions, recalled at ≈ 0.15–0.3 W implied. They should be added once they can be read.

### 10.2 Category average

| Set | n | Mean | Median | Range |
|---|---|---|---|---|
| **SE, default mode** (one value per model: JM21 base variant; R5 II economy excluded as an outlier mode) | 10 | **14.5 h** | 13.5 h | 11–21 h |
| SE including the R5 II economy mode | 11 | 16.4 h | 14 h | 11–35 h |
| SE, Linux (HiByOS) players only | 3 | 18.3 h | 19 h | 15–21 h |
| SE, Android players only | 7 | 12.9 h | 13 h | 11–15 h |
| **Balanced** (M11 Plus LTD, JM21, M21, R6 III) | 4 | **11.0 h** | 11.0 h | 9.5–12.5 h |

**The category average used below is 14.5 h SE and 11 h balanced.** Wh figures for the other averages can be scaled linearly from §11.2.

### 10.3 What the benchmark says about the model

1. The two closest architectural neighbours imply ≈ 0.65–0.66 W, the same as the T1's typical model (0.662 W):
   - the **FiiO JM21** (2× CS43198, discrete amplifiers);
   - the **HiBy R3Pro II** (2× CS43198 + 4× OPA1622, the T1's exact audio chain).
2. **Risk:** the R3Pro II uses an Ingenic X1600E, a far lighter SoC than the RK3566.
   - If its 0.65 W is mostly the audio chain, the T1's chain is heavier than modelled. A T1 built like it would draw closer to **≈ 0.85 W**.
   - That is the **risk case** carried into §11.
   - Makers usually measure at low volume, so the true gap is unknown until §12, step 3.
3. The Linux players (R1, R3 II, R3Pro II) reach 15–21 h. That shows what a lean, non-Android Linux stack can do. It is also the bar the T1 is most likely to be compared against.

---

## 11. Autonomy against the category, and body sizing

### 11.1 Each candidate against the average

Figures are typical (§6.1 and §6.2): new pack, 90 % usable. The risk case is a 0.85 W draw (§10.3).

| Pack | Label energy | SE runtime | % of the 14.5 h average | BAL runtime | % of the 11 h average | SE % of average in the risk case |
|---|---|---|---|---|---|---|
| Adafruit 2011 | 7.4 Wh | 10.1 h | **69 %** | 8.5 h | 78 % | 54 % |
| 705050-class (20 mm) | ≈ 8.5 Wh | 11.5 h | **80 %** | 9.8 h | 89 % | 62 % |
| Adafruit 328 | 9.4 Wh | 12.8 h | **88 %** | 10.8 h | 99 % | 69 % |
| USE-104060-3K | 11.1 Wh | 15.1 h | **104 %** | 12.8 h | 116 % | 81 % |
| 105060-class 4.2 V | ≈ 12.0 Wh | 16.3 h | **112 %** | 13.8 h | 126 % | 88 % |
| 105060-class LiHV | ≈ 13.5 Wh | 18.3 h | **126 %** | 15.6 h | 142 % | 99 % |

**SE is the binding case.** In this model, balanced reaches its own average with ≈ 10 % less energy.

### 11.2 Minimum battery energy for each target

The formula is: label Wh = target hours × draw ÷ 0.9.

| Target | Typical model (0.66 W) | Risk case (0.85 W) | Best case (0.39 W) |
|---|---|---|---|
| **100 % of average** (14.5 h SE) | **10.7 Wh** | 13.7 Wh | 6.3 Wh |
| **85 % of average** (12.3 h SE) | **9.1 Wh** | 11.6 Wh | 5.4 Wh |
| 100 % of the balanced average (11 h) | 9.5 Wh | — | — |

### 11.3 What the body can hold

**Fixed constraints:**

- **Width:** the Focus E35GH module is 54.46 mm wide. With walls, the body is at least **≈ 59.5 mm** wide (display report). A credit-card footprint (85.6 × 54 mm) cannot hold this display.
- **Height:** at least ≈ 91.5 mm for the module (82.94 mm) and its frame (display report). The card ratio (≈ 1.585) at 59.5 mm gives **94.3 mm**. The largest card-ratio body within T1-031 is ≈ 59.9 × 95 mm.
- **Card-ratio footprint:** at 59.5 × 94.3 mm it is **≈ 21 % larger in area than a credit card**. The shortest display-limited body, 59.5 × 91.5 mm (ratio 1.54, inside T1-033's 1.52–1.59), is ≈ 18 % larger.

**Method:** the battery report's pocket arithmetic (§2.2–2.4), applied to these footprints:

- 11.7 mm of fixed stack, 0.3 mm of foam and +10 % swelling: t_max = (T − 12) / 1.1.
- 1.6 mm walls, 2.5 mm side margins, a 17 mm bottom band, and 0.4 mm clearances.
- 2.5 mm for leads and the connector.
- **HAT device-side piece:** stacked over the bottom band only when the body is ≥ 23.7 mm thick. Below that it goes in the top band, which grows to 13 mm. That costs 6 mm of pack length.
- **Geometric ceiling:** 390–440 Wh/L applied to the maximum pack box. It is an upper bound; LiHV adds about 10 %.

| Body (W × H × T, mm) | Ratio | Max pack (t × w × L, mm) | Geometric ceiling, 4.2 V (LiHV) | Practical catalogue pack that fits |
|---|---|---|---|---|
| 59.5 × 91.5 × 20 | 1.54 | 7.3 × 50.5 × 55.0 | 7.9–8.9 Wh (8.7–9.8) | 705050, ≈ 7.4–8.5 Wh (LiHV ≈ 8.5–9.5), quote only |
| **59.5 × 94.3 × 20** | **1.585** | 7.3 × 50.5 × 57.8 | 8.3–9.3 Wh (9.1–10.3) | **705055**, ≈ 8.1–9.3 Wh (LiHV ≈ 8.9–10.2), quote only (**UNVERIFIED** size code) |
| 59.5 × 91.5 × 21 | 1.54 | 8.2 × 50.5 × 55.0 | 8.9–10.0 Wh (9.7–11.0) | 805050, ≈ 8.5–9.6 Wh, quote only (**UNVERIFIED**) |
| 59.5 × 94.3 × 22 | 1.585 | 9.1 × 50.5 × 57.8 | 10.3–11.7 Wh (11.4–12.8) | 905055, ≈ 10–11 Wh (LiHV ≈ 11–12), quote only (**UNVERIFIED**, uncommon code) |
| 59.5 × 91.5 × 23.5 | 1.54 | 10.3 × 50.5 × 55.0 | ≈ 11–12.5 Wh | **105050**, ≈ 10.4–11.5 Wh (common code), quote only |
| 59.5 × 91.5 × 24 | 1.54 | 10.9 × 50.5 × 61.0 | 13.1–14.8 Wh (14.4–16.3) | **Adafruit 328 (9.4 Wh, stocked)**; 105060 does **not** fit (needs a 62.5 mm pack, so H ≥ 93) |
| **59.5 × 94.3 × 24** | **1.585** | 10.9 × 50.5 × 63.8 | 13.7–15.5 Wh (15.1–17.0) | **105060**, 11.1–13.0 Wh (LiHV 12.7–14.6); USE-104060-3K 11.1 Wh (stocked); Adafruit 328 |
| 59.5 × 94.3 × 25 | 1.585 | 11.8 × 50.5 × 63.8 | 14.9–16.8 Wh (16.3–18.4) | 115060 LiHV, ≈ 14.5–15.5 Wh, quote only |
| 62.5 × 93.75 × 25 (battery report D25) | 1.50 | 11.8 × 53.5 × 63.25 | 15.6–17.6 Wh | Same 105060 / 115060 packs: the extra 3 mm of width fits no standard 50 mm code |

### 11.4 Smallest body for each target

The tables follow the maintainers' priorities: closest to card size first, then thinnest, then card ratio, with small ratio changes shown where they buy something.

#### Target: 85 % of the average (≥ 9.1 Wh, typical model)

| Option | Body | Pack | Reaches | Trade-off |
|---|---|---|---|---|
| **A. Thinnest, card ratio** | **59.5 × 94.3 × 20** | LiHV 705055-class, quoted | ≈ 83–96 % (8.9–10.2 Wh) | Meets the target only with LiHV near the top of its range. Quote-only, with no stocked fallback at 20 mm. No margin for the risk case (≈ 65–75 %) |
| B. Shorter, slightly thicker | 59.5 × 91.5 × 21 (ratio 1.54) | 805050-class, quoted | ≈ 80–90 % | 2.8 mm shorter (closer to card size) but 1 mm thicker. Also quote-only |
| C. Stocked today | 59.5 × 91.5 × 24 (ratio 1.54) | Adafruit 328 | 88 % | Validated stock and UN 38.3 now, but 24 mm thick |

#### Target: 100 % of the average (≥ 10.7 Wh, typical model)

| Option | Body | Pack | Reaches | Trade-off |
|---|---|---|---|---|
| D. Thinnest, card ratio | 59.5 × 94.3 × 22 | LiHV 905055-class, quoted | ≈ 103–112 % | Uncommon size code: single-source risk (against T1-028). **UNVERIFIED** that any maker catalogues it |
| **E. Shortest, common code** | **59.5 × 91.5 × 23.5** (ratio 1.54) | **105050**, quoted | ≈ 97–108 % | Closest to card size for this target. The HAT piece goes in the top band. 105050 is a common code with several makers |
| **F. Card ratio, commercial target** | **59.5 × 94.3 × 24** | **105060 4.2 V** (LiHV later), quoted; USE-104060-3K and Adafruit 328 fit the same bay | ≈ 104–122 % (4.2 V), up to 137 % LiHV | The battery report's commercial size code. It shares one bay with every stocked prototype pack. 2.8 mm taller than E |

**Risk case (0.85 W) at 100 % (≥ 13.7 Wh):** only **59.5 × 94.3 × 25** with a 105060 LiHV (≈ 13.5 Wh, borderline) or a 115060 LiHV (≈ 15 Wh, quote only).

### 11.5 Reading for the maintainers' rule

1. **The display, not the battery, sets the footprint.** Every option is 59.5 mm wide.
   - The card ratio (94.3 mm) is what lets the 105060 class and the stocked 60 mm packs fit (they need H ≥ 93 mm at 24 mm thick).
   - The shorter 91.5 mm body saves 2.8 mm of height but limits the pack to ≤ 55 mm long below 23.7 mm thickness, or ≤ 61 mm above it.
   - Widening to 62.5 mm (ratio 1.50) buys no standard cell, because size codes are 50 mm wide. That is not worth it on energy grounds.
2. **Thickness is the real decision.**
   - 20 mm reaches ≈ 85 % of average only with a quote-only LiHV pack and no margin.
   - 23.5–24 mm reaches ≈ 100–120 % with common packs.
3. **The decision hinges on the measured draw.**
   - At the best-case draw (0.39 W), even 20 mm gives ≈ 135 % of average.
   - At the risk case (0.85 W), 24 mm gives only ≈ 88 %.
   - The RK3566 idle floor and the amplifier chain (§12, steps 1 and 3) can be measured on dev-kit hardware before the case is committed.

**Recommendation (labelled):**

- Design the bay for **option F (59.5 × 94.3 × 24 mm, card ratio, 105060 class)** as the current plan.
- Keep **option A (59.5 × 94.3 × 20 mm)** as the shrink path. Take it only if the measured scenario-1 draw is **≤ 0.55 W**, which gives ≥ 100 % of average with ≈ 9 Wh, and a LiHV 705055 pack can be quoted with certificates.
- Between those, **option E** (23.5 mm, 105050) is the smallest body that still reaches the average with a common pack.

---

## 12. Prototype measurements to validate this model

This plugs into promise **P7** of the [verification instrumentation report](https://github.com/opendap-project/toolkit-verify/blob/main/docs/research/2026-10-verification-instrumentation.md): published autonomy for named scenarios, met within ±10 % on hardware.

### 12.1 Design-for-test (needed before layout)

1. **Shunt points**, or 0 Ω links that can take a shunt, on:
   - the battery rail and SYS;
   - vdd_cpu, vdd_logic and the DDR rail;
   - the DAC 1.8 V rail and the MCLK 3.3 V rail;
   - the OPA1622 ± rails;
   - the backlight boost input;
   - the RP2350 supply and the radio module supply.
2. **BRD-040 test pads** (MCLK, BCLK, LRCK, SDATA), for bit-perfect captures during power runs.
3. The MAX17055 ALRT line on a wake-capable GPIO.

### 12.2 Measurement steps

Each step ends in a matrix row: case | steps | expected | observed | evidence.

1. **SoC floor first, on a dev kit (L3), now.**
   - Hardware: an RK3566 board with mainline. Quartz64 or Radxa ZERO 3 on a bench supply; RG353 or PineNote as a battery-fed reference.
   - Instrument: Joulescope JS220.
   - Cases: screen-off idle; FLAC 16/44.1 to an I2S codec with ALSA buffers of 20 ms, 100 ms, 500 ms and 2 s.
   - Also record cpuidle residency and wakeups per second (`powertop`, `wakeup_sources`).
   - **This decides between options A, E and F in §11.4.**
2. **Per-rail budget on the T1 prototype.** Measure each state of every §4 row:
   - the OPA1622 EN-off current;
   - both oscillators, one gated and one running;
   - the CS43198 per format (PCM 44.1, 192 and 384 kHz; DSD128 native and DoP);
   - RK809 light-load efficiency;
   - the backlight at 20, 200 and 800 nit, with luminance measured.
3. **Amplifier stage against §5.**
   - Use a load box (16 Ω, 32 Ω, 300 Ω, open) with pink noise and real music, at 0.1, 1 and 10 mW per channel, SE and balanced.
   - Fit *k*. Reconcile with the R3Pro II benchmark (§10.3) by measuring a JM21 or R3Pro II at matched level, if one can be borrowed.
4. **Scenario runs** with `opendap power check` and a scenario runner (to build).
   - Each scenario in §6, for 30 minutes on a bench supply at 3.8 V.
   - Then **full rundowns** on the real pack at 25 °C, from 100 % to shutdown, with a fixed looped playlist.
   - Pass: within ±10 % of the model (P7).
5. **Fuel-gauge logging during rundowns.**
   - Log MAX17055 `AvgCurrent`, `VCell`, `RepSOC`, `RepCap` and `TTE` every 60 s while the screen is off (slow, alert-driven reads).
   - Compare with the Joulescope to calibrate TTE (the user-facing "hours left").
   - The gauge shows trends; mW-level differences need the external analyser ([DOC] blackbird-os §8.6).
6. **Captured samples alongside power.**
   - Capture I2S passively on the BRD-040 pads during each optimised run.
   - Validate bit-perfect output with the toolkit's validators (PCM compare, `dop-check`) and count xruns.
   - A configuration that saves power but drops samples **fails**.
7. **Benchmark parity (later).**
   - Run one category device (JM21 or R3Pro II) and the T1 through the same rundown: same files, same level at the jack, measured with the QA403.
   - That replaces makers' claims with a like-for-like comparison before any runtime is published.
8. **Ageing and temperature (later):** repeat scenario 1 at 0 °C and 35 °C, and on a pack aged to 80 % state of health.

---

## 13. Open questions and risks

1. **SoC idle floor (biggest risk).** If RK3566 runtime idle is above ≈ 0.4 W, every typical runtime drops by 10–20 %. It also pushes the body to 24–25 mm.
2. **Audio power tree (BOM gap):** the ± rail generator, the rail voltages per gain mode, and the DAC 1.8 V pre-regulation. Together they are worth ≈ 100–200 mW.
3. **OPA1622 topology:** how many channels each output enables (2 or 4 for SE).
4. **Dual mono or a single DAC for SE** (BRD-022, Draft): a ≈ 40 mW lever.
5. **Benchmark quality.** The average uses makers' claims with unstated conditions, from 2 brands (FiiO, HiBy). Shanling, iBasso, Sony and A&K could not be read. Adding Sony would pull the average up.
6. **Battery:** the 105060, 105050, 705055 and 905055 classes are quote-only. The current stocked basis is the Adafruit 328 (88 % of average).
7. **RK809 gauge statement:** the RK809 datasheet (Rev 1.01, §1.1) says a fuel gauge is integrated. The component-selection report says "no charger, no gauge". The board documents should agree (**UNVERIFIED** for the RK809-5 sold today).
8. **Datasheets still to read:** CS43198 per-mode power, MAX17055, MAX98357A, LSM6DSV16X, GT911 currents, ILI9488 panel current, IW612, Crystek CCHD-957, and the Samsung LPDDR4X and eMMC IDD tables.

---

## 14. Sources

**Component datasheets and pages** (read 2026-10-22):

- TI OPA1622 datasheet: Iq 2.6 / 3.3 mA per channel, shutdown 5 / 10 µA per channel, EN thresholds, ±2 to ±18 V. <https://www.ti.com/lit/ds/symlink/opa1622.pdf>
- Cirrus Logic CS43198 product page: 26 mW playback, 1.8 V. <https://www.cirrus.com/products/cs43198/>
- TI BQ25895 datasheet: quiescent currents, 11 mΩ BATFET. <https://www.ti.com/lit/ds/symlink/bq25895.pdf>
- onsemi FUSB302B datasheet: Idisable 0.37 µA, Itog 25 / 40 µA, PD standby 40 µA. <https://www.onsemi.com/pdf/datasheet/fusb302b-d.pdf>
- Rockchip RK809 datasheet Rev 1.01 (mirror): standby 25 µA, buck efficiencies. <https://rockchip.fr/RK809%20datasheet%20V1.01.pdf>
- NDK NZ2520SDA catalogue: 15 mA max, 35 µA standby, 4 ms start-up. <https://www.ndk.com/images/products/catalog/c_NZ2520SDA_e.pdf>
- Raspberry Pi RP2350 datasheet, §14.9.7, Tables 1692–1693. <https://datasheets.raspberrypi.com/rp2350/rp2350-datasheet.pdf>
- TI TPS2121 datasheet. <https://www.ti.com/lit/ds/symlink/tps2121.pdf>
- TI TPS25947 datasheet. <https://www.ti.com/lit/ds/symlink/tps25947.pdf>
- Vishay VEML6030 datasheet. <https://www.vishay.com/docs/84366/veml6030.pdf>
- Goodix GT911 programming guide (Crystalfontz mirror). <https://www.crystalfontz.com/controllers/GOODIX/GT911ProgrammingGuide/478/>

**DAP makers' claims** (read 2026-10-22):

- FiiO M11: <https://www.fiio.com/m11>
- FiiO M11 Plus LTD: <https://www.fiio.com/m11plusltd>
- FiiO M11S: <https://www.fiio.com/m11s>
- FiiO JM21: <https://www.fiio.com/jm21>
- FiiO M21: <https://www.fiio.com/m21>
- FiiO M23 (runtime in a chart only): <https://www.fiio.com/m23>
- FiiO M15S (no runtime figure): <https://www.fiio.com/m15s>
- HiBy R1: <https://store.hiby.com/products/hiby-r1>
- HiBy R3 II (2025): <https://store.hiby.com/products/hiby-r3-ii-2025>
- HiBy R3Pro II: <https://store.hiby.com/products/hiby-r3pro-ii>
- HiBy R4: <https://store.hiby.com/products/hiby-r4>
- HiBy R5 II: <https://store.hiby.com/products/hiby-r5-gen-2>
- HiBy R6 III (2025): <https://store.hiby.com/products/hiby-r6-iii-2025>

**Not obtained** (§13, item 8): the CS43198 full datasheet, MAX17055, MAX98357A, LSM6DSV16X, IW612, CCHD-957, the Samsung memory tables, the Shanling, iBasso, Sony and Astell&Kern runtime specs, and the RK3566 comparable-device measurements.
