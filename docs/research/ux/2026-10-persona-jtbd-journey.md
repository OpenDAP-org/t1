# UX research: personas, jobs to be done and the user journey of the T1

- **Date:** 2026-10-22
- **Status:** Desk research. Nothing here was tested with users. Personas are archetypes built from public user reports and from the T1 requirements; they are not survey results, and no figure in this document describes how many people share a trait. Every specific claim about other players links to its source; anything without a primary source read in this session is marked **UNVERIFIED**.
- **Question:** who the OpenDAP T1 is for, what they hire a digital audio player (DAP) to do, and where, from first hearing about it to the end of its life, the experience is won or lost.
- **Inputs read (OpenDAP):**
  - [T1 requirements](../../requirements/README.md) (T1-001 to T1-045) and [controls and status LED](../../specs/controls-and-status-led.md) (Draft);
  - [system requirements](https://github.com/opendap-project/system/blob/main/docs/requirements/README.md) (SYS-001 to SYS-029), [OpenDAP OS requirements](https://github.com/opendap-project/opendap-os/blob/main/docs/requirements/README.md) (OS-001 to OS-022), [desktop requirements](https://github.com/opendap-project/desktop/blob/main/docs/requirements/README.md) (DESK-001 to DESK-005);
  - [board requirements](https://github.com/opendap-project/board/blob/main/docs/requirements/README.md), [board ADR 0012](https://github.com/opendap-project/board/blob/main/docs/adr/0012-headphone-output-stage.md) (headphone output stage), the T1 [battery-life analysis](../2026-10-battery-life-analysis.md), the faceplate requirements (touch, cassette, classic).
- **External sources:** listed in §8. Read directly with HTTP fetches on 2026-10-22; web search was not available in this session.
- **Access limits:** reddit.com (r/DigitalAudioPlayer) refused every fetch (login wall); head-fi.org thread pages redirected to the forum home page; rockbox.org returned an anti-bot page (the Rockbox manual and sources were read from the project's GitHub mirror instead); Shanling's download page returned HTTP 403; FiiO's firmware page renders its list with JavaScript and showed no changelogs; Sony's help guide URL tried returned HTTP 404. Head-fi and Reddit culture is therefore represented only where Audio Science Review (ASR) threads quote or link it. This is the main gap of this study.
- **Conventions:** quotes are verbatim from the linked page, including the authors' spelling. Cayin's changelogs are published in German; the English renderings here are this document's translations and are marked *(translated)*. "User report" means one person's post, not a measured fact.

---

## 1. Summary

1. **The T1 competes with the phone and a dongle, not only with other DAPs.** On ASR the recurring argument is that a phone plus a good USB DAC sounds the same; what a DAP still wins, in users' own words, is output power, battery life, quick start-up and ease of use ([ASR, cell phones vs DAP][asr-phones]). Those four are experience properties, so the T1 is judged on execution, which matches its promise.
2. **The software is the product.** Users choosing between players say the interface and software matter more than the sound ([ASR, DX180 or DX340][asr-dx180]); they describe Android on DAPs as laggy or maddening ([ASR, DAP comparison][asr-compare]); and one user rewrote Astell&Kern's firmware to fix library speed and EQ ([ASR, AKCFW][asr-akcfw]).
3. **Hardware ageing ends DAPs, and repair rarely saves them.** Reported deaths are batteries (one failed replacement, one swollen cell) and an old SoC that makes the device "almost a piece of E-waste" ([ASR, DAPs dead][asr-dead]; [ASR, DAP comparison][asr-compare]). A replaceable standard cell, mainline Linux and a lean image are the T1's answer, and they need to be visible in the journey, not only in the BOM.
4. **The moments of truth are small and physical:** the first sound, the volume step, silence with sensitive IEMs, the wait before the first track, the library on day one, and the first update that does not break anything (§6).
5. **The primary persona, the engineer audiophile, wants proof, not adjectives.** Measurements, bit-perfect indicators, changelogs that say what changed, and limits that are documented and visible (T1-026, T1-045) are part of the experience for this persona.

---

## 2. Method

- **Personas** combine the brief's primary persona with the use contexts and gear that recur in ASR threads and reviews of current players (§8).
- **Jobs to be done** are written as "When …, I want to …, so I can …" and tagged functional, emotional or social.
- **The journey** runs from awareness to end of life. Each stage lists actions, touchpoints, pain points seen on other players (cited) and opportunities for the T1 (tied to requirements where one exists).
- **Changelogs** were read as complaint logs: a fix names a problem users had. Cayin, HiBy and iBasso pages could be read; FiiO's and Shanling's could not (see access limits).
- **Not covered:** usability testing, interviews, quantitative surveys, and any topic the maintainers have not opened (software stacks, services). Accounts are out of scope by brief.

---

## 3. Personas

### 3.1 Primary: the engineer audiophile

Demanding about sound, curious about how the player works, and wants both an excellent first use and the freedom to explore, customise and extend it. Not necessarily a professional engineer: the trait is the engineering mindset, not the job title.

| Aspect | Description |
|---|---|
| Goals | Transparent playback of a local library, provably bit-perfect (OS-020); enough clean power for every headphone owned; understand and control every stage (gain, filters, EQ, sample rate); extend the device when an idea comes (HATs, capabilities, scripts). |
| Context of use | Desk, commute, travel, home listening; sessions of hours; also evenings spent configuring, measuring or reading docs. |
| Gear | Sensitive multi-driver IEMs and full-size headphones; a 4.4 mm balanced cable for at least one of them; often a desktop DAC/amp, an audio interface or an ADC to measure with ([ASR, JM21 as USB DAC][asr-jm21], where a user measures a DAP with a Topping E2X2 Pro). |
| Frustrations seen on other players | Measured shortfalls hidden behind marketing: limited power at 32 Ω and poor gain management on a flagship ([ASR, Kann Ultra review][asr-kann]); 2 V instead of 4 V balanced output and no gain step to lower residual noise ([ASR, AP80 Pro Max review][asr-ap80]); noise rising when a USB cable is attached ([ASR, JM21][asr-jm21]); changelogs that say only "system optimisation" *(translated)* ([Cayin N8 II firmware][cayin-n8ii]) or no changelog at all ([iBasso DX180 firmware][ibasso-dx180]). |
| Already tinkering | Rooting and "optimization guides" for iBasso players ([head-fi thread, linked from ASR][asr-dx180]); custom firmware for Astell&Kern with AutoEQ, DSP, library and battery work ([ASR, AKCFW][asr-akcfw]); Rockbox on iPods and on current players such as the FiiO M3K, HiBy R1 and R3 Pro II and Shanling Q1 ([Rockbox `tools/configure`][rb-configure]). |
| What "extraordinary" means | The numbers on the box are the numbers on the bench; nothing locked (T1-045); one device that plays perfectly out of the box and opens all the way down when wanted, with the documentation and tools to do it (T1-025, OS-019, DESK-002). |

### 3.2 Secondary: the commuter with IEMs

| Aspect | Description |
|---|---|
| Goals | Music with one hand, often without looking; long battery; small and light in a pocket; silence between tracks with sensitive IEMs. |
| Context of use | Train, bus, walking, gym; also in the car. One user asked specifically for physical buttons for car use after several players failed ([ASR, DAPs dead][asr-dead]). |
| Gear | Sensitive IEMs, sometimes Bluetooth earphones; phone in the other pocket. |
| Frustrations seen on other players | Large Android players described as too big and heavy to carry ([ASR, lack of players][asr-lack]); hiss on very sensitive IEMs as a buying concern ([ASR, small DAP with low noise][asr-dlna]); no fast rewind through inline controls for spoken content ([ASR, inline controls][asr-inline]); random reboots on skip or volume change on a small player (user report, [ASR, JM21 thread, post 24][asr-jm21]). |
| What "extraordinary" means | Pocket the T1 and forget it: track and volume control by feel with the screen off (Action combinations in the [controls spec](../../specs/controls-and-status-led.md) §1.3), no hiss, no surprises, a battery that outlasts the day. |

### 3.3 Secondary: the collector with a large lossless and DSD library

| Aspect | Description |
|---|---|
| Goals | Carry the whole library (tens of thousands to 100k+ tracks, FLAC, hi-res PCM, DSD); browse it the way it is tagged; keep it in sync with the master copy on a PC or NAS. |
| Context of use | Home listening sessions; library maintenance on a computer; rare but long imports. |
| Gear | Large microSD cards, a NAS or a PC library, full-size headphones, sometimes a home system fed over USB or line out. |
| Frustrations seen on other players | A database rebuilt on every power-on, about 20 s each time ([ASR, Sony NW-WM1A review][asr-wm1a]); a song list slowed by full-resolution artwork, fixed by a third party, who also reports halving the time of a full library rebuild ([ASR, AKCFW][asr-akcfw]); scanning moved into settings and made configurable, and library search added later by firmware ([Cayin N3 Ultra firmware][cayin-n3u]); left and right swapped and random noise on DSD512 fixed by firmware *(translated)* ([Cayin N3 Ultra firmware][cayin-n3u]); a microSD card not recognised after an update ([HiBy R3 firmware][hiby-dl]). |
| What "extraordinary" means | The library is there, complete, correctly sorted and instantly scrollable on the first day, and stays that way through every update. |

### 3.4 Secondary: the listener who wants it to just work

| Aspect | Description |
|---|---|
| Goals | Press play. No settings, no file management if possible. |
| Context of use | Home, bed, travel. A user describes an iPod nano as "great for sleeping with" ([ASR, iPod nano 7G measurements][asr-nano]). |
| Gear | Whatever came with the phone, wired earphones, a Bluetooth speaker or headphones. |
| Frustrations seen on other players | Loading music is a barrier: "requires pre-loading and I don't have the organisational skills to really use it" ([ASR, listening without a smartphone][asr-nophone]); unwanted features: "why does my music player have a location feature?!" ([ASR, DAP without Wi-Fi][asr-nowifi]); on macOS, an Android player needs a separate transfer application before it appears as storage ([Cayin N3 Ultra download page][cayin-n3u]). |
| What "extraordinary" means | It feels like a dedicated product, not a small phone (SYS-017), and everything advanced stays out of the way until asked for (DESK-003). |

The **tinkerer or modder** is treated as the far end of the primary persona rather than a separate one: the same person on a weekend, building the DIY edition (T1-006), writing a capability (SYS-001) or designing a HAT (T1-002).

---

## 4. Jobs to be done

Type: **F** functional, **E** emotional, **S** social. Persona: **P** primary, **C** commuter, **L** collector (library), **J** just works.

### 4.1 Listening

| Job | Type | Persona | Evidence or requirement |
|---|---|---|---|
| When I start a track, I want to hear it exactly as it was mastered, so I can trust what I hear. | F, E | P, L | OS-020 bit-perfect PCM and DSD; phones resample unless the app bypasses it ([ASR, cell phones vs DAP][asr-phones]) |
| When I plug in a new headphone, I want the right gain and a safe starting level, so I can listen without a jolt or hiss. | F, E | all | per-gain volume curves on the FiiO JM21 ([FiiO JM21][fiio-jm21]); gain and noise with sensitive IEMs ([ASR, AP80 Pro Max][asr-ap80]) |
| When I listen at night or to sensitive IEMs, I want fine volume steps at the bottom of the range, so I can find a quiet level. | F | C, J | the iPod nano's lowest selectable volume is −76 dBFS ([ASR, iPod nano][asr-nano]); Rockbox's perceptual mode gives smaller dB steps at high volume and larger at low ([Rockbox manual, system options][rb-system]) |
| When an album plays through, I want no gaps and no clicks between tracks or sample-rate changes, so I can listen to live and classical albums whole. | F | P, L | gapless playback is a Rockbox baseline ([Wikipedia, Rockbox][wiki-rockbox]) |
| When I return to a long podcast or audiobook, I want it to resume where I left it and skip back a few seconds, so I can follow dense content. | F | C | [ASR, inline controls][asr-inline]; Rockbox automatic resume and skip length ([autoresume][rb-autoresume], [playback options][rb-playback]) |
| When I listen for hours, I want to know the device protects my hearing without nagging, so I can listen without worry. | F, E | all | SYS-012 (EN 50332 weekly dose) |

### 4.2 Discovering

| Job | Type | Persona | Evidence or requirement |
|---|---|---|---|
| When I have a large library, I want to rediscover albums I forgot, so I can enjoy what I already own. | E | L | **UNVERIFIED**: no source for rediscovery behaviour was read; included as a hypothesis to test |
| When I search, I want results by artist, album, genre and work as I type, so I can find anything in seconds. | F | L, P | search by album, artist and genre added by a later Cayin firmware *(translated)* ([Cayin N3 Ultra][cayin-n3u]) |

### 4.3 Organising

| Job | Type | Persona | Evidence or requirement |
|---|---|---|---|
| When I add music on my computer, I want the T1 to update itself without a manual rescan, so I can keep one master library. | F | L, J | DESK-002 (desktop app covers library and sync, scope to define) |
| When my tags are messy, I want the T1 to show what the tags say and let me choose sort rules, so I can browse my own way. | F | L, P | Rockbox's database sort criteria are "completely customised" ([Rockbox manual, database][rb-tagcache]) |
| When I store non-music files on the card, I want to exclude folders from the library, so I can keep scans fast. | F | P, L | Rockbox `database.ignore` ([Rockbox manual, database][rb-tagcache]) |

### 4.4 Carrying

| Job | Type | Persona | Evidence or requirement |
|---|---|---|---|
| When I leave home, I want the device to last the whole day of listening, so I can stop thinking about charging. | F, E | C | T1-039 (6 h red line, 8 h fine, 12 h ideal); 11 h on 3.5 mm quoted for the HiBy R4 ([HiBy R4][hiby-r4]) |
| When I carry it with my phone, I want it small enough for a jeans pocket, so I can take it everywhere. | F, E | C | T1-031 to T1-033 (credit-card proportions); a user values players that fit "the coin/watch/fifth pocket of my jeans" ([ASR, DAP without Wi-Fi][asr-nowifi]) |
| When I control it in a pocket or a car, I want buttons I can use by feel, so I can keep my eyes elsewhere. | F | C | [controls spec](../../specs/controls-and-status-led.md); [ASR, DAPs dead][asr-dead] |

### 4.5 Sharing

| Job | Type | Persona | Evidence or requirement |
|---|---|---|---|
| When I find a good setting for a headphone, I want to export it and share it, so I can help others and compare notes. | F, S | P | AutoEQ integration requested and built by users ([ASR, AKCFW][asr-akcfw]) |
| When I play music for someone, I want to send it to a speaker or the car, so I can share without handing over my headphones. | F, S | J, C | the T1 has Bluetooth and a mono speaker (BRD-029, T1-016) |
| When I talk about the T1 on forums, I want evidence I can link (measurements, specs, source code), so I can defend my choice. | S | P | ASR threads are argued with measurements ([ASR, cell phones vs DAP][asr-phones]) |

### 4.6 Measuring

| Job | Type | Persona | Evidence or requirement |
|---|---|---|---|
| When I doubt a claim, I want to measure the output and the digital stream myself, so I can verify it. | F, E | P | BRD-040 (I2S/DSD test pads), BRD-041 (passive capture), T1-026 |
| When I use the T1 as a USB DAC, I want the same noise floor as standalone and no added latency, so I can use one device everywhere. | F | P | noise rises with USB attached and audio delay in DAC mode are reported on FiiO players ([ASR, JM21][asr-jm21]) |
| When a limit applies, I want to see it and why, so I can trust the device. | E | P | T1-045; [ADR 0012](https://github.com/opendap-project/board/blob/main/docs/adr/0012-headphone-output-stage.md) output ceiling with the interim battery (note, 2026-10-22: removed by [board ADR 0014](https://github.com/opendap-project/board/blob/main/docs/adr/0014-two-cell-battery.md), the T1 moved to a 2S pack) |

### 4.7 Tinkering

| Job | Type | Persona | Evidence or requirement |
|---|---|---|---|
| When I want a feature, I want to build it as a capability with the SDK, so I can extend the player without forking it. | F, E | P | SYS-001, SYS-007, SYS-019 |
| When I break something, I want a guaranteed way back, so I can experiment without fear. | F, E | P | OS-017, BRD-043, MaskROM and recovery combinations ([controls spec](../../specs/controls-and-status-led.md) §1.2) |
| When I build or mod hardware, I want documented interfaces, so I can design a HAT or faceplate. | F, S | P | T1-002, T1-018, T1-023 |

---

## 5. User journey

Each stage lists what the person does, where it happens, what goes wrong on current players, and what the T1 can do about it. "Touchpoints" use the T1's own surfaces: the device, OpenDAP Desktop (DESK-001), the `opendap` CLI (T1-025), the docs and the community.

### 5.1 Awareness and evaluation

- **Actions:** reads measurements and reviews, compares output power and noise at 32 Ω and with IEMs, asks in forums which player to buy.
- **Touchpoints:** ASR reviews and threads, head-fi, Reddit, YouTube, the OpenDAP repositories and docs.
- **Pain points elsewhere:** specification sheets that omit power ("could not see any power specifications from Sony", [ASR, NW-WM1A][asr-wm1a]); measured flaws only found after purchase (one channel much worse than the other, [ASR, xDuoo X5][asr-x5]); a debate that dismisses DAPs against phones and dongles ([ASR, cell phones vs DAP][asr-phones]).
- **Opportunities:** publish complete, measured figures for both outputs, every gain and the main loads, with the method, before anyone asks (T1-026); state the plain case for a DAP the way users state it: power, battery, quick start, ease of use ([ASR, cell phones vs DAP][asr-phones]); keep the early-development status explicit.

### 5.2 Unboxing and first boot

- **Actions:** opens the box (or builds the DIY kit), presses Main, waits.
- **Touchpoints:** device, status LED, quick-start sheet **UNVERIFIED** (no packaging decision exists).
- **Pain points elsewhere:** a long database step at every boot ([ASR, NW-WM1A][asr-wm1a]); an update that asks the user to restart manually again after the automatic restart *(translated)* ([Cayin N3 Ultra][cayin-n3u]).
- **Opportunities:** a boot that shows progress (white breathing LED, [controls spec](../../specs/controls-and-status-led.md) §2.2) and ends on a usable screen; the DIY edition's first boot doubles as a self-test of what the builder assembled, reported in plain words.

### 5.3 Setup

- **Actions:** picks a language, sets date and time, chooses where music lives (microSD, internal storage, a NAS or a computer), pairs headphones if Bluetooth.
- **Touchpoints:** device (on-screen keyboard, SYS-021), OpenDAP Desktop, CLI for advanced users.
- **Pain points elsewhere:** on macOS, Android players need an extra application to appear as storage ([Cayin N3 Ultra][cayin-n3u]); "requires pre-loading" as a barrier for less organised users ([ASR, without a smartphone][asr-nophone]).
- **Opportunities:** setup is short and skippable; language first, with the five launch languages and right-to-left from day one (SYS-025, SYS-028); copying music works from macOS, Linux and Windows hosts without a vendor tool (requirement to state; mechanism not chosen here); OpenDAP Desktop offers to sync a folder once and keeps it in sync after (DESK-002).

### 5.4 First listen

- **Actions:** plugs in headphones, presses play, adjusts volume.
- **Touchpoints:** device, both jacks, volume buttons.
- **Pain points elsewhere:** volume control that needs "dozens" of turns from maximum to zero ([ASR, AP80 Pro Max][asr-ap80]); a volume display that "does not keep up" ([ASR, Kann Ultra][asr-kann]); audible residual noise with sensitive IEMs ([ASR, AP80 Pro Max][asr-ap80]).
- **Opportunities:** see §6 (first sound, volume step, silence). The first track should start at a safe level for the detected output and remembered per output (SYS-012 profiles).

### 5.5 Daily use

| Context | What the person needs | Pain points elsewhere | Opportunity for the T1 |
|---|---|---|---|
| Commute | Control by feel, screen off; resume after interruptions; unplug pauses | random reboots on skip or volume change (user report, [ASR, JM21 thread][asr-jm21]) | Action combinations for track control ([controls spec](../../specs/controls-and-status-led.md) §1.3); pause on unplug, as Rockbox offers ([Rockbox manual, playback][rb-playback]) |
| Gym | Small, light, secure in a pocket or on a strap; sweat | none read in this session | IP65 target for the Commercial edition (T1-017, Draft) |
| Home | Long sessions with full-size headphones; maybe the T1 as a source for a home system | delay after unplug, volume change and replug *(translated)* ([Cayin N6iii firmware][cayin-n6iii]); USB DAC noise and latency ([ASR, JM21][asr-jm21]) | line-level and USB modes that keep the on-board measurements |
| Car | Physical controls, reliable aux or Bluetooth | a user replacing failed players for car use asked for physical buttons ([ASR, DAPs dead][asr-dead]) | body buttons work without the screen; the cassette and classic faceplates use physical controls (FPC-008, FPK-008) |
| Travel | Battery, offline library, flight mode | none read in this session | radios off by default when not in use (BRD-008) |
| Sleep | Very low volume, sleep timer, no light | lowest volume floors on small players ([ASR, iPod nano][asr-nano]) | fine low-level steps; the status LED stays dark in normal operation ([controls spec](../../specs/controls-and-status-led.md) §2.2) |

### 5.6 Library growth and maintenance

- **Actions:** adds albums, fixes tags, replaces the microSD with a larger one, rescans.
- **Touchpoints:** OpenDAP Desktop, device library settings, CLI.
- **Pain points elsewhere:** full-resolution artwork caching that slows the songs view and can leave blank placeholders ([ASR, AKCFW][asr-akcfw]); a microSD not recognised after an update ([HiBy R3][hiby-dl]); a rescan moved and made configurable after release ([Cayin N3 Ultra][cayin-n3u]).
- **Opportunities:** incremental, background, resumable scanning, as Rockbox already does: the initial scan runs in the background and continues after power-off ([Rockbox manual, database][rb-tagcache]); thumbnails generated at display size; a library that survives updates and card swaps; SD/TF with eMMC and RAM cache levels (OS-022).

### 5.7 Gear pairing

| Situation | What decides the experience | Evidence | Opportunity for the T1 |
|---|---|---|---|
| Sensitive IEMs | residual noise, gain, output impedance | output impedance below 1/8 of the headphone impedance; on one multi-driver IEM, 43 Ω of output impedance produced a 12 dB response variation ([NwAvGuy][nwavguy]) | publish output impedance and noise per output and gain (**UNVERIFIED** until measured, T1-026) |
| Full-size headphones | voltage and current into 32 Ω and 300 Ω | 2 V balanced limiting high-impedance headphones ([ASR, AP80 Pro Max][asr-ap80]); clipping into low impedance ([ASR, Kann Ultra][asr-kann]) | [ADR 0012](https://github.com/opendap-project/board/blob/main/docs/adr/0012-headphone-output-stage.md) targets about 240 mW SE and 640 mW balanced into 32 Ω (desk estimate) |
| Balanced vs single-ended | which jack, which gain, what changes | the JM21 lists separate battery life for SE and balanced ([FiiO JM21][fiio-jm21]) | per-output volume memory and a clear indication of the active output (BRD-024) |
| USB DAC or transport | bit-perfect out, sample-rate following, noise | noise rises with USB attached; DAC-mode delay ([ASR, JM21][asr-jm21]) | OS-020: over USB, bit-perfect up to the external DAC's limit; conversions labelled |
| Bluetooth | codec, range, dropouts | dropouts reported on a small player (user report, [ASR, JM21 thread][asr-jm21]) | show the codec and rate actually negotiated |

### 5.8 Updates

- **Actions:** gets notified or checks, reads what changed, installs, verifies nothing broke.
- **Touchpoints:** device, OpenDAP Desktop, CLI, release notes.
- **Pain points elsewhere:** an update that broke microSD recognition, with a manual recovery path ([HiBy R3][hiby-dl]); firmware for a small player not offered over the air (user report, [ASR, JM21 thread][asr-jm21]); changelogs reduced to "system optimisation" *(translated)* ([Cayin N8 II][cayin-n8ii]); firmware pages with no changelog and firmware images of about 900 MB ([iBasso DX170][ibasso-dx170]); users blocking vendor update checks in custom firmware ([ASR, AKCFW][asr-akcfw]).
- **Opportunities:** signed, versioned releases (OS-011) with notes that name the problem fixed; A/B updates with automatic rollback (OS-017); updates from the distribution and OpenDAP repositories (OS-013); the library, settings and capabilities survive every update.

### 5.9 Troubleshooting and recovery

- **Actions:** the device freezes, will not boot, or misbehaves; the person looks for help.
- **Touchpoints:** status LED, recovery mode, OpenDAP Desktop, `opendap` CLI and `doctor` (TK-008), docs, community.
- **Pain points elsewhere:** freezing and shutting down ([ASR, DAPs dead][asr-dead]); a spontaneous shutdown ([ASR, JM21][asr-jm21]); a factory reset and microSD reflash as the way out (user report, [ASR, JM21 thread][asr-jm21]).
- **Opportunities:** recovery that never needs special tools: hardware power-off and MaskROM in hardware, recovery from SD/TF (BRD-043, [controls spec](../../specs/controls-and-status-led.md) §1.2); errors that state the next step; logs the user can export when asking for help.

### 5.10 Battery ageing and repair

- **Actions:** notices shorter runtime, checks battery health, replaces the cell or has it replaced.
- **Touchpoints:** device battery screen, docs, spare parts.
- **Pain points elsewhere:** a failed battery replacement and a swollen battery ending two players ([ASR, DAPs dead][asr-dead]); unknown battery history as the main risk of buying used ([ASR, DAP without Wi-Fi][asr-nowifi]); Rockbox lets owners tell it the capacity of a replacement battery ([Rockbox manual, system options][rb-system]); iPod owners still replace drives and fit extended batteries ([iFlash][iflash]).
- **Opportunities:** a standard commercial cell (T1-012, T1-038) with a connector, replaceable by the user, as the EU rule requires from 18 February 2027 ([battery selection research](https://github.com/opendap-project/board/blob/main/docs/research/2026-10-battery-selection.md), citing Regulation (EU) 2023/1542 Art. 11); battery health from the fuel gauge shown in plain terms; charge-limit options **UNVERIFIED** (not specified).

### 5.11 Extension

- **Actions:** adds a HAT (NAS, recorder, radio, powerbank, …), swaps a faceplate, installs or writes a capability.
- **Touchpoints:** device, OpenDAP Desktop, CLI, SDK docs, community.
- **Pain points elsewhere:** extension on current players happens against the vendor: rooting guides and custom firmware ([ASR, DX180 or DX340][asr-dx180]; [ASR, AKCFW][asr-akcfw]).
- **Opportunities:** extension by design (SYS-001, SYS-018); faceplates are not swapped often (T1-019), so the swap is a guided, documented procedure rather than a daily gesture; a HAT appears where its capabilities appear, without a separate app (SYS-017).

### 5.12 End of life

- **Actions:** the T1 is replaced, handed down, sold or repurposed.
- **Touchpoints:** device, docs.
- **Pain points elsewhere:** older Android players described as e-waste because of their SoC ([ASR, DAP comparison][asr-compare]); players kept alive for years when built well ([ASR, DAP without Wi-Fi][asr-nowifi], a seven-year-old player still in use).
- **Opportunities:** mainline kernel and standard Debian base (OS-001, OS-005) as the basis for long software life; a documented secure wipe before resale; repurposing as a general Linux gadget (T1-002).

---

## 6. Moments of truth

The details that decide whether the T1 feels extraordinary. "How to check" is the measurement or test that turns each one into a verifiable claim (T1-026). Thresholds are not set here: they are for the maintainers.

| # | Moment | What decides it | Seen elsewhere | How to check |
|---|---|---|---|---|
| 1 | Pop-free power-on, plug and unplug | no click on power-on, jack insertion, output switch, gain change or sample-rate switch | delayed audio after unplug and replug fixed by firmware *(translated)* ([Cayin N6iii][cayin-n6iii]) | capture the output with an ADC during each transition, both jacks, every gain |
| 2 | First sound | the first track starts at a safe level, on the right output, without a gap | none read for this specific case | scripted first-boot flow on the virtual and real T1 (OS-018, OS-019) |
| 3 | Volume step granularity | small steps at low levels with sensitive IEMs; quick sweep with buttons held; display keeps up | "dozens" of turns ([AP80 Pro Max][asr-ap80]); display lag ([Kann Ultra][asr-kann]); perceptual stepping ([Rockbox][rb-system]) | step size in dB across the range per gain; latency from button to level change |
| 4 | Silence | no audible hiss with sensitive IEMs at the lowest gain | residual noise with ultra-sensitive IEMs ([AP80 Pro Max][asr-ap80]) | noise in µV per output and gain, A-weighted, published |
| 5 | Time to first track after cold boot | seconds from Main press to music | database rebuild of about 20 s at each power-on ([NW-WM1A][asr-wm1a]); a user lists quicker start-up among a DAP's advantages ([ASR, cell phones vs DAP][asr-phones]) | stopwatch from power-on to first sample, empty and 100k-track library |
| 6 | Resume after screen-off or pause | instant | none read | latency from Action press to sound |
| 7 | Scrolling and searching a 100k-track library | smooth scrolling, artwork in place, results as you type | artwork caching slowing lists ([AKCFW][asr-akcfw]) | frame times while flinging a 100k-track list on the 320 × 480 display (OS-018); search latency |
| 8 | First library import | runs in the background, resumable, music playable during the scan | Rockbox's background, resumable scan ([Rockbox manual][rb-tagcache]) | import time and playback continuity for a reference library |
| 9 | Bit-perfect, visibly | the screen states rate, format and whether anything converts | DSD channel swap and noise fixed later *(translated)* ([Cayin N3 Ultra][cayin-n3u]) | digital capture at the I2S test pads compared with the file (BRD-040) |
| 10 | Unplug while playing | pauses, does not blast the speaker | Rockbox pause on unplug ([Rockbox manual][rb-playback]) | test both jacks with the speaker present (T1-016) |
| 11 | The first update | nothing lost, nothing broken, a clear note of what changed | microSD lost after an update ([HiBy R3][hiby-dl]) | upgrade and rollback tests (OS-017) with a full library and capabilities installed |
| 12 | USB DAC mode | same noise as standalone, no added delay | noise rise and delay ([ASR, JM21][asr-jm21]) | noise and latency measured with a host, both modes |
| 13 | A limit appears | the user sees which limit, why and what lifts it | (T1-045 requirement) | every safety cap has a visible state and a doc page; [ADR 0012](https://github.com/opendap-project/board/blob/main/docs/adr/0012-headphone-output-stage.md) output ceiling is one (note, 2026-10-22: removed by [board ADR 0014](https://github.com/opendap-project/board/blob/main/docs/adr/0014-two-cell-battery.md); safety limits remain) |
| 14 | The device is stuck | a guaranteed way back without a computer specialist | factory reset and SD reflash as the fix (user report, [ASR, JM21 thread][asr-jm21]) | recovery tests from every failure state, including MaskROM |
| 15 | Battery after two years | runtime still close to new, or a cheap, simple cell swap | dead and swollen batteries ending players ([ASR, DAPs dead][asr-dead]) | battery replacement time and tools, timed on a real unit |

---

## 7. Implications and open questions for the maintainers

These follow from the research and stay at the level of needs; no technology is proposed.

1. **Measurements as a product surface (T1-026).** Publish per-output, per-gain power, noise, output impedance and distortion with method and raw data. The primary persona reads them before buying and after every update.
2. **Gain and volume as one designed system.** Gain levels, step size, per-output memory, the hearing-safety dose (SYS-012) and visible safety caps (T1-045) are one feature, not five settings.
3. **A library engine for 100k tracks.** Background, incremental, resumable scanning; artwork at display size; search as you type; tags shown as they are.
4. **One mode, not a "pure music mode".** Other players split Android and dedicated modes ([FiiO JM21][fiio-jm21]; iBasso's separate Mango OS firmware, [iBasso downloads][ibasso-dl]). SYS-017 already points the T1 to a single dedicated experience.
5. **Release notes that name the bug.** Each release should say what users will notice, in plain words; the contrast with "system optimisation" changelogs is itself part of the trust.
6. **File transfer from every host.** The just-works persona needs copying music to work from macOS, Linux and Windows without a vendor tool. How is a decision for the maintainers.
7. **Streaming and network playback.** ASR users ask for streaming services on DAPs and for a DLNA/UPnP renderer ([ASR, lack of players][asr-lack]; [ASR, small DLNA DAP][asr-dlna]). Whether and how the T1 serves this is a topic the maintainers have not opened; it is recorded here as an observed need only.
8. **Battery repair as a journey step.** User replacement is required by law from 2027; the docs, the spare part and the on-device health view should be designed with it.
9. **Validate with people.** Next steps: interviews with each persona, a usability test of first boot and first listen on the virtual T1 (OS-018), and a diary study once prototypes exist.

---

## 8. Sources

All fetched on 2026-10-22.

### Forums and reviews (Audio Science Review)

- `asr-phones`: ASR, "Cell phones vs DAP players". <https://www.audiosciencereview.com/forum/index.php?threads/cell-phones-vs-dap-players.69249/>
- `asr-dx180`: ASR, "Ibasso DX180 or Ibasso DX340" (links the head-fi DX180 and rooting threads). <https://www.audiosciencereview.com/forum/index.php?threads/ibasso-dx180-or-ibasso-dx340.69194/>
- `asr-compare`: ASR, "A comparison of some random DAPs on my hand: iBasso DX200, HiBy R4, FiiO JM21". <https://www.audiosciencereview.com/forum/index.php?threads/a-comparison-of-some-random-daps-on-my-hand-ibasso-dx200-hiby-r4-fiio-jm21.70118/>
- `asr-akcfw`: ASR, "AKCFW: Astell&Kern custom firmware". <https://www.audiosciencereview.com/forum/index.php?threads/akcfw-astell-kern-custom-firmware-development-releases-discussion.73066/>
- `asr-dead`: ASR, "DAPs dead". <https://www.audiosciencereview.com/forum/index.php?threads/daps-dead.64892/>
- `asr-jm21`: ASR, "FiiO JM21 DAP USB DAC performance". <https://www.audiosciencereview.com/forum/index.php?threads/fiio-jm21-dap-usb-dac-performance.58712/>
- `asr-kann`: ASR, "Astell&Kern Kann Ultra DAP review". <https://www.audiosciencereview.com/forum/index.php?threads/astell-kern-kann-ultra-dap-review.62216/>
- `asr-ap80`: ASR, "Hidizs AP80 Pro Max DAP review". <https://www.audiosciencereview.com/forum/index.php?threads/hidizs-ap80-pro-max-dap-review.68852/>
- `asr-wm1a`: ASR, "Sony NW-WM1A review". <https://www.audiosciencereview.com/forum/index.php?threads/sony-nw-wm1a-review-digital-audio-player.18099/>
- `asr-x5`: ASR, "xDuoo X5 DAP review". <https://www.audiosciencereview.com/forum/index.php?threads/xduoo-x5-digital-audio-player-dap-review.64298/>
- `asr-nano`: ASR, "Apple iPod nano 7th generation measurements". <https://www.audiosciencereview.com/forum/index.php?threads/apple-ipod-nano-7th-generation-measurements.70217/>
- `asr-lack`: ASR, "Is there a lack of audio players on the market?". <https://www.audiosciencereview.com/forum/index.php?threads/is-there-a-lack-of-audio-players-on-the-market.53560/>
- `asr-dlna`: ASR, "Small DAP with low noise floor and DLNA/UPnP renderer support". <https://www.audiosciencereview.com/forum/index.php?threads/small-dap-with-low-noise-floor-and-dlna-upnp-renderer-support.72102/>
- `asr-inline`: ASR, "DAP with inline controls (fast forward, rewind)". <https://www.audiosciencereview.com/forum/index.php?threads/dap-with-inline-controls-fast-forward-rewind.67349/>
- `asr-nophone`: ASR, "Listening to audio out and about without a smartphone". <https://www.audiosciencereview.com/forum/index.php?threads/listening-to-audio-out-and-about-without-a-smartphone.69677/>
- `asr-nowifi`: ASR, "DAP without Wi-Fi or Bluetooth". <https://www.audiosciencereview.com/forum/index.php?threads/dap-without-wifi-or-bluetooth.69000/>

### Manufacturers (product pages and changelogs)

- `fiio-jm21`: FiiO JM21 product page. <https://www.fiio.com/jm21>
- `hiby-r4`: HiBy R4 product page. <https://store.hiby.com/products/hiby-r4>
- `hiby-dl`: HiBy download page (R3 firmware V1.15 note). <https://store.hiby.com/pages/download>
- `cayin-n3u`: Cayin N3 Ultra downloads and firmware (German). <https://cayin.com/n3-ultra-downloads-bedienungsanleitung-firmware/>
- `cayin-n6iii`: Cayin N6iii downloads and firmware (German). <https://cayin.com/n6iii-mk3-downloads-firmware-updates/>
- `cayin-n8ii`: Cayin N8 II downloads and firmware (German). <https://cayin.com/n8ii-downloads-firmware-updates/>
- `ibasso-dl`: iBasso downloads. <https://ibasso.com/downloads/>
- `ibasso-dx180`: iBasso DX180 firmware. <https://ibasso.com/dx180-firmware/>
- `ibasso-dx170`: iBasso DX170 firmware. <https://ibasso.com/dx170-firmware/>

### Rockbox, iPod modding and references

- `wiki-rockbox`: Wikipedia, "Rockbox". <https://en.wikipedia.org/wiki/Rockbox>
- `rb-configure`: Rockbox `tools/configure` (target list). <https://github.com/Rockbox/rockbox/blob/master/tools/configure>
- `rb-tagcache`: Rockbox manual, database. <https://github.com/Rockbox/rockbox/blob/master/manual/rockbox_interface/tagcache.tex>
- `rb-autoresume`: Rockbox manual, automatic resume. <https://github.com/Rockbox/rockbox/blob/master/manual/configure_rockbox/autoresume.tex>
- `rb-playback`: Rockbox manual, playback options. <https://github.com/Rockbox/rockbox/blob/master/manual/configure_rockbox/playback_options.tex>
- `rb-system`: Rockbox manual, system options. <https://github.com/Rockbox/rockbox/blob/master/manual/configure_rockbox/system_options.tex>
- `iflash`: iFlash adapters for iPod. <https://www.iflash.xyz/>
- `nwavguy`: NwAvGuy, "Headphone & amp impedance". <http://nwavguy.blogspot.com/2011/02/headphone-amp-impedance.html>

[asr-phones]: https://www.audiosciencereview.com/forum/index.php?threads/cell-phones-vs-dap-players.69249/
[asr-dx180]: https://www.audiosciencereview.com/forum/index.php?threads/ibasso-dx180-or-ibasso-dx340.69194/
[asr-compare]: https://www.audiosciencereview.com/forum/index.php?threads/a-comparison-of-some-random-daps-on-my-hand-ibasso-dx200-hiby-r4-fiio-jm21.70118/
[asr-akcfw]: https://www.audiosciencereview.com/forum/index.php?threads/akcfw-astell-kern-custom-firmware-development-releases-discussion.73066/
[asr-dead]: https://www.audiosciencereview.com/forum/index.php?threads/daps-dead.64892/
[asr-jm21]: https://www.audiosciencereview.com/forum/index.php?threads/fiio-jm21-dap-usb-dac-performance.58712/
[asr-kann]: https://www.audiosciencereview.com/forum/index.php?threads/astell-kern-kann-ultra-dap-review.62216/
[asr-ap80]: https://www.audiosciencereview.com/forum/index.php?threads/hidizs-ap80-pro-max-dap-review.68852/
[asr-wm1a]: https://www.audiosciencereview.com/forum/index.php?threads/sony-nw-wm1a-review-digital-audio-player.18099/
[asr-x5]: https://www.audiosciencereview.com/forum/index.php?threads/xduoo-x5-digital-audio-player-dap-review.64298/
[asr-nano]: https://www.audiosciencereview.com/forum/index.php?threads/apple-ipod-nano-7th-generation-measurements.70217/
[asr-lack]: https://www.audiosciencereview.com/forum/index.php?threads/is-there-a-lack-of-audio-players-on-the-market.53560/
[asr-dlna]: https://www.audiosciencereview.com/forum/index.php?threads/small-dap-with-low-noise-floor-and-dlna-upnp-renderer-support.72102/
[asr-inline]: https://www.audiosciencereview.com/forum/index.php?threads/dap-with-inline-controls-fast-forward-rewind.67349/
[asr-nophone]: https://www.audiosciencereview.com/forum/index.php?threads/listening-to-audio-out-and-about-without-a-smartphone.69677/
[asr-nowifi]: https://www.audiosciencereview.com/forum/index.php?threads/dap-without-wifi-or-bluetooth.69000/
[fiio-jm21]: https://www.fiio.com/jm21
[hiby-r4]: https://store.hiby.com/products/hiby-r4
[hiby-dl]: https://store.hiby.com/pages/download
[cayin-n3u]: https://cayin.com/n3-ultra-downloads-bedienungsanleitung-firmware/
[cayin-n6iii]: https://cayin.com/n6iii-mk3-downloads-firmware-updates/
[cayin-n8ii]: https://cayin.com/n8ii-downloads-firmware-updates/
[ibasso-dl]: https://ibasso.com/downloads/
[ibasso-dx180]: https://ibasso.com/dx180-firmware/
[ibasso-dx170]: https://ibasso.com/dx170-firmware/
[wiki-rockbox]: https://en.wikipedia.org/wiki/Rockbox
[rb-configure]: https://github.com/Rockbox/rockbox/blob/master/tools/configure
[rb-tagcache]: https://github.com/Rockbox/rockbox/blob/master/manual/rockbox_interface/tagcache.tex
[rb-autoresume]: https://github.com/Rockbox/rockbox/blob/master/manual/configure_rockbox/autoresume.tex
[rb-playback]: https://github.com/Rockbox/rockbox/blob/master/manual/configure_rockbox/playback_options.tex
[rb-system]: https://github.com/Rockbox/rockbox/blob/master/manual/configure_rockbox/system_options.tex
[iflash]: https://www.iflash.xyz/
[nwavguy]: http://nwavguy.blogspot.com/2011/02/headphone-amp-impedance.html
