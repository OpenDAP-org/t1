# Electronics integration and verification

- **Status:** Draft v0.1 (2026-10-24), decided by Claude under the maintainers' delegation: specialised modules per domain are built, provided the integration is verified as well and never inferred from the parts. Task E5 of the [electronics plan](../plans/2026-10-electronics-plan.md).
- **Scope:** the T1's electronics, from parts to the assembled product. Case, mechanical and physical-simulation work is out of scope; the levels that need it (L5, L6) list only their electrical evaluations.

## 0. Principles

1. **Parts passing does not make the whole pass.** An interaction between two domains is signed off only at the lowest level that reproduces its physical mechanism (conduction, proximity, shared ground, shared clock, heat, software timing). A mechanism that a level cannot reproduce is marked there as *not representative*, never as passed.
2. **What is tested is what ships.** Every module is built from the same hierarchical schematic sheets as the main board, and its layout block is copied from or into the main board's, with the same stackup class, part numbers and values. A module that diverges from the board is re-tested after the board is laid out (§2, rule M4).
3. **Every row traces to a requirement or a constraint.** Each evaluation row names the IDs it verifies (T1-, BRD-, PCB-, ADR, BOM §7 item) and carries the T1-048 thresholds (ideal, warning, hard limit) where the metric has them.
4. **Regression follows the dependency graph.** A change to a block reruns its L1 and L2 rows and every L3 and L4 row of each interaction it takes part in (§3).
5. **Measurement before shrinking.** A level's exit gate is evidence (data file, capture, photo), stored with the component that was tested; a result without evidence counts as not run.

## 1. Integration levels

| Level | What is integrated | Where it lives | Representative for | Not representative for | Exit gate |
|---|---|---|---|---|---|
| **L0 Part** | One part against its datasheet | parts-lib sheet, kicad-lib symbol/footprint, fab rule test boards | Pin map, land pattern, ratings, model availability | Anything about the circuit | Sheet transcribed, footprint checked against the drawing, availability dated (PLIB-003) |
| **L1 Block** | One circuit block (a hierarchical sheet) | `sim/` studies (`odapkit sim`), schematic review | Topology, values, margins, stability, levels | Layout parasitics, coupling, real part spread | Every `checks.toml` row passes; ERC clean; datasheet review checklist done |
| **L2 Domain module** | One domain on its own test board, real parts and real layout block | Module boards (§2) | Layout-dependent performance of one domain, part spread, bench items of BOM §7 | Interactions with other domains | Every L2 row of the domain passes with evidence |
| **L3 Domain pair** | Two or more modules connected through the core module | Module stack (§2.2) | Conducted coupling (supply, ground return through defined interconnect), protocol and timing, power sequencing, software bring-up | Proximity and radiated coupling, shared copper planes, thermal coupling, final ground geometry | Every L3 row of the pair passes; conducted-coupling rows recorded as upper bounds |
| **L4 Integrated board** | Main board (prototype 1) with faceplate and HAT device-side piece on the bench | board | Everything electrical, including proximity, radiated and thermal coupling on the real board | Enclosure effects (antenna detuning, case thermal, ESD through the case) | Every L4 row passes; every interaction of §3 signed off |
| **L5 Product assembly** | Board, faceplate, battery pack and HAT piece in the case | t1 | Electrical behaviour in the enclosure: radio in the case, ESD at the product surface, thermal at the surface, audio with the final jacks and ground | Long-term behaviour | Every L5 row passes |
| **L6 Product in use** | The T1 running Blackbird OS through real sessions | t1, toolkit-verify | Autonomy (T1-039), endurance and scale (T1-046, T1-047), experience budgets (T1-048), compliance pre-scans | — | Release gate: no hard limit crossed |

## 2. Domain modules (L2) and the module stack (L3)

### 2.1 Modules

