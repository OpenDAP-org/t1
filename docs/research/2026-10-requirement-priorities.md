# Audit of requirement priorities

- **Date:** 2026-10-22
- **Status:** Proposal. No requirements file was edited. Every change to a **Confirmed** requirement is a proposal for the maintainers, because confirmed priorities change only with their decision. Changes to **Draft** requirements are proposed more freely.
- **Scope:** every requirement with status Confirmed or Draft in the public requirements files: `t1`, `board`, `case`, `faceplate-touch`, `faceplate-cassette`, `faceplate-classic`, `hat-standard`, the seven product HATs, `system`, `blackbird-os`, `desktop`, `blackbird-ui`, the four libraries and the four toolkit repositories (`toolkit/host` has no requirements yet; `rfcs` has none). Requirements with status Moved or Withdrawn are listed per repository but not audited. Private repositories are out of scope.
- **Inputs:** the workspace rule on priorities, the T1 product rules ([T1 requirements](../requirements/README.md), T1-040 to T1-048), the [UX research](ux/) (persona, jobs to be done and moments of truth in [`2026-10-persona-jtbd-journey.md`](ux/2026-10-persona-jtbd-journey.md) §6, and the [candidate requirements](ux/2026-10-candidate-requirements.md)), and the [experience budgets](../specs/experience-budgets.md).

## Definition applied

- **Must:** the T1 fails its promise without it. It blocks a release and is never traded away.
- **Should:** nice to have. Delivered when schedule and the hardware allow, and the first candidate to drop or, for hardware, to become a population option under the board's assembly variants (BRD-053).

The T1's promise, as the requirements state it: a mid-range DAP whose audiophile experience is kept at 100 % (T1-040), with safe power first (BRD-049), 100 % of the hardware and no software limits (T1-045), no degradation with scale or long sessions (T1-046, T1-047), its own software ecosystem (SYS-035), fully open and running on Linux (T1-001), and extensible with faceplates and HATs (T1-002, T1-003).

Each requirement was put through three questions:

1. Does the T1 fail that promise, or lose one of the 15 moments of truth (MoT), without it?
2. If not, is it a constraint that cannot be retrofitted once hardware or a public interface ships (an interface, an architecture rule, a sourcing or licensing rule)? Such constraints stay **must**, because getting them wrong later breaks the product or third parties.
3. Otherwise it is a **should**. A hardware should says whether it becomes a **population option** (designed in, fitted on prototypes, left unpopulated or fitted in some production variants) or is **dropped**.

Requirements that only block one component (a HAT, an optional faceplate) were judged against that component's own promise, not against the T1 release. Rules about how the project works (tooling, libraries, licensing) were kept as must when they are rules, and flagged when their wording is only a permission.

## Summary

| Repository | Audited | Keep | Must → should | Should → must |
|---|---|---|---|---|
| `t1` | 46 | 45 | 1 | 0 |
| `board` | 52 | 49 | 3 | 0 |
| `case` | 6 | 5 | 1 | 0 |
| `faceplate-touch` | 21 | 19 | 1 | 1 |
| `faceplate-cassette` | 9 | 9 | 0 | 0 |
| `faceplate-classic` | 9 | 9 | 0 | 0 |
| `hat-standard` | 20 | 19 | 0 | 1 |
| Product HATs (7) | 36 | 36 | 0 | 0 |
| `system` | 31 | 25 | 6 | 0 |
| `blackbird-os` | 27 | 25 | 1 | 1 |
| `desktop` | 8 | 5 | 3 | 0 |
| `blackbird-ui` | 3 | 3 | 0 | 0 |
| Libraries (4) | 7 | 6 | 1 | 0 |
| `toolkit` (cli, hardware, verify) | 17 | 15 | 2 | 0 |
| **Total** | **292** | **270** | **19** | **3** |

Of the 22 proposed changes, 15 touch Confirmed requirements (proposals for the maintainers) and 7 touch Draft ones. The set of musts is mostly right: the product rules, the audio path, power safety, recovery and every interface that third parties will build against are must, and should stay so.

## Most consequential proposals

