# T1 electronics work plan

- **Date:** 2026-10-24
- **Status:** Active. Tasks change status as work lands; the plan itself changes only with the maintainers.
- **Scope:** the T1's electronics: main board, touch-faceplate electronics, HAT device-side piece and the flat cables (board, faceplate-touch, hat-standard), with their libraries (kicad-lib, parts-lib) and tools (toolkit-hardware, toolkit-hooks).
- **Out of scope:** the case, mechanical CAD and physical simulations (thermal, structural, acoustic), the other faceplates and the HATs themselves. The electronics still own what those need from them, as data: board outline and keep-outs taken in, height zones per side, board STEP and per-part dissipation handed out (E4.9, E4.10).

## Where the work starts

The electronics exist as documents: board BOM v1.7, power tree v0, a v0 pin plan, 15 board ADRs, HAT interface v1 with 4 ADRs, 128 parts-lib spec sheets and three ngspice studies (board `sim/`). There is no KiCad project, no symbol or footprint, no fab rule set for HDI or flex and no `component.yaml` yet. The BOM lists items it must close before schematic capture (§7.22, §7.28) and before layout (§7.1, §7.29–§7.31).

## Quality levels

Every rule, check, verification row and certification item carries the level at which it becomes mandatory, from Q0 (bench-safe) to Q4 (reference), per [RFC 0010](https://github.com/opendap-project/rfcs/blob/main/text/0010-quality-levels.md). A build passes everything at its target level; what lies above is visible debt, not a blocker, and a level reached is never lost. Targets: risk-reduction modules and prototype 1 at **Q1**; pilot units and the DIY kit at **Q2**; the Commercial edition at **Q3**; every revision moves toward **Q4**.

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
| E1.4 | BOM §7.22: BQ28Z610-R1 gauge-only configuration from TI's reference manual | board | todo |
| E1.5 | BOM §7.28: effective output capacitance of the 5.1 V TPS62903 (≥ 12 µF after DC bias) | board | todo |
| E1.6 | BOM §7.1: SPICE of the master-clock attenuator into the CS43198 clock input | board | todo |
| E1.7 | RK809-5 power-up defaults and RK860-0 data: ask Rockchip, else follow the ROCK 3C reference and plan a power-up capture | board | todo: measured on M-CORE (V2-C01) instead of asked; the ROCK 3C reference meanwhile |
| E1.8 | Flat-cable specification board ↔ HAT device-side piece (BRD-013): stack-up, impedance, length, bend, stiffeners | board | blocked (E2, E1.3) |
| E1.9 | Faceplate flex specification: FH35C tail (0.2 ± 0.03 mm, 15.6 ± 0.05 mm), DSI pairs, stiffener | faceplate-touch | blocked (E1.2) |
| E1.11 | Retrieve the Rockchip RK3566 hardware design guide and the Murata Type 2EL hardware design guide (constraints §9 items 2 and 3) | parts-lib | done: Rockchip (parts-lib `9be21a0`, board `7be75ba`), Murata notes (parts-lib `c65b1bb`, board `3b4e1fd`); the 2EL datasheet is still missing |
| E1.12 | Power tree: a 1.8 V power rail for the radio module (AVDD18, peaks above 1 A) and the LPDDR4X 1.10 V VDD2 at the SoC; check against power tree v0 and the BOM | board | todo |
| E1.13 | Antennas from Murata's certified lists for the target regions (the Molex 146187 is on Japan's only), and the SAR / permissive-change path for handheld use | board, t1 | todo |
| E1.10 | Battery pack quotes from two makers (ADR 0014 item 2) | board | deferred: vendor contact after the engineering work (maintainers, 2026-10-24) |

### E2. PCB design constraints

One specification per board, deriving every limit from the product requirements, the parts and the fab. Each constraint has an ID, a target, a hard limit, its source, and the rule or check that enforces it. Values without a source are marked **UNVERIFIED**.