| Module | Contents (BOM v1.7) | Main questions it answers | IDs |
|---|---|---|---|
| **M-CORE** | RK3566, LPDDR4X, eMMC, RK809-5, RK860-0, 24 MHz crystal, boot and MaskROM circuit, USB-C OTG0 for flashing, UART pads | HDI stackup and BGA escape; LPDDR4X at full speed; PMIC power-up defaults (E1.7, measured instead of asked); SoC power by mode, the measurement t1 ADR 0003 waits for; MaskROM and eMMC timing | BRD-001, BRD-016, BRD-019, BRD-035, BRD-043, BRD-047; PCB-S, PCB-G-02/03, PCB-I-05/06 |
| **M-PWR** | 2S battery path, BQ25798, BQ28Z610-R1, TPS2121, FUSB302B + USB-C, TPS62136, TPS62903 ×2, eFuses ×3, fuse, ship FET, protections | Charging and power path across sources; full output budget on battery; converter stability (§7.27), thermal (§7.24); VBUS surge; eFuse limits and inrush | BRD-007, BRD-026, BRD-049, BRD-050; ADR 0014; PCB-P |
| **M-AUD** | 2 × CS43198, 6 × OPA1622, TPS65131, LMR33630C, LT3045/LT3094 pairs, AUDIO_2V2, master clocks and attenuator, both jacks with detection, output networks | Noise floor, distortion, output power and impedance, channel balance, pops at every transition, rail and clock behaviour; BOM §7.1, §7.29–§7.31 | BRD-011, BRD-022 to BRD-024, BRD-052, T1-049, T1-050, ADR 0010, 0012, 0015; PCB-A |
| **M-RF** | Murata Type 2EL, U.FL ×2, matching networks, antennas | Throughput and range per band, Bluetooth audio, coexistence; adherence to Murata's reference design (pre-certification) | BRD-025, BRD-029; PCB-R |
| **M-UI** | Board side of the faceplate interface: FH35C, backlight driver, LP5521, level shifting; with the touch faceplate prototype | DSI 1-lane bring-up, touch over I2C, HID-over-I2C controls, LED, PDM microphone | T1-018, T1-036, BRD-048; faceplate interface v0 |
| **M-IO** | RP2354A, buttons, IMU, DRV2605L + LRA, NTAG / PN7160, SLB9673 TPM, MAX98357A + speaker, microSD (SDR104) | Coprocessor flashing from the SoC; button behaviour with the screen off; peripherals on mainline or DKMS drivers; SD at SDR104 | BRD-014, BRD-015, BRD-036, BRD-051, BRD-056 to BRD-059, T1-060 |
| **M-HAT** | DF40 pair, FPC, device-side piece with the pogo field, a test HAT with loopbacks for USB 3, PCIe, CSI and power classes | Signal integrity at 5 Gb/s through DF40, FPC and pogo pins (hat-standard ADR 0004 coupon); power classes and protection; conformance L1–L3 | BRD-006, BRD-013, BRD-026, BRD-044, BRD-045; hat-standard interface v1 |
| **M-SI** | Impedance and loss coupons of the main-board HDI stackup and of the two flat cables | Impedance per class (TDR), loss, microvia reliability (cross-section) | PCB-I-01 to PCB-I-07, PCB-S |

Module rules:

- **M1.** Each module is a KiCad project in the repository of the component it serves, built from the same hierarchical sheets as the main board (no re-drawn copies).
- **M2.** M-CORE uses the main board's stackup and fab process; the other modules use the cheapest stackup that keeps their critical layout identical (same layer adjacency under the block). M-AUD keeps the main board's ground and plane arrangement under the audio block.
- **M3.** Modules connect to M-CORE through one documented module connector per boundary, carrying the nets that cross that boundary on the main board, with ground on every second contact and the supply returns sized for the full budget. The connector map is a machine-readable interface file (E4.6), checked against the main-board netlist.
- **M4.** After the main board is laid out, a module whose layout block differs from the board's is either updated and re-tested, or its L2 results are marked *superseded by L4*.

### 2.2 Module stack (L3)

M-CORE is the hub: every other module attaches to it, so pairs and the full stack run real Blackbird OS. M-PWR powers the stack through the same rails as on the board. The stack is representative for conducted effects through its defined interconnect only; every proximity, radiated or thermal interaction is first signed off at L4.

## 3. Interaction matrix

Each cell names the coupling mechanism, the level where it is first testable, and the level where it is signed off. Domains: PWR (power and charging), CORE (SoC, memory, storage), AUD (DAC, amplifiers, clocks, jacks), RF (radio), UI (faceplate), HAT, IO (coprocessor and peripherals), EXT (USB-C and the outside world).