| # | Requirement | Status | Proposal | Impact |
|---|---|---|---|---|
| 1 | SYS-028, with SYS-026 and SYS-029 | Confirmed | must → should | Shipping five languages, composing input methods for Chinese, Japanese and Korean and progressive language loading at the first release is the largest software scope item that no moment of truth depends on. The i18n-first architecture (SYS-025, DS-006) stays must, so languages can follow without rework. |
| 2 | BRD-031 ambient-noise microphone | Confirmed | must → should, population option | No listening job or moment of truth needs a second microphone; it is a gadget input. Designed in and fitted on prototypes, optional in production variants. |
| 3 | BRD-032 IMU | Confirmed | must → should, population option | Wake on motion and gadget uses are useful, not part of the promise. Same treatment as BRD-031. |
| 4 | BRD-014 RP coprocessor | Confirmed | must → should, population option (conditional) | The SoC boots without it (BRD-019) and it may never sit in a data path (BRD-005), so the product works without it. Holds only while no must function (buttons, power sequencing, HAT detection) is assigned to it; otherwise keep must. BRD-015 and BRD-041 then apply when it is fitted. |
| 5 | HCS-022 reserved contacts | Confirmed | should → must | The HAT interface is a public standard for third parties (HCS-002) meant to outlive the board (HCS-021). Contacts cannot be added after HATs exist, as T1-024 already requires for the command set. |
| 6 | OS-007 prebuilt kernel updates | Draft | should → must | With no custom kernel (OS-002) and updates from the distribution (OS-013), a kernel that cannot be updated as a package stalls security and driver fixes; it underpins the first update (MoT 11). |
| 7 | T1-027 newest part generation | Confirmed | must → should | The experience does not depend on the newest radio or sensor generation. Mainline support and availability (T1-028, T1-029) already decide; the newest generation becomes the tie-breaker instead of a release blocker. |
| 8 | DESK-006 to DESK-008 background service | Draft | must → should | Sync and downloads while the app is closed are convenient; the desktop app and the CLI (DESK-001, DESK-002, TK-002) already cover every job. |
| 9 | OS-023 own optimised package builds | Confirmed | must → should | Carrying OpenDAP builds of distribution packages is an optimisation path, and it pulls against the most-standard rule (T1-011). Keep it where a budget needs it, not as a release condition. |
| 10 | FPT-021 touch technology | Confirmed | split: core must, extras should | Projected-capacitive multi-touch with palm rejection is the must (MoT 7, scrolling); "best available technology" and glove or wet operation become should, dropped when no available panel offers them. |

## Consistency findings

These are not priority changes, but they affect how priorities read:

- **Duplicate IDs (fixed 2026-10-22: the verify pair is now TK-016, TK-017):** `toolkit/hardware` and `toolkit/verify` both defined TK-014 and TK-015. The T1 requirements and the experience budgets cite TK-014 and TK-015 as the scale and endurance tests (`toolkit/verify`). The `toolkit/hardware` pair needs new IDs.
- **Permissions labelled must:** TK-014 and TK-015 in `toolkit/hardware` say "may"; a permission cannot block a release (proposed should below, or recorded as context).
- **Mixed wording:** BRD-036 ("should be accessible … at least the Action button shall"), FPT-018 ("should have a mainline driver", priority must) and DESK-005 ("shall run on UNIX … should also support Windows") each mix a must and a should in one row; splitting them makes the priority exact.
- **Statements rather than requirements:** FPC-007, FPC-008, FPK-007 and FPK-008 ("will most likely") describe expectations; they read better as context until the maintainers decide.
- **Overlaps:** FPT-010 (should, Draft) repeats the ambient light sensor and microphone that FPT-014, FPT-017, FPT-023 and FPT-024 require as must; BRD-016 is met by BRD-035.

## Moments of truth without a must today

The audit found no must that covers these moments of truth directly. Each has a candidate in the [candidate requirements](ux/2026-10-candidate-requirements.md), and the decision on them is for the maintainers:

