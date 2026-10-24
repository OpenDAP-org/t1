# T1 electronics work plan

- **Date:** 2026-10-24
- **Applies:** the [hardware engineering system plan](https://github.com/opendap-project/toolkit-hardware/blob/main/docs/plans/2026-10-hardware-engineering-system-plan.md) (toolkit-hardware): the methods, standards and tools every OpenDAP device is built with. This plan keeps only the T1's own tasks; tasks moved to the system plan on 2026-10-24 keep their line here, pointing to their new ID.
- **Status:** Active. Tasks change status as work lands; the plan itself changes only with the maintainers.
- **Scope:** the T1's electronics: main board, touch-faceplate electronics, HAT device-side piece and the flat cables (board, faceplate-touch, hat-standard), with their libraries (kicad-lib, parts-lib) and tools (toolkit-hardware, toolkit-hooks).
- **Out of scope:** the case, mechanical CAD and physical simulations (thermal, structural, acoustic), the other faceplates and the HATs themselves. The electronics still own what those need from them, as data: board outline and keep-outs taken in, height zones per side, board STEP and per-part dissipation handed out (E4.9, E4.10).

## Where the work starts

The electronics exist as documents: board BOM v1.7, power tree v0, a v0 pin plan, 15 board ADRs, HAT interface v1 with 4 ADRs, 128 parts-lib spec sheets and three ngspice studies (board `sim/`). There is no KiCad project, no symbol or footprint, no fab rule set for HDI or flex and no `component.yaml` yet. The BOM lists items it must close before schematic capture (§7.22, §7.28) and before layout (§7.1, §7.29–§7.31).

## Quality levels

Levels per [RFC 0010](https://github.com/opendap-project/rfcs/blob/main/text/0010-quality-levels.md), as in the [system plan](https://github.com/opendap-project/toolkit-hardware/blob/main/docs/plans/2026-10-hardware-engineering-system-plan.md#quality-levels). T1 targets: risk-reduction modules and prototype 1 at **Q1**; pilot units and the DIY kit at **Q2**; the Commercial edition at **Q3**; every revision moves toward **Q4**.

## Critical path

1. **Battery:** two pack makers quote the 2S pack (board ADR 0014) → battery connector and pocket edge freeze → board outline.
2. **Pins:** pinout v1 against the full RK3566 datasheet → faceplate contact positions and HAT DF40 numbering → schematic.
3. **Rules:** PCB design constraints (E2) → fab and product rule sets (E3) → KiCad project with stackup and net classes (E4.1) → schematic and layout.

## Workstreams

Task status: `todo`, `doing`, `done`, `blocked` (with its blocker), `decision` (waits for the maintainers).

### E0. Consolidate the documents

Contradictions are fixed before anything is built on them. The governing source is the most recent Accepted ADR or Confirmed requirement; anything else is flagged, not resolved.

| ID | Task | Repository | Status |
|---|---|---|---|
| E0.1 | Board: bring BOM §7 and §9, the power tree, ADR 0008/0009/0010 notes, the pin plan header and the 2S study up to ADR 0014 and ADR 0015 (unused TPS65131 inverter, BQ25798 driver state, DF40 power pin count, FH35C-51S, stale line references) | board | done (board `9498a39`) |
| E0.2 | Product, case and touch notes: body 59.5 × 94.3 mm (t1 ADR 0003) where 91.5 mm remains; battery basis in t1 ADR 0003 after board ADR 0014; stale "Draft" notes in case and faceplate-touch requirements | t1, case, faceplate-touch | done (t1 `5147da8`, faceplate-touch `ab8c492`); case notes left, out of this scope |
| E0.3 | HAT standard: one contact field everywhere (4 × 25 at 2.54 mm, Harwin P70, Mill-Max 0965; hat-standard ADR 0004) in interface v1, the requirements and board BOM line 67 | hat-standard, board | done (hat-standard `42fcf14`, board `9498a39`) |
| E0.4 | List what needs a maintainers' call: requirements still Draft behind Accepted ADRs (BRD-020, BRD-022, BRD-024, BRD-037, FPT-008), BRD-001 against the kernel configuration in BOM v1.7, the LP5521 drive direction against the faceplate interface | all | done: resolved on 2026-10-24 (board `6e4d540`, faceplate-touch `aee106a`) |

### E1. Freeze the inputs to the schematic

| ID | Task | Repository | Status |
|---|---|---|---|
| E1.1 | Pinout v1: check every pin of the v0 plan against the full RK3566 datasheet (IO domains, muxing, boot strapping) | board | todo |
| E1.2 | Faceplate contact positions on the FH35C-51S from pinout v1 (RFC 0001 group order) | t1, board | blocked (E1.1) |
| E1.3 | HAT DF40 100-pin and 50-pin contact numbering, pull-up ownership and power classes (interface v1 §10) | hat-standard | blocked (E1.1) |
| E1.4 | BOM §7.22: BQ28Z610-R1 gauge-only configuration from TI's reference manual | board | doing: narrowed (board `4cf3306`, parts-lib `a2f1dfc`); open: PACK tied to BAT+ without FETs (gauge never shuts down above 3.0 V), 5 mΩ sense resistor outside TI's 1–3 mΩ, defaults to read back from a chip |
| E1.5 | BOM §7.28: effective output capacitance of the 5.1 V TPS62903 (≥ 12 µF after DC bias) | board | done: one GRM32ER71E226KE15L keeps ≥ 13.9 µF at 5.1 V worst tolerance (board `4cf3306`) |
| E1.6 | BOM §7.1: SPICE of the master-clock attenuator into the CS43198 clock input | board | doing: study `sim/mclk-attenuator` (board `4cf3306`); passes with a typical oscillator, fails with the weakest output NDK allows (`xo-load`); see E1.14 |
| E1.7 | RK809-5 power-up defaults and RK860-0 data: ask Rockchip, else follow the ROCK 3C reference and plan a power-up capture | board | todo: measured on M-CORE (V2-C01) instead of asked; the ROCK 3C reference meanwhile |
| E1.8 | Flat-cable specification board ↔ HAT device-side piece (BRD-013): stack-up, impedance, length, bend, stiffeners | board | blocked (E2, E1.3) |
| E1.9 | Faceplate flex specification: FH35C tail (0.2 ± 0.03 mm, 15.6 ± 0.05 mm), DSI pairs, stiffener | faceplate-touch | blocked (E1.2) |
| E1.11 | Retrieve the Rockchip RK3566 hardware design guide and the Murata Type 2EL hardware design guide (constraints §9 items 2 and 3) | parts-lib | done: Rockchip (parts-lib `9be21a0`, board `7be75ba`), Murata notes (parts-lib `c65b1bb`, board `3b4e1fd`); the 2EL datasheet is still missing |
| E1.12 | Power tree: a 1.8 V power rail for the radio module (AVDD18, peaks above 1 A) and the LPDDR4X 1.10 V VDD2 at the SoC; check against power tree v0 and the BOM | board | todo |
| E1.13 | Antennas from Murata's certified lists for the target regions (the Molex 146187 is on Japan's only), and the SAR / permissive-change path for handheld use | board, t1 | todo |
| E1.14 | Master-clock duty: the NSC5083D symmetry spec (40–60 %) is wider than the CS43198's 45–55 % whatever the attenuator does, and two DAC pins can exceed the oscillator's 15 pF load. Options to study and decide: an oscillator with a 45–55 % symmetry spec, a buffer per DAC, or an oscillator at twice the frequency divided by two (exact 50 %), each with its phase-noise and jitter budget against ADR 0010 | board | todo |
| E1.15 | Feature-block gaps G1–G21 of the [block catalog](https://github.com/opendap-project/board/blob/main/docs/specs/feature-blocks.md) (IMU wake rail, DAC I2C back-powering, MCLK into an unpowered DAC, RP2354A taps, TPM and NFC supplies, PN7160 supply, faceplate pull-ups, per-block measurement points) into the power tree and the schematic | board | todo |
| E1.10 | Battery pack quotes from two makers (ADR 0014 item 2) | board | deferred: vendor contact after the engineering work (maintainers, 2026-10-24) |

### E2. PCB design constraints

One specification per board, deriving every limit from the product requirements, the parts and the fab, per the [PCB design constraints method](https://github.com/opendap-project/toolkit-hardware/blob/main/docs/specs/pcb-design-constraints-method.md) (system plan HW3.5). Each constraint has an ID, a target, a hard limit, its source, and the rule or check that enforces it. Values without a source are marked **UNVERIFIED**.

| ID | Task | Repository | Status |
|---|---|---|---|
| E2.1 | Main board constraints: outline and keep-outs taken from the product, height zones per side, stackup and build, impedance classes, length matching, current capacity, audio partition and grounding, clocks, RF keep-out, power integrity, ESD and protection, test access, assembly and DFM, variants | board | doing: Draft v0.1 (board `6b0d1bc`, BRD-060); open items in its §9 |
| E2.2 | Flat-cable and HAT device-side piece constraints | board, hat-standard | blocked (E1.3) |
| E2.3 | Touch-faceplate electronics constraints | faceplate-touch | blocked (E1.2) |
| E2.5 | Front-side height under the display (constraints PCB-Z-03, §9 item 1): grow the gap, single-sided under the display, or a recess | board, t1 | doing: decision delegated to Claude (2026-10-24) |
| E2.4 | Machine-readable form of the constraints (one data file per board) so rule sets, net classes and checks are generated from it, not copied | board | done: `hardware/rules/netclasses.toml` (board `46c3a66`); project generation in E4.1 |

### E3. Design rules (DRC)

The fab rule sets, the composition tool and the rule test boards are system work ([system plan](https://github.com/opendap-project/toolkit-hardware/blob/main/docs/plans/2026-10-hardware-engineering-system-plan.md) HW1, HW2.1). The T1's product rules are composed on top of the fab set.

| ID | Task | Repository | Status |
|---|---|---|---|
| E3.1 | Moved: system plan HW1.1 (`pcbway-hdi`) | kicad-lib | done |
| E3.2 | Moved: system plan HW1.2 (`pcbway-flex`) | kicad-lib | done |
| E3.3 | Moved: system plan HW1.3 (`pcbway-multilayer`) | kicad-lib | done |
| E3.4 | T1 main-board product rules (from E2.1) and test board | board | done: v0.1, 21 rules, 21/21 fire (board `46c3a66`) |
| E3.5 | Moved: system plan HW2.1 (`odapkit dru`, hook `kicad-dru-composed`) | toolkit-hardware, toolkit-hooks | done |
| E3.6 | Moved: system plan HW1.4 (PCBWay stackups and unpublished limits) | kicad-lib | deferred |

### E4. Engineering cycle tooling (electronics)

Each design iteration runs the local loop of the [system plan](https://github.com/opendap-project/toolkit-hardware/blob/main/docs/plans/2026-10-hardware-engineering-system-plan.md#the-engineering-cycle). The generic tools are system work; these are the T1's tasks.

| ID | Task | Repository | Status |
|---|---|---|---|
| E4.1 | Board KiCad project skeleton: kicad-lib as a pinned submodule, stackup, net classes and rules generated from E2.4, hierarchical sheets per subsystem | board | blocked (E2.1, E3.4) |
| E4.2 | Symbols, footprints and 3D models for the critical parts first: RK3566, RK809-5, LPDDR4X, eMMC, CS43198, OPA1622, RP2354A, DF40, FH35C, USB4105, jacks, Micro-Fit, P70 | kicad-lib | todo |
| E4.3 | Moved: system plan HW2.7 (parts-lib availability field, PLIB-003) | parts-lib | done |
| E4.4 | Moved: system plan HW2.2 (`odapkit sim`); the T1 board checks (15 pass, 1 blocked, board `ddaf1d5`) are its first application | board, toolkit-hardware | done |
| E4.5 | Moved: system plan HW2.4 (signal-integrity loop), checked against the T1 constraints (E2) | toolkit-hardware | todo |
| E4.6 | Machine-readable interface files (faceplate, HAT, battery connector, board ↔ case envelope) with checkers that both sides match | t1, hat-standard, board | todo |
| E4.7 | Moved: system plan HW2.3 (`component.yaml` schema) | toolkit-hardware | done |
| E4.8 | Moved: system plan HW2.5 (fab pipeline dry run) | toolkit-hardware | blocked (HW1.1) |
| E4.9 | Board export for integration: STEP and glTF per the export contract | board | blocked (E4.1) |
| E4.11 | Moved: system plan HW2.6 (toolkit-hooks v0.4.0) | toolkit-hooks | todo |
| E4.12 | Sim inputs: `--json` output for `vsys-bucks/run.py`, parts-lib SPICE model entries (SLVMC09, SBOM958, SNVMBG5) and the extracted model hashes for the OPA1622 and LMR33630C | board, parts-lib | todo |
| E4.10 | Power and dissipation data per part and per use case, exported for the out-of-scope thermal work and the battery-life model | board | todo |

### E5. Modules and integration levels

Specialised modules per domain, with the integration verified level by level and never inferred from the parts: the T1's [electronics integration and verification](../specs/electronics-integration-verification.md) (modules, interaction matrix, evaluation matrices), applying the shared [method](https://github.com/opendap-project/toolkit-hardware/blob/main/docs/specs/integration-and-verification.md) (levels L0–L6, module rules, row format, gates; system plan HW3.4).

| ID | Task | Repository | Status |
|---|---|---|---|
| E5.0 | T1 modules, interaction matrix and evaluation matrices (the generic method moved to system plan HW3.4) | t1 | doing: Draft v0.1 |
| E5.1 | M-SI coupons: main-board stackup and both flat cables; with M-HAT, the hat-standard ADR 0004 coupon | board, hat-standard | blocked (E2.1, E3.6 stackup) |
| E5.2 | M-AUD: audio, clocks, rails and jacks (BOM §7.1, §7.29–§7.31) | board | blocked (E1.6, E6.1 audio sheets) |
| E5.3 | M-PWR: charging, power path, converters, eFuses | board | blocked (E1.4, E1.5, E6.1 power sheets) |
| E5.4 | M-CORE: SoC, LPDDR4X, eMMC, PMIC, boot; the hub of the module stack and the first SoC power measurement | board | blocked (E1.1, E1.11, E6.1 core sheets) |
| E5.5 | M-RF, M-UI, M-IO, M-HAT | board, faceplate-touch, hat-standard | blocked (E1.11, E1.2, E1.3) |
| E5.6 | Module connector maps as interface files checked against the main-board netlist (rule M3) | board | blocked (E4.6, E6.1) |

### E6. Schematics

| ID | Task | Repository | Status |
|---|---|---|---|
| E6.1 | Main board schematic, per subsystem, with ERC and a review against each datasheet | board | blocked (E1, E4.1) |
| E6.2 | Touch-faceplate schematic | faceplate-touch | blocked (E1.2, E2.3) |
| E6.3 | HAT device-side piece schematic and flat cable | hat-standard, board | blocked (E1.3, E2.2) |

### E7. Layout

| ID | Task | Repository | Status |
|---|---|---|---|
| E7.1 | Placement against the outline, keep-outs and height zones | board | blocked (E6.1, board outline) |
| E7.2 | Routing: LPDDR4X, eMMC, SD, USB 3 and PCIe to the DF40, DSI, audio partition | board | blocked (E7.1) |
| E7.3 | SI and power-integrity sign-off against E2 | board | blocked (E7.2) |

### E9. Benchmarks and compliance

| ID | Task | Repository | Status |
|---|---|---|---|
| E9.1 | Expert DFM, mechanical and reliability rules in the product DRC (holes, edges, assembly, routing hygiene, voltage spacing, thermal placement) | board | doing |
| E9.2 | Moved: system plan HW3.1 (PCB craft standard, rule set and check script) | kicad-lib, toolkit-hardware | doing |
| E9.3 | Moved: system plan HW3.2 (industry wisdom benchmark, design-review checklist); the T1 gaps note (board `427d860`) is the T1 application | toolkit-hardware, board | doing |
| E9.4 | Design for compliance of the T1: regimes, conflicts and the path through Q1–Q4 (the reusable regime catalog moved to system plan HW3.6) | t1 | done: research (t1 `3c19e1c`); T1-071 to T1-086 (t1 `62f5cfc`), SYS-012 (system `1b916a2`), board constraints v0.5 (board `2dc4c59`), verification rows (t1 `9f9469d`) |
| E9.5 | Moved: system plan HW3.3 (quality-level tagging, RFC 0010) | all | todo |

### E10. Feature blocks and variants

Every function is a feature block that can be enabled, disabled, swapped or measured alone; prototype 1 is the full variant, and every other variant is derived from it as data ([RFC 0011](https://github.com/opendap-project/rfcs/blob/main/text/0011-feature-blocks-and-variants.md)).

| ID | Task | Repository | Status |
|---|---|---|---|
| E10.1 | T1 block catalog: interface, power domain and switch, absent state, measurement point, alternates, software description per block; gaps for the schematic | board | done: 49 blocks, gaps G1–G21 (board `8ead864`) |
| E10.2 | Variant files: `full` (prototype 1), `production`, `lite` (proposal) | board | done: `full`, `production`, `lite` (proposal) (board `8ead864`) |
| E10.3 | Moved: system plan HW2.8 (`odapkit variant`) | toolkit-hardware, toolkit-hooks | todo |
| E10.4 | Moved: system plan HW2.9 (virtual variants) | virtual-devices | todo |
| E10.5 | Verification per variant in the integration matrices | t1 | todo |

### E8. Prototype 1

| ID | Task | Repository | Status |
|---|---|---|---|
| E8.1 | Fab and assembly package, PCBWay DFM review | board | blocked (E7) |
| E8.2 | Bring-up plan: power-up sequence capture, MaskROM and eMMC timing, rail measurements, SoC draw for T1-039 and the body thickness (t1 ADR 0003) | board | todo |

## Decisions for the maintainers

Delegated to Claude on 2026-10-24: front-side height (E2.5), the requirement mismatches (resolved in E0.4), and modules before the full board (yes, with the integration levels of E5). Vendor contacts (E1.7, E1.10, E3.6) wait until the engineering work needs them. Still open:

1. HCS-012: whether the HAT device-side piece is fitted in every T1 (board ADR 0013).