| | CORE | AUD | RF | UI | HAT | IO | EXT |
|---|---|---|---|---|---|---|---|
| **PWR** | Load transients of DVFS on VSYS_LV and vdd_cpu; sequencing; suspend and wake. First L3, signed off L4 | Converter ripple and PFM bursts into the amplifier rails and DAC supplies (PSRR budget, §7.31); ground return of the switchers. First L3 (conducted), signed off L4 (proximity) | Switching harmonics in the radio bands (desense). L4 | Backlight current steps on VSYS_LV. L3 | HAT inrush and fault on the shared 5.1 V converter; powerbank HAT as a source. L3, signed off L4 | Haptic and speaker current pulses. L3 | Source switching, PD negotiation, surge and ESD on VBUS. L2, signed off L5 |
| **CORE** | | I2S/DSD timing, DAC as clock master, bit-perfect paths, sample-rate changes without pops. First L3. DDR and SoC noise coupling into analog: L4 | DDR and SDIO harmonics into the radio bands; SDIO timing. L3 (protocol), L4 (desense) | DSI timing, touch interrupt latency against the experience budgets. L3 | USB 3 and PCIe link training through the cable and pogo field; DLN2 bridge. L3, signed off L4 | I2C bus loading and addresses; coprocessor flashing (BRD-015). L3 | USB OTG roles, MaskROM recovery, USB serial console (BRD-046). L3 |
| **AUD** | | | Radio bursts (TDMA, BT) demodulated in the amplifiers: audible buzz. L4, signed off L5 | Display and backlight noise into the headphone path. L4 | HAT audio on the DAC clock domain; line-out ground. L3, signed off L4 | Speaker amplifier and haptics against the headphone path; jack detect and speaker never chosen unasked (T1-053). L3 | Ground loops when USB-connected to a computer (USB DAC mode, T1-054). L4, signed off L5 |
| **RF** | | | | Faceplate and display as antenna loading. L5 | HAT shielding and metal near the antennas. L5 | NFC field against nearby metal and the TPM I2C. L4 | Antenna in the enclosure; certification adherence. L5 |
| **UI** | | | | | — | Buttons and faceplate controls agree (one press, one action). L3 | ESD into the faceplate through the cover. L5 |
| **HAT** | | | | | | — | ESD and hot-plug at the pogo field. L4, signed off L5 |
| **IO** | | | | | | | ESD at the side buttons and the microSD slot. L5 |

## 4. Evaluation matrices

Columns: `ID | case | steps | expected (with thresholds) | observed | evidence`. The tables below fix the cases and their expected values; *observed* and *evidence* are filled when the level runs, stored next to the module or board that was tested. Thresholds come from the requirement, the ADR or the constraint named; where none exists yet, the case says what sets it.

### 4.1 L0 Part

| ID | Case | Expected |
|---|---|---|
| V0-01 | Footprint against the manufacturer's drawing (pads, courtyard, pin 1, 3D model height) | Matches; reviewed by a second reading |
| V0-02 | Symbol pin map against the datasheet pin table | Matches; Manufacturer and MPN fields (PCB-T-05) |
| V0-03 | Availability dated, second source named | PLIB-003, T1-028, T1-029 |
| V0-04 | Simulation model available and its hash recorded | `odapkit sim fetch` succeeds or the block's checks are marked blocked |

### 4.2 L1 Block

| ID | Case | Expected |
|---|---|---|
| V1-01 | Converter stability over line, load and temperature corners | Phase margin ≥ 45° (board sims), peak currents ≤ 80 % continuous and ≤ 100 % at a limiter trip (BRD-049) |
| V1-02 | MaskROM entry and normal boot over corners | `sim/maskrom` checks |
| V1-03 | Master clock levels, duty and edges into the CS43198 | `sim/mclk-attenuator` checks (E1.6) |
| V1-04 | Headphone stage at full scale into 16 and 32 Ω, both outputs | ADR 0012 and 0015 figures; rail headroom |
| V1-05 | Every block reviewed against its datasheet application circuit | Checklist item per part, deviations justified |

### 4.3 L2 Domain modules