| MoT | Moment | Covered today by | Candidate |
|---|---|---|---|
| 1 | Pop-free transitions | experience budgets (Draft values) | UX-010 |
| 3 | Volume step granularity | experience budgets (Draft values) | UX-016, UX-017 |
| 6 | Resume after screen-off or pause | button latency budget only | UX-005, UX-013 |
| 8 | First library import in the background | T1-046 in general terms | UX-030 |
| 10 | Unplug while playing | BRD-052 (detection hardware only) | UX-011 |
| 12 | USB DAC mode | BRD-034 (port capability only) | UX-043 |
| 15 | Battery after two years | T1-012 (standard cell, not user replacement) | UX-058 |

## Per repository

Abbreviations: **C** Confirmed, **D** Draft.

### `t1`

Not audited: T1-013 (Withdrawn), T1-034 (Moved to FPT-020).

| ID | Now | Status | Proposal | Reason |
|---|---|---|---|---|
| T1-001 | must | C | keep | Defines the product: open, open hardware, 100 % Linux. |
| T1-002 | must | C | keep | The gadget and HAT ecosystem is part of the promise. |
| T1-003 | must | C | keep | Interchangeable faceplates are product identity; the interface cannot be retrofitted. |
| T1-004 | must | C | keep | One design for both editions is a promise. |
| T1-005 | must | C | keep | Defines the Commercial edition. |
| T1-006 | must | C | keep | Defines the DIY edition, the tinkerer end of the primary persona. |
| T1-007 | must | C | keep | Without integration from exports the whole product cannot be verified (T1-026). |
| T1-008 | must | C | keep | Audiophile experience at 100 %. |
| T1-009 | must | C | keep | Autonomy is a core job of the commuter persona. |
| T1-010 | must | C | keep | A part without availability stops production. |
| T1-011 | must | C | keep | Standard, trick-less software is what keeps updates safe (MoT 11). |
| T1-012 | must | C | keep | A standard cell is the answer to battery ageing (MoT 15). |
| T1-014 | must | C | keep | Fixed button layout; control by feel. |
| T1-015 | must | C | keep | Fixed button layout. |
| T1-016 | must | C | keep | Both jacks and the port layout are fixed by the promise. |
| T1-017 | should | D | keep | No ingress rating is promised. A case design target if pursued, not a board option; otherwise dropped. |
| T1-018 | must | C | keep | The interface every faceplate depends on. |
| T1-019 | must | C | keep | Design assumption that allows screws and a flat cable. |
| T1-020 | should | C | keep | Preference for the identification method; that identification works is part of the interface. |
| T1-021 | must | D | keep | Shared mechanical convention; parts do not fit without it. |
| T1-022 | must | C | keep | The Action button is used by feel with the screen off (MoT 6). |
| T1-023 | must | C | keep | Standard commands make faceplates portable; cannot be retrofitted. |
| T1-024 | must | C | keep | Reserved space keeps the command set extensible. |
| T1-025 | must | C | keep | Primary persona: open all the way down. |
| T1-026 | must | C | keep | Proof, not adjectives, for the primary persona. |
| T1-027 | must | C | **→ should** | The experience does not depend on the newest generation; availability and mainline support decide first. |
| T1-028 | must | C | keep | Sourcing rule tied to production. |
| T1-029 | must | C | keep | Sourcing rule tied to production. |
| T1-030 | must | C | keep | The body governs; a display never resizes it. |
| T1-031 | must | C | keep | Maximum envelope; product identity. |
| T1-032 | should | C | keep | Ideal size, bounded by T1-031 and autonomy. |
| T1-033 | should | C | keep | Proportion ideal, explicitly adjustable. |
| T1-035 | must | C | keep | Battery capacity is the main autonomy lever. |
| T1-036 | must | C | keep | Board and faceplate decoupling cannot be added after the board ships. |
| T1-037 | should | C | keep | Shrink rule subordinate to T1-039. |
| T1-038 | must | C | keep | Sourcing; custom parts threaten production and repair. |
| T1-039 | must | C | keep | Autonomy red line. |
| T1-040 | must | C | keep | Product rule. |
| T1-041 | must | C | keep | Product rule; it is what makes the should category possible. |
| T1-042 | must | C | keep | Product rule. |
| T1-043 | must | C | keep | Power, restart and recovery without a screen (MoT 14). |
| T1-044 | must | C | keep | The only state feedback when the screen is off or the system has not booted (MoT 14). |
| T1-045 | must | C | keep | Product rule; MoT 13. |
| T1-046 | must | C | keep | Product rule; MoT 7. |
| T1-047 | must | C | keep | Product rule. |
| T1-048 | must | C | keep | Product rule; turns the others into release gates. |

