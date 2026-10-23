# UX research: what the modding and custom-firmware communities reveal

- **Date:** 2026-10-22
- **Status:** Desk research. Nothing here is decided. Every "candidate requirement" below is a proposal for the maintainers, not an entry in any requirements table.
- **Question:** what do people who install custom firmware, mod hardware or tune Linux audio appliances work around, and what does that say about features DAP makers neglect or lock? How should the OpenDAP T1 answer each item, and where in the architecture does the answer live?
- **Rules this research serves:**
  - T1-045 and the workspace rule "100 % of the hardware, never software limits": software never locks, caps or hides a capability the hardware has; limits exist only for safety, and they are documented and visible.
  - T1-040: the audiophile experience is never sacrificed. The T1 is a DAP first.
  - T1-001, T1-025, OS-010, OS-017, OS-020, SYS-001, SYS-010 to SYS-012, SYS-017 to SYS-019, HCS-007 to HCS-013, blackbird-os ADR 0004 and ADR 0005.
- **Inputs read:**
  - [`t1/docs/requirements/README.md`](../../requirements/README.md)
  - [`t1/docs/specs/faceplate-interface.md`](../../specs/faceplate-interface.md) and [`t1/docs/specs/controls-and-status-led.md`](../../specs/controls-and-status-led.md)
  - [`system/docs/requirements/README.md`](../../../../system/docs/requirements/README.md)
  - [`system/docs/research/2026-10-system-architecture.md`](../../../../system/docs/research/2026-10-system-architecture.md) (capabilities, slots and templates, §5.11 audio engine, §5.12 security model, developer mode)
  - [`blackbird-os/docs/requirements/README.md`](../../../../blackbird-os/docs/requirements/README.md), [ADR 0004](../../../../blackbird-os/docs/adr/0004-image-lifecycle.md), [ADR 0005](../../../../blackbird-os/docs/adr/0005-trust-model-for-now.md)
  - [`hats/standard/docs/requirements/README.md`](../../../../hats/standard/docs/requirements/README.md)
  - [`board/docs/requirements/README.md`](../../../../board/docs/requirements/README.md) (BRD-017, BRD-023, BRD-035, BRD-043, BRD-049)

## 0. How evidence is marked

- Every specific claim carries a link to the page or file it comes from. Sources were read on 2026-10-22.
- **UNVERIFIED** marks a claim that no reachable primary source confirmed. It is kept only where it matters for a decision, and never used as the basis of a candidate requirement on its own.
- Community repositories describe themselves. Their claims (battery gains, track limits, measured outputs) are reported as the authors' claims, not as facts verified here.
- **Blocked sources.** rockbox.org (all pages, including the wiki and release notes) returned an anti-bot "Access Denied" page; Rockbox facts come from its GitHub mirror instead. head-fi.org returned a verification page. xdaforums.com returned HTTP 403. reddit.com cannot be fetched. EUR-Lex (Decision 2009/490/EC) returned empty pages. Web search was not available in this session, so discovery used GitHub repository search (`gh search repos`) and direct fetches.

## 1. Key findings (read this first)

