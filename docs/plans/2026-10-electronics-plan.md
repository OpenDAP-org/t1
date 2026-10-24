# T1 electronics work plan

- **Date:** 2026-10-24
- **Status:** Active. Tasks change status as work lands; the plan itself changes only with the maintainers.
- **Scope:** the T1's electronics: main board, touch-faceplate electronics, HAT device-side piece and the flat cables (board, faceplate-touch, hat-standard), with their libraries (kicad-lib, parts-lib) and tools (toolkit-hardware, toolkit-hooks).
- **Out of scope:** the case, mechanical CAD and physical simulations (thermal, structural, acoustic), the other faceplates and the HATs themselves. The electronics still own what those need from them, as data: board outline and keep-outs taken in, height zones per side, board STEP and per-part dissipation handed out (E4.9, E4.10).

## Where the work starts

The electronics exist as documents: board BOM v1.7, power tree v0, a v0 pin plan, 15 board ADRs, HAT interface v1 with 4 ADRs, 128 parts-lib spec sheets and three ngspice studies (board `sim/`). There is no KiCad project, no symbol or footprint, no fab rule set for HDI or flex and no `component.yaml` yet. The BOM lists items it must close before schematic capture (§7.22, §7.28) and before layout (§7.1, §7.29–§7.31).

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
| E0.1 | Board: bring BOM §7 and §9, the power tree, ADR 0008/0009/0010 notes, the pin plan header and the 2S study up to ADR 0014 and ADR 0015 (unused TPS65131 inverter, BQ25798 driver state, DF40 power pin count, FH35C-51S, stale line references) | board | todo |
| E0.2 | Product, case and touch notes: body 59.5 × 94.3 mm (t1 ADR 0003) where 91.5 mm remains; battery basis in t1 ADR 0003 after board ADR 0014; stale "Draft" notes in case and faceplate-touch requirements | t1, case, faceplate-touch | todo |
| E0.3 | HAT standard: one contact field everywhere (4 × 25 at 2.54 mm, Harwin P70, Mill-Max 0965; hat-standard ADR 0004) in interface v1, the requirements and board BOM line 67 | hat-standard, board | todo |
| E0.4 | List what needs a maintainers' call: requirements still Draft behind Accepted ADRs (BRD-020, BRD-022, BRD-024, BRD-037, FPT-008), BRD-001 against the kernel configuration in BOM v1.7, the LP5521 drive direction against the faceplate interface | all | todo |

### E1. Freeze the inputs to the schematic

| ID | Task | Repository | Status |
|---|---|---|---|
| E1.1 | Pinout v1: check every pin of the v0 plan against the full RK3566 datasheet (IO domains, muxing, boot strapping) | board | todo |
| E1.2 | Faceplate contact positions on the FH35C-51S from pinout v1 (RFC 0001 group order) | t1, board | blocked (E1.1) |
| E1.3 | HAT DF40 100-pin and 50-pin contact numbering, pull-up ownership and power classes (interface v1 §10) | hat-standard | blocked (E1.1) |
| E1.4 | BOM §7.22: BQ28Z610-R1 gauge-only configuration from TI's reference manual | board | todo |
| E1.5 | BOM §7.28: effective output capacitance of the 5.1 V TPS62903 (≥ 12 µF after DC bias) | board | todo |
| E1.6 | BOM §7.1: SPICE of the master-clock attenuator into the CS43198 clock input | board | todo |
| E1.7 | RK809-5 power-up defaults and RK860-0 data: ask Rockchip, else follow the ROCK 3C reference and plan a power-up capture | board | decision (contacting Rockchip) |
| E1.8 | Flat-cable specification board ↔ HAT device-side piece (BRD-013): stack-up, impedance, length, bend, stiffeners | board | blocked (E2, E1.3) |
| E1.9 | Faceplate flex specification: FH35C tail (0.2 ± 0.03 mm, 15.6 ± 0.05 mm), DSI pairs, stiffener | faceplate-touch | blocked (E1.2) |
| E1.10 | Battery pack quotes from two makers (ADR 0014 item 2) | board | decision (outward contact) |

### E2. PCB design constraints

One specification per board, deriving every limit from the product requirements, the parts and the fab. Each constraint has an ID, a target, a hard limit, its source, and the rule or check that enforces it. Values without a source are marked **UNVERIFIED**.

