# T1 product requirements

Product requirements (PRD) for the OpenDAP T1. Component requirements live in each component's repository.

| ID | Requirement | Priority | Status |
|---|---|---|---|
| T1-001 | The T1 shall be a digital audio player that is fully open source and open hardware, and runs 100% on Linux. | must | Confirmed |
| T1-002 | The T1 shall also be a general-purpose Linux gadget, extensible with HATs mounted on its back. | must | Confirmed |
| T1-003 | The T1 shall accept interchangeable faceplates. The default is the touch faceplate. | must | Confirmed |
| T1-004 | The T1 shall come in a Commercial and a DIY edition built from exactly the same design, differing only in manufacturing precision. | must | Confirmed |
| T1-005 | The Commercial edition shall be assembled. | must | Confirmed |
| T1-006 | The DIY edition shall be a kit the user builds. | must | Confirmed |
| T1-007 | The complete product shall be assembled and simulated from the exports of its components, each acting as a single unit. | must | Confirmed |
| T1-008 | Audio quality is a top priority: the T1 shall maximize sound quality and offer the features with real impact for audiophiles. | must | Confirmed |
| T1-009 | Battery autonomy is a top priority: power management shall maximize battery life. | must | Confirmed |
| T1-010 | Hardware shall prefer commercial, highly available, easy-to-source parts. | must | Confirmed |
| T1-011 | Software shall prefer the most standard Linux, open-source, trick-less option. | must | Confirmed |
| T1-012 | The battery shall be a standard commercial cell, not custom, with at least 2,000 mAh. | must | Confirmed |
| T1-013 | The device footprint shall be approximately that of a credit card (ISO/IEC 7810 ID-1: 85.60 × 53.98 mm). | must | Withdrawn |
| T1-014 | The left edge shall carry the **Action**, **Volume +** and **Volume −** buttons. | must | Confirmed |
| T1-015 | The right edge shall carry the **Main / Power** button. | must | Confirmed |
| T1-016 | The bottom edge shall carry, from left to right: a 3.5 mm jack, a 4.4 mm jack, a USB-C port (centered) and the integrated mono speaker (right side). | must | Confirmed |
| T1-017 | The Commercial edition should reach at least IP65, or the ingress protection standard of its product category. | should | Draft |
| T1-018 | The faceplate interface (`docs/specs/faceplate-interface.md`) shall define: flat-cable connector pinout, dimensions and anchoring with screws. | must | Confirmed |
| T1-019 | Faceplates are not meant to be swapped regularly. | must | Confirmed |
| T1-020 | Faceplate identification should rely on the faceplate interface pinout; an EEPROM only if the pinout cannot do it. | should | Confirmed |
| T1-021 | Coordinate system for every mechanical part: origin at the device center, x across the width, y along the length, z through the thickness with +z towards the faceplate. General tolerances ISO 2768-m. | must | Draft |
| T1-022 | The Action button shall have a deeper travel and a clearer click; the other buttons need no special click or travel. | must | Confirmed |
| T1-023 | Faceplate controls shall not send raw signals: the faceplate interface shall standardize binary commands that a DAP needs. | must | Confirmed |
| T1-024 | The command set shall have a well-standardized core that is not minimal, plus reserved space for new commands. | must | Confirmed |
| T1-025 | The T1 shall have an ADB-style command-line tool for advanced users (specified in `toolkit`). | must | Confirmed |
| T1-026 | Every technical promise shall be verified at least in software (or simulation) until it can be measured on hardware, and the tooling to debug it shall exist. | must | Confirmed |
| T1-027 | Hardware parts shall be the most recent generation available (for example the latest Wi-Fi and Bluetooth versions) as long as they stay compatible with the project: mainline Linux support, interfaces the SoC provides, and commercial availability. | must | Confirmed |
| T1-028 | Among compatible parts, prefer the ones with the best availability; when a part is not available, look for the best commercially available alternative. | must | Confirmed |
| T1-029 | Every hardware part shall be validated against real availability before it is adopted, and replaced by a part with availability when it has none. | must | Confirmed |
| T1-030 | Displays shall fit the product's dimensions; the product is never resized to fit a display. | must | Confirmed |
| T1-031 | Maximum device dimensions, excluding the side buttons and add-on HATs: 62.5 mm wide × 95 mm tall × 25 mm thick. | must | Confirmed |
| T1-032 | Ideal device dimensions, for the main body only (buttons may protrude): a credit-card footprint, rounded to 54 mm wide (ISO/IEC 7810 ID-1 is 85.60 × 53.98 mm), and up to 20 mm thick. Designs aim for the ideal and may grow up to the maximum (T1-031) only when needed. | should | Confirmed |
| T1-033 | The device should keep the proportions of a credit card (height-to-width ≈1.59, as 85.6 × 54 mm), also when it grows toward the maximum. The proportion is the ideal: adjusting it slightly is valid when it genuinely helps. | should | Confirmed |
| T1-034 | Displays shall keep an aspect ratio in about the same range as the body (height-to-width ≈1.5 to 1.6 in portrait), with **2:3 preferred**. | must | Moved |
| T1-035 | Battery capacity shall be as large as fits the defined dimensions. | must | Confirmed |
| T1-036 | The faceplate interface shall carry the signals for almost any display and touch connection standard, so that faceplates can change display or touch parts without changing the board. | must | Confirmed |
| T1-037 | Once the battery is chosen and autonomy re-estimated, the body shall shrink toward the ideal as long as audiophile-session autonomy stays around the average for this category of device (slightly below is acceptable), keeping the credit-card proportions; otherwise it stays at the size that meets autonomy. | should | Confirmed |
| T1-038 | No custom or made-to-order components: every part shall be an off-the-shelf catalog part. The only possible exception is the display cover glass. | must | Confirmed |
| T1-039 | Audiophile-session autonomy thresholds: **6 h is the red line** (minimum admissible); below 8 h is a warning; 8 h or more is fine; more than 12 h is ideal. They also bound how far the body may shrink (T1-037). | must | Confirmed |
| T1-040 | The audiophile experience is the one thing that is never sacrificed: it is required at 100 %. This is an OpenDAP. | must | Confirmed |
| T1-041 | Everything else is balanced by cost and benefit, with concessions where they give the best possible product and make the most of the hardware. | must | Confirmed |
| T1-042 | No decision shall depend on obtaining a development kit (for example a Radxa ZERO 3); kits only refine estimates when available. | must | Confirmed |
| T1-043 | The T1 shall define a standard set of button combinations for system functions (power, power menu, restart, boot modes, screenshots and others), inspired by Android's conventions; for example, Volume − + Main held opens the power/restart menu and Main held powers on. | must | Confirmed |
| T1-044 | The T1 shall define a status LED code with at least a base set of states, based on existing standards where possible. | must | Confirmed |