1. **The communities are alive in 2026, and they are mostly rebuilding the player, not adding apps.** On GitHub, recent projects replace the stock player on Linux-based DAPs: Sonix Player and Compás Player for HiBy ([sonix-player](https://github.com/Jepl4r/sonix-player), [compas-player](https://github.com/Starnished66/compas-player)), Cinder and Wampy for Sony Walkmans ([Cinder](https://github.com/superwilso/Cinder), [Wampy](https://github.com/unknown321/wampy)), diskOS for the FiiO Snowsky Disc ([diskos](https://github.com/b0hemia/diskos)), Q2 Pod for the Shanling Q2 ([q2-pod](https://github.com/DiamondBond/q2-pod)) and ReChord for the FiiO Echo Mini ([ReChord](https://github.com/brnomt/ReChord)). Rockbox reached 4.0 on 2025-03-31 ([tags](https://github.com/Rockbox/rockbox/tags)) and still gets forks such as Rockpod ([rockpod](https://github.com/nuxcodes/rockpod)).
2. **What they unlock is mostly what the hardware could already do.** Examples:
   - A HiBy R1 teardown found a hidden Bluetooth-receiver screen, a factory test menu and a library limit that a flag raises from 80,000 to 200,000 songs ([Hiby-R1-Mod](https://github.com/bidhata/Hiby-R1-Mod)).
   - Cinder runs USB-DAC input and LDAC output at the same time: "Sony's app blocks this; the hardware doesn't" ([Cinder](https://github.com/superwilso/Cinder)).
   - RepriseOS removes the EU volume limit on European iPods ([osos-lab](https://github.com/lgreitis/osos-lab)).

   This is the exact failure T1-045 forbids.
3. **Volume caps are the most emotional limit, and measurement shows they are the only "region" difference that is real.** On a Sony NW-A50, 28 output measurements across 19 regions and the Walkman One sound signatures found that only the KR3 and CEW2 regions (a quieter volume table "to comply with local regulation laws") and one gain mode change the signal. "Everything else (other regions, plus modes, dac mode, sound signatures) produces the same signal" ([Wampy, making of volume tables](https://github.com/unknown321/wampy/blob/master/MAKING_OF_VOLUME_TABLES.md)). Two lessons follow: show the user exactly which limit applies and why, and publish measurements instead of letting folklore fill the gap.
4. **Bit-perfect output and freedom from forced resampling are the second big driver.**
   - USB Audio Player PRO ships its own USB driver that "bypasses the Android audio system completely" ([UAPP](https://www.extreamsd.com/index.php/products/usb-audio-player-pro)).
   - Android added a bit-perfect mixer mode only in Android 14, and only for USB ([AOSP](https://source.android.com/docs/core/audio/preferred-mixer-attr)).
   - Rockpod advertises bit-perfect PCM over the iPod dock ([rockpod](https://github.com/nuxcodes/rockpod)), and llusbdac shows a CRC32 of the received audio on a Walkman so the user can check it ([llusbdac](https://github.com/zhangboyang/llusbdac)).

   OS-020 and the `SignalPath` proposal (architecture §5.11) already answer this. The community adds one idea: let the user see the proof (a checksum or a pass/fail), not only a label.
5. **Recovery is the community's weakest point.**
   - The Walkman backup tool warns that restoring another model's backup "will brick your device. There are no checks" ([wbrt](https://github.com/unknown321/wbrt)).
   - Sonix warns that "the recovery system does not check what it is given" ([sonix-player](https://github.com/Jepl4r/sonix-player)).
   - HiBy's update images carry only MD5 checksums and "no signature" ([Hiby-R1-Mod](https://github.com/bidhata/Hiby-R1-Mod)).
   - The ZX505 USB-DAC mod "unlocks the bootloader, replaces the kernel and erases the player twice" ([ZX500-USB-DAC](https://github.com/remove/ZX500-USB-DAC)).

   The T1 already has hardware MaskROM entry, SD recovery and A/B rollback (BRD-035, BRD-043, OS-017). The gap is image-to-device compatibility checks and protected per-unit factory data.
6. **Openness on the T1 is mostly a matter of not closing things.** Communities spend years reverse-engineering what a maintainer could publish: firmware formats, SoC datasheets and schematics (HiBy OS Crack collects datasheets and a development-board schematic, [hiby_os_crack](https://github.com/hiby-modding/hiby_os_crack)), bootrom exploits (iPod: [wInd3x](https://github.com/freemyipod/wInd3x)) and undocumented chips (iPod "Mikey", [Hackaday](https://hackaday.com/2026/09/27/reverse-engineering-apples-mikey-chip/)). OpenDAP's documented interfaces, reproducible image (OS-010) and signed-but-unlocked trust model (ADR 0005) remove that work up front.
7. **Two tensions with the current proposals need the maintainers' call (§10).**
   - Communities strongly want **themes and alternative now-playing layouts**: Rockbox WPS, Winamp skins in Wampy, palettes in Cinder, `.theme` files in Compás. SYS-009 (one design system) and SYS-017 (product UI, not an OS) have to make room for that without losing coherence.
   - They also want to **replace the whole player**. The architecture proposal allows no third-party activities.

## 2. Scope

| # | Area | What was read |
|---|---|---|
| 3 | Rockbox | GitHub mirror: manual sources, codec list, plugin tree, target list, tags; Wikipedia |
| 4 | iPod modding | iFlash store and compatibility chart, iFixit, Hackaday, freemyipod, Rockpod, RepriseOS, USB-C board, Tangara |
| 5 | Android DAPs | UAPP, AOSP audio docs and source, JamesDSP, ViPER4Android, iBasso and FiiO community repositories |
| 6 | Linux DAPs and appliances | HiBy, Sony Walkman, FiiO Snowsky, Shanling community firmware; moOde, piCorePlayer, Lyrion, Mopidy; foobar2000, Roon |
| 7 | Hardware tinkering | DIY players (Tangara, PiPod, Retro-Audio, JPL.mp3), iBasso amp modules, AutoEq, CamillaDSP |
| 8 | Synthesis | Neglected features and artificial limits mapped to OpenDAP answers |
| 9 | Risks | Bricking, warranty, hearing safety, and how to offer freedom safely |

## 3. Rockbox

**What it is.** Rockbox started on 2002-06-02 "due to owner frustration with severe limitations in the manufacturer-supplied user interface" of the Archos Jukebox, and is GPLv2-or-later ([Wikipedia](https://en.wikipedia.org/wiki/Rockbox)). Version 4.0 was tagged on 2025-03-31 ([tags](https://github.com/Rockbox/rockbox/tags)).

### 3.1 What it adds over stock firmware

| Area | What Rockbox offers | Source |
|---|---|---|
| Formats | 41 decoder source files, including FLAC, ALAC, WavPack, APE, TTA, Opus, Vorbis, MPC, WMA Pro, DSD, chip-tune formats (SPC, NSF, SID, VGM); 4 encoders | [codecs/SOURCES](https://github.com/Rockbox/rockbox/blob/master/lib/rbcodec/codecs/SOURCES) |
| Gapless and crossfade | "all track changes are gapless" when crossfade is off; configurable crossfade | [playback_options.tex](https://github.com/Rockbox/rockbox/blob/master/manual/configure_rockbox/playback_options.tex) |
| ReplayGain, A–B repeat, cue sheets, skip length | ReplayGain per track or album; A–B repeat "typically used by musicians"; cue-sheet track markers; skip buttons that jump by time | Same |
| EQ and DSP | Up to ten parametric bands (low shelf, peaking, high shelf); crossfeed to make headphones "more similar to listening to music with stereo speakers"; dithering with noise shaping; timestretch; compressor; Haas surround; perceptual bass enhancement; auditory fatigue reduction | [sound_settings.tex](https://github.com/Rockbox/rockbox/blob/master/manual/configure_rockbox/sound_settings.tex) |
| Themes | The While Playing Screen is declared from tags: metadata (`%ia` artist), progress bar (`%pb`), viewports `%V(x,y,w,h)`, conditionals `%?xx<…\|…>`, album art `%Cl()`/`%Cd` | [wps_tags.tex](https://github.com/Rockbox/rockbox/blob/master/manual/appendix/wps_tags.tex) |
| Plugins and games | 80+ entries: Doom, a Game Boy emulator (rockboy), chess, sudoku, a calculator, a metronome, an MPEG player, an image viewer, an LRC lyrics player and Lua scripting | [apps/plugins](https://github.com/Rockbox/rockbox/tree/master/apps/plugins) |
| Database | Builds a database from file tags | [Wikipedia](https://en.wikipedia.org/wiki/Rockbox) |
| Dual boot | "Rockbox has a dual-boot feature"; each target documents a key to start the original firmware | [rockbox_interface/main.tex](https://github.com/Rockbox/rockbox/blob/master/manual/rockbox_interface/main.tex) |

### 3.2 Devices

47 targets are marked stable, among them the iPod 1G to 6G Classic, Mini and Nano 1G/2G, iriver H100/H300/H10, SanDisk Sansa (Clip, Fuze, e200 and variants), Creative Zen, Toshiba Gigabeat and, more recently, the xDuoo X3/X3ii/X20, FiiO M3K, AIGO Eros Q/K and Shanling Q1. Another 24 are unstable and 8 unusable ([builds.pm](https://github.com/Rockbox/rockbox/blob/master/tools/builds.pm)). Community ports continue outside the tree, for example an experimental ARM64 port that runs on top of a rooted Android Shanling M3X ([rockbox-shanling-m3x](https://github.com/doriangironde/rockbox-shanling-m3x)).

### 3.3 Why people still install it

- **Revival of hardware the maker abandoned.** All stable targets are discontinued players. Rockpod adds SSD-aware power management for iFlash-modded iPods and auto-poweroff on USB because "stock blocks poweroff indefinitely on any USB" ([rockpod](https://github.com/nuxcodes/rockpod)).
- **Audio control the stock firmware never offered:** the DSP set in §3.1, on players whose stock EQ was a few presets (stock EQ details are **UNVERIFIED** per model).
- **Looks.** Theme projects such as adwaitapod, built on GNOME's interface guidelines, are maintained in 2026 ([adwaitapod](https://github.com/D0-0K/adwaitapod)).

### 3.4 Limits

- It does not play DRM files, and on some platforms the user must dual-boot into the original firmware to charge or transfer files ([Wikipedia](https://en.wikipedia.org/wiki/Rockbox)).
- The theme engine is a fixed tag language. A theme author lists missing pieces: colour tokens for a unified light/dark mode, themable menu colours, tiled images, forced line breaks ([d00k, missing features](https://d00k.net/wiki/rockbox_advanced/missing_features/)). The same author notes that the Rockbox development team "is small, and have little free time" ([adwaitapod](https://github.com/D0-0K/adwaitapod)).
- Plugins are reached through one Plugins entry, and Rockpod promotes Cover Flow out of "the plugins menu" to a top-level entry ([rockpod](https://github.com/nuxcodes/rockpod)). This matches the architecture research's point that a plugin list is an OS-like seam (§3.7 there).
- It targets old microcontroller-class hardware: no Wi-Fi streaming and no modern Bluetooth codec on those targets (**UNVERIFIED** per target).

## 4. iPod modding

| Mod | What people do | Source |
|---|---|---|
| Solid-state storage | iFlash adapters (Solo, Dual, Quad, SATA, CF) replace the iPod's hard drive with microSD, CF or M.2 SATA, for iPod Video, Classic and Photo | [iFlash](https://www.iflash.xyz/) |
| Storage limits | The compatibility chart says 6th-generation models "max out at 128GB due to LBA28 addressing", and that 30 GB models handle "anywhere from 10000 tracks to 30000 tracks" | [iFlash compatibility](https://www.iflash.xyz/store/iflash-compatibility/) |
| Battery | The 650 mAh cell is replaced with a modern one; iFlash links a "3rd party extended Battery guide" | [Hackaday](https://hackaday.com/2024/11/29/modernizing-an-apple-ipod-or-a-modern-day-ship-of-theseus/), [iFlash store](https://www.iflash.xyz/store/) |
| Repair | iFixit lists battery, click wheel, display, hard drive and headphone jack guides, plus "How to install Rockbox on an iPod Classic" | [iFixit](https://www.ifixit.com/Device/iPod_Classic) |
| Bluetooth, USB-C, display, haptics, case | One build added a Bluetooth transmitter, replaced the 30-pin connector with USB-C, fitted a modern LCD, a "taptic mod", and a transparent case and touch wheel; afterwards "the parts left of the original iPod were most of the mainboard and some flex cable" | [Hackaday](https://hackaday.com/2024/11/29/modernizing-an-apple-ipod-or-a-modern-day-ship-of-theseus/) |
| USB-C board | An open-source USB-C breakout PCB for the 4th-generation iPod (gerbers, BOM, wiring) | [ipod_usb_c](https://github.com/makhowastaken/ipod_usb_c) |
| Headset remote | Reverse-engineering Apple's "Mikey" chip so Rockbox can read three-button headset remotes | [Hackaday](https://hackaday.com/2026/09/27/reverse-engineering-apples-mikey-chip/) |
| Firmware | Rockbox on iPods; Rockpod adds bit-perfect digital out over the dock; RepriseOS patches Apple's own firmware (custom EQ, Play Next, "EU volume limit removal") and boots Apple OS, RepriseOS or Rockbox by the key held | [rockpod](https://github.com/nuxcodes/rockpod), [osos-lab](https://github.com/lgreitis/osos-lab) |
| Unlocking | Newer Nanos and the Classic need a bootrom exploit to run unsigned code (wInd3x, s5late) | [wInd3x](https://github.com/freemyipod/wInd3x) |
| DAC or amplifier swaps | **UNVERIFIED.** No reachable source documented a DAC or headphone-amplifier swap inside an iPod | — |

**What drives the scene** (from the projects' own words, not a survey):

- **Ownership and repair:** keep a device working after the maker stopped. Storage, battery and connector replacements dominate the list above.
- **Nostalgia and tactile interaction:** a DIY ESP32 player was built because its maker missed "the tactile experience and portability of iPods" ([Hackaday, JPL.mp3](https://hackaday.com/2025/08/01/diy-mp3-player-inspired-by-the-ipod/)). Tangara is "the music player you wish you had in the early 2000s" ([Crowd Supply](https://www.crowdsupply.com/cool-tech-zone/tangara)). Q2 Pod puts an "iPod classic experience" on a Shanling Q2 ([q2-pod](https://github.com/DiamondBond/q2-pod)).
- **Customisation:** transparent shells, new wheels, themes.

## 5. Android DAP customisation

| Practice | What it works around | Source |
|---|---|---|
| Own USB audio driver (USB Audio Player PRO) | Android's mixer: UAPP "bypasses the Android audio system completely" and claims "bit-perfect 24-bit audio playback and all sample rates that the chip supports without resampling" on internal audio, plus native DSD | [UAPP](https://www.extreamsd.com/index.php/products/usb-audio-player-pro) |
| Android 14 bit-perfect mode | Optional for vendors, "Only USB devices", no software volume ("the hardware controls the volume"), no sample-rate conversion or effects | [AOSP preferred mixer attributes](https://source.android.com/docs/core/audio/preferred-mixer-attr) |
| Manufacturer "bit-perfect" modes | HiBy markets "Direct Transport Architecture (systemwide bit-perfect audio)"; FiiO markets "no global SRC" and an exclusive "Pure Music mode". Internals are proprietary (**UNVERIFIED**) | As cited in [system architecture research §3.6](../../../../system/docs/research/2026-10-system-architecture.md) |
| System-wide DSP (JamesDSP, ViPER4Android) | No system-wide EQ or convolution in stock firmware. JamesDSP offers a partitioned convolver, FIR EQ with AutoEQ, crossfeed, compressor and user-written effects in EEL2. Its Android manager targets Android 5–10 and installs through system modification or Magisk. ViPER4Android's app source "is currently not open source" | [JDSP4Linux](https://github.com/Audio4Linux/JDSP4Linux), [JamesDSPManager](https://github.com/james34602/JamesDSPManager), [ViPER4Android](https://github.com/AndroidAudioMods/ViPER4Android) |
| Rootless DSP | RootlessJamesDSP uses internal audio capture. Limits: apps that block capture (Spotify, Chrome) stay unprocessed, it conflicts with other effect apps, and latency increases | [RootlessJamesDSP](https://github.com/ThePBone/RootlessJamesDSP) |
| Safe-volume warnings | AOSP's safe media volume depends on a "country option", can be set inactive once the user confirms, and is re-enabled after 20 hours of music at unsafe volume (`UNSAFE_VOLUME_MUSIC_ACTIVE_MS_MAX = 20 * 3600 * 1000`). Android also computes a sound dose over "a seven-day continuously rolling window" (EN 50332-3, IEC 62368-1) and, without acknowledgement, "the volume lowers to the predefined … RS1 value" | [SoundDoseHelper.java](https://github.com/aosp-mirror/platform_frameworks_base/blob/main/services/core/java/com/android/server/audio/SoundDoseHelper.java), [AOSP sound dose](https://source.android.com/docs/core/audio/sound-dose) |
| ADB tweaks without root | An iBasso DX340 playbook debloats over `adb shell`, applies kernel and OS tweaks for "low-latency audio playback" with `adb root` ("Rooting is NOT required"), and whitelists music apps from Doze | [DX340 playbook](https://github.com/Stouthart/DX340) |
| Root modules | FiiOat, a Magisk module for the FiiO M21/JM21: scheduler and uclamp tweaks, highest priority for essential processes, zswap removal, debloat | [FiiOat](https://github.com/WheresWaldo/FiiOat) |
| Modified firmware | iBasso DX150 builds that lower CPU and RAM speed with the screen off ("more than a hour" more playback, per the author), add USB mass storage instead of MTP, and add a "Pure Mango OS" mode. Flashed with Rockchip FactoryTool or `rkflashtool` | [DX150-firmware](https://github.com/Lurker00/DX150-firmware) |
| Custom ROMs and kernels, bootloader policy | **UNVERIFIED.** XDA was not reachable. No primary source on FiiO, HiBy, Shanling or iBasso bootloader-unlock policy was found. The repositories above show that some devices allow `adb root` and Rockchip USB flashing, and others need Magisk or EDL boot writes ([rockbox-shanling-m3x](https://github.com/doriangironde/rockbox-shanling-m3x)) | — |

## 6. Linux-based DAPs and appliance hacking

### 6.1 HiBy OS (Linux, Ingenic X1600E)

- **System.** Linux 4.4.94, Buildroot 2020.02.1, BusyBox, a read-only squashfs root. An init script "reboots the device if the player ever exits". Update images are ISO 9660 files whose "Integrity is MD5 only. No signature" ([Hiby-R1-Mod](https://github.com/bidhata/Hiby-R1-Mod)).
- **Locked or hidden:** a "PEQ Combined Settings" debug dialog with 32 bands; a factory test menu; a hidden Bluetooth-receiver screen; feature flags, one of which raises the library limit from 80,000 to 200,000 songs; a volume configuration line `volume: default 20, warn 34, lock 50 (headset)` (its meaning is the author's reading, **UNVERIFIED**). ADB is behind an easter egg: tap "About" 10 times ([Hiby-R1-Mod](https://github.com/bidhata/Hiby-R1-Mod), [hiby-mods](https://github.com/hiby-modding/hiby-mods)).
- **Community firmware:**
  - HiBy Mods ships database tools, a unified theme, extended Unicode, a "Most Played" list and ADB. Its known limits are text rendering ("does not support bidirectional text") and a 50,000-track limit "under investigation" ([hiby-mods](https://github.com/hiby-modding/hiby-mods)).
  - Two full replacement players exist. Sonix exposes the CS43198's "digital filters, DRE, NOS" on the R3 Pro II ([sonix-player](https://github.com/Jepl4r/sonix-player)). Compás is GPL-3.0, with Lua plugins, an on-device plugin store, themes, a remote-control API, gapless, ReplayGain, a 10-band PEQ and USB DAC mode ([compas-player](https://github.com/Starnished66/compas-player)).
- **Relevance to the T1.** The R3 Pro II uses two CS43198, the DAC the T1 plans (BRD-022, Draft) ([sonix-player](https://github.com/Jepl4r/sonix-player)). Users of that chip already ask for its filter, NOS and DRE controls.

### 6.2 Sony Walkman (Android-based NW-A, ZX and WM players)

- **Region and volume.** Volume tables depend on region, and only the CEW and KR3 variants are quieter "to comply with local regulation laws". Sound signatures and plus modes measured identical at the output ([Wampy, volume tables](https://github.com/unknown321/wampy/blob/master/MAKING_OF_VOLUME_TABLES.md)). Cinder "removes Sony's regional volume limit by default", with the region-limited curve available as an option ([Cinder](https://github.com/superwilso/Cinder)).
- **Walkman One** (Mr.Walkman's custom firmware for the A30, A40, A50, ZX300 and WM1A/Z, linked from [Wampy](https://github.com/unknown321/wampy)) changes the key Sony's updater accepts. Its plus modes are "a 3-byte change to Sony's audio library", and it drops some Sony effects ([Cinder](https://github.com/superwilso/Cinder)). The primary Walkman One pages and head-fi threads were not fetched (**UNVERIFIED** beyond these statements).
- **Interface add-ons:** Wampy offers Winamp 2 skins, cassette skins, a volume table editor, per-song audio options, all audio filters, FM radio and recording, and region-free ATRAC. On the A40/A30, stock firmware "is broken, outdated GPU driver; use Walkman One" ([Wampy](https://github.com/unknown321/wampy)).
- **Unlocked hardware roles:**
  - USB-DAC input with LDAC output at once ([Cinder](https://github.com/superwilso/Cinder)).
  - A native UAC2 USB DAC up to 384 kHz on the ZX505, with "A configurable ceiling caps how loud the computer can drive the hardware" ([ZX500-USB-DAC](https://github.com/remove/ZX500-USB-DAC)).
  - A low-latency USB DAC that shows "sample rate, bit depth, and CRC32 checksum" ([llusbdac](https://github.com/zhangboyang/llusbdac)).
- **Recovery** depends on a backup made beforehand with a Windows-only tool: "There is no other recovery path for this device" ([Cinder](https://github.com/superwilso/Cinder), [wbrt](https://github.com/unknown321/wbrt)).

### 6.3 FiiO Snowsky and Shanling

- diskOS flashes a new interface over Ingenic mask-ROM USB, keeps the stock audio engine, offers eight themes, and lets the user boot stock once or restore the saved stock image ([diskos](https://github.com/b0hemia/diskos)).
- ReChord is a from-source GPL-3.0 firmware for the FiiO Echo Mini. Gapless works "within one format"; a rate change "stays a hard boundary by design, because the DAC has to re-lock" ([ReChord](https://github.com/brnomt/ReChord)). The engine proposal in architecture §5.11 makes the same choice.
- Q2 Pod puts an iPod-style interface on a Shanling Q2, with a parametric EQ "with up to 30 bands", AutoEQ and Equalizer APO import, scrobbling, an "optional 80% charge limit" and a standby limit ([q2-pod](https://github.com/DiamondBond/q2-pod)).

### 6.4 Network players and MPD-class software

| Project | What its users value | Source |
|---|---|---|
| moOde | GPL-3.0; "does not serve Ads, nag for subscriptions, use cookies, analytics/tracking"; built with standard Debian packaging and pi-gen | [moode README](https://github.com/moode-player/moode) |
| piCorePlayer | Runs "entirely in RAM", "you can simply pull the power without any risk of corruption" | [picoreplayer.org](https://www.picoreplayer.org/) |
| Lyrion (formerly Logitech Media Server) | The community took over the Squeezebox server after Logitech and renamed it, so Squeezebox hardware keeps working | [lyrion.org](https://lyrion.org/), [slimserver](https://github.com/LMS-Community/slimserver) |
| Mopidy | An "extensible music server" controlled "from any MPD or web client" | [mopidy](https://github.com/mopidy/mopidy) |
| MPD | Bit-perfect only with ALSA `hw:`, no software mixer, no ReplayGain and no forced format; per-format output policy | As cited in [system architecture research §3.6](../../../../system/docs/research/2026-10-system-architecture.md) |
| Volumio | Plugin kinds: audio interface, music service, system hardware, system controller, user interface | Same |
| CamillaDSP, AutoEq | IIR/FIR engine "for crossovers, room correction etc."; AutoEq (about 16,000 GitHub stars) generates headphone EQ from measurements, and DAP firmwares import its format (HiBy PEQ parses `ParametricEQ.txt`) | [camilladsp](https://github.com/HEnquist/camilladsp), [AutoEq](https://github.com/jaakkopasanen/AutoEq), [Hiby-R1-Mod](https://github.com/bidhata/Hiby-R1-Mod) |

### 6.5 Desktop players as the reference for expectations

- **foobar2000:** about 300 components tagged by function (DSP, decoder, resampler, output, UI panel, visualisation, tagging, statistics, lyrics) ([components](https://www.foobar2000.org/components)).
- **Roon:** classifies the signal path as Lossless ("from the file to the device without being modified"), Enhanced (processing "because you asked for them"), High Quality and Low Quality ([signal path](https://help.roonlabs.com/portal/en/kb/articles/signal-path)).
- **JRiver:** not researched (no source fetched).

## 7. Hardware tinkering and what makes a platform hackable

| Project or practice | What it shows | Source |
|---|---|---|
| Tangara | "100% open hardware running open-source software"; FreeCAD design files on sourcehut and Codeberg; UI written in Lua; standard LiPo pouch with a JST connector, "replacing or upgrading your battery a breeze"; display and touch wheel on a separate PCB connected by ribbon | [Crowd Supply](https://www.crowdsupply.com/cool-tech-zone/tangara) |
| PiPod, Retro-Audio, JPL.mp3 | Raspberry Pi and ESP32 players built for a specific interaction (click wheel, tape-recorder buttons, classical-music metadata) | [PiPod](https://github.com/BramRausch/PiPod), [Retro-Audio](https://github.com/CoretechR/Retro-Audio), [Hackaday](https://hackaday.com/2025/08/01/diy-mp3-player-inspired-by-the-ipod/) |
| iBasso amp modules | Swappable amplifier modules (AMP8 MK2, AMP13–AMP18) and faceplate options for the DX340 | [ibasso.com](https://ibasso.com/) (which modules fit which player: **UNVERIFIED**) |
| DIY headphone amplifiers | Open designs on GitHub, for example an INA1620 USB-C DAC/amp and CMoy variants | [DIY USB-C DAC amp](https://github.com/daumemo/DIY-Micro-USB-C-DAC-Amplifier), [uC3Moy](https://github.com/UC3Music/uC3Moy) |
| Op-amp rolling | **UNVERIFIED.** No reachable source was fetched. The T1's OPA1622 headphone amplifiers are small surface-mount packages (BRD-022, Draft), so socketed op-amp swapping does not apply to the T1 as drafted | — |

**What made a platform hackable, observed across §3 to §7:**

| Property | Positive example | Cost when missing |
|---|---|---|
| Documented firmware format and update path | Compás and Sonix update from the device; HiBy update images are plain ISO 9660 | Months of reverse-engineering; firmware formats documented by outsiders ([hiby-mods](https://github.com/hiby-modding/hiby-mods)) |
| Schematics and datasheets | Tangara publishes them | HiBy OS Crack collects SoC datasheets and a reference-board schematic to infer the hardware ([hiby_os_crack](https://github.com/hiby-modding/hiby_os_crack)) |
| Unlockable boot that the owner controls | HiBy (no signature, at the cost of no integrity); Rockchip USB flashing on iBasso | iPod needs a bootrom exploit; the ZX505 mod wipes the device twice |
| A simulator or host build | Sonix builds "host" and "target" from one tree; HiBy OS Crack has an experimental QEMU setup | Testing on the only unit; Cinder has "One test unit" |
| A plugin or extension API | Compás's versioned `plugin.*` Lua API; Rockbox plugins and Lua; foobar2000 components | Whole-firmware forks for small features |
| A shell and root for the owner | ADB on HiBy (easter egg) and Walkman | Magisk or EDL boot writes |
| A community place | HiBy OS modding Discord; head-fi; Rockbox forums | Knowledge scattered across one-person repositories that ask for maintainers ([hiby-mods](https://github.com/hiby-modding/hiby-mods)) |

## 8. Neglected features and artificial limits, mapped to OpenDAP

Legend for **Lives in:** **OS** (Blackbird OS image and services), **engine** (the audio engine, architecture §5.11), **cap** (a capability, SYS-001), **tpl** (template or slot, architecture §5.7), **theme** (design-system tokens and layouts; see the §10 tension), **dev** (developer mode, architecture §5.12), **tool** (the `opendap` CLI, T1-025 / TK-002), **HAT**, **FP** (faceplate), **HW** (board), **docs** (published documentation and measurements).

Candidate requirements use "shall" for consistency with the requirements tables. They are proposals only.

| # | Limit or neglected feature the community works around | Evidence | Existing coverage | Candidate requirement | Lives in |
|---|---|---|---|---|---|
| 1 | **Volume caps by region, silent and fixed** (EU iPods, Sony CEW/KR3, Android per-country safe volume) | [osos-lab](https://github.com/lgreitis/osos-lab), [Wampy](https://github.com/unknown321/wampy/blob/master/MAKING_OF_VOLUME_TABLES.md), [SoundDoseHelper](https://github.com/aosp-mirror/platform_frameworks_base/blob/main/services/core/java/com/android/server/audio/SoundDoseHelper.java) | T1-045 (limits only for safety, visible); SYS-012 (dose) | Blackbird OS shall show, wherever volume is set, every active volume limit, its reason and the rule it implements, and shall let the user choose the volume curve; a limit required by law shall be the only kind applied by default, with the exact legal behaviour, and never a silent region table. | engine, OS, docs |
| 2 | **Forced resampling and mixing** (Android SRC, "bit-perfect modes" users cannot verify) | [UAPP](https://www.extreamsd.com/index.php/products/usb-audio-player-pro), [AOSP](https://source.android.com/docs/core/audio/preferred-mixer-attr) | OS-020, BRD-023, `SignalPath` proposal | Blackbird OS shall let the user verify bit-perfect playback on the device: the signal path shows each stage, and a test mode compares a known test file with what the output receives (pass/fail), as llusbdac shows a CRC32. | engine, tool, docs |
| 3 | **Locked or toy EQ** (presets only; a 32-band PEQ hidden behind a debug dialog on HiBy) | [Hiby-R1-Mod](https://github.com/bidhata/Hiby-R1-Mod), [JamesDSP](https://github.com/Audio4Linux/JDSP4Linux), [q2-pod](https://github.com/DiamondBond/q2-pod) | Architecture §5.11 lists an optional DSP chain | The T1 shall offer a system-wide parametric EQ (shelf and peaking filters) that imports AutoEQ and Equalizer APO formats, per-headphone profiles, and crossfeed; the DSP chain shall be off by default, and when it is on the signal path shall say so (Roon's "Enhanced"). | engine, tpl |
| 4 | **No user DSP extension** (convolution, user effects require root on Android) | [JamesDSP](https://github.com/Audio4Linux/JDSP4Linux), [CamillaDSP](https://github.com/HEnquist/camilladsp) | SYS-001 capabilities | Blackbird OS should let a capability provide a DSP stage (for example a convolver with user impulse responses) inside the engine's chain, under a declared permission, and the signal path shall name that stage. | cap, engine |
| 5 | **Hidden DAC and output controls** (CS43198 filters, NOS, DRE; gain tables; output modes) | [sonix-player](https://github.com/Jepl4r/sonix-player), [Wampy](https://github.com/unknown321/wampy/blob/master/MAKING_OF_VOLUME_TABLES.md), [Compás Gain Mode](https://github.com/Starnished66/compas-player) | T1-045; faceplate extension candidates "gain, DAC filter" | The T1 shall expose every DAC filter, gain setting and output mode its audio hardware supports as user settings, with a plain description of each, and faceplates may bind them to controls. | engine, tpl, FP |
| 6 | **Missing gapless, or gapless broken at format changes** | [Rockbox](https://github.com/Rockbox/rockbox/blob/master/manual/configure_rockbox/playback_options.tex), [ReChord](https://github.com/brnomt/ReChord) | Architecture §5.11 gapless | The T1 shall play gapless between tracks of the same rate and format, and shall document and measure the silence at rate-family changes. | engine, docs |
| 7 | **No themes, or a closed fixed look** (Rockbox WPS, Winamp skins, palettes, `.theme` files, iPod-style UIs) | [wps_tags](https://github.com/Rockbox/rockbox/blob/master/manual/appendix/wps_tags.tex), [Wampy](https://github.com/unknown321/wampy), [Cinder](https://github.com/superwilso/Cinder), [compas-player](https://github.com/Starnished66/compas-player), [diskos](https://github.com/b0hemia/diskos) | SYS-009 (one design system), SYS-018 | Blackbird OS should support user-installable themes, defined as design-system token sets (colour, type, density) plus alternative now-playing layouts built from templates, so that a theme cannot break accessibility or the input profiles. **Needs a maintainers' decision (§10).** | theme, tpl |
| 8 | **No plugin system**, or plugins hidden in one menu | [apps/plugins](https://github.com/Rockbox/rockbox/tree/master/apps/plugins), [Compás plugins](https://github.com/Starnished66/compas-player), [foobar2000](https://www.foobar2000.org/components) | SYS-001, SYS-007, SYS-019 | Already covered by capabilities and slots. Add: a capability's settings and tasks shall be discoverable from the place they act (now playing, library, output), never only from an extensions list. | cap, tpl |
| 9 | **Replacing the whole player** (Sonix, Compás, Cinder, diskOS, Rockbox ports) | Projects in §6 | OS-010 (image from code and configuration) | Blackbird OS shall document how to build and boot an image with a different product shell, and the recovery path shall restore the official image whatever the user installed. **Whether alternative shells are first-class is a maintainers' decision (§10).** | OS, docs |
| 10 | **Abandoned firmware and dead platforms** (Squeezebox, A30/A40 "outdated GPU driver", one-person mod projects) | [lyrion.org](https://lyrion.org/), [Wampy](https://github.com/unknown321/wampy), [hiby-mods](https://github.com/hiby-modding/hiby-mods) | OS-001, OS-002, OS-011, OS-013 | Each Blackbird OS release shall publish its full source and build inputs, so that anyone can rebuild and keep updating a T1 image without the project. | OS, docs |
| 11 | **Locked boot or bootrom exploits needed** (iPod, ZX505 bootloader unlock with wipes) | [wInd3x](https://github.com/freemyipod/wInd3x), [ZX500-USB-DAC](https://github.com/remove/ZX500-USB-DAC) | ADR 0005 (unlocked for now, nothing burned in OTP) | If verified boot is enforced later, the owner shall be able to enrol their own keys or turn enforcement off from the device, without erasing user data and without a key from OpenDAP, and the device shall show which keys it trusts. | OS, HW |
| 12 | **No root shell, or a shell behind an easter egg** | [hiby-mods](https://github.com/hiby-modding/hiby-mods), [DX340](https://github.com/Stouthart/DX340) | T1-025, TK-002, OS-019 (SSH) | The T1 shall give its owner a documented root shell (USB and SSH) enabled from settings, with a persistent indicator while it is on. | dev, tool |
| 13 | **Hidden hardware roles** (Bluetooth receiver hidden on HiBy; USB-DAC in plus LDAC out blocked on Sony) | [Hiby-R1-Mod](https://github.com/bidhata/Hiby-R1-Mod), [Cinder](https://github.com/superwilso/Cinder), [ZX500-USB-DAC](https://github.com/remove/ZX500-USB-DAC) | T1-045; architecture §5.11 mentions a later USB-in UAC2 gadget | The T1 shall offer every audio role its hardware supports, alone or combined where the hardware can (for example USB audio input with any output, Bluetooth source or sink), and shall state which combinations it cannot do and why. | engine, OS |
| 14 | **Library and storage limits** (80,000 or 50,000 tracks on HiBy; 10,000–30,000 on iPods; 128 GB on LBA28) | [Hiby-R1-Mod](https://github.com/bidhata/Hiby-R1-Mod), [hiby-mods](https://github.com/hiby-modding/hiby-mods), [iFlash](https://www.iflash.xyz/store/iflash-compatibility/) | OS-022 | The library shall have no fixed track or capacity limit; its measured performance at large sizes shall be published. | OS, docs |
| 15 | **MTP-only or proprietary transfer; long rescans** | [DX150-firmware](https://github.com/Lurker00/DX150-firmware), [hiby-mods](https://github.com/hiby-modding/hiby-mods) | OS-022 | The T1 should expose its music storage as a standard USB mass-storage or network file share, and the library index format shall be documented so a computer can build or read it. | OS, docs |
| 16 | **Battery and power policy the user cannot set** (charge limit, no charging in DAC mode, screen-off CPU limits, auto-poweroff on USB) | [q2-pod](https://github.com/DiamondBond/q2-pod), [Compás](https://github.com/Starnished66/compas-player), [Hiby-R1-Mod](https://github.com/bidhata/Hiby-R1-Mod), [DX150-firmware](https://github.com/Lurker00/DX150-firmware), [rockpod](https://github.com/nuxcodes/rockpod) | T1-009, T1-045 | The T1 shall let the user set a charge limit, charging behaviour while connected as a USB device, and idle and auto-off behaviour; the power-safety limits of BRD-049 stay fixed and are shown. | OS, HW |
| 17 | **Unrepairable battery and parts** (iPod battery and storage swaps; Tangara's JST cell) | [iFixit](https://www.ifixit.com/Device/iPod_Classic), [Tangara](https://www.crowdsupply.com/cool-tech-zone/tangara) | T1-012 (standard commercial cell), T1-038 | The T1 should let a user replace the battery with common tools, and the repair procedure shall be published. | HW, docs |
| 18 | **Fixed physical identity** (custom shells, wheels, transparent cases, cassette skins) | [Hackaday](https://hackaday.com/2024/11/29/modernizing-an-apple-ipod-or-a-modern-day-ship-of-theseus/), [Wampy](https://github.com/unknown321/wampy) | T1-003, T1-018, T1-023, T1-024 (faceplates, HID commands) | Already covered by faceplates (touch, cassette, classic). Add: the faceplate interface and mechanical drawings shall be published so third parties can make faceplates. | FP, docs |
| 19 | **Sound "signature" folklore** (region and firmware tunings that measure identical) | [Wampy](https://github.com/unknown321/wampy/blob/master/MAKING_OF_VOLUME_TABLES.md) | T1-026 | Every audio claim about a T1 setting shall come with a published measurement, and settings that do not change the signal shall not be presented as if they did. | docs, engine |
| 20 | **No remote or scripting API** (Compás remote-control API, MPD clients) | [compas-player](https://github.com/Starnished66/compas-player), [mopidy](https://github.com/mopidy/mopidy) | SYS-004 (bus API), T1-025 | Covered by the bus and the CLI. Add: playback control and library browsing shall be a documented, versioned API reachable from another device. | OS, tool |

## 9. Risks the community runs into, and how to offer the freedom safely

| Risk | Community evidence | OpenDAP answer | Existing coverage | Gap (candidate) |
|---|---|---|---|---|
| **Bricking from a wrong or interrupted flash** | "No checks"; restoring another unit's backup overwrites the serial number "with no chance of recovery" ([wbrt](https://github.com/unknown321/wbrt)); the recovery "does not check what it is given" ([sonix-player](https://github.com/Jepl4r/sonix-player)); an interrupted flash may need mask-ROM recovery ([diskos](https://github.com/b0hemia/diskos)) | Recovery that always works and does not depend on the installed OS | MaskROM by hardware (BRD-035, controls spec: "Safety-related functions never depend only on software"); SD recovery (BRD-043); A/B rollback (OS-017) | Images shall declare the hardware they target, and every flashing path (recovery, CLI, update) shall refuse a mismatch unless the owner forces it. Per-unit factory data (serial number, calibration) shall live where no image flash can overwrite it, with a documented backup. |
| **No integrity at all** | MD5-only update images ([Hiby-R1-Mod](https://github.com/bidhata/Hiby-R1-Mod)) | Signed images by default, verified by the boot chain (blackbird-os ADR 0004), while the owner keeps control (ADR 0005) | ADR 0004, ADR 0005, architecture §5.12 | §8 row 11: owner-enrolled keys if enforcement ever comes |
| **Warranty void** | "it voids your warranty" ([ZX500-USB-DAC](https://github.com/remove/ZX500-USB-DAC)); "No warranty at all" ([DX150-firmware](https://github.com/Lurker00/DX150-firmware)) | Modifying software through documented paths is normal use, not tampering | Not covered (a commercial policy, out of scope here) | Maintainers' decision, not an engineering item (§10) |
| **Hearing damage when caps are removed** | Mods remove region tables and EU limits by default (Cinder, RepriseOS); Android re-arms its warning after 20 hours and lowers volume to RS1 without acknowledgement ([SoundDoseHelper](https://github.com/aosp-mirror/platform_frameworks_base/blob/main/services/core/java/com/android/server/audio/SoundDoseHelper.java), [sound dose](https://source.android.com/docs/core/audio/sound-dose)) | Safety limits are the one allowed limit: documented, visible, applied the same way to every control | SYS-012 (weekly dose, 85 dB(A), EN 50332); controls spec ("Volume limits and the hearing-safety dose (SYS-012) apply to button volume changes") | §8 row 1. Which parts of the dose behaviour the user may override is a legal question: the EU decision text was not reachable (**UNVERIFIED**), so the override rules must come from the standard itself |
| **Damage from unsafe hardware settings** (charge current, voltages) | A HiBy flag turns off charging in DAC mode ([Hiby-R1-Mod](https://github.com/bidhata/Hiby-R1-Mod)) | User power policy within BRD-049 margins | BRD-049 | §8 row 16: limits shown, not hidden |
| **Lock-in to one maintainer** | "maintainers and contributors wanted" ([hiby-mods](https://github.com/hiby-modding/hiby-mods)); "I can't promise features or time" ([Cinder](https://github.com/superwilso/Cinder)) | The official firmware is itself the open project, so there is no fork to orphan | OS-010, OS-011, licensing rules | §8 row 10 |
| **Security exposure from mods** | A web server "runs as root on port 4399 with CGI on … and no chroot" during Wi-Fi transfer ([Hiby-R1-Mod](https://github.com/bidhata/Hiby-R1-Mod)) | Sandboxed capabilities, a permission broker, developer mode with an indicator | Architecture §5.12 | Root shell and developer mode always visible (§8 row 12) |

**"Try before you flash."** Rockbox dual-boot, RepriseOS's three-way boot and diskOS's "boot the stock UI once" show that users want to try an alternative without committing. The T1 can already boot from SD for recovery (BRD-043). A candidate: *the T1 should boot an Blackbird OS image from the SD/TF card without modifying internal storage*, so that alternative or development images can be tried and removed by taking the card out. Whether RK3566 boot order and the A/B scheme allow this cleanly is **UNVERIFIED**.

## 10. Tensions with current requirements, and questions for the maintainers

> **Resolved (2026-10-22).** The maintainers answered every question below:
>
> 1. Themes: one official theme, with community themes supported from the start as design-system token sets and template-based layouts (SYS-030, DS-007), and WCAG AA contrast in every theme (DS-010).
> 2. Alternative shells: community scope. The architecture allows them (documented image build, separable board support, owner-controlled boot), but OpenDAP does not support them (SYS-035).
> 3. Side-loading: possible in developer mode, which must be simple to turn on (SYS-031).
> 4. Verified boot: if it is ever enforced, the owner can enrol their own keys or turn enforcement off without losing data (blackbird-os OS-025).
> 5. Hearing safety: the limits required by law (EN 50332) are fixed and always visible; everything beyond them is adjustable (SYS-012 context, T1-045).
> 6. Warranty: a commercial policy, decided outside this repository.
>
> The questions are kept as they were asked.

1. **Themes vs one design system (SYS-009, SYS-017).** The strongest pattern in this research is visual customisation: Rockbox themes, Winamp and cassette skins, palettes, iPod-style shells. A theme system that only swaps design-system tokens and chooses among host-rendered now-playing layouts keeps accessibility and input profiles intact (architecture §5.7). The architecture research already cites Sailfish ambiences as a precedent for one theme driving the system (§3.3 there). Should user themes be allowed, and at which depth (tokens only, layouts, custom components)?
2. **Alternative product shells vs "no third-party activities" (architecture Q5).** Communities replace the whole player. Inside Blackbird OS, the proposal keeps the product shell first-party. Should an alternative shell be a supported image variant (a build option of OS-010), an unsupported but documented path, or out of scope?
3. **Package endpoint vs installing from anywhere (SYS-010, SYS-011).** Users of Compás install Lua plugins by copying a file to the SD card, and other projects install from a PC. SYS-011 sends every download through the image's package endpoint, and architecture §5.12 accepts unsigned capabilities in developer mode. Is developer mode the only door for side-loading, and is it acceptable to everyday users who just want a community theme or plugin?
4. **Future verified-boot enforcement (ADR 0005).** The community record argues for owner-controlled keys (§8 row 11). This should be settled before any lock is considered.
5. **Hearing-safety override scope.** Which parts of SYS-012 the user may change (dose budget, warnings, per-headphone profiles) needs the EN 50332 text and a legal reading; this research could not reach the EU decision.
6. **Warranty and modification.** Whether flashing a self-built image affects the Commercial edition's warranty is a policy decision, not an engineering one.

## 11. Not covered and why

- **Head-fi, XDA and Reddit discussions** were not reachable. The prevalence of each practice (how many users, which complaints dominate) is therefore not quantified. The ranking in the final summary reflects how many independent projects work on an item, not a survey.
- **Manufacturer bootloader policies** (FiiO, HiBy, Shanling, iBasso, Sony) were not confirmed from primary sources.
- **iPod DAC or amplifier swaps, op-amp rolling and JRiver** have no fetched source.
- **Rockbox's own wiki** ("Why Rockbox", release notes, target status pages) was blocked by an anti-bot page. The GitHub mirror replaced it for facts, but not for the project's own wording of its motivation.
- **Business, pricing and cloud topics** are out of scope by instruction.

## 12. Sources (accessed 2026-10-22)

- **Rockbox:** [tags](https://github.com/Rockbox/rockbox/tags) · [builds.pm](https://github.com/Rockbox/rockbox/blob/master/tools/builds.pm) · [codecs/SOURCES](https://github.com/Rockbox/rockbox/blob/master/lib/rbcodec/codecs/SOURCES) · [apps/plugins](https://github.com/Rockbox/rockbox/tree/master/apps/plugins) · [sound_settings.tex](https://github.com/Rockbox/rockbox/blob/master/manual/configure_rockbox/sound_settings.tex) · [playback_options.tex](https://github.com/Rockbox/rockbox/blob/master/manual/configure_rockbox/playback_options.tex) · [wps_tags.tex](https://github.com/Rockbox/rockbox/blob/master/manual/appendix/wps_tags.tex) · [rockbox_interface/main.tex](https://github.com/Rockbox/rockbox/blob/master/manual/rockbox_interface/main.tex) · [Wikipedia](https://en.wikipedia.org/wiki/Rockbox) · [adwaitapod](https://github.com/D0-0K/adwaitapod) · [d00k missing features](https://d00k.net/wiki/rockbox_advanced/missing_features/) · [rockbox-shanling-m3x](https://github.com/doriangironde/rockbox-shanling-m3x)
- **iPod:** [iFlash](https://www.iflash.xyz/) · [iFlash compatibility](https://www.iflash.xyz/store/iflash-compatibility/) · [iFlash store](https://www.iflash.xyz/store/) · [iFixit iPod Classic](https://www.ifixit.com/Device/iPod_Classic) · [Hackaday, modernizing an iPod](https://hackaday.com/2024/11/29/modernizing-an-apple-ipod-or-a-modern-day-ship-of-theseus/) · [Hackaday, Mikey](https://hackaday.com/2026/09/27/reverse-engineering-apples-mikey-chip/) · [Hackaday, JPL.mp3](https://hackaday.com/2025/08/01/diy-mp3-player-inspired-by-the-ipod/) · [ipod_usb_c](https://github.com/makhowastaken/ipod_usb_c) · [rockpod](https://github.com/nuxcodes/rockpod) · [osos-lab (RepriseOS)](https://github.com/lgreitis/osos-lab) · [wInd3x](https://github.com/freemyipod/wInd3x)
- **Android:** [USB Audio Player PRO](https://www.extreamsd.com/index.php/products/usb-audio-player-pro) · [AOSP preferred mixer attributes](https://source.android.com/docs/core/audio/preferred-mixer-attr) · [AOSP sound dose](https://source.android.com/docs/core/audio/sound-dose) · [SoundDoseHelper.java](https://github.com/aosp-mirror/platform_frameworks_base/blob/main/services/core/java/com/android/server/audio/SoundDoseHelper.java) · [JDSP4Linux](https://github.com/Audio4Linux/JDSP4Linux) · [JamesDSPManager](https://github.com/james34602/JamesDSPManager) · [RootlessJamesDSP](https://github.com/ThePBone/RootlessJamesDSP) · [ViPER4Android](https://github.com/AndroidAudioMods/ViPER4Android) · [DX340 playbook](https://github.com/Stouthart/DX340) · [FiiOat](https://github.com/WheresWaldo/FiiOat) · [DX150-firmware](https://github.com/Lurker00/DX150-firmware)
- **Linux DAPs:** [Hiby-R1-Mod](https://github.com/bidhata/Hiby-R1-Mod) · [hiby-mods](https://github.com/hiby-modding/hiby-mods) · [hiby_os_crack](https://github.com/hiby-modding/hiby_os_crack) · [sonix-player](https://github.com/Jepl4r/sonix-player) · [compas-player](https://github.com/Starnished66/compas-player) · [Cinder](https://github.com/superwilso/Cinder) · [Wampy](https://github.com/unknown321/wampy) · [Wampy, making of volume tables](https://github.com/unknown321/wampy/blob/master/MAKING_OF_VOLUME_TABLES.md) · [wbrt](https://github.com/unknown321/wbrt) · [llusbdac](https://github.com/zhangboyang/llusbdac) · [ZX500-USB-DAC](https://github.com/remove/ZX500-USB-DAC) · [diskos](https://github.com/b0hemia/diskos) · [ReChord](https://github.com/brnomt/ReChord) · [q2-pod](https://github.com/DiamondBond/q2-pod)
- **Appliances and desktop:** [moOde](https://github.com/moode-player/moode) · [piCorePlayer](https://www.picoreplayer.org/) · [Lyrion](https://lyrion.org/) · [slimserver](https://github.com/LMS-Community/slimserver) · [Mopidy](https://github.com/mopidy/mopidy) · [CamillaDSP](https://github.com/HEnquist/camilladsp) · [AutoEq](https://github.com/jaakkopasanen/AutoEq) · [foobar2000 components](https://www.foobar2000.org/components) · [Roon signal path](https://help.roonlabs.com/portal/en/kb/articles/signal-path)
- **Hardware:** [Tangara](https://www.crowdsupply.com/cool-tech-zone/tangara) · [PiPod](https://github.com/BramRausch/PiPod) · [Retro-Audio](https://github.com/CoretechR/Retro-Audio) · [iBasso](https://ibasso.com/) · [DIY USB-C DAC amp](https://github.com/daumemo/DIY-Micro-USB-C-DAC-Amplifier) · [uC3Moy](https://github.com/UC3Music/uC3Moy)
- **Blocked or empty:** rockbox.org (anti-bot), head-fi.org (verification page), xdaforums.com (HTTP 403), reddit.com (not fetchable), eur-lex.europa.eu Decision 2009/490/EC (empty), Wikipedia "EN 50332" (HTTP 404).