| ID | Task | Repository | Status |
|---|---|---|---|
| E2.1 | Main board constraints: outline and keep-outs taken from the product, height zones per side, stackup and build, impedance classes, length matching, current capacity, audio partition and grounding, clocks, RF keep-out, power integrity, ESD and protection, test access, assembly and DFM, variants | board | todo |
| E2.2 | Flat-cable and HAT device-side piece constraints | board, hat-standard | blocked (E1.3) |
| E2.3 | Touch-faceplate electronics constraints | faceplate-touch | blocked (E1.2) |
| E2.4 | Machine-readable form of the constraints (one data file per board) so rule sets, net classes and checks are generated from it, not copied | board | todo |

### E3. Design rules (DRC)

The fab rule sets live in kicad-lib (KLIB-002), one per PCBWay process. The product rules live with each board and are composed on top of the fab set, because a KiCad project reads one `.kicad_dru` file. Every rule set has a test board that triggers every rule, checked with `kicad-cli pcb drc`.

| ID | Task | Repository | Status |
|---|---|---|---|
| E3.1 | `pcbway-hdi` fab rule set and test board, from the published capabilities (research 2026-10 PCBWay) | kicad-lib | todo |
| E3.2 | `pcbway-flex` fab rule set and test board | kicad-lib | todo |
| E3.3 | `pcbway-multilayer` fab rule set and test board (device-side piece, rigid faceplate boards) | kicad-lib | todo |
| E3.4 | T1 main-board product rules (from E2.1) and test board | board | blocked (E2.1) |
| E3.5 | Composition tool and check: project `.kicad_dru` = fab set + product rules, with a stale-file check in pre-commit | toolkit-hardware, toolkit-hooks | todo |
| E3.6 | Ask PCBWay for the HDI and flex stackups and the unpublished limits (microvia land, flex with stiffener tolerance, pogo pin placement) | kicad-lib | decision (outward contact) |

### E4. Engineering cycle tooling (electronics)

Each design iteration runs the same local loop: constraints → ERC/DRC → simulation regression → checks across interfaces → exports for integration → fab package.

| ID | Task | Repository | Status |
|---|---|---|---|
| E4.1 | Board KiCad project skeleton: kicad-lib as a pinned submodule, stackup, net classes and rules generated from E2.4, hierarchical sheets per subsystem | board | blocked (E2.1, E3.4) |
| E4.2 | Symbols, footprints and 3D models for the critical parts first: RK3566, RK809-5, LPDDR4X, eMMC, CS43198, OPA1622, RP2354A, DF40, FH35C, USB4105, jacks, Micro-Fit, P70 | kicad-lib | todo |
| E4.3 | Availability field in the parts-lib spec-sheet template (T1-010, T1-029), with date and source | parts-lib | todo |
| E4.4 | Simulation regression: one runner for the ngspice studies with pass/fail thresholds taken from the constraints and ADRs, vendor models fetched by script, never committed | board, toolkit-hardware | todo |
| E4.5 | Signal-integrity loop: stackup impedance calculation, length and skew reports from the layout, checked against E2 | toolkit-hardware | todo |
| E4.6 | Machine-readable interface files (faceplate, HAT, battery connector, board ↔ case envelope) with checkers that both sides match | t1, hat-standard, board | todo |
| E4.7 | `component.yaml` schema of the component export contract | toolkit-hardware | todo |
| E4.8 | Fab pipeline dry run: run `fab-package` on a rule-set test board end to end | toolkit-hardware | blocked (E3.1) |
| E4.9 | Board export for integration: STEP and glTF per the export contract | board | blocked (E4.1) |
| E4.10 | Power and dissipation data per part and per use case, exported for the out-of-scope thermal work and the battery-life model | board | todo |

### E5. Risk-reduction boards

| ID | Task | Repository | Status |
|---|---|---|---|
| E5.1 | Signal-integrity coupon: DF40 at 5 Gb/s and the pogo field (required by hat-standard ADR 0004 before the pad map freezes) | hat-standard | decision (build it) |
| E5.2 | Power and audio test board: TPS65131 boost at ≈ 99 %, LMR33630C idle and heat, LDO current sharing, VSYS_LV ripple into the DAC (BOM §7.29–§7.31) | board | decision (build it) |

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

### E8. Prototype 1

| ID | Task | Repository | Status |
|---|---|---|---|
| E8.1 | Fab and assembly package, PCBWay DFM review | board | blocked (E7) |
| E8.2 | Bring-up plan: power-up sequence capture, MaskROM and eMMC timing, rail measurements, SoC draw for T1-039 and the body thickness (t1 ADR 0003) | board | todo |

## Decisions for the maintainers

1. Contacting Rockchip (E1.7), the pack makers (E1.10) and PCBWay (E3.6).
2. Whether to build the risk-reduction boards (E5) before the full board.
3. The requirement and ADR mismatches listed by E0.4.
4. HCS-012: whether the HAT device-side piece is fitted in every T1 (board ADR 0013).
