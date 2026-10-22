# Review of the earlier experiments

The earlier experiments (claude.ai artifacts, 18–22 October 2026: research, datasheets, a OpenDAP OS prototype and 12 3D models) are references only, and many of their decisions were wrong. This review checks every item that was considered for adoption against the decisions already taken, the product rules (mainline-only, no custom kernel, commercially available parts, standard trick-less software, audio and battery first) and, where possible, the mainline Linux sources. Checked on 2026-10-21.

**Verdicts:** *Keep* (kept as a Draft requirement), *Withdraw* (wrong, conflicting or not validated), *Open* (turned into an open question).

## Electronics (`board`)

| Item | Verdict | Reason / evidence |
|---|---|---|
| RK809 PMIC | Keep | Companion PMIC of the RK3566; supported by the mainline `rk8xx` MFD driver (`drivers/mfd/rk8xx-core.c`). |
| 4 GB LPDDR4X, 32 GB eMMC | Withdraw → Open | Sizing came from the experiment's compute module; not validated against workloads (SDR and video recording) or cost. |
| 2 × CS43198 dual mono + 4 × OPA1622 | Keep | `cirrus,cs43198` is handled by mainline `sound/soc/codecs/cs43130.c`; dual mono and dedicated headphone amps serve the audio priority (T1-008). Availability still to check. |
| Two audio master clocks (45.1584 / 49.152 MHz), DAC as master | Keep | Standard way to play both sample-rate families bit-perfect without resampling (T1-008). |
| 3.5 mm single-ended, 4.4 mm balanced | Keep | Matches the confirmed ports; 4.4 mm (Pentaconn) is balanced by definition. |
| Pre-certified Wi-Fi/BT module | Keep | Required for certification of both editions (`toolkit` compliance research). |
| Murata Type 2EL (NXP IW612) | Keep as candidate | Mainline `nxpwifi` driver supports IW61x over SDIO (`drivers/net/wireless/nxp/nxpwifi`). Whether Debian's backports kernel enables it is not checked. |
| eFuse on each HAT rail | Keep | Consistent with the protection requirements. |
| Microphone privacy LED wired to microphone power | Keep | Hardware-enforced, cannot be faked by software. |
| BQ25895 / BQ27Z561 / TPS2121 / FUSB302B | Open | Common TI/onsemi parts with mainline drivers, but the desk research proposes BQ25792 (can also source power to a powerbank HAT). Evaluate together. |
| RP2354B as the RP2350 variant | Open | RP2350 is decided (board ADR 0004); the in-package-flash variant is a sourcing choice. |
| Compute module (8-layer HDI) + audio module + 6-layer base board | Withdraw | The maintainers' first intention is a single PCB with SoC and DAC chipset on it, no system-on-module. |

## Mechanics (`case`)

| Item | Verdict | Reason |
|---|---|---|
| Coordinate system (origin at center, z toward the faceplate), ISO 2768-m | Keep | A convention every mechanical part needs; no conflict. |
| Footprint 92.5 × 55 mm, R 8.5 | Withdraw → Open | Derived from the experiment's internal layout (its compute module, 42-pin HAT connector, battery), which no longer holds. CASE-001 (about a credit card) stands. |
| Thickness 19 mm, wall 1.6 mm | Withdraw → Open | Same reason: a consequence of a layout that changed. |
| Button sizes and positions, two-stage Action button | Withdraw → Open | Layout-derived; a two-stage switch also narrows sourcing. CASE-002/003 stand. |
| Bottom-edge x positions | Withdraw → Open | Layout-derived. CASE-004 (order) stands. |
| Four M1.6 screws for the faceplate | Withdraw → Open | The faceplate interface (CASE-009) defines anchoring; the size and count follow from a layout that changed. |
| Graphite, grip-textured polymer as default; PC or ABS first; crystal clear, carbon and anodized aluminum later | Keep (confirmed) | Confirmed by the maintainers (CASE-020 to CASE-022). |
| 654060 cell with protection on the board | Withdraw → Open | Conflicts with the compliance research, which recommends a certified pack (IEC 62133-2, UN 38.3) with its own protection and a connector, for the EU user-replaceable battery rule. |