| ID | Module | Case | Expected |
|---|---|---|---|
| V2-C01 | M-CORE | Power-up sequence captured on every rail | Matches the RK809-5 defaults recorded for the board; rails within RK3566 recommended ranges |
| V2-C02 | M-CORE | LPDDR4X memory test at the highest supported rate, hot and cold | No errors over the test duration; eye or margin data where the SoC exposes it |
| V2-C03 | M-CORE | eMMC HS200 and boot; MaskROM and SD recovery | BRD-016, BRD-043; boot every time over 100 power cycles |
| V2-C04 | M-CORE | SoC and memory power per mode (idle, playback-equivalent load, suspend) | Feeds the battery-life model; T1-039 thresholds evaluated with the measured draw |
| V2-C05 | M-CORE | Impedance coupon on the same panel | PCB-I classes within PCBWay ±10 % or the standard's window |
| V2-P01 | M-PWR | Charge from USB-C PD and from a HAT source, every transition | No brown-out of VSYS_LV; TPS2121 interlock holds (power tree finding 10) |
| V2-P02 | M-PWR | Full output budget on battery: HAT_5V 1.5 A + HAT_3V3 1.0 A + headphone full scale | VSYS ≥ 6.6 V shed point respected; every part within 70–80 % of its rating (BRD-049, BRD-050) |
| V2-P03 | M-PWR | Converter load steps and Bode plots | Phase margin ≥ 45°, closes BOM §7.27 |
| V2-P04 | M-PWR | eFuse limits, short circuit, inrush with 470 µF | ILIM within the RILM tolerance; no upstream disturbance |
| V2-P05 | M-PWR | VBUS surge and ESD at the connector | No damage; downstream absolute maxima respected (power tree risk 5) |
| V2-P06 | M-PWR | Thermal at the full budget, 30 min | Part temperatures within ratings with margin (BRD-049); measured, not simulated |
| V2-A01 | M-AUD | Noise floor, both outputs, into 16 Ω and 32 Ω and open | Sets the T1-048 thresholds (experience budgets §Audio output) from ADR 0012's design figures |
| V2-A02 | M-AUD | THD+N and output power versus level and load | ADR 0012 and 0015 figures |
| V2-A03 | M-AUD | Output impedance | ≤ 0.5 Ω ideal, > 2 Ω hard limit (experience budgets) |
| V2-A04 | M-AUD | Channel balance at low volume, volume step | ≤ 0.1 dB ideal / > 1 dB hard; ≤ 0.5 dB step (T1-050) |
| V2-A05 | M-AUD | Pops at power, play, pause, seek, rate change, PCM/DSD change, output change, gain change, plug and unplug | None audible (T1-049); measured peak at the jack recorded |
| V2-A06 | M-AUD | Both clock families, every sample rate, PCM and DSD | Bit-perfect (OS-020); DAC locked; no resampling (BRD-023) |
| V2-A07 | M-AUD | Boost, inverter and LDO bench items | Closes BOM §7.29–§7.31 |
| V2-R01 | M-RF | Throughput and range per band, Bluetooth audio stability | Murata reference figures **UNVERIFIED** until E1.11 |
| V2-R02 | M-RF | Layout against Murata's reference design | No deviation (BRD-025) |
| V2-U01 | M-UI | Display at full frame rate, touch report rate and latency | T1-056 and the experience budgets |
| V2-U02 | M-UI | Faceplate identification, controls over HID, LED states | Faceplate interface v0, controls and status LED spec |
| V2-I01 | M-IO | Coprocessor flashed and updated from the SoC; SoC boots without it | BRD-015, BRD-019 |
| V2-I02 | M-IO | microSD at SDR104 | BRD-051; sustained read rate recorded |
| V2-I03 | M-IO | Every peripheral bound by its mainline or DKMS driver | BRD-001 |
| V2-H01 | M-HAT | USB 3 and PCIe through DF40, FPC and pogo field: eye or link margin, error rate | hat-standard ADR 0004 coupon criteria |
| V2-H02 | M-HAT | Power classes, eFuse trips, hot-plug | hat-standard interface v1; BRD-026 |
| V2-S01 | M-SI | TDR per impedance class, microvia cross-section | PCB-I targets; PCBWay acceptance class |