| ID | Task | Repository | Status |
|---|---|---|---|
| E2.1 | Main board constraints: outline and keep-outs taken from the product, height zones per side, stackup and build, impedance classes, length matching, current capacity, audio partition and grounding, clocks, RF keep-out, power integrity, ESD and protection, test access, assembly and DFM, variants | board | doing: Draft v0.1 (board `6b0d1bc`, BRD-060); open items in its §9 |
| E2.2 | Flat-cable and HAT device-side piece constraints | board, hat-standard | blocked (E1.3) |
| E2.3 | Touch-faceplate electronics constraints | faceplate-touch | blocked (E1.2) |
| E2.5 | Front-side height under the display (constraints PCB-Z-03, §9 item 1): grow the gap, single-sided under the display, or a recess | board, t1 | doing: decision delegated to Claude (2026-10-24) |
| E2.4 | Machine-readable form of the constraints (one data file per board) so rule sets, net classes and checks are generated from it, not copied | board | done: `hardware/rules/netclasses.toml` (board `46c3a66`); project generation in E4.1 |

### E3. Design rules (DRC)

The fab rule sets live in kicad-lib (KLIB-002), one per PCBWay process. The product rules live with each board and are composed on top of the fab set, because a KiCad project reads one `.kicad_dru` file. Every rule set has a test board that triggers every rule, checked with `kicad-cli pcb drc`.

| ID | Task | Repository | Status |
|---|---|---|---|
| E3.1 | `pcbway-hdi` fab rule set and test board, from the published capabilities (research 2026-10 PCBWay) | kicad-lib | done (kicad-lib `bec830a`, 16/16 rules fire) |
| E3.2 | `pcbway-flex` fab rule set and test board | kicad-lib | done (kicad-lib `bec830a`, 6/6) |
| E3.3 | `pcbway-multilayer` fab rule set and test board (device-side piece, rigid faceplate boards) | kicad-lib | done (kicad-lib `bec830a`, 13/13 at 18 µm and at 35 µm) |
| E3.4 | T1 main-board product rules (from E2.1) and test board | board | done: v0.1, 21 rules, 21/21 fire (board `46c3a66`) |
| E3.5 | Composition tool and check: project `.kicad_dru` = fab set + product rules, with a stale-file check in pre-commit | toolkit-hardware, toolkit-hooks | done: `odapkit dru`, hook `kicad-dru-composed` (toolkit-hooks `f5edd58`, `e540f4e`) |
| E3.6 | Ask PCBWay for the HDI and flex stackups and the unpublished limits (microvia land, flex with stiffener tolerance, pogo pin placement) | kicad-lib | deferred: published capabilities used meanwhile (maintainers, 2026-10-24) |

### E4. Engineering cycle tooling (electronics)

Each design iteration runs the same local loop: constraints → ERC/DRC → simulation regression → checks across interfaces → exports for integration → fab package.

| ID | Task | Repository | Status |
|---|---|---|---|
| E4.1 | Board KiCad project skeleton: kicad-lib as a pinned submodule, stackup, net classes and rules generated from E2.4, hierarchical sheets per subsystem | board | blocked (E2.1, E3.4) |
| E4.2 | Symbols, footprints and 3D models for the critical parts first: RK3566, RK809-5, LPDDR4X, eMMC, CS43198, OPA1622, RP2354A, DF40, FH35C, USB4105, jacks, Micro-Fit, P70 | kicad-lib | todo |
| E4.3 | Availability field in the parts-lib spec-sheet template (T1-010, T1-029), with date and source | parts-lib | done: PLIB-003 (parts-lib `a3afb78`); existing sheets migrate when edited |
| E4.4 | Simulation regression: one runner for the ngspice studies with pass/fail thresholds taken from the constraints and ADRs, vendor models fetched by script, never committed | board, toolkit-hardware | done: `odapkit sim`; board checks 15 pass, 1 blocked on the LMR33630C model (board `ddaf1d5`) |
| E4.5 | Signal-integrity loop: stackup impedance calculation, length and skew reports from the layout, checked against E2 | toolkit-hardware | todo |
| E4.6 | Machine-readable interface files (faceplate, HAT, battery connector, board ↔ case envelope) with checkers that both sides match | t1, hat-standard, board | todo |
| E4.7 | `component.yaml` schema of the component export contract | toolkit-hardware | done: schema and validator (toolkit-hardware `6860055`); open points in the contract |
| E4.8 | Fab pipeline dry run: run `fab-package` on a rule-set test board end to end | toolkit-hardware | blocked (E3.1) |
| E4.9 | Board export for integration: STEP and glTF per the export contract | board | blocked (E4.1) |
| E4.11 | Tag toolkit-hooks v0.4.0 and move the hardware repos to it, so `kicad-dru-composed` runs on commit | toolkit-hooks | todo |
| E4.12 | Sim inputs: `--json` output for `vsys-bucks/run.py`, parts-lib SPICE model entries (SLVMC09, SBOM958, SNVMBG5) and the extracted model hashes for the OPA1622 and LMR33630C | board, parts-lib | todo |
| E4.10 | Power and dissipation data per part and per use case, exported for the out-of-scope thermal work and the battery-life model | board | todo |