### `board`

Not audited: BRD-021 (Withdrawn).

| ID | Now | Status | Proposal | Reason |
|---|---|---|---|---|
| BRD-001 | must | C | keep | Mainline drivers keep updates safe (MoT 11). |
| BRD-002 | must | C | keep | Mid-range positioning. |
| BRD-003 | must | C | keep | General-purpose platform (T1-002). |
| BRD-004 | must | C | keep | Recovery and 100 % of the hardware. |
| BRD-005 | must | C | keep | No helper may limit a capability (T1-045). |
| BRD-006 | must | C | keep | HAT bandwidth (HCS-004); cannot be retrofitted. |
| BRD-007 | must | C | keep | Safe power first. |
| BRD-008 | must | C | keep | Autonomy. |
| BRD-009 | must | C | keep | Autonomy and standby drain budget. |
| BRD-010 | must | C | keep | Thermal limits must exist before they affect audio. |
| BRD-011 | must | C | keep | Both outputs are part of the promise. |
| BRD-012 | must | C | keep | Keeps the speaker out of the audio path. |
| BRD-013 | must | C | keep | Ownership of a shared interface. |
| BRD-014 | must | C | **→ should, population option** | Not in the boot or data path (BRD-005, BRD-019). Conditional: stays must if any must function is assigned to it. |
| BRD-015 | must | C | keep | Applies whenever the coprocessor is fitted. |
| BRD-016 | must | C | keep | Recovery (MoT 14); met by BRD-035. |
| BRD-017 | must | C | keep | Recovery (MoT 14). |
| BRD-018 | must | C | keep | Flashing and recovery over USB. |
| BRD-019 | must | C | keep | Keeps the product working whatever the coprocessor does. |
| BRD-020 | must | D | keep | Power-tree foundation; safe power first. |
| BRD-022 | must | D | keep | The audio stage is the audiophile core. |
| BRD-023 | must | C | keep | Bit-perfect at every rate (OS-020). |
| BRD-024 | must | D | keep | Defines both outputs. |
| BRD-025 | must | C | keep | Radio certification is a release condition. |
| BRD-026 | must | D | keep | Safe power first. |
| BRD-027 | must | D | keep | Privacy guarantee for the main microphone; applies to every fitted microphone. |
| BRD-028 | should | C | keep | Design intention, not a population option. |
| BRD-029 | must | C | keep | Updates, Bluetooth audio and the gadget role. |
| BRD-030 | should | C | keep | Population option if it needs its own radio (BOM line 103 is open); dropped if only the main radio can do it and its standby draw breaks the budget. |
| BRD-031 | must | C | **→ should, population option** | No listening job needs a second microphone; gadget input. |
| BRD-032 | must | C | **→ should, population option** | Wake on motion and gadget uses are not part of the promise. |
| BRD-033 | should | C | keep | Population option if an on-board PCIe device is ever fitted. |
| BRD-034 | must | C | keep | Charging, flashing, file transfer and DAC mode (MoT 12). |
| BRD-035 | must | C | keep | Recovery without extra buttons. |
| BRD-036 | must | C | keep | Primary persona; split the wording (see findings). |
| BRD-037 | should | D | keep | Capacity is a variant choice; its floor is set by the must budgets (T1-046 to T1-048). |
| BRD-038 | must | C | keep | Collector persona: the library lives on the card. |
| BRD-039 | must | C | keep | Debugging through the one user port. |
| BRD-040 | must | C | keep | Lets users verify bit-perfect output (MoT 9). |
| BRD-041 | should | D | keep | Firmware on the coprocessor; available where it is fitted. |
| BRD-042 | must | C | keep | Standard, trick-less high-speed paths. |
| BRD-043 | must | C | keep | Recovery without a computer specialist (MoT 14). |
| BRD-044 | must | C | keep | Bus allocation; cannot be retrofitted. |
| BRD-045 | must | C | keep | HAT bandwidth; cannot be retrofitted. |
| BRD-046 | should | C | keep | Convenience; BRD-047 covers bring-up. |
| BRD-047 | must | C | keep | Bring-up of the earliest boot stages. |
| BRD-048 | must | C | keep | Faceplate decoupling (T1-036). |
| BRD-049 | must | C | keep | Safe power first. |
| BRD-050 | must | C | keep | Safe power first. |
| BRD-051 | must | C | keep | Library scale and scan time (T1-046). |
| BRD-052 | must | C | keep | Pause on unplug, never to the speaker (MoT 10). |
| BRD-053 | must | C | keep | The mechanism every hardware should relies on. |