## Context

- T1-043, T1-044: specified in [`../specs/controls-and-status-led.md`](../specs/controls-and-status-led.md) (Draft).
- **OpenDAP** is the brand; **T1** is the model and the product.
- Official faceplates: touch (default), cassette, classic. Official HATs: powerbank, NAS, camera, recorder, oscilloscope, VNA, radio.
- T1-013 (was CASE-001), T1-014 (was CASE-002), T1-015 (was CASE-003), T1-016 (was CASE-004), T1-017 (was CASE-007), T1-018 (was CASE-009), T1-019 (was CASE-010), T1-020 (was CASE-011), T1-021 (was CASE-012), T1-022 (was CASE-023), T1-023 (was CASE-026), T1-024 (was CASE-027): moved from the `case` repository on 2026-10-21, because they are product requirements, not physical-case ones.
- T1-013 (about a credit-card footprint) was replaced on 2026-10-22 by T1-031 (maximum) and T1-032 (ideal): finding a suitable display inside a credit-card footprint proved too hard.
- T1-031 and T1-032 are outer dimensions of the finished device: they already include the case walls, the display's inner frame, bezels and every other layer. Display modules and internals must fit inside them.
- T1-034 moved to `faceplate-touch` as FPT-020 on 2026-10-22: the aspect-ratio rule applies to the touch faceplate; other faceplates have their own design rules.
- Body size (2026-10-22): designs tend toward the ideal minimum, and may grow up to the maximum (T1-031) whenever the requirements need it (for example battery capacity, T1-035). At the card ratio, a 95 mm maximum height gives about 59.9 mm of width.
- The closer to credit-card size, the better, keeping the card's proportions (maintainers, 2026-10-22). This replaces an earlier preference for a 1.50 ratio.
- The thinner, the better (maintainers, 2026-10-22): thickness tends toward the 20 mm ideal or less, and grows toward 25 mm only when autonomy needs it.
- T1-039 applies to the core audiophile session (screen off, local playback). Until the RK3566 draw is measured, the check uses the pessimistic estimate: the [battery life analysis](research/2026-10-battery-life-analysis.md) puts the Adafruit 328 at about 12.8 h typical but about 5.7 h pessimistic, below the red line, so the measurement gate in ADR 0003 is decisive.
- "Credit card sized" is part of the product's identity; that is why the size ranges (T1-031, T1-032, T1-033) are strict. Any growth beyond the ideal must be justified and reversed as soon as possible.