### E5. Modules and integration levels

Specialised modules per domain, with the integration verified level by level and never inferred from the parts: [electronics integration and verification](../specs/electronics-integration-verification.md) (levels L0–L6, modules, interaction matrix, evaluation matrices).

| ID | Task | Repository | Status |
|---|---|---|---|
| E5.0 | Integration levels, modules, interaction matrix and evaluation matrices | t1 | doing: Draft v0.1 |
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
| E9.2 | OpenDAP PCB craft standard with its rule set and check script, shared by every board | kicad-lib, toolkit-hardware | doing |
| E9.3 | Industry wisdom benchmark and the phase-staged design-review checklist | toolkit-hardware, board | doing |
| E9.4 | Design for compliance: regimes, conflicts and the path through Q1–Q4 | t1 | done: research (t1 `3c19e1c`); T1-071 to T1-086 (t1 `62f5cfc`), SYS-012 (system `1b916a2`), board constraints v0.5 (board `2dc4c59`), verification rows (t1 `9f9469d`) |
| E9.5 | Tag every existing item with its quality level as each document is next edited (RFC 0010) | all | todo |

### E10. Feature blocks and variants

Every function is a feature block that can be enabled, disabled, swapped or measured alone; prototype 1 is the full variant, and every other variant is derived from it as data ([RFC 0011](https://github.com/opendap-project/rfcs/blob/main/text/0011-feature-blocks-and-variants.md)).

| ID | Task | Repository | Status |
|---|---|---|---|
| E10.1 | T1 block catalog: interface, power domain and switch, absent state, measurement point, alternates, software description per block; gaps for the schematic | board | doing |
| E10.2 | Variant files: `full` (prototype 1), `production`, `lite` (proposal) | board | doing |
| E10.3 | `odapkit variant`: per-variant BOM, pick-and-place, devicetree overlays and device description; absent-state check and its hook | toolkit-hardware, toolkit-hooks | todo |
| E10.4 | Virtual variants from the same files | virtual-devices | todo |
| E10.5 | Verification per variant in the integration matrices | t1 | todo |

### E8. Prototype 1

| ID | Task | Repository | Status |
|---|---|---|---|
| E8.1 | Fab and assembly package, PCBWay DFM review | board | blocked (E7) |
| E8.2 | Bring-up plan: power-up sequence capture, MaskROM and eMMC timing, rail measurements, SoC draw for T1-039 and the body thickness (t1 ADR 0003) | board | todo |

## Decisions for the maintainers

Delegated to Claude on 2026-10-24: front-side height (E2.5), the requirement mismatches (resolved in E0.4), and modules before the full board (yes, with the integration levels of E5). Vendor contacts (E1.7, E1.10, E3.6) wait until the engineering work needs them. Still open:

1. HCS-012: whether the HAT device-side piece is fitted in every T1 (board ADR 0013).