## Faceplate (`faceplate-touch`)

| Item | Verdict | Reason / evidence |
|---|---|---|
| 3.5" 480 × 800 ST7701S panel "with mainline support" | **Withdraw (wrong)** | Mainline `panel-sitronix-st7701.c` supports only six specific panels, each with its own init sequence. Any other ST7701S panel needs a kernel patch, which violates OS-002. Replaced by a requirement that the panel already be supported in mainline; RK3566 handhelds in mainline (for example the Anbernic RG353P panel, driven by `panel-newvision-nv3051d`) show which panels qualify. |
| Goodix GT911 touch | Keep | Generic mainline `goodix` touchscreen driver. |
| Graphite tray that also takes SoC heat | Withdraw | Faceplates are interchangeable: the SoC's heat path cannot depend on which faceplate is fitted. |
| Ambient light sensor | Keep | Lets the backlight follow ambient light, which serves battery autonomy (T1-009). |
| Outline 53 × 90.5 × 3.6 mm | Withdraw → Open | Layout-derived. |

## HAT Case Standard (`hat-standard`)

| Item | Verdict | Reason |
|---|---|---|
| 42-contact connector (Raspberry Pi 40 + GND + PRESENT#), no PCIe or camera | Withdraw (wrong) | Conflicts with HCS-004 and HCS-006. |
| Passive HATs with flat pads | Keep | Already decided (ADR 0001). |
| Power classes A / B / F | Keep | Needed for a powerbank HAT (source) and for budgeting power. |
| Raspberry Pi HAT+ EEPROM format + OpenDAP record (reverse-domain ID, Ed25519, manifest hash) | Keep | A documented, standard EEPROM format, extended to meet HCS-009/010. |
| Power-up sequence ending in "apply overlay" at hot-plug | Withdraw (wrong) | Mainline Linux applies overlays from the bootloader, not at runtime; the sequence must be redesigned around boot-time detection or another hot-plug mechanism. |
| Four M2 screws, steel slider lock, release button on the right edge | Withdraw → Open | The right edge carries the power button; the mechanism was never validated. A latch is needed anyway for the pogo spring force. |

## OpenDAP OS and UI (`opendap-os`, `system`, `ui-components`)

The maintainers discards the experiments' **UI prototype and design system entirely**. Only items required for other reasons survive.

| Item | Verdict | Reason |
|---|---|---|
| MaskROM recovery, automatic A/B rollback | Keep | Already decided (opendap-os ADR 0004). |
| Alpine + dinit + DRM/KMS without Wayland | Withdraw (wrong) | Conflicts with Debian (opendap-os ADR 0002) and with Wayland. |
| Hearing safety: weekly dose, 85 dB(A), per-headphone profiles (EN 50332) | Keep | Required for personal music players (EN 50332), independently of the prototype. |
| Verified-HAT card on connect | Withdraw | From the discarded UI prototype. |
| Error messages: human sentence and next step | Withdraw | From the discarded UI prototype. |
| Button gesture map (Action ×1/×2/×3, hold to capture) | Withdraw → Open | UX not designed yet. "Volume − always lowers the volume" is worth keeping as a principle once UX is defined. |
| Activities Listen / Watch / Read / Create / Day / Advanced | Withdraw | Names never confirmed, and "Watch" conflicts with the T1 not being a video player. |
| IBM Plex, palette, layout anatomy, 480 × 800 reference screen | Withdraw | Visual design not started; the reference screen depends on a mainline-supported panel. |

## Not considered

Product variants (Lite, Pro), the extended catalogue of about 38 HATs, the dropped HATs (capture, LoRa, audio), and every number that appeared in only one artifact.
