# Research: radio performance of the T1

- **Date:** 2026-10-24
- **Status:** Desk research. Engineering input, not a decision. Nothing here has been measured on T1 hardware. The proposed requirement (§7) is for the main session to number and integrate; the board changes (§8) are recommendations for the board's owners. This note edits no requirement, board ADR, board constraint or board requirement.
- **Method:** this note is the T1's application of the shared [radio performance method](https://github.com/opendap-project/toolkit-hardware/blob/main/docs/research/2026-10-radio-performance-methods.md) (toolkit-hardware), which holds the link-budget terms, the benchmark protocol, the measurement tiers (RM-C, RM-B, RM-O, field courses) and the rule that turns results into targets. This note keeps what is specific to the T1: where its design stands, the benchmark numbers that apply, the levers ranked for the T1, the recommended changes and the targets.
- **Question (maintainers, 2026-10-24):** radios strong enough to compete fully with, and beat, commercial devices that degrade quickly with distance or walls. Transmit power is capped by spectrum law (T1-045 as amended, T1-075), so the advantage has to come from engineering.
- **Inputs read:** [T1 requirements](../requirements/README.md) (T1-027, T1-040, T1-045, T1-048, T1-075, T1-076); [experience budgets](../specs/experience-budgets.md); [electronics integration and verification](../specs/electronics-integration-verification.md) (M-RF, V2-R01, V4-02, V5-01); board [ADR 0018](https://github.com/opendap-project/board/blob/main/docs/adr/0018-antennas-and-certification-route.md) and its [antennas and RF exposure study](https://github.com/opendap-project/board/blob/main/docs/research/2026-10-antennas-and-rf-exposure.md); the board [component selection](https://github.com/opendap-project/board/blob/main/docs/research/2026-10-component-selection.md) (radio options, SDIO); Murata CER-0248 Rev. 6.0 and E2B-09-1413 Rev. 9.0 (hashes in `parts-lib/manifest.toml`); the Ezurio *Sona MT320 datasheet* v1.3; mainline Linux (`torvalds/linux` master, Makefile 7.3, read 2026-10-24) and `linux-firmware` (main, read 2026-10-24); [RFC 0010](https://github.com/opendap-project/rfcs/blob/main/text/0010-quality-levels.md) and [RFC 0011](https://github.com/opendap-project/rfcs/blob/main/text/0011-feature-blocks-and-variants.md); case requirements CASE-020 and CASE-022.
- **Conventions:** as in the method: **UNVERIFIED**, "(vendor claim)", "(secondary)", and arithmetic shown as such. "Model" marks a link-budget estimate built from stated assumptions, not a measurement.

## 0. Summary

1. **Where the T1 starts.** One Murata Type 2EL (NXP IW612: Wi-Fi 6 1×1, Bluetooth 5.4 BR/EDR/LE) on one Molex 146153 antenna inside the top wall, Wi-Fi and Bluetooth sharing it (board ADR 0018). The module's Bluetooth receiver is among the best available (BR −94 to −95 dBm, 3-DH5 −89 dBm, LE Coded −107 dBm, E2B-09-1413 Table 16). Its certified power tables are much lower for EDR, the modulation that carries LDAC at 660 and 990 kb/s, than for BR and LE: FCC 13 ± 3 dBm for BR and LE but 5 ± 3 dBm for EDR; EU 3 ± 3 dBm for BR and LE and 0 ± 3 dBm for EDR (CER-0248 Tables 9, 41).
2. **Benchmark.** Commercial Digital Audio Players publish neither Bluetooth range nor output power; reviewers report range only as anecdote. What public measurements do show: LDAC 990 and 660 kb/s need about −60 dBm at the headphones and 330 kb/s holds to about −80 dBm (SoundGuys 2025); a phone in a trouser pocket swings by 15–20 dB with body orientation and a 15 cm blockwork wall costs about 20 dB at 2.4 GHz (Leith 2020); Android phones average 308–369 ms of Bluetooth audio latency (SoundGuys); 2×2 phones carry 150–160 Mb/s at 15 m through walls on a Wi-Fi 6 access point (Keenan 2019). The T1's own benchmark set has to be measured with the method (§2.4).
3. **Levers, ranked for the T1** (§3): (1) antenna efficiency and placement in the main body frame, several dB to over 10 dB between a poor and a good built-in antenna; (2) LE Audio over the LE PHY, which the certified tables let transmit 8 dB more than EDR (FCC) and whose receivers are about 10 dB more sensitive than 3-DH5, about 17 dB of forward budget (model); (3) a dedicated Bluetooth antenna, which removes airtime contention: LDAC 990 already fills 45–68 % of Bluetooth airtime before any Wi-Fi; (4) body-aware placement, 3–10 dB; (5) codec and link-adaptation policy, about 20 dB of RSSI between LDAC 990 and 330 kb/s; (6) the EU power table, +5 dB for EDR and +10 dB for BR and LE if OpenDAP tests its own EN 300 328 reports; (7) desense control, one dB per dB; (8) a 2×2 Wi-Fi module, a Wi-Fi gain that costs Bluetooth power, size and Japan certification.
4. **No mainline-SDIO module beats the IW612 for Bluetooth audio today.** The only 2×2 Wi-Fi 6 option with a mainline SDIO driver and a certified catalog module is the Ezurio Sona MT320 (MediaTek MT7921, `mt7921s` and `btmtksdio` in mainline since 2021). It transmits Bluetooth at 7 dBm maximum in every mode: 2 dB more than the IW612's FCC EDR setting, 6 dB less for BR and LE; and it is 20 × 14 mm against 8.8 × 7.7 mm. Infineon CYW5557x and Realtek RTL8852BS have no mainline SDIO support (checked 2026-10-24).
5. **Recommended T1 changes** (§5): make the dedicated Bluetooth antenna the target configuration and reserve a second antenna zone in the main body frame now; keep every antenna in the frame behind dielectric windows in any metal case; LE Audio as the long-range path; an EU variant with its own Bluetooth power tests; Japan's high-power setting; tuned matching and efficiency targets; a desense budget; link-adaptation and coexistence policy in Blackbird OS; the Type 2EL stays the radio of the full and production variants, with the MT320 only for a Wi-Fi-first variant.
6. **Targets** (§6): thirteen rows added to the "Radio" section of the experience budgets, each also held against the benchmark median (ideal ≥ 2×, hard limit at 1.0×).

## 1. Where the T1 starts

| Item | Value | Source |
|---|---|---|
| Module | Murata Type 2EL (LBES5PL2EL-923), NXP IW612; Wi-Fi 802.11a/b/g/n/ac/ax 1×1, up to 600 Mb/s PHY; Bluetooth 5.4 BR/EDR/LE; 802.15.4 not wired | E2B-09-1413 §2 |
| Host links | SDIO 3.0 for Wi-Fi; UART and PCM for Bluetooth; RK3566 SDMMC1 is SDIO 3.0, about 50 MB/s practical | E2B-09-1413 §2; board component selection |
| Antenna | One Molex 146153-0050 (3.2 / 4.25 dBi registered) on the inside of the top wall, shared Wi-Fi and Bluetooth (10 pF from pin 18 to pin 23); the dedicated configuration (Bluetooth on ANT1) is kept open in the layout and on the prototype | Board ADR 0018 |
| Keep-out | ≥ 5 mm from any metal, provisional | Board ADR 0018 decision 5 |
| Body | 59.5 × 94.3 × 25.0 mm; maximum 62.5 × 95 × 25 mm | Board ADR 0016; T1-031 |
| Bluetooth sensitivity (typical, at the module port) | DH5 −94/−95, 2-DH5 −94/−95, 3-DH5 −89, LE 1M −101/−102, LE 2M −99/−100, LE Coded 125k −106/−107 dBm | E2B-09-1413 Table 16 |
| Wi-Fi sensitivity | 2.4 GHz HE20 MCS0 −92, MCS11 −62/−63; 5 GHz HE80 MCS0 −85, MCS11 −57 dBm | E2B-09-1413 Tables 11, 15 |
| Wi-Fi throughput (Murata's test) | 5 GHz HE80: 419 Mb/s TCP TX, 384 Mb/s TCP RX; 2.4 GHz HE20: 119 / 116 Mb/s; at about 0.9 m from an ASUS RT-AX88U, on an NXP i.MX 8M Mini board | E2B-09-1413 Table 25 |
| Bluetooth power tables | FCC: BR 13.0 ± 3.0, **EDR 5.0 ± 3.0**, LE and LE 2M 13.0 ± 3.0 dBm (ISED Table 25 identical); EU: BR 3.0 ± 3.0, EDR 0.0 ± 3.0, LE 3.0 ± 3.0 dBm; Japan (typical): BR 5, EDR 2, LE 5 dBm, or with BR/EDR only a high-power setting of BR 12, EDR 5 dBm | CER-0248 Tables 9, 25, 41, 50, 51 |
| Coexistence | IW612 "enhanced collaborative coexistence hardware mechanisms and algorithms" | E2B-09-1413 §2 (vendor claim) |

## 2. Benchmark

### 2.1 Bluetooth audio range and dropouts

| Finding | Number | Source |
|---|---|---|
| RSSI needed at the headphones | LDAC 990 / 660 kb/s: stutter risk just below −60 dBm; 330 kb/s and standard codecs drop packets around −80 dBm; Sony WH-1000XM2/XM3 measured about −49 to −51 dBm on average from a phone, minimum near −60 dBm when the body obstructs | R. Triggs, SoundGuys, 2025-10-01 |
| Phone bitrate defaults | Most phones default to LDAC 660 kb/s or lower; one (LG V30+) reached 990 kb/s automatically; Pixel 3 / 3 XL stayed at 330 kb/s | Same |
| Body and pocket | Phone in the left trouser pocket, person rotating 1 m from another phone: about 20 dB swing; facing about −60 dBm, back turned about −80 dBm; a person standing close behind a handset (as for a rear pocket): about 15 dB lower than in front | Leith and Farrell 2020, §III |
| Walls | Plasterboard stud wall: little attenuation; blockwork wall about 15 cm thick: about 20 dB (2.4 GHz, phones 1 m apart) | Same |
| Body loss statistics | Mean body loss about 10, 6 and 3 dB for helix, whip and patch antennas; up to 10 dB between users (1.89 GHz, 200 users) | Nielsen et al. 2001 |
| Antenna losses dominate | Handheld devices in data mode: absorption in the body around 50 %; losses in built-in antennas often dominating, up to 99 % total for one smartphone user; "a better antenna would reduce the total losses to less than 50 %" | Andersen et al. 2016 |
| DAP specifications | Range and Bluetooth output power not published (Sony NW-ZX707 help guide; FiiO M23 parameters). A HiBy R6 Pro II development specification indexed by a search engine lists 10 ± 2.5 dBm for DH5 and 8 ± 2.5 dBm for EDR; the document could not be retrieved | **UNVERIFIED** for HiBy |
| DAP reviews | Range reported as "quite good" without distances (iBasso DX180, Mobile Audiophile 2024-09-07) | — |

### 2.2 Wi-Fi throughput at range

| Finding | Number | Source |
|---|---|---|
| 2×2 phones, Wi-Fi 6 access point, 50 ft (15 m) indoors through walls and furniture | Galaxy S10e 160 Mb/s average (176 peak); iPhone Xs (Wi-Fi 5) 150 Mb/s; with a Wi-Fi 5 access point at the same spot: 35 and 65 Mb/s | Keenan Systems, 2019-08-06 |
| A 1×1 module close in | IW612 384 Mb/s TCP RX at about 0.9 m, 5 GHz HE80 | E2B-09-1413 Table 25 |
| DAPs | No public throughput-at-range data found | — |

### 2.3 Latency

| Finding | Number | Source |
|---|---|---|
| Android Bluetooth audio, four phones | SBC 308 ms, aptX 316 ms, LDAC 324 ms, AAC 369 ms on average | SoundGuys, Android Bluetooth latency (page read 2026-10-24) |
| aptX Adaptive | Below about 90 ms claimed | Qualcomm (vendor claim, secondary) |

### 2.4 The T1's benchmark set (to measure)

Per the method §3: three current DAPs across the T1's price segment, two phones, two reference sinks (over-ear, true wireless) with LDAC, one Wi-Fi 6 access point; every row of §6 measured on each. Until then the relative criteria of §6 cannot be evaluated and the absolute values stay Draft. Proposed at Q1 for the shielded-box tier (RM-B1 to RM-B3) and Q2 for the field courses.

### 2.5 What the numbers mean for the T1 (model)

Illustrative link budget for the forward link, indoors with ITU-R P.1238 N = 28, headphones at −3 dBi, 10 dB fade margin. EDR rows carry LDAC 660 and 990 kb/s with an assumed headphone 3-DH5 sensitivity of −86 dBm; the LE rows carry LE Audio with an assumed LE 2M sensitivity of −95 dBm. The headphone figures, the "typical DAP" EDR power of 6 dBm and the antenna losses are assumptions (**UNVERIFIED**), chosen to show sensitivities, not to predict range.

| Case | Conducted | Antenna loss | Body | Allowed path loss | Range, indoor N = 28 | Through a 20 dB wall |
|---|---|---|---|---|---|---|
| Typical DAP, EDR (assumed) | 6 dBm | −6 dB | 0 | 73 dB | 15 m | 3.0 m |
| Typical DAP, EDR, in a pocket | 6 dBm | −6 dB | −15 dB | 58 dB | 4.5 m | 0.9 m |
| T1, FCC EDR, good antenna | 5 dBm | −2 dB | 0 | 76 dB | 20 m | 3.8 m |
| T1, FCC EDR, good antenna, in a pocket | 5 dBm | −2 dB | −15 dB | 61 dB | 5.7 m | 1.1 m |
| T1, EU EDR today, good antenna | 0 dBm | −2 dB | 0 | 71 dB | 13 m | 2.5 m |
| T1, EU EDR today, in a pocket | 0 dBm | −2 dB | −15 dB | 56 dB | 3.8 m | 0.7 m |
| T1, FCC LE (LE Audio), good antenna | 13 dBm | −2 dB | 0 | 93 dB | 80 m | 15 m |
| T1, FCC LE (LE Audio), good antenna, in a pocket | 13 dBm | −2 dB | −15 dB | 78 dB | 23 m | 4.5 m |

(Arithmetic: allowed loss = P + antenna loss + body + G_rx − S − margin; d = 10^((L − 39.75)/28).) The model says four things that measurement must confirm: for LDAC the T1 cannot win on transmit power, because its certified EDR power is no higher than a typical DAP's, so its advantage has to come from the antenna, placement and the receive side; the EU EDR table costs another 5 dB; the pocket costs more than any single design lever gains; and LE Audio, on the LE PHY's higher power and better sensitivity, has about 17 dB more forward budget than EDR, roughly 4× the range at N = 28, at the cost of LC3's lower bitrates.

## 3. Levers for the T1, ranked

| Rank | Lever | Effect for the T1 | Basis | Cost and constraints |
|---|---|---|---|---|
| 1 | **Antenna efficiency and placement in the main body frame** | Several dB to more than 10 dB; unknown for the T1 until RM-O3. It counts on both directions of the link, so it also helps the weak headphone-to-T1 path | Andersen 2016; Molex 146153 efficiency in an enclosure not obtained (**UNVERIFIED**) | Keep-out to battery, pocket floor, microSD shell, screw inserts; never the faceplate (display, touch, metal frame) and not the back cover (HATs mount there); tuning per enclosure material |
| 2 | **LE Audio over the LE PHY** | About +8 dB of certified transmit power over EDR (FCC LE 13 against EDR 5 dBm) and about 10 dB better T1-side sensitivity (LE 2M −99/−100 against 3-DH5 −89 dBm): about 17 dB of forward budget in the model, 4× the range at N = 28 | CER-0248 Table 9; E2B-09-1413 Table 16; §2.5 | LC3 bitrates (up to 320 kb/s per channel) below LDAC's top steps; headphone support; BlueZ and controller support on the T1 **UNVERIFIED**; IW612 LE Audio is a vendor claim |
| 3 | **Dedicated Bluetooth antenna (ANT1)** | Removes the 2.4 GHz airtime contention: LDAC 990 needs 45 % (3-DH5) to 68 % (2-DH5) of Bluetooth airtime, leaving 1.2 or 0.46 retransmissions per packet before Wi-Fi takes any; also places the Bluetooth antenna for the body case independently of Wi-Fi | Method §1.4 (arithmetic); E2B-09-1413 Figures 2, 10 | A second keep-out zone that ADR 0018 found no room for at 5 mm; second antenna (W24P-U $1.48 or 146153 $2.27, Digi-Key 2026-10-24, from the board study); simultaneous-transmission SAR analysis; dedicated calibration file `WlanCalData_ext_dANT.conf` |
| 4 | **Body-aware placement** | 3–10 dB mean body loss by antenna; 15–20 dB swings in a pocket | Nielsen 2001; Leith 2020 | The top wall stays out of the hand in portrait use and faces up in a pocket (to confirm in RM-O5); a second antenna at the other upper corner or side gives the link a second path when one end is shadowed |
| 5 | **Codec and link-adaptation policy** | About 20 dB of RSSI between the needs of LDAC 990 and 330 kb/s | SoundGuys 2025 | Adaptive by default with the bitrate visible; fixed bitrates always available (T1-045); prefer 3-DH5 at 990 kb/s (45 % against 68 % airtime) |
| 6 | **EU Bluetooth power table** | EDR from 0 to 5 dBm (+5 dB, the FCC EDR setting) and BR and LE from 3 to 13 dBm (+10 dB): 1.5× and 2.3× range at N = 28. EIRP with 3.2 dBi stays under EN 300 328's 20 dBm | CER-0248 Tables 9, 41; EN 300 328 (secondary). Murata's EU BR and LE setting (9.2 dBm EIRP at the tolerance's top) sits just under the 10 dBm EIRP above which adaptivity requirements apply; that this is Murata's reason, and that 5 dBm is the IW612's EDR linearity limit rather than a regulatory choice, are inferences (**UNVERIFIED**) | Above 10 dBm EIRP the EU variant needs its own conducted EN 300 328 reports, adaptivity included (whether Bluetooth AFH satisfies them as tested is **UNVERIFIED**); SAR assessed at the new level (T1-076). FCC, ISED and Japan tables are fixed by Murata's grants |
| 7 | **Desense control** | One dB of sensitivity per dB of in-channel noise; protects the IW612's −89 dBm 3-DH5 and −85 dBm HE80 MCS0 | Arithmetic; V4-02 | SDIO and DDR harmonics, switching regulators, display; a budget is needed (§5.6) |
| 8 | **Coexistence software** | Recovers airtime: Wi-Fi on 5 GHz while Bluetooth audio plays, no background 2.4 GHz scans while streaming | Method §1.4 | Whether the IW612 shares airtime between Bluetooth and 5 GHz Wi-Fi in the shared configuration is **UNVERIFIED**; measured in COEX |
| 9 | **2×2 Wi-Fi module (MT7921, Sona MT320)** | Wi-Fi only: up to about 10 dB from diversity at 1 % outage in fading and a second spatial stream; peak still bounded by SDIO 3.0 (about 400 Mb/s) | Method §1.3; MT320 datasheet | Bluetooth 7 dBm in every mode (EDR +2 dB, BR and LE −6 dB against the IW612 FCC table); 20 × 14 × 2.42 mm module; two antennas for Wi-Fi plus Bluetooth shared on one of them; Japan certification "to be added" |

## 4. Radio modules against the aim

Mainline status checked on 2026-10-24 against `torvalds/linux` master (Makefile 7.3) and `linux-firmware` main. "Mainline SDIO" means the SDIO bus driver and the chip's IDs are in the tree.

| Module (chip) | Wi-Fi | Bluetooth TX / sensitivity | Mainline SDIO | Firmware in linux-firmware | Size | Verdict for the T1 |
|---|---|---|---|---|---|---|
| **Murata Type 2EL (NXP IW612)** | Wi-Fi 6 1×1 | FCC table BR and LE 13 ± 3, EDR 5 ± 3 dBm; EU 3 / 0 ± 3 dBm / DH5 −94/−95, 3-DH5 −89, LE 2M −99/−100 dBm | Yes: `nxpwifi` (first commit 2026-07-15, in 7.3), `btnxpuart` (2023) | `nxp/sd_w61x_v1.bin.se`, `nxp/uartspi_n61x_v1.bin.se` | 8.8 × 7.7 mm outline (DXF) | **Keep** for full and production variants: best Bluetooth link budget available on a mainline SDIO path |
| u-blox MAYA-W271 (IW612) | Wi-Fi 6 1×1 | **UNVERIFIED** | Same chip and drivers | Same | **UNVERIFIED** | Second source (board component selection) |
| **Ezurio Sona MT320 (MediaTek MT7921)** | Wi-Fi 6 **2×2**, one antenna Wi-Fi only, the other shared with Bluetooth | 7 dBm maximum / 1DH5 −94, 3DH5 −88, LE Coded −104 dBm | Yes: `mt7921s` (commit 2021-10-20), `btmtksdio` | `mediatek/WIFI_RAM_CODE_MT7961_1.bin`, `BT_RAM_CODE_MT7961_1_2_hdr.bin` | 20 × 14 × 2.42 mm (M.2 1420 LGA) | Candidate only for a Wi-Fi-first variant (RFC 0011 higher-tier or application kind); for Bluetooth audio about equal on EDR, worse on LE; FCC, IC, CE, UKCA; Japan pending (datasheet v1.3) |
| NXP IW610 modules | Wi-Fi 6 1×1, 20 MHz | Bluetooth **LE only** | Yes (`nxpwifi` IW610 IDs) | **UNVERIFIED** | — | Out: no BR/EDR, so no A2DP with most headphones |
| Murata Type 2EA (Infineon CYW55573) | Wi-Fi 6E 2×2 | **UNVERIFIED** | **No**: no CYW5557x entry in `brcmfmac` SDIO (Infineon ships out-of-tree patches) | — | 12.5 × 9.4 × 1.2 mm (vendor claim) | Watch list: re-check when upstream |
| Realtek RTL8852BS modules | Wi-Fi 6 2×2 | **UNVERIFIED** | **No**: `rtw89` has PCIe and USB, no SDIO | — | — | Watch list |
| Broadcom BCM43752 modules (for example Ampak AP6275S) | Wi-Fi 6 2×2 | **UNVERIFIED** | Chip known to `brcmfmac` SDIO | **No** `brcmfmac43752-sdio.bin` | — | Out until firmware is redistributable in linux-firmware |
| MediaTek MT7902 SDIO modules | **UNVERIFIED** | **UNVERIFIED** | Yes since 2026-03 (Wi-Fi) and 2026-04 (Bluetooth) | **UNVERIFIED** | — | Watch list: no certified catalog module found |
| NXP 88W8997 modules | Wi-Fi 5 2×2 | **UNVERIFIED** | Yes (`mwifiex` sd8997) | — | — | Older generation (T1-027) |

PCIe and USB radios stay out for the reasons in the board component selection (PCIe is taken by the HAT; USB 2 caps throughput and idles higher).

## 5. Recommended T1 changes

### 5.1 Dedicated Bluetooth antenna as the target configuration

ADR 0018 chose the shared configuration for space and kept the dedicated one open, switching only if the coexistence gate fails. Against the aim, the airtime arithmetic of §3 argues the other way: LDAC 990 leaves little retransmission room even without Wi-Fi, and any 2.4 GHz Wi-Fi activity on a shared radiator takes from it. Recommendation:

1. **Target configuration: dedicated** (Wi-Fi on ANT0, Bluetooth on ANT1), confirmed or rejected by measurement on prototype 1 with both configurations fitted (RM-B1, RM-B3, COEX).
2. **Reserve the second antenna zone in the main body frame now**, so the choice needs no case change later. Candidates, each with its cost:

   | Option | Keep-out obtained | Cost |
   |---|---|---|
   | Upper part of a side wall, long axis along y, orthogonal to ANT0 (polarisation and spatial separation) | ≈ 2.9 mm today (board study §3.2) | Below the provisional 5 mm; acceptable only if RM-O3 at 3, 4 and 5 mm spacers shows the loss is small |
   | Same, body widened toward T1-031's 62.5 mm | Up to ≈ 5 mm with about 2.1 mm more width | Size: against T1-032 and T1-033; maintainers' call |
   | Same, battery pocket narrowed by about 2 mm | ≈ 5 mm | Capacity (T1-035, T1-039); maintainers' call |
   | Second antenna on the top wall | Does not fit (board study §3.2) | — |

   The prototype answers which keep-out is enough; the case work keeps the zone free and polymer-windowed until then.
3. If measurement shows no gain, shared stays, as ADR 0018 decided.

### 5.2 Antenna rules for the case, including a metal case

- Antennas live in the **main body frame** only: never in the faceplate (display, touch and their metal), not in the back cover (HATs mount there and bring metal).
- In any metal or metallised frame (CASE-022's anodised aluminium option, the metal-case work in progress), each radiating element sits behind a **dielectric window** in the frame at least as large as the element plus its keep-out on every side; no metal, coating or metallic paint over the window; screws and inserts outside the keep-out. A frame used as the antenna itself (split rims, as in metal phones) would not be a Murata-listed antenna type: FCC and ISED would need a permissive change through Murata and Japan would need Murata to add it (board study §4), so it is not recommended for the T1.
- The window and keep-out positions are inputs to the case (E4.6) and to the HAT standard's antenna keep-out (T1-085).

### 5.3 Regional power tables

- **EU variant:** test Bluetooth at the FCC table's settings (BR and LE 13 dBm, EDR 5 dBm conducted; 16.2 and 8.2 dBm EIRP with 3.2 dBi, under 20 dBm), with OpenDAP's own EN 300 328 conducted and adaptivity reports and SAR at that level. Q3 item of the EU regional variant (RFC 0011 §4).
- **Japan variant:** use Murata's high-power setting when Bluetooth runs BR/EDR only (BR 12, EDR 5 dBm, CER-0248 Table 51), and the standard table (LE 5 dBm) when LE is in use; how the T1 switches between them is a software point (**UNVERIFIED** whether Murata's condition "only BT (BR/EDR)" means no LE at all while the setting is active).
- **USA and Canada:** Murata's FCC and ISED tables are the hardware's best within the grant. Why EDR is 8 dB below BR there (likely the 8DPSK linearity of the power amplifier) is **UNVERIFIED**; it is the reason LE Audio is the long-range path.
- Blackbird OS shows the active table and its legal basis (T1-045, T1-075).

### 5.4 Antenna design targets (verification rows, not experience budgets)

| Quantity | Ideal | Warning | Hard limit | Measure |
|---|---|---|---|---|
| Radiation efficiency in the case, free space, 2.4 and 5 GHz | ≥ 50 % (−3 dB) | < 32 % (−5 dB) | < 20 % (−7 dB) | RM-O3 |
| Extra loss with a hand phantom | ≤ 3 dB | > 5 dB | > 8 dB | RM-O2 |
| Isolation ANT0–ANT1 at 2.44 GHz (dedicated) | ≥ 25 dB | < 20 dB | < 15 dB | RM-O4 |
| Return loss in band after tuning | ≥ 10 dB | < 8 dB | < 6 dB | RM-O4 |

All are proposals (**UNVERIFIED** against Molex's and NXP's guidance, which was not obtained); gain after tuning stays at or below the registered 3.2 / 4.25 dBi (board ADR 0018).

### 5.5 Software and firmware (Blackbird OS, system)

- Link adaptation: LDAC adaptive by default, preferring 3-DH5 at 990 kb/s; bitrate, codec and RSSI one step away in the interface (audiophile first, engineer one step away); fixed bitrates always selectable (T1-045).
- Coexistence: prefer 5 GHz for Wi-Fi while Bluetooth audio plays; no background 2.4 GHz scans while streaming; PTA priorities for A2DP over Wi-Fi bulk traffic. How the mainline `nxpwifi` and `btnxpuart` expose coexistence settings is **UNVERIFIED**.
- LE Audio LC3 with retransmissions and packet-loss concealment when controller, BlueZ and headphones support it: the long-range path (§3 rank 2), offered when the link would otherwise drop LDAC to its lowest step, as a choice shown to the listener, never forced.
- Logging of codec steps, retransmissions, flushes and RSSI for the field courses (method §6.2).

### 5.6 Desense budget

V4-02 has no number yet. Proposal: in-channel noise rise with every domain active ≤ 1 dB (ideal), > 2 dB (warning), > 3 dB (hard limit), per band, measured as conducted sensitivity loss (RM-C3).

### 5.7 Radio module per variant (RFC 0011)

| Variant kind | Radio block | Reason |
|---|---|---|
| `full` (prototype 1) | Type 2EL, both antennas fitted, both configurations measurable | Measures the dedicated option; best Bluetooth budget |
| `production` | Type 2EL, configuration chosen by §5.1's measurement | Same |
| Regional (EU, USA, Canada, Japan) | Type 2EL with each market's table; EU with its own Bluetooth tests (§5.3) | RFC 0011 §4 |
| Higher-tier or application, Wi-Fi first | Sona MT320 as an alternate radio block behind the same SDIO, supplies and RF ports, if the radio block's interface is drawn for it (RFC 0011 rule 6: a 20 × 14 mm land and a second Wi-Fi antenna) | 2×2 Wi-Fi; accepts −6 dB Bluetooth power and Japan pending |

## 6. Targets

Added to the "Radio" section of the [experience budgets](../specs/experience-budgets.md), Draft until calibrated on the prototype and the benchmark set. Courses and IDs are the method's. Reference sinks, access point and benchmark set per method §3. Every range or throughput row also carries the relative criterion: ideal ≥ 2× the benchmark median, warning < 1.5×, hard limit ≤ 1.0×; latency rows the inverse.

| Metric (condition) | Ideal | Warning | Hard limit | Measure | Level of hard limit |
|---|---|---|---|---|---|
| Bluetooth audio on the body: T1 in a front then back trouser pocket, reference headphones worn, 30 min each, LDAC 990 kb/s held | 0 dropouts | any dropout | > 3 dropouts in 30 min | BODY | Q2 |
| Bluetooth range, line of sight, LDAC 990 kb/s held, no dropout | ≥ 15 m | < 10 m | < 5 m | LOS | Q2 |
| Bluetooth range, line of sight, LDAC adaptive, no dropout | ≥ 40 m | < 25 m | < 15 m | LOS | Q2 |
| Bluetooth range through one interior wall, LDAC adaptive, no dropout | ≥ 20 m | < 12 m | < 8 m | W1 | Q2 |
| Bluetooth range through two interior walls, LDAC adaptive, no dropout | ≥ 12 m | < 8 m | < 5 m | W2 | Q2 |
| Bluetooth audio with Wi-Fi streaming 20 Mb/s on the same T1, LDAC 990 kb/s, 3 m, 60 min, Wi-Fi on 2.4 then 5 GHz | 0 dropouts | — | any dropout | COEX | Q1 (ADR 0018 gate) |
| Wi-Fi TCP downlink, 5 GHz 80 MHz, 1 m line of sight | ≥ 350 Mb/s | < 250 Mb/s | < 150 Mb/s | LOS | Q2 |
| Wi-Fi TCP downlink, 10 m through one wall | ≥ 200 Mb/s | < 120 Mb/s | < 60 Mb/s | W1 | Q2 |
| Wi-Fi TCP downlink, 15 m through two walls | ≥ 100 Mb/s | < 50 Mb/s | < 25 Mb/s | W2 | Q2 |
| Range with sustained TCP ≥ 20 Mb/s, two-wall course extended | ≥ 25 m | < 18 m | < 12 m | W2 | Q2 |
| Bluetooth audio latency, A2DP (SBC, AAC, LDAC), reference headphones | ≤ 150 ms | > 200 ms | > 300 ms | Loopback | Q2 |
| Bluetooth audio latency, LE Audio (LC3), when supported | ≤ 60 ms | > 100 ms | > 150 ms | Loopback | Q3 |
| Wi-Fi round trip to the access point under load, 99th percentile | ≤ 20 ms | > 50 ms | > 150 ms | Under load | Q2 |

Notes:

- Wall losses on W1 and W2 are measured and recorded (target about 5–10 dB per wall at 2.44 GHz); results are only comparable on the same course.
- A 1×1 T1 is not expected to beat 2×2 phones in throughput at range; the Wi-Fi rows aim at the DAP benchmark. The phones' 150–160 Mb/s at 15 m (Keenan 2019) is a reference, not a target.
- The A2DP latency hard limit (300 ms) sits at the Android average for SBC (308 ms); the ideal is half of it.
- The EU variant with today's Murata EU table (EDR 0 dBm, BR and LE 3 dBm) may miss the Bluetooth range rows; §5.3 is the remedy.
- The LDAC rows ride on EDR at 5 dBm under the FCC table; the T1 beats the benchmark on them only through §3 ranks 1, 3, 4, 5 and 7.

## 7. Proposed product requirement (for the main session)

> **T1-NEXT.** The T1's Wi-Fi and Bluetooth shall beat the typical commercial Digital Audio Player in Bluetooth audio stability and range (on the body, in line of sight and through walls), Wi-Fi throughput at range and radio latency, measured with OpenDAP's radio performance method against a dated benchmark set, and shall get there through engineering (antennas and their placement, receiver path, desense, coexistence, link adaptation and codecs) within each market's certified transmit power (T1-045, T1-075). Thresholds: experience budgets, "Radio". Level Q2 (hard limits), Q3 (warnings), Q4 (ideal).

Priority is the maintainers' call; the aim of 2026-10-24 reads as **must**. T1-048's list of experience metrics (start-up, responsiveness, library scale, audio output, power, endurance) would gain radio; the "Radio" section of the experience budgets is already written for it.

## 8. Board changes recommended (not edited here)

| Item | Change | Owner |
|---|---|---|
| ADR 0018 decision 2 | Reassess: dedicated configuration as the target, decided by prototype measurement (§5.1); shared as the fallback | Board (new ADR or amendment) |
| ADR 0018 decision 5, PCB-R-07 | A second antenna zone (ANT1) in the main body frame with its keep-out; the measurement of 3, 4 and 5 mm keep-outs on the prototype | Board, case |
| BOM line 57 | Prototype: second antenna fitted on ANT1 (146153-0050 or W24P-U) | Board |
| BOM line 56 | ANT1 U.FL fitted on prototype (already planned) and, if §5.1 confirms, in production | Board |
| Radio block (feature-blocks spec) | Draw the radio block's interface so an alternate 2×2 module (Sona MT320: 20 × 14 mm, two RF ports, SDIO 3.0 for Wi-Fi and Bluetooth) fits behind it, or record that it does not (RFC 0011 rule 6) | Board |
| Verification rows | V2-R01, V4-02, V5-01 get numbers from §5.4, §5.6 and §6; add RM-B1 to RM-B3 at L2/L4 and RM-O1 to RM-O5 at L5 | t1 (main session) |
| Board requirements | A desense budget per radio band (§5.6) | Board |
| hat-standard | The antenna keep-out covers both antenna zones (T1-085) | hat-standard |

## 9. Open points

1. Molex 146153 efficiency in an enclosure and its application specification (clearances): not obtained (also open in the board study).
2. Whether the IW612 time-shares Bluetooth with 5 GHz Wi-Fi in the shared configuration, and what coexistence controls mainline `nxpwifi` and `btnxpuart` expose.
3. Whether Bluetooth AFH satisfies EN 300 328 V2.2.2 adaptivity as tested, and the cost of OpenDAP's own EU conducted reports.
4. MAYA-W271 Bluetooth power and sensitivity; MT7902 module availability; Infineon CYW5557x upstream status.
5. The benchmark set's models, bought and measured at Q1–Q2.

## Sources

Shared sources are listed in the [method](https://github.com/opendap-project/toolkit-hardware/blob/main/docs/research/2026-10-radio-performance-methods.md#9-sources). Specific to this note:

- Murata CER-0248 Rev. 6.0 and E2B-09-1413 Rev. 9.0 (hashes in `parts-lib/manifest.toml`, LBES5PL2EL-923).
- Ezurio, *Sona MT320 (M.2 1420 and M.2 2230) datasheet* v1.3, <https://ezurio.com/documentation/datasheet-sona-mt320-m-2-1420-and-m-2-2230>, sha256 `6271560926c84763ab894cc53ca324989976206f208a0b1d134dd75230ff3424`; [Sona MT320 in stock](https://www.ezurio.com/resources/blog/now-in-stock-sona-mt320-wi-fi-6-bluetooth-5-4-module), Ezurio, 2024-05-23.
- Mainline Linux, `torvalds/linux` master read 2026-10-24: `drivers/net/wireless/nxp/nxpwifi/` (first commit 2026-07-15), `drivers/bluetooth/btnxpuart.c` (2023-04-24), `drivers/net/wireless/mediatek/mt76/mt7921/sdio.c` (2021-10-20; MT7902 2026-03-24), `drivers/bluetooth/btmtksdio.c` (MT7902 2026-04-10), `drivers/net/wireless/realtek/rtw89/` (no `sdio.c`), `drivers/net/wireless/broadcom/brcm80211/brcmfmac/sdio.c` (no CYW5557x), `drivers/net/wireless/marvell/mwifiex/sdio.c`, `include/linux/mmc/sdio_ids.h`.
- `linux-firmware`, <https://gitlab.com/kernel-firmware/linux-firmware>, main read 2026-10-24.
- [Murata Type 2EA announcement](https://www.murata.com/en-us/news/connectivitymodule/wi-fi-bluetooth/2023/0724), 2023-07-24 (size, vendor claim); [NXP IW610](https://www.nxp.com/products/IW610) (Bluetooth LE only).