### `case`

Not audited: CASE-001 to CASE-005, CASE-007 to CASE-012, CASE-023, CASE-026, CASE-027 (Moved to `t1`); CASE-013 to CASE-019 (Withdrawn).

| ID | Now | Status | Proposal | Reason |
|---|---|---|---|---|
| CASE-006 | must | C | keep | HAT ecosystem (T1-002). |
| CASE-020 | must | C | keep | Material constraint for the first cases. |
| CASE-021 | must | C | **→ should** | A colour and texture do not decide the promise. Not a board option: an alternative finish, or dropped. |
| CASE-022 | should | C | keep | Later material options. |
| CASE-024 | must | C | keep | Repairability and the battery swap (MoT 15). |
| CASE-025 | must | C | keep | The plain back cover swaps for a HAT case. |

### `faceplate-touch`

Not audited: FPT-007, FPT-009, FPT-011, FPT-012 (Withdrawn).

| ID | Now | Status | Proposal | Reason |
|---|---|---|---|---|
| FPT-001 | must | C | keep | Default faceplate (T1-003). |
| FPT-002 | must | C | keep | Defines the faceplate. |
| FPT-003 | must | C | keep | Carries the status LED code (T1-044). |
| FPT-004 | must | C | keep | Main microphone. |
| FPT-005 | must | C | keep | Interface compliance. |
| FPT-006 | must | C | keep | Repository scope. |
| FPT-008 | should | D | keep | Part choice; FPT-021 governs the technology. |
| FPT-010 | should | D | **→ must** | Repeats sensors that FPT-014, FPT-017, FPT-023 and FPT-024 require as must; align it or withdraw it as a duplicate. |
| FPT-013 | must | C | keep | Display selection rule. |
| FPT-014 | must | C | keep | Display and dimming. |
| FPT-015 | must | C | keep | Keeps the board display-agnostic. |
| FPT-016 | must | C | keep | Primary persona. |
| FPT-017 | must | C | keep | Main microphone position. |
| FPT-018 | must | C | keep | Mainline display support; split the wording (see findings). |
| FPT-019 | must | C | keep | Availability first. |
| FPT-020 | must | C | keep | Display fits the body (T1-030). |
| FPT-021 | must | C | **split: core must, extras → should** | Projected-capacitive multi-touch with palm rejection is the must (MoT 7); best available technology and glove or wet operation are should, dropped when no available panel has them. |
| FPT-022 | must | C | keep | The stack must fit and seal. |
| FPT-023 | must | C | keep | Fixed front layout. |
| FPT-024 | must | C | keep | Fixed front layout. |
| FPT-025 | should | C | keep | RGB is optional; T1-044 codes must stay legible on a single colour. |

### `faceplate-cassette` and `faceplate-classic`

The cassette and classic faceplates are not the default; their musts block their own release, not the T1's.

