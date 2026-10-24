# Research: design for compliance of the T1

- **Date:** 2026-10-24
- **Status:** Desk research. Engineering input only, not legal advice and not a decision. Nothing here has been tested. Task E9.4 of the [electronics plan](../plans/2026-10-electronics-plan.md). The proposals in §5 are for the main session to number and integrate; this note edits no requirement, specification or plan.
- **Question (maintainers, 2026-10-24):** what the T1 would need to pass certification completely if it becomes a formal product. This note covers the engineering side: what each regime requires of the hardware and software, the test levels and the margins experienced teams keep in pre-compliance, the documents a technical file needs, and what the T1 already does, misses or contradicts.
- **Product as assessed:** a portable Digital Audio Player running Linux, with Wi-Fi 6 and Bluetooth 5.4 on a pre-certified Murata Type 2EL module (IEEE 802.15.4 present in the module, not wired), an NFC dynamic tag and an optional NFC reader, a user-replaceable 2S Li-ion pack, USB-C with USB PD charging and sourcing, a 3.5 mm single-ended and a 4.4 mm balanced headphone output, a mono speaker, the touch faceplate and the HAT interface ([BOM v1.7](https://github.com/opendap-project/board/blob/main/docs/specs/bom-v1.md)).
- **Quality levels:** every proposal carries the lowest [RFC 0010](https://github.com/opendap-project/rfcs/blob/main/text/0010-quality-levels.md) level at which it is mandatory (Q0 bench-safe, Q1 prototype, Q2 pre-compliance, Q3 compliant, Q4 reference). §6 gives the path through the levels.
- **Conventions:** a standard is cited by number, edition and clause where known. **UNVERIFIED** marks what was not checked against the primary text in this pass: an edition that may have been superseded in the Official Journal, a value from a secondary source, or a value from memory. A secondary source is named as such. Arithmetic is shown as such.
- **Inputs read:** [T1 requirements](../requirements/README.md), [electronics integration and verification](../specs/electronics-integration-verification.md), [board requirements](https://github.com/opendap-project/board/blob/main/docs/requirements/README.md), [PCB design constraints](https://github.com/opendap-project/board/blob/main/docs/specs/pcb-design-constraints.md), [BOM v1.7](https://github.com/opendap-project/board/blob/main/docs/specs/bom-v1.md) and its [power tree](https://github.com/opendap-project/board/blob/main/docs/specs/power-tree-v0.md), board ADRs [0012](https://github.com/opendap-project/board/blob/main/docs/adr/0012-headphone-output-stage.md), [0014](https://github.com/opendap-project/board/blob/main/docs/adr/0014-two-cell-battery.md) and [0015](https://github.com/opendap-project/board/blob/main/docs/adr/0015-amplifier-rails-from-vsys.md), the [Murata LBES5PL2EL-923 sheet](https://github.com/opendap-project/parts-lib/blob/main/Murata/LBES5PL2EL-923.md) (Murata CER-0248 Rev. 6.0 and E2B-09-1413 Rev. 9.0), [system requirements](https://github.com/opendap-project/system/blob/main/docs/requirements/README.md) (SYS-012, SYS-055, SYS-056), [Blackbird OS requirements](https://github.com/opendap-project/blackbird-os/blob/main/docs/requirements/README.md) (OS-020, OS-025, OS-030, OS-031).

## 0. Summary

1. **Hearing safety is the regime that touches the audiophile promise most.** At digital full scale, the ADR 0012 stage gives about 2.8 Vrms single-ended and 5.6 Vrms balanced (arithmetic, §2.6). The fixed classification route of IEC 62368-1 clause 10.6 caps a player sold without headphones at 150 mV (100 dB(A)), 25–31 dB below that. The **dose-based route** (EN 50332-3:2017, through EN IEC 62368-1:2020+A11:2020 clause 10.6) has no fixed ceiling: it warns when the momentary level passes 100 dB(A) and at every 100 % of the weekly dose, and falls back to the safe level only when the listener does not acknowledge. That route keeps T1-045 and T1-040 whole: the limit is documented, visible and owned by the listener, and the full hardware output stays one acknowledgement away. SYS-012 should name that route explicitly (§3, C1).
2. **The Murata module does not certify the T1 on its own.** Its FCC and ISED grants are for mobile use (≥ 20 cm from the body), so the handheld T1 needs SAR testing and a Class II (FCC) / Class 4 (ISED) permissive change through Murata. The Molex 146187 antenna in the BOM is on neither the FCC nor the ISED list, and its 43.3 mm cable is shorter than the ≥ 50 mm registered in Japan. Murata's RF copy rules (230–240 µm to the reference ground) do not fit the 50 µm HDI dielectrics without plane cut-outs. Murata's certified TX-power tables target NXP's out-of-tree driver; no loader was found in mainline `nxpwifi` (**UNVERIFIED**). In the EU, Murata supplies conducted reports only: radiated tests run on the T1 (§2.2, §2.3).
3. **RF exposure needs SAR.** Murata's 18 dBm Wi-Fi and 13 dBm Bluetooth conducted levels are far above the FCC portable-device exemption threshold at handheld distances (about 2.7 mW at 2.45 GHz and 1.4 mW at 5.5 GHz for 5 mm, arithmetic, §2.3), and above the EN 62479 low-power level, so EN 50566 and FCC 2.1093 SAR evaluation are needed.
4. **The NFC reader (PN7160) is a chip-down radio** with no pre-certification. It already sits as a population option (BRD-053); keeping it unfitted in a certified variant avoids a separate 13.56 MHz certification (§2.2, C5).
5. **Battery and power are mostly in place.** The 2S pack with PCM, keyed Micro-Fit connector, fuse, ship FET and user replacement (T1-055, ADR 0014) match IEC 62368-1 Annex M and the EU Battery Regulation's Article 11. What is missing: IEC 62133-2 for every pack that ships (ADR 0014 asks it for the Commercial edition only), the pack labelling and spare-pack specification, and an output limiter that keeps the HAT and USB-C source outputs at PS1 under a single fault (§2.4, §2.5).
6. **USB PD is mandatory in the EU for the T1** because it charges at 9 V (Directive 2022/2380 applies above 5 V). The T1 does PD; it needs the charging-power label and pictogram, and PD conformance pre-tests (§2.8).
7. **EMC and ESD** need fix-option footprints that can be fitted without a respin (BRD-053), pre-scans with a 6 dB margin, and ESD at the EN 55035 levels (±4 kV contact, ±8 kV air) on the product surface and the user-touchable HAT pogo field (§2.1, §2.10).
8. **Two product rules need a wording decision:** T1-045 allows limits "only for safety", but radio rules impose legal limits (TX power per country, DFS, indoor-only bands) that are not hearing or electrical safety (C2); and RFC 0010 places the DIY kit at Q2 while the regimes here apply to any product placed on the market, kit or not (C11).

## 1. Regimes at a glance

| Regime | Instrument and main standards | What it forces in the T1 | First level |
|---|---|---|---|
| EMC, unintentional emissions and immunity | FCC 47 CFR Part 15 Subpart B (Class B, 15.109); ICES-003 Issue 7; EN 55032:2015+A11:2020 Class B; EN 55035:2017+A11:2020; EN 301 489-1/-17 under RED (editions **UNVERIFIED**) | Shielding and filter options, clock spreading, cable and flat-cable control, tested configurations | Q2 |
| Radio, spectrum | FCC Part 15C (15.247), 15E (15.407), modular approval 15.212; RED 2014/53/EU Art. 3.2 with EN 300 328 V2.2.2, EN 301 893 V2.1.1, EN 300 330 (NFC); ISED RSS-247 Issue 3, RSS-Gen Issue 5; Japan Radio Act (MIC); Korea Radio Waves Act (KC) (editions **UNVERIFIED**) | Murata integration rules, listed antennas, regulatory domain handling in Linux, NFC reader variant | Q2 |
| RF exposure | EU 1999/519/EC limits via EN 62479:2010, EN 50663:2017, EN 50566:2017, EN IEC/IEEE 62209-1528:2021; FCC 47 CFR 1.1307(b)(3), 2.1093; ISED RSS-102 Issue 6 (**UNVERIFIED** edition) | SAR of the handheld; antenna placement; permissive change | Q2 (assessment), Q3 (test) |
| Radio cybersecurity | Delegated Regulation (EU) 2022/30 under RED Art. 3.3(d)(e)(f); EN 18031-1/-2/-3:2024 with the restrictions of Decision (EU) 2025/138 | No default credentials, secure updates, minimal exposed services, data deletion | Q3 |
| Product safety | IEC 62368-1:2018 (ed. 3), EU harmonised EN IEC 62368-1:2020+A11:2020; IEC 62368-1:2023 (ed. 4) exists, its EU citation **UNVERIFIED** | Energy source classes, Annex M batteries, touch temperatures, clause 10.6 acoustics | Q2 (pre-assessment), Q3 |
| Batteries | IEC 62133-2:2017 (+A1:2021); UN Manual of Tests and Criteria Part III 38.3; Regulation (EU) 2023/1542 Art. 11, 13, 17 | Certified pack, transport tests, user replacement with commercial tools, labelling, spare pack | Q2 (UN 38.3), Q3 |
| Hearing safety | IEC 62368-1 clause 10.6; EN 50332-1:2013, EN 50332-2:2013, EN 50332-3:2017 | Calibrated output levels, dose accounting, warnings and acknowledgement, safe default level | Q2 |
| Chemicals and environment | RoHS 2011/65/EU + (EU) 2015/863, EN IEC 63000:2018; REACH (EC) 1907/2006 Art. 33 and SCIP; WEEE 2012/19/EU, EN 50419; California Proposition 65 | Lead-free parts and assembly, declarations per BOM line, marking | Q2 (parts), Q3 (file) |
| USB and common charger | Directive (EU) 2022/2380 amending RED (Annex Ia), EN IEC 62680-1-3 (USB Type-C), EN IEC 62680-1-2 (USB PD); USB-IF compliance programme (voluntary) | USB-C receptacle, USB PD for 9 V charging, charging-power label and pictogram | Q3 |
| Wi-Fi Alliance | Voluntary certification | Only if the logo is used | Q4 |
| ESD | IEC 61000-4-2:2008 at the EN 55035 / EN 301 489-1 levels | Protection placement, discharge paths, no acoustic transient | Q2 |
| HAT ecosystem | Same regimes, per configuration | Tested configurations, host and accessory responsibilities, PS1 outputs | Q3 |

## 2. Regimes in detail

### 2.1 EMC: FCC Part 15B, ICES-003, EN 55032 and EN 55035

**What it requires.** The T1 is a Class B digital device (residential use).

- Radiated emissions from 30 MHz up to the frequency the highest internal clock sets: FCC 15.33 and CISPR 32 both extend the range to 6 GHz when internal frequencies exceed 1 GHz (the LPDDR4X and the 2.4/5 GHz radio qualify; 15.33 rule **UNVERIFIED** in detail).
- Conducted emissions apply to AC mains ports. A battery device charged over USB-C is tested on mains only through its charger, which is not part of the T1 when it ships without one (**UNVERIFIED** whether a lab tests with a representative charger in the EU).
- Immunity (EN 55035): ESD (§2.10), radiated RF immunity 80 MHz–6 GHz at 3 V/m, and the audio-specific criterion: the acoustic output during immunity tests is judged against a demodulated-interference limit for audio functions (EN 55035 Annex for audio; value **UNVERIFIED**).
- Radio equipment adds EN 301 489-1/-17 under RED, with the same ESD and RF immunity levels and radio-specific performance criteria.
- FCC: Supplier's Declaration of Conformity for the host, with the compliance statement in the manual or e-label (47 CFR 2.1077, 15.19, 15.105). ISED: ICES-003 compliance label.

**Margins experienced teams keep.** Pre-scans in a non-accredited setup carry several dB of uncertainty (antenna factors, site, cables), so a pre-scan peak at least 6 dB under the Class B limit is the usual go criterion (secondary, common practice; **UNVERIFIED** as a figure from a standard). Formal results with less than about 3 dB margin are treated as a production-spread risk.

**What the T1 does.**

- ESD arrays at every external connector (PCB-E-01), shielding-can frames around the SoC/DDR and the radio, DNP first (PCB-T-06), one unsplit ground (PCB-A-01), switching loops per datasheet (PCB-P-07).
- High-speed buses on controlled impedance with reference planes (PCB-S-04, PCB-I-*).

**What it misses.**

- **Fix options without a respin.** BRD-053 allows DNP parts. EMC fixes found at pre-scan usually need footprints that are already there: common-mode filter positions (0 Ω by default) on the USB 2 pair at the USB-C, on the faceplate DSI at the FH35C and on the HAT high-speed entry; ferrite positions on the HAT power lines and the faceplate supplies; shield-can frames around the switching converters near the radio and the headphone stage. Proposed as PCB-EMC-01 to -03 (§5.2).
- **Clock spreading.** Spread-spectrum on the DDR and display clocks is a software and device-tree setting where the RK3566 supports it (**UNVERIFIED**); its effect on audio is nil because the DAC is the clock master (BRD-023). It needs a measured on/off comparison (V4-07).
- **Front zone under the display has no shield cans** (PCB-Z-03, ADR 0016). The SoC decoupling sits there; radiated emission through the faceplate gap is a pre-scan item, not a known failure.
- **Tested configurations.** ANSI C63.4 and CISPR 32 test the equipment with its ports exercised by typical peripherals. For the T1 that means at least: no HAT, a representative HAT on the flat cable, charging over USB-C, USB device mode to a computer, headphones on each jack, Wi-Fi and Bluetooth active (§2.11).

### 2.2 Radio: FCC 15C/15E, RED, ISED, Japan, Korea

**Modular approval reuse (US, Canada).** 47 CFR 15.212 lets a host carry a module's grant if the module is used within its grant conditions: the listed antennas (type and gain no higher), the trace design for a limited approval, and the RF exposure conditions. The host then needs only the "Contains FCC ID: …" label, a Part 15B SDoC and a check of co-located transmitters (FCC KDB 996369 is the integration guidance, number **UNVERIFIED**). A change outside the grant is a permissive change filed by the grantee (Murata): Class I for an equivalent antenna of the same or lower gain, Class II for a new RF exposure condition (CER-0248 pp. 26–28, via the parts-lib sheet). ISED follows the same pattern with "Contains IC: …" and Class 4 changes (CER-0248 p. 35).

What this means for the T1, from Murata CER-0248 Rev. 6.0 and E2B-09-1413 Rev. 9.0 (parts-lib sheet):

| Condition of the Murata approval | T1 today | Gap |
|---|---|---|
| Mobile use, antenna ≥ 20 cm from the body (FCC, ISED) | Handheld; antennas within a few millimetres of the hand | SAR test of the T1 and a Class II / Class 4 permissive change through Murata (Bluetooth-only use is excepted for the FCC, CER-0248 §2.1.3.6) |
| Antennas from the FCC/ISED lists: Molex 146153, Molex 219611, Unictron WT32D1-KX, Inventek W24P-U, Murata trace antenna | Molex 146187-0050 (BOM line 57) | Not listed for the US or Canada: change to a listed antenna, or a Class I permissive change (same type, same or lower gain) |
| Japan: Molex 146187 with a cable of 50 mm or more | 146187-0050, cable 43.3 mm | Outside the Japan registration as specified |
| RF trace copied from Murata's u.FL layout: 50 Ω, 230–240 µm to the ground layer, 7–25 mm, width ±0.25 mm, passives in Murata's positions (0603), ≥ 0.8 mm keep-out under the U.FL | HDI stackup with about 50 µm dielectrics (PCB-S-05); 0402 matching parts (BOM line 58) | Plane cut-outs to reach a 230–240 µm reference; 0603 matching positions; proposed PCB-R-03 |
| Certified TX power per region through Murata's `txpower_*.bin` and `rutxpower_*.bin` files | Mainline `nxpwifi` (v7.3); no loader for those files found | **UNVERIFIED** that certified powers are applied; measure per region (V4-09); carry a patch if needed (OS-023) |

**802.15.4.** The module carries an IEEE 802.15.4 radio; the T1 does not wire its SPI (BOM line 55). It stays disabled in firmware, so it adds no co-location case. Whether Murata's grants cover it is **UNVERIFIED** and only matters if a later board wires it.

**NFC.**

- The NTAG I2C plus (BOM line 125) is a passive tag powered by the reader's field. Whether a passive tag in a host is radio equipment under RED is **UNVERIFIED**; it emits nothing by itself.
- The PN7160 reader (BOM line 127) transmits at 13.56 MHz and is chip-down. Fitted, it needs its own authorisation: FCC 15.225 (certification), EN 300 330 and EN 301 489-3 under RED, ISED RSS-210 (**UNVERIFIED** annex), and the Japanese and Korean equivalents. It is already a population option (BRD-053), so the certified variant can leave it unfitted (C5).

**EU (RED).** There is no modular grant. Murata provides conducted test reports (EN 300 328, EN 301 893, EN 300 440, EN 62311); radiated measurements and the declaration of conformity are the product maker's (CER-0248 §2.3, p. 59). Points that touch the T1's design and software:

- **Adaptivity and receiver category (EN 300 328 V2.2.2, EN 301 893 V2.1.1):** they are properties of the module firmware; the T1 must ship the firmware Murata tested with, or retest.
- **DFS and indoor-only bands at 5 GHz:** the 5150–5350 MHz band is indoor-only in the EU, and a device acting as an access point on DFS channels must detect radar. A general-purpose Linux device can be put in access-point mode, so Blackbird OS has to restrict access-point channels to what the module is certified for (C2). The indoor-only restriction is stated on the packaging and in the manual (RED Art. 10(10)).
- **Regulatory domain:** Linux `cfg80211` with the wireless regulatory database sets the allowed channels and power per country. The country is a setting of the device, not of the user's network, and must not be set to a less restrictive domain than where the device is.

**Canada.** RSS-247 Issue 3 for 2.4 and 5 GHz, RSS-Gen Issue 5 for labelling and receiver rules, ICES-003 for the host (**UNVERIFIED** editions). The bilingual RSS-Gen statement goes in the manual or e-label.

**Japan.** The Radio Act requires the technical conformity mark with the module's certification number, and the module used within its certification conditions (antenna and cable as registered). VCCI is voluntary. Whether the mark may be shown only on screen is **UNVERIFIED**.

**Korea.** KC under the Radio Waves Act: radio conformity, KC EMC (KS C 9832 and KS C 9835, the CISPR 32/35 equivalents) and the KC battery scheme. Whether a module's KC certificate removes the host's radio tests is **UNVERIFIED**; plan for host testing.

### 2.3 RF exposure

**FCC.** The T1 is a portable device (47 CFR 2.1093: used within 20 cm of the body). A portable device is exempt from SAR evaluation only below the threshold of 47 CFR 1.1307(b)(3)(i): 1 mW at any distance, or Pth from the formula of 1.1307(b)(3)(i)(B) (transcribed here, **UNVERIFIED** against the eCFR text in this pass):

- Pth = ERP20cm × (d / 20 cm)^x, with x = −log10(60 / (ERP20cm × √f)), f in GHz, ERP20cm = 3060 mW for 1.5–6 GHz, d ≥ 0.5 cm.
- At 2.45 GHz and d = 0.5 cm (arithmetic): √f = 1.565; x = −log10(60 / 4,790) = 1.90; Pth = 3060 × 0.025^1.90 ≈ **2.7 mW**.
- At 5.5 GHz and d = 0.5 cm (arithmetic): √f = 2.345; x = −log10(60 / 7,177) = 2.08; Pth = 3060 × 0.025^2.08 ≈ **1.4 mW**.

Murata measures 18 dBm (63 mW) for 802.11b and 13 dBm (20 mW) for Bluetooth, conducted, before antenna gain (E2B-09-1413 Tables 18 and 20). SAR testing is needed. Limits: 1.6 W/kg over 1 g for the body, 4 W/kg over 10 g for extremities (2.1093(d)). A T1 in a pocket during playback is a body-worn case; a T1 in the hand is an extremity case.

**EU.** EN 62479:2010 exempts equipment below a low-power level (20 mW average for the general public, **UNVERIFIED** value); the T1 is above it. EN 50663:2017 is the generic assessment and EN 50566:2017 the product standard for hand-held and body-mounted devices, with SAR measured per EN IEC/IEEE 62209-1528:2021. The basic restrictions of Council Recommendation 1999/519/EC are 2 W/kg over 10 g for head and trunk and 4 W/kg for limbs.

**Canada.** RSS-102 has its own exemption thresholds, different from the FCC's (**UNVERIFIED** values for Issue 6).

**Design levers.**

- Antenna position: at the edges or the top band, away from the surface the hand and the body touch, and away from HAT metal. The case and faceplate set the distance; it should be fixed with a SAR pre-assessment before the case freezes (PCB-R-05, V5-06).
- Simultaneous transmission: with dedicated antennas (ANT0 Wi-Fi, ANT1 Bluetooth), Wi-Fi and Bluetooth transmit at once; the SAR sum or a separation ratio is evaluated.
- Time-averaged power: SAR is assessed at the maximum time-averaged power the software allows. A per-band power back-off is a lever; its mainline support for this module is **UNVERIFIED**.

### 2.4 Product safety: IEC 62368-1

**Energy source classes (IEC 62368-1:2018).**

- **Electrical (clause 5):** every voltage in the T1 is ES1: USB PD at 9 V maximum (TPS2121 OV at 11.1–12.6 V, power tree), VSYS 6.0–8.8 V, amplifier rails ±7 V. No basic safeguard against electric shock is needed.
- **Fire (clause 6):** power sources are classed by the power available after 3 s into a worst-case load: PS1 ≤ 15 W, PS2 ≤ 100 W, PS3 above. The pack's PCM trips at 4.0–5.0 A (ADR 0014), about 34–42 W at 8.4 V (arithmetic), so the battery circuit is PS2 at most if the PCM is a reliable limiter (**UNVERIFIED** that the PCM qualifies; otherwise PS3). PS2 circuits need V-1 or better materials near potential ignition sources, or the clearances of clause 6.4. The PCB laminate is FR-4 V-0; evidence that the fab's laminate is UL-recognised is **UNVERIFIED**.
- **HAT and USB-C source outputs:** 5.1 V at the 1.66 A eFuse ceiling is 8.5 W, and 3.3 V at 1.15 A is 3.8 W (arithmetic, BOM line 44): PS1 while the eFuse works. Under a single fault (eFuse shorted), the TPS62903's high-side limit of up to 5.5 A gives about 28 W at 5.1 V (arithmetic, power tree §4.6): PS2. A HAT fed from a PS2 output needs fire-enclosure evaluation of its own; a PS1 output frees every HAT from it. A second, independent limiter per output (a fuse or a PTC sized to open within 3 s at the buck's limit) keeps the outputs PS1 under a single fault (PCB-P-11). This makes the HAT ecosystem much simpler to certify (§2.11).
- **Thermal (clause 9):** accessible surfaces stay within the TS1 limits of IEC 62368-1 Table 38, which depend on material and contact time; a surface held continuously for long periods is limited to about 43 °C for any material (**UNVERIFIED** value and row). The worst case is charging at 15 W with full-scale headphone output, a HAT drawing its full budget and Wi-Fi transmitting (V5-08).
- **Acoustics (clause 10.6):** §2.6.
- **Coin cells:** none in the BOM, so the coin-cell rules (IEC 62368-1 4.8; US 16 CFR 1263) do not apply. Keep it so.

**Annex M, batteries.** Charging stays inside the cell's specified voltage, current and temperature window, also under a single fault (M.4, **UNVERIFIED** clause numbers in ed. 3). The T1's safeguards:

- pack PCM (over-charge, over-discharge, over-current, short circuit, balancing);
- charger thermal regulation and TS window on the pack NTC;
- the 5 A board fuse;
- the gauge's per-cell monitoring through the mid-cell tap;
- the keyed Micro-Fit connector, which prevents reverse insertion and fitting a 1S pack (ADR 0014).

ADR 0014 item 6 leaves the BQ25798 watchdog disabled and relies on safeguards that work without the OS. The hazard analysis has to state that software is not a safeguard, and show that a corrupted charger register (a single fault) is caught by the PCM; enabling the watchdog so the charger returns to its pin-set defaults if the OS stops is a Q4 option (C9). User replacement of the pack adds the IEC 62368-1 marking for replaceable batteries (risk of fire if replaced by an incorrect type) and the requirement that the replacement is safe for an ordinary person.

**Documentation.** A CB test report and certificate (IECEE CB scheme) is the usual evidence, reusable across markets; the hazard-based safeguard analysis (energy sources, safeguards, single faults) goes in the technical file.

### 2.5 Batteries

| Item | Requirement | T1 today | Gap |
|---|---|---|---|
| IEC 62133-2:2017 (+A1:2021) | Cell and pack safety tests | ADR 0014: required for the Commercial edition | Every pack that ships is the same part, so require it for every pack (C7) |
| UN 38.3 | Transport tests, test summary available down the supply chain; air transport per IATA PI 966/967 | ADR 0014: UN 38.3 report required | None; Q2 because pilot units and kits ship |
| Regulation (EU) 2023/1542 Art. 11 | From 18 Feb 2027: portable batteries removable and replaceable by the end user with commercially available tools (or tools supplied free), no heat or solvents, instructions provided, battery available as a spare part for the period the guidelines set (Commission guidelines C/2025/214; details **UNVERIFIED**) | T1-055 (common tools, standard part); Micro-Fit connector; no glue in the plan | Pack specification published so any maker can build it, and Blackbird OS accepts any pack meeting it (OS-031); replacement procedure in the manual; V5-09 |
| Art. 13 labelling | General information label (from 18 Aug 2026), capacity label, crossed-out bin, QR code (from 18 Feb 2027) (**UNVERIFIED** dates for each item) | Not specified | Pack label specification (C7) |
| CE marking of the battery | A pack placed on the market as a spare is itself a product under the Regulation (DoC, technical documentation) (**UNVERIFIED** module) | Not specified | Spare pack as its own conformity item |
| Battery passport (Art. 77) | Light means of transport, industrial over 2 kWh and EV batteries only | Portable battery | Not applicable |
| Japan | J62133-2 for the battery under the electrical appliance safety law (PSE) (**UNVERIFIED** category for a pack sold inside a host vs as a spare) | — | Q3 if Japan is declared |

### 2.6 Hearing safety: IEC 62368-1 clause 10.6, EN 50332

**Scope.** Clause 10.6 covers personal music players: portable, intended for ordinary persons, mainly for listening with headphones or earphones. The T1 is one. The speaker output is out of scope. A line-level output that is not intended for headphones (the HAT line out, BOM line 69) is outside it as well (**UNVERIFIED** reading).

**Two routes.**

1. **Classification route** (IEC 62368-1 10.6.2, EN 50332-1 and EN 50332-2):
   - by default the output stays at RS1: 85 dB(A), or 27 mV wide-band voltage for a player sold without headphones;
   - the listener may go up to RS2 (100 dB(A), 150 mV) after acknowledging a warning;
   - the warning returns after about 20 h of listening above RS1;
   - RS2 is a ceiling.

   Values from secondary sources (150 mV corroborated by a manufacturer's user guide; 27 mV and the 20 h re-warning **UNVERIFIED** against the standard text). Sony's EU and Korean players and Android's safe-media-volume behave this way (secondary, [community mods research](ux/2026-10-community-mods-customisation.md)).
2. **Dose-based route** (IEC 62368-1:2018 10.6.3 and EN 50332-3:2017; the EU harmonised EN IEC 62368-1:2020+A11:2020 amends clauses 3.3.19 and 10.6 with the momentary exposure level and the calculated sound dose, secondary: standards catalogue listing):
   - the player integrates the momentary exposure level (MEL) into a calculated sound dose (CSD) over a rolling 7-day window, where 100 % corresponds to 80 dB(A) for 40 h a week;
   - it warns at 100 % CSD and at every further 100 %;
   - it warns each time the MEL exceeds RS2 (100 dB(A));
   - if the listener does not acknowledge, it lowers the level to RS1.

   Android's sound-dose framework implements this route and states those rules: warning on each 100 % of CSD, reduction to RS1 if unacknowledged, a momentary warning above RS2 = 100 dB(A), and the rolling 7-day window (secondary: [AOSP sound dose](https://source.android.com/docs/core/audio/sound-dose)). Whether the dose route alone is enough in the EU, with no RS2 ceiling, is **UNVERIFIED** against the A11 text and is the first question for the test lab.

**The T1's output against the limits** (arithmetic from BOM line 37 and ADR 0012):

| Output | Full scale (digital 0 dBFS) | Into 32 Ω (ADR 0012 estimate) | Above 150 mV (RS2) | Above 27 mV (RS1) |
|---|---|---|---|---|
| 3.5 mm single-ended | 1.73 Vrms × 1.62 = 2.80 Vrms | 240 mW → √(0.24 × 32) = 2.77 Vrms | 20·log10(2.80 / 0.15) = 25.4 dB | 40.3 dB |
| 4.4 mm balanced | 2 × 2.80 = 5.6 Vrms (unloaded) | 640 mW → 4.53 Vrms | 20·log10(5.6 / 0.15) = 31.4 dB | 46.3 dB |

EN 50332-2 measures with programme-simulation noise (IEC 60268-1), whose level at full scale is lower than a sine's, so the measured wide-band voltage is below the sine figures above (by how much is **UNVERIFIED** until V4-10). The gap remains 20 dB or more.

**How compliant players handle it** (secondary, observed practice): a default safe level; a warning dialog the listener acknowledges to go above it; the warning re-armed after cumulative listening; in some EU and Korean variants, a lower maximum. A permanent lower maximum would break T1-045 and T1-040; the warning-and-acknowledgement pattern does not.

**Resolution that keeps the product rules** (proposed, C1):

- Use the dose-based route on both analog headphone outputs.
- At power-on the level is at or below RS1 (T1-050 already asks for a safe level after a power cycle).
- MEL and CSD are computed per output from the DAC volume, the gain and the headphone profile (SYS-012); an unknown headphone takes the EN 50332-2 reference sensitivity.
- Warnings come at MEL > RS2 and at each 100 % CSD; acknowledgement gives the full hardware output.
- With no acknowledgement, the level lowers gradually to RS1 (UX-019 asks for an announced, gradual reduction).
- The dose, the thresholds and the reason for every reduction are visible (SYS-038).
- The metering is passive: it reads the signal and the volume and never changes samples. The only action is the DAC volume. Bit-perfect playback (OS-020) and USB DAC mode (T1-054) are untouched, and the dose runs in USB DAC mode too.
- The safety function lives in open software that owners can modify. The technical file states that the shipped configuration is the one assessed.

**Hardware consequences.**

- The amplifier EN defaults to disabled at reset and the DAC powers up muted, so a crashed or absent OS can never drive a loud output (PCB-A-13).
- No ESD or power event may produce an acoustic transient at the jack (V5-05).
- Bluetooth output to wireless headphones: the dose then belongs to the listening device's own compliance (**UNVERIFIED**); the T1 can still meter it.

### 2.7 Chemicals and environment

- **RoHS** (2011/65/EU, Annex II as amended by (EU) 2015/863): 10 restricted substances, 0.1 % by weight per homogeneous material (cadmium 0.01 %). Exemptions such as high-melting solder or lead in glass and ceramic parts have expiry dates (**UNVERIFIED** status per exemption). Technical documentation per EN IEC 63000:2018: supplier declarations or test reports per part. For the T1: RoHS-compliant parts, lead-free assembly and a lead-free finish (ENIG, PCB-S-07) on every build that leaves the lab (Q2); a declaration per BOM line in the technical file (Q3). The BOM gains a compliance field per line (PCB-T-07).
- **REACH:** substances of very high concern above 0.1 % w/w per article trigger the Art. 33 information duty and a SCIP notification (Waste Framework Directive 2008/98/EC Art. 9(1)(i)). Typical carriers: cable insulation, rubber parts, gaskets. The Annex XVII restrictions to check are PAHs in rubber or plastic parts in skin contact (entry 50: buttons, gaskets) and nickel release from metal parts in prolonged skin contact (entry 27) (**UNVERIFIED** entry numbers).
- **WEEE** (2012/19/EU): crossed-out wheeled-bin mark per EN 50419 on the product, and on the pack under the Battery Regulation.
- **California Proposition 65:** a warning is needed if a listed chemical is present above the safe harbour (for example lead in brass jack contacts or DEHP in cables); suppliers' declarations decide it (**UNVERIFIED** per part).

### 2.8 USB-C, USB PD and the EU common charger

**Directive (EU) 2022/2380** (amending RED, applicable since 28 Dec 2024 for the categories listed, which include portable audio and handheld devices; scope reading **UNVERIFIED** for a DAP):

- a USB Type-C receptacle per EN IEC 62680-1-3 (USB4105, BOM line 61);
- if charging needs more than 5 V, more than 3 A or more than 15 W, USB PD per EN IEC 62680-1-2, and full charging speed with any compliant charger. The T1 charges as a PD sink at 9 V (power tree), so **USB PD is mandatory**. It has it (FUSB302B with Linux TCPM);
- the product offered without a charger, with the pictogram showing whether a charger is included;
- a label with the minimum and maximum charging power the device needs, and "USB PD" when it supports it. For the T1 the maximum is 15 W (input cap, power tree); the minimum is the power that still charges at full speed, to be measured.

**Engineering points.**

- The PD sink is negotiated by Linux. With the OS not running (dead battery, early boot), the T1 must still charge from a 5 V Type-C source; the dead-battery Rd (TPD8S300, BOM line 63) and the BQ25798 buck-boost from 5 V allow it (V4-13).
- **Source role:** the T1 sources 5 V at 1.5 A on battery; the Type-C source rules (vSafe0V before attach, Rp advertisement, no back-feed through the TPS2121) are pre-tested.
- **USB-IF certification and logos** are voluntary. Certification requires the USB-IF compliance tests (Type-C functional, PD) at an authorised lab; the logos may be used only after it. Plain technical names in text are a separate question (**UNVERIFIED** USB-IF trademark rules).

### 2.9 Radio cybersecurity: RED delegated act and EN 18031

Delegated Regulation (EU) 2022/30 applies since 1 Aug 2025 to radio equipment that can communicate over the internet. EN 18031-1 covers network protection (Art. 3.3(d)) and EN 18031-2 personal data (Art. 3.3(e)). EN 18031-3 covers monetary transactions (Art. 3.3(f)) and becomes relevant if T1-067's on-device purchase is ever enabled. Decision (EU) 2025/138 cites them with restrictions: a design that allows no password at all cannot rely on the default-password clauses, and the presumption of conformity is lost for those points.

| EN 18031 theme | T1 today | Gap |
|---|---|---|
| No default or shared credentials | SYS-056 (no default credentials; services off until enabled) | Developer mode (SYS-031) and any SSH must follow the same rule |
| Minimal exposed services | SYS-055 (nothing reachable from the internet by default) | Port audit on the release image (V6-05) |
| Secure update | OS-030 (staged, deferrable updates) | Signed updates and the update chain described in the technical file; OS-025 owner keys keep it compatible with GPLv3 |
| Key storage | OS-032 keystore, BRD-059 TPM | — |
| Logging, data deletion | SYS-050 (recoverable deletion) | Factory reset that deletes personal data; security event logging documented |
| Personal data | SYS-055 (data stays local) | Asset and threat inventory in the technical file |

The UK PSTI regime and the EU Cyber Resilience Act cover the same ground with their own documents (support period, vulnerability contact); T1-070 already asks for a published support commitment.

### 2.10 ESD: IEC 61000-4-2

- **Levels.** EN 55035 and EN 301 489-1 test the enclosure port at ±4 kV contact and ±8 kV air (IEC 61000-4-2:2008 levels 2 and 3), performance criterion B: temporary degradation, self-recovery, no loss of stored data (levels **UNVERIFIED** against the current editions). Experienced teams pre-test handhelds at IEC 61000-4-2 level 4 (±8 kV contact, ±15 kV air) for margin (Q4).
- **Discharge points:** USB-C shell, jack sleeves, button caps, the faceplate cover-glass gap, case seams, the microSD slot, and the HAT pogo field, which is user-touchable when no HAT is fitted (BOM line 68 places ESD there).
- **T1 today:** ESD arrays per connector (PCB-E-01), VBUS TVS (PCB-E-02), button ESD (BOM line 74), headphone ESD after the OPA1622 feedback node (BOM line 36), V4-05 and V5-02.
- **Missing:**
  - The discharge current path: in a plastic case, connector shells and ESD diode returns need a short, low-inductance path to the ground plane (PCB-E-04).
  - A stated level for V4-05 and V5-02.
  - The audio criterion: an ESD event must not produce a loud transient at the headphones (acoustic shock) (V5-05).

### 2.11 The HAT ecosystem

- **Each HAT is its own product.** It carries its own declaration, tests and marks. A radio HAT is its own radio equipment, and the T1's grants never cover it.
- **The T1 is certified in declared configurations.** The technical file lists the configurations tested: without a HAT, and with a representative HAT exercising the flat cable and pogo field (USB 3, PCIe and power at its class). EMC and SAR are tested with the worst-case representative HAT (metal near the antennas). A HAT that is not in the list does not void the T1's own compliance. It does make the combination the user's assembly, and the HAT maker carries the HAT's compliance in that combination.
- **Host responsibilities** (proposed for the [hat-standard](https://github.com/opendap-project/hat-standard)):
  - PS1 outputs under a single fault (PCB-P-11), so HATs need no fire enclosure for power drawn from the T1;
  - ESD at the pogo field;
  - the HAT ID, so the software knows what is attached and can apply a HAT's declared limits;
  - published EMC conditions for HAT makers (emission budget at the flat cable, shielding of high-speed lines).
- **Accessory responsibilities:**
  - a HAT that sources power to the T1 (the powerbank HAT on TPS2121 IN2) meets the T1's input limits;
  - a HAT with a battery carries its own battery compliance (§2.5);
  - a radio HAT carries its own radio and RF-exposure compliance;
  - a HAT keeps the T1's antenna keep-out.

## 3. Conflicts and gaps against current decisions

| # | Current decision | Conflict or gap | Proposed resolution, within the product rules |
|---|---|---|---|
| C1 | ADR 0012 and ADR 0015 (full output, no ceiling); SYS-012 (weekly dose, "85 dB(A) limit"); T1-045, T1-040 | The classification route would cap the output at 150 mV, 25–31 dB below full scale; SYS-012's wording reads as a fixed 85 dB(A) limit | Dose-based route (§2.6): RS1 default at power-on, warnings at MEL > RS2 and each 100 % CSD, acknowledgement unlocks full hardware output, gradual return to RS1 if unacknowledged, all visible. Reword SYS-012 to name EN 50332-3 and the dose route; confirm with the lab that A11 accepts it without an RS2 ceiling |
| C2 | T1-045: limits exist only for safety | Radio law imposes limits that are not safety in the T1-045 sense: TX power per country, DFS, indoor-only channels, access-point channels | Amend T1-045 to "only for safety or by law (spectrum rules), documented and visible"; the regulatory domain is shown in the UI (V6-06) |
| C3 | BRD-025, BOM lines 55–58 (pre-certified module, Molex 146187-0050, 0402 matching) | Mobile-only grants; antenna not listed for the US and Canada; cable short for Japan; matching parts and stackup differ from Murata's copy rules | Listed antenna (Molex 146153 or 219611, fit to check) or a Class I permissive change; SAR and Class II / Class 4 change through Murata; RF trace with plane cut-outs to 230–240 µm and 0603 matching positions (PCB-R-03, PCB-R-04) |
| C4 | Mainline `nxpwifi` (BOM line 55) | No loader for Murata's certified per-country TX-power tables found (**UNVERIFIED**) | Measure conducted power per band and domain with the shipped driver (V4-09); if it exceeds the certified values, carry a patch (OS-023) or apply limits through the regulatory database |
| C5 | BOM line 127, PN7160 reader fitted on prototypes | Chip-down 13.56 MHz transmitter with no pre-certification | Leave it unfitted in the certified variant until a certification is planned; the NTAG tag keeps tap-to-pair in every variant |
| C6 | 802.15.4 in the module | Certification and co-location scope | Keep it disabled in firmware while the SPI is not wired |
| C7 | ADR 0014 item 2 (IEC 62133-2 for the Commercial edition); T1-055 (a standard part); T1-038 (quoted pack) | The same pack ships in both editions; a quoted pack is not a catalog part; Battery Regulation labelling and spare-pack duties | IEC 62133-2 and UN 38.3 for every pack; a published pack specification (cells, PCM, connector, label) that any maker can build to, accepted by the OS (OS-031); pack label per Art. 13 |
| C8 | Power tree §4.6 (HAT and USB-C outputs limited by eFuses) | Under a single fault the outputs reach PS2 (about 28 W) | Second independent limiter per output (PCB-P-11), verified in V4-12 |
| C9 | ADR 0014 item 6 (charger watchdog disabled) | Annex M single-fault analysis must not count on software | Hazard analysis shows PCM and charger hardware limits as the safeguards; enabling the watchdog is a Q4 option |
| C10 | PCB-E-01, V4-05, V5-02 | No ESD levels stated; no acoustic criterion; no discharge-path rule | PCB-E-04; V4-05 and V5-02 amended to EN 55035 levels; V5-05 audio criterion |
| C11 | RFC 0010 (the DIY kit at Q2) | The regimes apply to a product placed on the market, kits included (47 CFR 15.23 does not exempt marketed kits; the EU treats a kit as a product) | The quality levels set engineering gates; the market status of each edition is a maintainers' decision. Until it is taken, a kit placed on a market is assumed to need that market's Q3 items |
| C12 | T1-067 (on-device purchase, opt-in) | Brings EN 18031-3 into scope when enabled | Keep it off by default and assess -3 before it ships |

## 4. Technical file and declarations (Q3)

The documentation a formal product needs, from the regimes above:

- **General description and variants:** models, assembly variants (BRD-053), declared configurations (faceplates, HATs), software version assessed.
- **Standards applied**, with editions.
- **Test reports:**
  - EMC, radio (conducted from Murata plus radiated from the T1), SAR;
  - IEC 62368-1 CB with Annex M and clause 10.6;
  - EN 50332 measurements per output;
  - EN 18031 assessment;
  - USB PD pre-tests;
  - pack IEC 62133-2 and UN 38.3.
- **Risk analyses:** IEC 62368-1 energy sources and safeguards; hearing-safety function; cybersecurity assets and threats.
- **Declarations:**
  - EU declaration of conformity under RED (with RoHS and the Battery Regulation for the pack);
  - FCC SDoC for the host, with the module's FCC ID on the label;
  - ISED and ICES-003 statements;
  - Japan and Korea marks where declared.
- **Labels and e-label:** FCC ID, IC, Japanese mark and number; CE; WEEE bin; battery marks; charging-power label and pictogram. A regulatory-information screen on the device serves the e-label where each authority allows it (**UNVERIFIED** per authority).
- **User information:** safety and hearing warnings, battery replacement procedure, RF exposure statement, the 5 GHz indoor-only restriction.
- **Supplier declarations:** RoHS, REACH and Proposition 65 per BOM line; the module's grants and reports; the pack's reports.

## 5. Proposed requirements, constraints and verification rows

IDs are placeholders for the main session to assign. Level = the lowest RFC 0010 level at which the item is mandatory.

### 5.1 Product requirements (t1, and system where noted)

| Proposal | Text | Priority | Level | Regime |
|---|---|---|---|---|
| P-T1-a | The T1 shall meet every mandatory regime of each market it is declared for, in the configurations listed in its technical file (faceplates, representative HATs, charging, USB device mode). | must | Q3 | All |
| P-T1-b | Both analog headphone outputs shall implement sound-dose management per EN 50332-3 through IEC 62368-1 clause 10.6: power-on level at or below RS1; warnings when the momentary level exceeds RS2 and at every 100 % of the 7-day dose; acknowledgement gives the full hardware output; without acknowledgement the level lowers gradually to RS1; dose, thresholds and every reduction visible. Amends SYS-012. | must | Q2 | Hearing safety |
| P-T1-c | Dose metering shall be passive: it never changes samples, and the only action it takes is the DAC volume, so bit-perfect playback (OS-020) and USB DAC mode (T1-054) are unchanged; the dose also runs in USB DAC mode. | must | Q1 | Hearing safety |
| P-T1-d | The maximum wide-band output voltage of each headphone output, at each gain and into 16 Ω, 32 Ω, 300 Ω and open circuit, shall be measured per EN 50332-2 and published. | must | Q2 | Hearing safety |
| P-T1-e | T1-045 amended: limits exist only for safety or because spectrum law requires them (radio power per country, DFS, indoor-only bands), and they are documented and visible. | must | Q2 | Radio |
| P-T1-f | The device shall apply the radio regulatory domain of the country it is in, with the certified TX power per band; access-point mode is offered only on channels the module is certified for without radar detection. | must | Q2 | Radio |
| P-T1-g | The handheld T1 shall pass RF exposure evaluation as a portable device (FCC 2.1093, EN 50566, RSS-102), body-worn and extremity, with simultaneous Wi-Fi and Bluetooth transmission and the representative HAT. | must | Q3 | RF exposure |
| P-T1-h | The certified variant shall contain no transmitter without a grant for the declared markets; the PN7160 reader stays unfitted until it has one, and 802.15.4 stays disabled while it is not wired. | must | Q3 | Radio |
| P-T1-i | The T1 shall meet EN 18031-1 and EN 18031-2, and EN 18031-3 before any payment function is enabled (T1-067). | must | Q3 | Cybersecurity |
| P-T1-j | Every battery pack shipped, in any edition, shall hold a UN 38.3 test summary (Q2) and an IEC 62133-2 report (Q3); the pack specification (cells, PCM, connector, label) shall be published so any maker can build a replacement. | must | Q2 / Q3 | Batteries |
| P-T1-k | The pack and the product shall carry the labels of Regulation (EU) 2023/1542 Art. 13 and the WEEE mark; the battery replacement procedure shall use commercially available tools only. | must | Q3 | Batteries |
| P-T1-l | The T1 shall carry the charging-power label ("min–15 W, USB PD" with the measured minimum) and the charger pictogram of Directive 2022/2380, and charge at full speed from any compliant USB PD charger and at reduced speed from any 5 V Type-C source, with or without the OS running. | must | Q3 | USB, common charger |
| P-T1-m | Every build that leaves the lab shall use RoHS-compliant parts and lead-free assembly; the technical file holds a RoHS, REACH and Proposition 65 declaration per BOM line. | must | Q2 / Q3 | Chemicals |
| P-T1-n | Accessible surfaces shall stay within IEC 62368-1 TS1 touch-temperature limits in the worst case: 15 W charging, full-scale headphone output, a HAT at its full budget and Wi-Fi transmitting. | must | Q2 | Safety |
| P-T1-o | A regulatory-information screen shall show the FCC ID, IC, Japanese certification number and other marks the authorities accept as an e-label. | should | Q3 | Labelling |
| P-T1-p | The HAT standard shall state host and accessory compliance responsibilities: PS1 host outputs, ESD at the pogo field, the EMC emission budget at the flat cable, the antenna keep-out, and that each HAT carries its own declaration. | must | Q3 | HAT ecosystem |
| P-T1-q | The hazard analysis shall show that no safety function depends on software alone (battery charging, acoustic output at power-on). | must | Q2 | Safety, hearing |

### 5.2 Board constraints (board, PCB design constraints)

| Proposal | Constraint | Level | Source |
|---|---|---|---|
| PCB-R-03 | Murata RF copy rules for each RF pin to its U.FL: 50 Ω microstrip over a solid ground 230–240 µm below (inner-plane cut-outs in the HDI stack), width ±0.25 mm, length 7–25 mm, matching positions where Murata places them and in 0603, ≥ 0.8 mm keep-out under the U.FL | Q1 | E2B-09-1413 §3.5.1, Figure 13; CER-0248 §2.1.6.2 |
| PCB-R-04 | Antennas from the FCC, ISED and Japan lists of the module's certification, with the listed cable length, or a Class I permissive change agreed before layout freeze | Q2 | CER-0248 Tables 20, 36, 56 |
| PCB-R-05 | Antenna feeds and radiators at the board edges or top band, away from the body-facing surface and from HAT metal; distance fixed by the SAR pre-assessment | Q2 | §2.3 |
| PCB-EMC-01 | Common-mode filter positions (0 Ω fitted by default) on the USB-C USB 2 pair, the faceplate DSI and the HAT high-speed entry, placed per the signal-integrity rules of each class | Q1 | §2.1, BRD-053 |
| PCB-EMC-02 | Ferrite positions on HAT_5V, HAT_3V3 and the faceplate supplies at their connectors | Q1 | §2.1 |
| PCB-EMC-03 | Shield-can frame footprints (DNP first) around the switching converters near the radio and the headphone stage, in addition to PCB-T-06 | Q1 | §2.1 |
| PCB-E-04 | Each connector shell and each ESD diode return reaches the ground plane through at least two vias directly at the pad; no ESD return shares a path with an audio ground return | Q1 | §2.10 |
| PCB-P-11 | HAT_5V, HAT_3V3 and the USB-C source output each have a second, independent limiter (fuse or PTC) so the available power after 3 s stays ≤ 15 W with the eFuse shorted | Q3 | IEC 62368-1 clause 6 (PS1); §2.4 |
| PCB-A-13 | Amplifier EN and the DAC reset default to off at power-up and reset (hardware pull), so no output is driven before the OS sets a level | Q1 | §2.6, T1-050 |
| PCB-S-09 | Laminate and solder mask flammability V-0, with recognition evidence from the fab | Q3 | IEC 62368-1 clause 6 |
| PCB-T-07 | Every BOM line carries RoHS and REACH status and a supplier declaration reference | Q2 | §2.7 |

### 5.3 Verification rows (electronics integration and verification, L4–L6)

| Proposal | Level (integration) | Quality level | Case | Expected |
|---|---|---|---|---|
| V4-05 (amend) | L4 | Q2 | ESD at every connector on the bare board | ±4 kV contact, ±8 kV air (IEC 61000-4-2): no damage, criterion B |
| V4-07 | L4 | Q2 | Near-field scan and radiated pre-scan, every domain active; spread-spectrum on and off; harmonics of every switching frequency listed against the radio bands | Peaks recorded; no harmonic in a radio channel above the desense budget of V4-02 |
| V4-09 | L4 | Q2 | Conducted TX power per band, rate and regulatory domain with the shipped driver and firmware | At or below Murata's certified values for each domain |
| V4-10 | L4 | Q2 | Maximum wide-band output voltage per EN 50332-2 (programme-simulation noise, A-weighting) on each output, gain and load; 1 kHz full-scale sine for reference | Recorded; sets the dose calibration and P-T1-d |
| V4-11 | L4 | Q2 | Annex M pre-assessment: charger register misprogrammed, pack NTC open and shorted, cell imbalance, output short | Cell limits never exceeded; the PCM or charger hardware stops each fault |
| V4-12 | L4 | Q3 | HAT and USB-C source outputs with the eFuse shorted | Available power after 3 s ≤ 15 W (PS1) |
| V4-13 | L4 | Q2 | USB Type-C and PD functional pre-test with a PD analyser: 5 V and 9 V sink, 5 V 1.5 A source, dead battery, OS not running, hot plug, legacy and non-PD chargers | Charges in every case; no back-feed; contracts as specified |
| V5-02 (amend) | L5 | Q2 | ESD at the product surface | ±4 kV contact, ±8 kV air, criterion B |
| V5-04 | L5 | Q2 | Radiated emissions pre-scan in the case, 30 MHz–6 GHz, configurations: no HAT, representative HAT, charging, USB device mode, playback on both jacks, Wi-Fi and Bluetooth transmitting | ≥ 6 dB below FCC 15.109 / CISPR 32 Class B in every configuration |
| V5-05 | L5 | Q2 (±4/±8 kV), Q4 (±8/±15 kV) | Headphone output during ESD at every discharge point, with a sensitive IEM load | No transient above the RS2 equivalent (value to set with the lab) |
| V5-06 | L5 | Q2 | RF exposure pre-assessment: antenna-to-surface distances, time-averaged power, SAR scan or lab pre-test with and without the representative HAT | Below the FCC, EU and ISED limits with margin; fixes PCB-R-05 |
| V5-07 | L5 | Q2 | Radiated spurious emissions of the radio in the case (FCC 15.205/15.209 restricted bands, EN 300 328 and EN 301 893 radiated limits) | ≥ 6 dB margin |
| V5-08 | L5 | Q2 | Surface temperatures in the worst case of P-T1-n, 2 h, at 25 °C ambient | Within the IEC 62368-1 TS1 limits |
| V5-09 | L5 | Q3 | Battery replacement by a person with no instructions other than the manual, using commercially available tools | Completed without heat, solvents or special tools; time recorded |
| V6-03 (amend) | L6 | Q2 | Compliance pre-scans repeated on each release candidate in the V5-04 configurations | No regression beyond 2 dB from the last pass |
| V6-04 | L6 | Q2 | Sound-dose function: CSD against EN 50332-3 test signals; warnings at 100 % increments and MEL > RS2; gradual return to RS1 without acknowledgement; persistence across power-off, crash and battery cut-off (T1-051); every output and USB DAC mode | Matches the reference within the tolerance the standard sets |
| V6-05 | L6 | Q3 | EN 18031-1/-2 assessment: credentials, open-port audit, update chain, logging, factory reset deletes personal data | Every applicable requirement met |
| V6-06 | L6 | Q2 | Regulatory domain: country change applies channels, power and access-point restrictions; the domain is shown | As P-T1-f |
| V6-07 | L6 | Q3 | Formal tests at accredited labs for the declared markets (§4) | Passed; reports in the technical file |

## 6. Path through the levels

- **Q0, bench-safe.** Battery handled under its safety rules: bench supply or a protected 2S pack (ADR 0014 item 4); fuse, PCM and keyed connector in place. No compliance item is due.
- **Q1, prototype.** The design keeps every compliance option open without a respin:
  - Murata RF copy rules (PCB-R-03);
  - EMC fix footprints (PCB-EMC-01 to -03);
  - ESD discharge paths (PCB-E-04);
  - safe audio power-up (PCB-A-13);
  - passive dose metering (P-T1-c) in the audio architecture from the first software.
  - Prototypes may still carry the unlisted antenna and the PN7160.
- **Q2, pre-compliance** (pilot units, units that leave the lab):
  - pre-scans with ≥ 6 dB margin in every declared configuration (V5-04, V5-07);
  - ESD at ±4 kV contact and ±8 kV air with no acoustic transient;
  - acoustic output measured per EN 50332-2 and the dose function working (V4-10, V6-04);
  - TX power verified with the shipped driver (V4-09);
  - an antenna on the certification lists (PCB-R-04);
  - SAR pre-assessment (V5-06);
  - touch temperatures (V5-08);
  - every pack UN 38.3 (transport-safe);
  - RoHS parts and lead-free assembly;
  - the regulatory domain applied and shown;
  - T1-045 reworded (P-T1-e).
- **Q3, compliant** (the product, per declared market):
  - formal EMC, radio, SAR and safety tests (CB report with Annex M and clause 10.6);
  - Murata's permissive changes filed;
  - IEC 62133-2 for every pack, pack and product labels, published pack specification;
  - PS1 HAT outputs under single fault (PCB-P-11);
  - EN 18031 assessment;
  - common-charger label;
  - supplier declarations complete;
  - the technical file and the declarations of §4;
  - the HAT standard's responsibilities published (P-T1-p);
  - no unlicensed transmitter fitted (P-T1-h).
- **Q4, reference:**
  - ≥ 6 dB margin in the formal EMC and radiated spurious results, not only in pre-scans;
  - ESD at IEC 61000-4-2 level 4 (±8 kV contact, ±15 kV air);
  - the charger watchdog returning to hardware defaults if the OS stops;
  - USB-IF certification of the PD implementation;
  - SAR well below the limits through placement rather than power back-off;
  - a certified PN7160 variant if the hand-off feature needs it.

## 7. Open items

1. EU: whether EN IEC 62368-1:2020+A11:2020 accepts the dose-based route without an RS2 ceiling (C1). First question for the test lab, with the A11 text in hand.
2. EN 50332-2 measurement details (programme-noise level relative to full scale) and the exact 27 mV and 20 h figures.
3. The FCC 1.1307(b)(3) formula transcription and the RSS-102 Issue 6 thresholds, checked against the primary texts.
4. Murata: SAR and permissive-change procedure for a portable host; antenna fit of Molex 146153 or 219611 in the case; the module datasheet (land pattern, supply ranges), still missing.
5. Mainline `nxpwifi` and Murata's TX-power tables (C4).
6. Whether the pack PCM qualifies as the PS2 limiter, and the fuse or PTC part for PCB-P-11.
7. Spread-spectrum clock support on the RK3566 DDR and display clocks.
8. Japan e-label acceptance and the KC host-testing rule.
9. Directive 2022/2380 scope reading for a DAP (expected in scope as a portable audio device with radio).

## Sources

- 47 CFR 1.1307 and 2.1093: <https://www.ecfr.gov/current/title-47/chapter-I/subchapter-A/part-1/subpart-I/section-1.1307>, <https://www.ecfr.gov/current/title-47/chapter-I/subchapter-A/part-2/subpart-J/subject-group-ECFR52efa600149ef42/section-2.1093>; secondary summary of the 2021 exemption rules: [Nemko](https://www.nemko.com/blog/understanding-rule-changes-for-human-exposure-to-radiofrequency-electromagnetic-fields)
- 47 CFR 15.23 (kits): <https://www.ecfr.gov/current/title-47/chapter-I/subchapter-A/part-15/subpart-A/section-15.23>
- Murata Type 2EL: CER-0248 Rev. 6.0 and E2B-09-1413 Rev. 9.0, as recorded in the [parts-lib sheet](https://github.com/opendap-project/parts-lib/blob/main/Murata/LBES5PL2EL-923.md) with their hashes
- Decision (EU) 2025/138 (EN 18031 citation and restrictions): <https://eur-lex.europa.eu/eli/dec_impl/2025/138/oj/eng>
- Commission guidelines on Battery Regulation Art. 11, C/2025/214: <https://eur-lex.europa.eu/eli/C/2025/214/oj/eng>
- Directive (EU) 2022/2380: <https://eur-lex.europa.eu/eli/dir/2022/2380/oj/eng>
- EN 50332-3:2017 scope (secondary, catalogue): <https://iteh.es/catalog/standards/sist/669af6ab-ca37-4ab1-a336-def08543a43b/sist-en-50332-3-2017>; EN IEC 62368-1:2020/A11:2020 scope (secondary, catalogue): <https://iteh.es/catalog/standards/sist/7f899651-a0c1-4092-a0d2-34161ce557d8/sist-en-iec-62368-1-2020-a11-2020>
- AOSP sound dose (secondary, an implementation of IEC 62368-1 ed. 3 and EN 50332-3): <https://source.android.com/docs/core/audio/sound-dose>
- 150 mV wide-band limit (secondary, a manufacturer's user guide citing EN 50332-2): <https://download.lenovo.com/pccbbs/pubs/t490/html_en/en/Using_headphones_or_earphones.html>