### 4.4 L3 Domain pairs

| ID | Pair | Case | Expected |
|---|---|---|---|
| V3-01 | PWR + CORE | DVFS and load steps from the SoC with the real converters | VSYS_LV and vdd_cpu within range; no reset |
| V3-02 | PWR + CORE | Suspend, resume, shutdown, battery cut-off, ship mode | Resume state kept (T1-051); no rail left floating |
| V3-03 | PWR + AUD | Converters running at worst load while measuring V2-A01 and V2-A02 | No degradation beyond the L2 values; spurs listed with their frequency (upper bound for L4) |
| V3-04 | CORE + AUD | Playback of every rate and format from Blackbird OS; rate changes; pause and seek | Bit-perfect; no pops (T1-049); latency within T1-057 budgets |
| V3-05 | CORE + RF | Wi-Fi and Bluetooth under SDIO and DDR load | Throughput within 10 % of V2-R01 **UNVERIFIED** threshold |
| V3-06 | CORE + UI | Interface at full frame rate during library scan and DSD playback | T1-056 |
| V3-07 | CORE + HAT | USB 3 and PCIe devices on the test HAT, DLN2 bridge, hot-plug | Links train at full rate; no error growth over 1 h |
| V3-08 | CORE + IO | Every I2C peripheral on its bus at once; coprocessor flashing during playback | No bus errors; audio unaffected |
| V3-09 | Full stack | Blackbird OS running every domain at once for 24 h | No resets, no driver errors, no memory growth (T1-047) |

### 4.5 L4 Integrated board

Every L2 and L3 row is repeated on the board (the L2/L3 values become the reference), and these are added:

| ID | Case | Expected |
|---|---|---|
| V4-01 | Audio noise and THD+N with every other domain active (radio transmitting, display on, HAT loaded, charging) | Within the T1-048 thresholds set at L2; no spur above the warning value |
| V4-02 | Radio sensitivity with the board's own noise (desense), per band | Within Murata's figures minus a documented margin **UNVERIFIED** |
| V4-03 | Thermal map at the full budget and at playback | Measured part temperatures; data handed to the out-of-scope thermal work (E4.10) |
| V4-04 | Power by use case on the board | Updates the battery-life model; T1-039 evaluated |
| V4-05 | ESD at every connector on the bare board | No damage, no lock-up |
| V4-06 | USB DAC mode connected to a computer | Bit-perfect, no added noise (T1-054); ground-loop noise recorded |

### 4.6 L5 Product assembly and L6 in use (electrical rows only)

| ID | Level | Case | Expected |
|---|---|---|---|
| V5-01 | L5 | Radio range and throughput in the case | Within the L4 values minus the enclosure loss, recorded |
| V5-02 | L5 | ESD at the product surface (cover, buttons, jacks, USB-C) | No damage, no lock-up, no audible event beyond a recovery |
| V5-03 | L5 | Audio rows of L4 with the final jacks and ground | No change beyond measurement repeatability |
| V6-01 | L6 | Core audiophile session autonomy | T1-039: > 12 h ideal, < 8 h warning, < 6 h hard limit |
| V6-02 | L6 | Endurance and scale | T1-046, T1-047, experience budgets |
| V6-03 | L6 | Compliance pre-scans (conducted and radiated emissions) | Below the applicable limits with margin, recorded |

## 5. Gates between levels

- A module goes to L3 only after its L2 rows pass or each failure has a recorded fix in the main-board design.
- The main board is released for fabrication (E8.1) only after every L2 bench item of BOM §7 is closed or carried into the board with a decision, and every L3 row that can fail on the board's design (not only on the module interconnect) has passed.
- The product enters L6 only when every interaction of §3 is signed off at its level.

## 6. Open

- Instruments and their residuals (audio analyser, TDR/VNA, current probes) are chosen with the module designs; a measurement's instrument residual must sit at least 10 dB below the expected value (**UNVERIFIED** margin, to confirm with the first instrument).
- KiCad 10 support for reusing a layout block between projects (rule M1/M4) is to be checked before the first module layout.
- Thresholds marked "set at L2" are filled from the first measurements and confirmed by the maintainers (experience budgets).