| ID | Now | Status | Proposal | Reason |
|---|---|---|---|---|
| FPC-001 | must | C | keep | Defines the faceplate. |
| FPC-002 | must | C | keep | Interface compliance. |
| FPC-003 | must | C | keep | Repository scope. |
| FPC-004 | must | C | keep | Every faceplate has a display. |
| FPC-005 | must | C | keep | Keeps the board display-agnostic. |
| FPC-006 | must | C | keep | Primary persona. |
| FPC-007 | should | C | keep | Expectation, not a requirement (see findings). |
| FPC-008 | should | C | keep | Expectation, not a requirement (see findings). |
| FPC-009 | must | C | keep | Standard commands (T1-023). |
| FPK-001 | must | C | keep | Defines the faceplate. |
| FPK-002 | must | C | keep | Interface compliance. |
| FPK-003 | must | C | keep | Repository scope. |
| FPK-004 | must | C | keep | Every faceplate has a display. |
| FPK-005 | must | C | keep | Keeps the board display-agnostic. |
| FPK-006 | must | C | keep | Primary persona. |
| FPK-007 | should | C | keep | Expectation, not a requirement (see findings). |
| FPK-008 | should | C | keep | Expectation, not a requirement (see findings). |
| FPK-009 | must | C | keep | Standard commands (T1-023). |

### `hat-standard`

Not audited: HCS-016, HCS-018 (Withdrawn).

| ID | Now | Status | Proposal | Reason |
|---|---|---|---|---|
| HCS-001 | must | C | keep | Scope of the standard. |
| HCS-002 | must | C | keep | Third parties are the point of a standard. |
| HCS-003 | must | C | keep | Mechanical principle. |
| HCS-004 | must | C | keep | 100 % of the hardware (T1-045). |
| HCS-005 | must | C | keep | Same rule, for compromises. |
| HCS-006 | must | C | keep | One interconnect; cannot change after HATs exist. |
| HCS-007 | must | C | keep | Identification is required for safe power-up. |
| HCS-008 | must | C | keep | Open ecosystem. |
| HCS-009 | must | C | keep | Capability discovery. |
| HCS-010 | must | C | keep | Security of the ecosystem. |
| HCS-011 | must | C | keep | Ties HATs to the package mechanism (SYS-010, SYS-011). |
| HCS-012 | must | C | keep | Mechanical principle. |
| HCS-013 | must | C | keep | OS as host of every chip (OS-008). |
| HCS-014 | must | D | keep | Wear on the HAT side only; safe contact. |
| HCS-015 | must | D | keep | Safe power first. |
| HCS-017 | must | D | keep | Third parties need one fixed format. |
| HCS-019 | should | C | keep | Exception, not a requirement to fulfil. |
| HCS-020 | must | C | keep | Mechanical principle. |
| HCS-021 | must | C | keep | The standard outlives the board. |
| HCS-022 | should | C | **→ must** | Contacts cannot be added once HATs exist; same reasoning as T1-024. |

### Product HATs

Each HAT's musts block that HAT, not the T1. The four common rows (complete project, custom software on popular open source, HAT Case Standard, optional on-board AI) are the same in every HAT and were judged once.

| ID | Now | Status | Proposal | Reason |
|---|---|---|---|---|
| CAM-001, NAS-001, OSC-001, PWR-001, RAD-001, REC-001, VNA-001 | must | C | keep | Defines each HAT as a complete project. |
| CAM-002, NAS-002, OSC-002, PWR-002, RAD-002, REC-002, VNA-002 | must | C | keep | Standard open-source base for each domain. |
| CAM-003, NAS-003, OSC-003, PWR-003, RAD-003, REC-003, VNA-003 | must | C | keep | Every HAT uses the standard. |
| CAM-004, NAS-004, OSC-004, PWR-004, RAD-004, REC-004, VNA-004 | should | D | keep | Optional on-board AI. |
| CAM-005 | must | C | keep | Defines the HAT. |
| NAS-005 | must | C | keep | Defines the HAT. |
| NAS-006 | must | C | keep | Defines the HAT. |
| OSC-005 | must | C | keep | Defines the HAT. |
| PWR-005 | must | C | keep | Defines the HAT. |
| RAD-005 | must | C | keep | Defines the HAT. |
| REC-005 | must | C | keep | Defines the HAT. |
| VNA-005 | must | C | keep | Defines the HAT. |

### `system`

Not audited: SYS-013 to SYS-016 (Withdrawn).

| ID | Now | Status | Proposal | Reason |
|---|---|---|---|---|
| SYS-001 | must | D | keep | Extension model; cannot be retrofitted. |
| SYS-002 | must | D | keep | Extension model. |
| SYS-003 | must | D | keep | Architecture rule. |
| SYS-004 | must | D | keep | Architecture rule. |
| SYS-005 | must | D | keep | Signed index; ecosystem security. |
| SYS-006 | must | D | keep | Interaction model of the product. |
| SYS-007 | must | D | keep | Extension without forking (primary persona). |
| SYS-008 | must | D | **→ should** | A runtime technique, not a promise; the budgets and SYS-033 decide what the UI must do. |
| SYS-009 | must | D | keep | Consistent product UI (SYS-017). |
| SYS-010 | must | D | keep | Own ecosystem without a store. |
| SYS-011 | must | D | keep | Update and package security (MoT 11). |
| SYS-012 | must | D | keep | Compliance for personal music players; MoT 13 asks that it be visible. |
| SYS-017 | must | C | keep | Dedicated product, not a small phone (just-works persona). |
| SYS-018 | must | C | keep | HATs and capabilities bring their UI; cannot be retrofitted. |
| SYS-019 | must | C | keep | Primary persona; extension through the SDK. |
| SYS-020 | must | C | keep | Protects the scale and endurance rules. |
| SYS-021 | must | C | keep | Needed for search (MoT 7) and network setup. |
| SYS-022 | must | C | **→ should** | Third-party keyboards are an ecosystem extra; the default keyboard covers every job. |
| SYS-023 | must | C | keep | Default layout. |
| SYS-024 | must | C | **split: core must, full set → should** | Text, search, number and password variants are must; email, URL, phone, PIN and context variants are should. |
| SYS-025 | must | C | keep | i18n-first architecture cannot be retrofitted. |
| SYS-026 | must | C | **→ should** | Composing input methods follow the languages (SYS-028); the architecture stays must under SYS-025. |
| SYS-027 | must | C | keep | Default language. |
| SYS-028 | must | C | **→ should** | No moment of truth depends on five languages at the first release; SYS-025 keeps them possible. |
| SYS-029 | must | C | **→ should** | Optimisation for SYS-028; follows its priority. |
| SYS-030 | must | C | keep | One theme is must; structuring tokens for community themes now avoids rework (theming is the most common mod in the community research). |
| SYS-031 | must | C | keep | 100 % of the hardware; primary persona. |
| SYS-032 | must | C | keep | A missing permission system never blocks capabilities. |
| SYS-033 | must | C | keep | Scale rule (T1-046); MoT 7. |
| SYS-034 | must | C | keep | Endurance rule (T1-047). |
| SYS-035 | must | C | keep | Own ecosystem; scope boundary. |

### `blackbird-os`

| ID | Now | Status | Proposal | Reason |
|---|---|---|---|---|
| OS-001 | must | C | keep | Mainline kernel. |
| OS-002 | must | C | keep | No custom kernel. |
| OS-003 | must | C | keep | Standard, trick-less OS. |
| OS-004 | must | C | keep | Start-up and endurance budgets. |
| OS-005 | must | C | keep | Standard base. |
| OS-006 | must | C | keep | Standard base. |
| OS-007 | should | D | **→ must** | With OS-002 and OS-013, a kernel that cannot be updated as a package stalls fixes (MoT 11). |
| OS-008 | must | C | keep | OS as host of every chip. |
| OS-009 | must | C | keep | Development and verification (T1-026). |
| OS-010 | must | C | keep | Reproducible images. |
| OS-011 | must | C | keep | Signed releases. |
| OS-012 | must | C | keep | Recovery and DIY flashing (MoT 14). |
| OS-013 | must | C | keep | Updates. |
| OS-014 | should | D | keep | Updates can reach the device through the desktop app and the CLI; OTA is a convenience. |
| OS-015 | must | C | keep | Lean default image. |
| OS-016 | must | C | keep | Development and verification. |
| OS-017 | must | D | keep | Nothing lost, nothing broken (MoT 11, MoT 14). |
| OS-018 | must | C | keep | Verification on the virtual T1 (TK-016, TK-017). |
| OS-019 | must | C | keep | Scripted verification. |
| OS-020 | must | C | keep | Bit-perfect (MoT 9). |
| OS-021 | must | C | keep | Lean default image; start-up budget. |
| OS-022 | must | C | keep | Start-up and library budgets with card storage. |
| OS-023 | must | C | **→ should** | An optimisation path that pulls against T1-011; use it where a budget needs it. |
| OS-024 | must | C | keep | Own ecosystem by default. |
| OS-025 | must | C | keep | Owner control. |
| OS-026 | must | C | keep | Scope boundary. |
| OS-027 | must | C | keep | Scope boundary. |

### `desktop`

| ID | Now | Status | Proposal | Reason |
|---|---|---|---|---|
| DESK-001 | must | C | keep | Library and sync for the collector and just-works personas. |
| DESK-002 | must | C | keep | Everything the device can do, from the computer. |
| DESK-003 | must | D | keep | Just-works persona first. |
| DESK-004 | must | D | keep | One core; a feature exists once. |
| DESK-005 | must | C | keep | POSIX first; the Windows clause is already should (see findings). |
| DESK-006 | must | D | **→ should** | Work while the app is closed is convenient; the app and the CLI cover every job. |
| DESK-007 | must | D | **→ should** | Follows DESK-006. |
| DESK-008 | must | D | **→ should** | Follows DESK-006. |

### `blackbird-ui`

Not audited: DS-002 to DS-005 (Withdrawn).

| ID | Now | Status | Proposal | Reason |
|---|---|---|---|---|
| DS-001 | must | C | keep | Consistent product UI. |
| DS-006 | must | C | keep | i18n-first components (SYS-025). |
| DS-007 | must | C | keep | Same reasoning as SYS-030. |

### Libraries

| ID | Now | Status | Proposal | Reason |
|---|---|---|---|---|
| BLIB-001 | must | C | keep | Project rule. |
| BLIB-002 | must | C | keep | Licensing rule. |
| FLIB-001 | must | C | keep | Project rule. |
| KLIB-001 | must | C | keep | Project rule. |
| KLIB-002 | must | D | **→ should** | Test boards for every rule strengthen the checks; the fab's own rules and DRC already gate a release. |
| PLIB-001 | must | C | keep | Every part validated before adoption (T1-029). |
| PLIB-002 | must | C | keep | Licensing rule. |

### `toolkit`

`toolkit/host` has no requirements.

| ID | Now | Status | Proposal | Reason |
|---|---|---|---|---|
| TK-001 (cli) | must | C | keep | Scope boundary. |
| TK-002 (cli) | must | C | keep | Primary persona (T1-025). |
| TK-003 (cli) | must | C | keep | Naming rule. |
| TK-004 (cli) | must | C | keep | Distribution rule. |
| TK-005 (cli) | must | C | keep | Installation path. |
| TK-006 (cli) | must | C | keep | Reproducible installs. |
| TK-007 (cli) | must | C | keep | Non-interactive use for agents (TK-009). |
| TK-008 (cli) | must | C | keep | Self-diagnosis. |
| TK-009 (cli) | must | C | keep | One domain model. |
| TK-010 (cli) | must | C | keep | Supported hosts. |
| TK-011 (cli) | must | C | keep | Supported hosts. |
| TK-014 (hardware) | must | C | **→ should** | A permission ("may"); duplicate ID (see findings). |
| TK-015 (hardware) | must | C | **→ should** | A permission ("may"); duplicate ID (see findings). |
| TK-012 (verify) | must | C | keep | Verification without lab instruments. |
| TK-013 (verify) | must | C | keep | Approved instrumentation. |
| TK-016 (verify, was TK-014) | must | C | keep | Scale gate (T1-046). |
| TK-017 (verify, was TK-015) | must | C | keep | Endurance gate (T1-047). |
