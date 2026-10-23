# UX research: playback, signal path, library, connectivity and device behaviour

- **Date:** 2026-10-22
- **Status:** Desk research. Nothing here is a decision. The "The T1 shall …" lines are **candidate** functional requirements for the maintainers to accept, change or drop; none of them is in a requirements table yet.
- **Requirements served:** T1-008 (audio quality first), T1-009 (battery), T1-026 (every promise verified), T1-040 (the audiophile experience at 100 %), T1-045 (100 % of the hardware, limits only for safety and visible), OS-020 (bit-perfect scope), SYS-012 (hearing-safety dose).
- **Inputs read:** [T1 requirements](../../requirements/README.md), [controls and status LED](../../specs/controls-and-status-led.md), [Blackbird OS requirements](https://github.com/opendap-project/blackbird-os/blob/main/docs/requirements/README.md), [DSD512 research](https://github.com/opendap-project/blackbird-os/blob/main/docs/research/2026-10-dsd512.md), [system requirements](https://github.com/opendap-project/system/blob/main/docs/requirements/README.md), [board ADR 0012](https://github.com/opendap-project/board/blob/main/docs/adr/0012-headphone-output-stage.md), [board requirements](https://github.com/opendap-project/board/blob/main/docs/requirements/README.md).

## 0. Method and limits

- Sources were fetched directly on 2026-10-22 (web search was not available). Each specific claim links to the page it was read from. Forum posts are reported as **user reports**, manufacturer pages as **manufacturer claims**, and Wikipedia as a secondary source.
- **Blocked in this session** (no content could be read, so no claim relies on them): reddit (old and new), head-fi.org (challenge page), rockbox.org and its wiki (access-denied page; the Rockbox manual was read from its LaTeX sources on GitHub instead), wiki.hydrogenaud.io (HTTP 403), gabriel.mp3-tech.org (LAME tag), web.archive.org, sony.com product pages, eur-lex.europa.eu (empty), lesbonscomptes.com (upmpdcli, HTTP 503), Shanling download pages (HTTP 403), most HiBy, iBasso, Astell&Kern and Shanling changelog pages (HTTP 404 or no changelog text), and most guessed Audio Science Review (ASR) review URLs (HTTP 404 or wrong thread).
- Consequence: measured per-model data (SINAD, output impedance, real battery life, boot time) and long-term owner complaints from reddit and head-fi are **missing**. The complaint evidence comes mainly from **manufacturers' own changelogs** (each fix names a problem users had), the Sony NW-A100 help guide, GitHub issue trackers of open-source players, ASR forum threads that did load, and reference documentation (Rockbox, MPD, Linux kernel, Android, specifications).
- "What good looks like" is a synthesis from the cited behaviour of the best references (mostly Rockbox, MPD and the cited changelogs), not a quotation.
- **UNVERIFIED** marks anything not read from a fetched source.

### Short names used in the tables

| Name | What it is |
|---|---|
| Rockbox | Open-source firmware for older DAPs; exists because of "severe limitations in the manufacturer-supplied user interface" ([Wikipedia: Rockbox][wp-rockbox]) |
| MPD | Music Player Daemon, open-source Linux player; its manual and issue tracker document Linux/ALSA behaviour directly relevant to Blackbird OS |
| NW-A100 | Sony Walkman NW-A100 series; its online help guide is the only Sony source that loaded |
| FiiO changelogs | FiiO's official firmware notes on forum.fiio.com |

## 1. Playback

| Item | What good looks like | Documented evidence and complaints | Right / wrong | Candidate requirement |
|---|---|---|---|---|
| Gapless, causes | Sample-exact joins for every format: decoder priming and padding trimmed, next track pre-decoded. | Gaps come from player latency (file fetch, metadata, decoder start, "half a second or more") and from lossy codecs' padding; MP3 has no standard field for delay/padding and needs the LAME tag or iTunes' iTunSMPB; Vorbis, Opus, FLAC and WAV are inherently gapless; AAC support varies by implementation ([Wikipedia: Gapless][wp-gapless]). Opus carries a "pre-skip" in its header and trims the end through the last page's granule position ([RFC 7845][rfc7845]). | Mature decoders regress: Media3 1.11 added a 12 ms gap between LAME MP3s that 1.9.2 played sample-exact ([media3 #3425][m3-3425], cause [PR #3200][m3-3200]); Opus "small hiccup" at the transition in Media3 1.6/1.8 ([media3 #2576][m3-2576]). Rockbox is gapless for MP3, AAC, Vorbis, FLAC, WAV/AIFF, ALAC, TTA and WavPack ([Rockbox file formats][rb-formats]). FiiO DISC: "enabling gapless playback for a long time could accidentally cause playback to pause" (fixed V2.21, [FiiO DISC][fiio-disc]). | **GP-1** The T1 shall play consecutive tracks of the same format and rate sample-exact, honouring encoder delay and padding (LAME/iTunSMPB, Opus pre-skip and end trimming, AAC priming), and this shall be checked by an automated bit-exact test on every release. |
| Gapless across rate or format changes | When the next track changes sample rate, family (44.1/48 kHz) or PCM/DSD, the join is not gapless by nature; the player still must not lose audio, pop or stall. | Some DACs need time to lock after a rate change, so the first moments of a track are lost unless silence is sent at the new rate first; only needed on rate changes ([MPD #1140][mpd-1140]). The first ≈ 0.25 s is lost when the ALSA device leaves the running state ([MPD #2588][mpd-2588]); with an on-board DAC, ≈ 1 s lost after idle and a pop on close ([MPD #1599][mpd-1599]). FiiO M11S fixed stutter at the start of a song when switching between sample rates in USB DAC mode (FW 1.0.5, [FiiO M11S][fiio-m11s]). | Wrong (later fixed): FiiO M11S. Open upstream: MPD #1140, #2588. | **GP-2** On a change of sample rate, rate family or PCM/DSD mode, the T1 shall mute, reconfigure and unmute without an audible click and without dropping the first samples of the new track, and shall otherwise keep the join as short as the hardware allows. |
| Crossfade | Optional, off by default, never applied to DSD; modes for automatic vs manual changes. | MPD crossfades only between songs of the same format; forcing a common format costs quality and CPU ([MPD manual][mpd-user]). DSD cannot be crossfaded; MPD's output died with "Cannot cross-fade format dsd" until a 2026-10-08 fix that plays those joins gapless instead ([MPD PR #2592][mpd-2592]). Rockbox separates crossfade on automatic changes, manual skips, shuffle, or always, with fade-in/fade-out settings ([Rockbox playback options][rb-playback]). | Right: Rockbox granularity. | **GP-3** Crossfade shall be off by default, selectable separately for automatic and manual track changes, and automatically replaced by a gapless join when either side is DSD or when it would require resampling a bit-perfect stream. |
| ReplayGain / R128 | Track, album and automatic (track when shuffling) modes, preamp, clipping prevention from stored peaks, a level for untagged files; R128 tags in Opus handled correctly. | Rockbox: Album (falls back to Track), Track, "Track Gain If Shuffling", Off; "Prevent Clipping"; preamp −12 to +12 dB in 0.5 dB steps ([Rockbox playback options][rb-playback]). MPD: track/album/auto, preamp ±15 dB, `replaygain_missing_preamp`, peak limit on by default ([MPD manual][mpd-user]). ReplayGain stores peaks to prevent clipping; newer tools measure with BS.1770/R128 ([Wikipedia: ReplayGain][wp-rg]). EBU R128 targets −23 LUFS, −1 dBTP ([EBU R128][ebu-r128]). Opus R128_TRACK_GAIN/ALBUM_GAIN add to the header gain, never replace it ([RFC 7845][rfc7845]). "ReplayGain is usually implemented with a software volume filter (which prevents bit-perfect playback)"; MPD can apply it through the hardware mixer instead ([MPD manual][mpd-user]). | Right: Rockbox, MPD. | **GP-4** The T1 shall support ReplayGain and R128 gain (track, album, track-when-shuffling, off), a preamp, a level for untagged files and peak-based clipping prevention, and shall apply the gain in the DAC's volume control where possible so the stream to the DAC stays bit-perfect; the UI shall say when a gain setting makes playback not bit-perfect. |
| Resume position | Resume the exact track and position after pause, power-off, crash or battery cut; per-track resume for long content. | MPD saves queue and position to `state_file` every 120 s by default; `restore_paused` starts paused ([MPD manual][mpd-user]). Rockbox stores a resume point per track in its database, limited to chosen folders such as podcasts if wanted ([Rockbox auto-resume][rb-autoresume]). FiiO M11 added "playback position and track remembering" only in FW 1.0.8 ([FiiO M11][fiio-m11]); FiiO ECHO lost the resume position when playing from Favorites (V1.5.0) and ECHO NANO restored shuffle to a fixed sequence after power-on (V1.3.0) ([FiiO ECHO][fiio-echo], [FiiO ECHO NANO][fiio-echo-nano]). | Wrong (fixed): FiiO M11, ECHO, ECHO NANO. | **GP-5** The T1 shall restore queue, shuffle order, repeat mode, track and position after a pause, a power-off, a crash or a battery cut-off, losing at most a few seconds of position, and shall start paused after a cold boot unless the user chose otherwise. |
| Queue, shuffle, repeat, A-B | Visible, editable queue; "play next"/"add to end"; shuffle that starts with the chosen track; repeat off/all/one; A-B loop. | Rockbox: repeat Off/All/One/Shuffle/A-B; "Play Selected First"; "Single Mode" pauses at a track, album, artist, composer, work or genre boundary; Party Mode ([Rockbox playback options][rb-playback]). NW-A100 has a play-queue screen with the codec of each track ([NW-A100 queue][sony-queue]). FiiO M11 added a "play single once" mode in FW 1.0.8 ([FiiO M11][fiio-m11]). Spotify's post on artist-spread shuffle could not be fetched (**UNVERIFIED**). | Right: Rockbox, NW-A100 queue. | **GP-6** The T1 shall provide an editable play queue (play next, add to end, reorder, remove), shuffle that keeps the chosen first track and a stable order until reshuffled, repeat off/all/one, A-B repeat, and stop-after-current (track, album or work). |
| Speed and pitch | Independent speed and pitch for spoken word and practice; remembered per book or per folder. | Rockbox timestretch: speed 35–250 % without changing pitch, pitch 50–200 %, persisted across power cycles ([Rockbox WPS][rb-wps]). | Right: Rockbox. | **GP-7** The T1 shall offer playback speed independent of pitch for spoken-word content, shall label it as not bit-perfect, and shall never apply it to music unless the user turns it on. |
| Audiobooks, podcasts, bookmarks | Per-item resume, bookmarks, rewind a few seconds on resume, skip by time. | Rockbox bookmarks per folder or playlist, saving speed and pitch; "Rewind Before Resume" 0–60 s and "Rewind on Pause" 0–15 s for speech ([Rockbox bookmarks][rb-bookmarks], [Rockbox playback options][rb-playback]); "Skip Length" skips by time on long tracks ([Rockbox playback options][rb-playback]). NW-A100 can add the queue to Bookmarks ([NW-A100 queue][sony-queue]). | Right: Rockbox. | **GP-8** The T1 shall keep a separate resume position and user bookmarks for each audiobook and podcast item, with configurable rewind-on-resume and time-based skip. |
| Sleep timer | Duration, visible countdown, fade-out, extend by a key press. | Rockbox: 5 min to 5 h in 5-min steps, remaining time shown, optional restart on key press ([Rockbox start-up/shutdown][rb-startup]). Fade-out is not documented in that source (**UNVERIFIED**). | Right: Rockbox. | **GP-9** The T1 shall provide a sleep timer with a visible remaining time, an end-of-track option, a fade-out and extension by a button press with the screen off. |
| Pops and clicks | No click on start, stop, pause, seek, rate change, PCM/DSD change, power-on or power-off. | DoP seeking clicked on essentially all of a reporter's USB DACs because the DAC fell back to PCM on stop and re-detected DSD on start; some players were clean ([MPD #578][mpd-578]). Native DSD clicks after a player change ([MPD #262][mpd-262]); DSD pops every 10–20 s ([MPD #1096][mpd-1096]); a loud one-channel click only with software volume enabled ([MPD #813][mpd-813]). MPD plays silence before stopping DSD because some DACs emit noise ([MPD plugins][mpd-plugins]). FiiO JM21 and M21 fixed pops on the first playback after power-on and on track change ([FiiO JM21][fiio-jm21], [FiiO M21][fiio-m21]); ECHO NANO fixed a pop on resume after start-up, ECHO MINI a pop when turning off, FiiO DISC noise when pausing with EQ on ([FiiO ECHO NANO][fiio-echo-nano], [FiiO ECHO MINI][fiio-echo-mini], [FiiO DISC][fiio-disc]). Rockbox fades on stop and pause ([Rockbox playback options][rb-playback]). The CS43198 driver work item already includes waiting for the DSD soft-mute ramp before power-down ([DSD512 research][os-dsd512]). | Wrong (fixed): FiiO JM21, M21, ECHO NANO, ECHO MINI, DISC. | **GP-10** The T1 shall produce no audible click or pop at power-on, power-off, play, pause, stop, seek, track change, sample-rate change, PCM/DSD change, output change or gain change, on both jacks, and this shall be measured on the prototype (T1-026). |
| Headphone unplug and speaker | Pause when headphones are removed; never move music to the speaker by surprise. | Android guidance: on unplug, audio reroutes to the speaker and can startle the user; music players should pause ([Android output][android-output]). Rockbox: Pause on Unplug Off / Pause / Pause-and-Resume, and no auto-resume at boot without headphones ([Rockbox playback options][rb-playback]). FiiO M11 and JM21 added auto-pause after 2 min of playback with nothing plugged in, "to avoid triggering playback by mistake" ([FiiO M11][fiio-m11], [FiiO JM21][fiio-jm21]). | Right: Rockbox, later FiiO firmware. | **GP-11** When the active headphone plug is removed, the T1 shall pause within one audio buffer and shall not route the music to the integrated speaker unless the user explicitly selects the speaker; resume on re-plug shall be an option. (Needs plug detection on both jacks; board support **UNVERIFIED**.) |
| Screen-off control and pocket presses | Volume, play/pause and track change work with the screen off; a lock that leaves chosen keys active; no accidental skip. | Rockbox "Advanced Key Lock" keeps chosen actions (volume, playback, skip) active under the software lock ([Rockbox system options][rb-system]); "Prevent Track Skipping" ([Rockbox playback options][rb-playback]). NW-A100 HOLD switch locks playback and volume buttons ([NW-A100 hold][sony-hold]). Screen-off bugs fixed in changelogs: FiiO DISC rebooted when the screen came on after a screen-off volume change ([FiiO DISC][fiio-disc]); ECHO volume malfunctions with the screen off ([FiiO ECHO][fiio-echo]); M27 unresponsive volume with the screen off ([FiiO M27][fiio-m27]); JM21 and M21 play/pause lock failing after sleep, and JM21 added a setting choosing which keys lock with the screen off ([FiiO JM21][fiio-jm21], [FiiO M21][fiio-m21]). FiiO M11 made the volume bar drag-only to avoid misoperation ([FiiO M11][fiio-m11]). | Right: Rockbox, NW-A100 (hardware hold). Wrong (fixed): FiiO DISC, ECHO, M27, JM21, M21. | **GP-12** With the screen off, the four body buttons shall control volume, play/pause and track change (controls spec §1.3) with the same response as with the screen on, and the T1 shall offer a key lock, toggled from the buttons, that lets the user choose which functions stay active. The T1 has no hold switch (T1-014, T1-015). |
| Control latency | A button press is heard within a fraction of a second, also from deep idle. | No fetchable source on current DAPs' button-to-audio latency (**UNVERIFIED**). FiiO M11S fixed no response on the first Play press after sleep was disabled ([FiiO M11S][fiio-m11s]). | — | **GP-13** Volume and play/pause shall take effect within a bounded, measured latency with the screen off and after idle; the target value is to be set by the maintainers and verified on the prototype. |
| Previous-track behaviour | "Previous" restarts the track after a few seconds, goes back before. | FiiO ECHO NANO changed "previous" to restart the current track only if under 3 s had played (V1.4.0, [FiiO ECHO NANO][fiio-echo-nano]); Rockbox "Rewind Across Tracks" within the first 3 s ([Rockbox playback options][rb-playback]). | Right: Rockbox. | Covered by GP-6 (behaviour to specify in the UI spec). |

## 2. Signal path

| Item | What good looks like | Documented evidence and complaints | Right / wrong | Candidate requirement |
|---|---|---|---|---|
| Bit-perfect, and proving it | The samples in the file reach the DAC unchanged, and the user can see it. | Android resamples in its mixer (e.g. 44.1 kHz content on a 48 kHz device), with resampler quality as a build setting ([Android SRC][android-src]); a true bit-perfect USB path only arrived in Android 14, optional, AIDL HAL only, with no software volume ([Android bit-perfect][android-bitperfect]). PipeWire runs its graph at 48 kHz by default; rate switching when idle is "not enabled by default for now" ([PipeWire conf][pw-conf]). MPD is bit-perfect by default; software volume, ReplayGain, forced format and resampling break it; users verify with `/proc/asound/card*/pcm*p/sub*/hw_params` because ALSA may "fool" the player ([MPD manual][mpd-user]). NW-A100 makes "Direct Source" all-or-nothing: it disables every other sound adjustment ([NW-A100 sound][sony-sound]). The T1 board can capture the I2S/DSD lines passively (BRD-040, BRD-041). | Right: MPD (documented checks). Wrong by design: Android's default mixer path. | **SP-1** The T1 shall show, on the now-playing screen and through the command-line tool, the exact source format, the format sent to the DAC and every processing step in between, with a single "bit-perfect" indicator that is true only when the stream is unmodified; a capture-based self-test (BRD-040/041) shall prove it. |
| Rate switching | Every rate at its native clock; no forced 48 kHz; no pop on the change. | The T1 already plans two master clocks so both families play without resampling (BRD-023). USB DACs need per-device delays at interface setup (`iface_delay`, 50 ms) or control messages (20 ms, 1–2 ms, 5–6 ms) through `snd-usb-audio` quirks ([kernel ALSA config][k-alsa]). | — | Covered by GP-2 and SP-1. |
| DSD | Native DSD where the DAC supports it; DoP only where needed; DSD→PCM labelled. | DoP v1.1 carries DSD64 as 24-bit PCM at 176.4 kHz with 0x05/0xFA markers; DSD128 needs 352.8 kHz; a DAC that misreads DoP as PCM outputs a tone around 88 kHz at about −34 dB, and PCM misread as DSD "may create less predictable clicks" ([DoP 1.1][dop]). MPD uses native DSD automatically and DoP only if enabled, since it cannot detect DAC support ([MPD manual][mpd-user]). Windows' UAC2 class driver lists PCM, float and IEC61937 but no DSD format ([Microsoft UAC2][ms-uac2]). Volume on DSD requires conversion or a DAC-side control ([Wikipedia: DSD][wp-dsd]); the CS43198 driver exposes a separate "Master DSD Playback Volume" ([cs43130.c][k-cs43130]). NW-A100 plays DSD files but outputs them "converted to Linear PCM" ([NW-A100 formats][sony-formats]). T1 scope: native DSD to DSD256 and DoP to DSD64 on-board; DSD512 converted and labelled; native over USB ([OS-020][os-req], [DSD512 research][os-dsd512]). | Wrong: NW-A100 (DSD as PCM, stated). | **SP-2** The T1 shall play DSD natively whenever the output supports it, shall use DoP only as a fallback and only towards a DAC known to accept it, shall never send DoP to a PCM-only DAC, and shall label every DSD→PCM conversion with its source and target rates. |
| Volume: where and how fine | Attenuation in the DAC or analog domain at high internal resolution, fine steps near the bottom, perfect channel balance at low level, remembered per output. | The CS43198 driver declares PCM and DSD volume as a dB scale from 0 to −127.5 dB with mute in 0.5 dB steps (reading of `DECLARE_TLV_DB_SCALE(…, -12750, 50, 1)` in [cs43130.c][k-cs43130]; the step size is our reading, not stated in prose). Rockbox volume is in dB with a user volume limit ([Rockbox sound settings][rb-sound]). ADR 0012 sets the level from a fixed-gain amplifier, so the volume control is the DAC's digital attenuator. Low-volume channel imbalance of analog potentiometers: no fetched source (**UNVERIFIED**). FiiO DISC added volume memory for USB audio mode ([FiiO DISC][fiio-disc]). | — | **SP-3** The T1 shall control volume inside the DAC (CS43198 PCM and DSD volume), in steps no coarser than the DAC provides, with identical left and right attenuation at every step, a remembered level per output (3.5 mm, 4.4 mm, speaker, USB, Bluetooth) and a safe default level after a power cycle. |
| Hiss with sensitive IEMs | Inaudible noise floor at the lowest gain. | ASR users choosing small DAPs treat the output noise into sensitive IEMs as a selection criterion and compare published noise figures (user reports, [ASR low-noise DAP][asr-lownoise], [ASR "dumb DAP"][asr-dumbdap]). T1 noise into IEMs is not measured (**UNVERIFIED**). | — | **SP-4** The T1 shall publish its measured output noise on both jacks at the lowest volume and into a sensitive-IEM load before claiming suitability for IEMs (T1-026). |
| Gain modes and output power | Enough voltage for high-impedance headphones; low gain for IEMs. | User report: a HiBy R6 III and iFi Hip-DAC 2 give "enough volume but not enough dynamics" with a 300 Ω headphone ([ASR dynamics][asr-dynamics]). ADR 0012 fixes amplifier gain at about 1.6 with a temporary output ceiling on the interim battery ([ADR 0012][brd-0012]). Whether the T1 has selectable analog gain is not decided. | — | **SP-5** Any output ceiling applied for power safety shall be shown to the user with its reason (T1-045); whether a selectable low-gain mode is needed for IEMs is an open question for the board. |
| Output impedance | Low enough not to colour multi-driver IEMs. | Rule of thumb: output impedance below 1/8 of the headphone impedance; on an IEM swinging 10–90 Ω, 43 Ω produced a 12 dB response variation and 10 Ω still "a very audible 6 dB" ([NwAvGuy][nwavguy]). | — | **SP-6** The T1 shall publish the measured output impedance of each jack and keep it low enough to meet the 1/8 rule for common multi-driver IEMs (target to be set from measurement). |
| Line out | A fixed, documented level that bypasses the volume control. | Rockbox documents a fixed line-out level setting ([Rockbox sound settings][rb-sound]). The T1 has no dedicated line-out jack (T1-016). | — | **SP-7** The T1 should offer a fixed-level mode on its headphone outputs (volume at 0 dB, user confirmation before enabling, never by default) for use with an external amplifier, guarded by SYS-012. |
| Balanced vs single-ended | Both outputs equally good; the active one obvious. | The T1 has 3.5 mm SE and 4.4 mm balanced (BRD-024). The NW-A100 page set has no per-output volume memory statement (**UNVERIFIED**). | — | Covered by SP-3 (volume per output). |
| EQ | Parametric EQ with preamp and clipping protection; importable headphone profiles; per-headphone presets; off means off. | AutoEq produces parametric, ten-band and convolution settings from oratory1990, crinacle and others, but "does not do the equalization itself" ([AutoEq][autoeq]); its results cover 6033 headphone models ([AutoEq results][autoeq-results]). HiBy R6 III: system-wide PEQ added by firmware, resolution fits oratory1990 profiles, but the negative preamp range was about 1 dB short for one user's profile; one user called it the best Android EQ they had used, after a third-party EQ "that would randomly not work" (user reports, [ASR R6 III PEQ][asr-r6iii]). FiiO M11S and M17 received AutoEQ only in 2026 firmware ([FiiO M11S][fiio-m11s], [FiiO M17][fiio-m17]). Rockbox EQ: low shelf, peaking bands, high shelf ([Rockbox sound settings][rb-sound]). NW-A100 keeps up to 3 sound presets ([NW-A100 sound][sony-sound]). | Right: HiBy R6 III (mostly). | **SP-8** The T1 shall provide a parametric EQ that imports AutoEq parametric profiles, with an automatic preamp derived from the profile's maximum boost so it cannot clip, presets assigned per headphone, and a bypass that restores a bit-perfect path. |
| Crossfeed | Optional, standard presets. | bs2b presets: default 700 Hz / 4.5 dB, Chu Moy 700 Hz / 6 dB, Jan Meier 650 Hz / 9.5 dB ([bs2b][bs2b]); Rockbox crossfeed with direct and cross gain and high-frequency attenuation ([Rockbox sound settings][rb-sound]). HiBy R6 III's native crossfeed was called "a little buggy" (user report, [ASR R6 III PEQ][asr-r6iii]). | Right: Rockbox. | **SP-9** The T1 should offer crossfeed with the standard bs2b presets, off by default and labelled as processing. |
| DAC filters | User-selectable DAC filters, explained. | The CS43198 driver exposes PCM filter speed (Fast/Slow), phase compensation, non-oversampling emulation, high-pass and de-emphasis, and DSD phase modulation ([cs43130.c][k-cs43130]). | — | **SP-10** The T1 shall expose every DAC filter and DSD option the CS43198 driver provides (T1-045), with a plain description and the measured response of each. |
| Dither and resampling | When processing is needed, do it at high resolution, dither on requantisation, use a documented high-quality resampler. | Rockbox: "Dithering adds low-level noise to the signal prior to throwing away the surplus bits" ([Rockbox sound settings][rb-sound]). libsoxr gives "very high quality results" with configurable passband, rejection and phase ([soxr][soxr]); MPD: resampling "reduces the quality" and is often the main CPU cost ([MPD manual][mpd-user]). | Right: Rockbox, MPD (documented). | **SP-11** Whenever the T1 must process audio (EQ, ReplayGain in software, speed, DSD→PCM, Bluetooth), it shall process at high internal resolution, dither on requantisation, use a documented resampler, and publish its filter specification. |
| MQA | No promise built on a declining format. | MQA Ltd entered administration on 2023-04-03; Lenbrook acquired its assets on 2023-09-19; Tidal replaced MQA with FLAC on 2024-07-24; tests and critics describe it as lossy and DRM-like ([Wikipedia: MQA][wp-mqa]). FiiO M23 still advertises MQA unfolding ([FiiO M23][fiio-m23]); NW-A100 lists MQA ([NW-A100 formats][sony-formats]). | — | **SP-12** The T1 shall play MQA files as the ordinary PCM they contain; MQA decoding is not a goal (decision for the maintainers). |

## 3. Library

| Item | What good looks like | Documented evidence and complaints | Right / wrong | Candidate requirement |
|---|---|---|---|---|
| Scale | 100k+ tracks without a ceiling or slowdown. | NW-A100 is "confirmed to operate normally with up to a total of 20,000 tracks" ([NW-A100 capacity][sony-capacity]). MPD's RAM was uncapped when building a database from scratch (issue title only, [MPD #902][mpd-902]); `metadata_to_use` reduces database size ([MPD manual (latest)][mpd-user-latest]). A custom-firmware developer for Astell&Kern claims a database rebuild nearly 50 % faster than stock (developer claim, [ASR AKCFW][asr-akcfw]). | Wrong: NW-A100 (20,000-track ceiling). | **LB-1** The T1 library shall support at least 200,000 tracks across internal storage and the card with bounded memory use, and scan time and browse latency at that size shall be measured and published. (Target to confirm.) |
| Scanning | Automatic, incremental, in the background, resumable, never blocking playback. | NW-A100 has no automatic scan: new files "may not appear in the library" until the user runs "Update database" ([NW-A100 update][sony-update]). Rockbox's first scan runs in the background and resumes across reboots, with folder exclusion (`database.ignore`) and separate update vs rebuild ([Rockbox database][rb-tagcache]). FiiO ECHO MINI could not update its library on some microSD cards (V2.8.0, [FiiO ECHO MINI][fiio-echo-mini]); HiBy R3 firmware 1.14 broke microSD recognition, fixed in 1.15 ([HiBy downloads][hiby-dl]). | Right: Rockbox. Wrong: NW-A100, FiiO ECHO MINI, HiBy R3 1.14. | **LB-2** The T1 shall update its library automatically and incrementally when storage changes, in the background, resumable after a reboot, without interrupting playback, and shall report files it could not read with the reason. |
| Tags | All standard fields in ID3v2.3, ID3v2.4, Vorbis comments, MP4 and APEv2; multi-value fields kept. | Vorbis comments repeat a field for multiple values and use UTF-8 ([Vorbis comment][xiph-vc]); ID3v2.4 separates values with a null byte and allows UTF-8, while v2.3 (UTF-16) is "the most widely used version", so a player must read both ([Wikipedia: ID3][wp-id3]). Picard's mapping defines album artist, multi-value artists, composer, work, movement name and number, compilation, disc number and total, sort fields (with the v2.3/v2.4 difference for album-artist sort), original date and lyrics across formats ([Picard tag mapping][picard]). Rockbox limits metadata strings to 90–500 bytes depending on device ([Rockbox file formats][rb-formats]). FiiO ECHO sorted m4a albums incorrectly (V1.6.0, [FiiO ECHO][fiio-echo]). Legacy-encoded CJK tags (Shift JIS, GBK): no fetched source (**UNVERIFIED**). | Right: Picard mapping as reference. Wrong (fixed): FiiO ECHO. | **LB-3** The T1 shall read the full Picard tag mapping in every supported container, keep multi-value fields as separate values, use sort tags for ordering and album artist for grouping, and shall not truncate tag text. |
| Classical | Browse by composer, work and movement; show the work title over its movements. | Picard defines WORK, MOVEMENTNAME, MOVEMENT and SHOWMOVEMENT across formats ([Picard tag mapping][picard]); Rockbox "Single Mode" can stop at a work or composer boundary ([Rockbox playback options][rb-playback]). | Right: Rockbox. | **LB-4** The T1 shall browse by composer, work and movement and group movements under their work. |
| Discs and compilations | Multi-disc albums as one album in disc order; compilations under one album. | Disc number and total, and the compilation flag, are standard fields ([Picard tag mapping][picard]). | — | Covered by LB-3. |
| CUE sheets | Single-file albums shown as tracks, gapless between them; external and embedded. | A CD can be one file plus a cue sheet; INDEX 01 is the track start, INDEX 00 the optional pregap; times in 75 frames per second; some non-standard sheets are rejected by players ([Wikipedia: Cue sheet][wp-cue]). FLAC defines a CUESHEET block ([RFC 9639][rfc9639]); MPD reads external and embedded cue sheets ([MPD plugins (latest)][mpd-plugins-latest]). Rockbox needs the same file name, works only with some tag types and needs a reboot after changing the setting ([Rockbox playback options][rb-playback], [Rockbox file formats][rb-formats]). DSDIFF may store a whole album as one stream with markers and has no native ID3 ([Wikipedia: DSD][wp-dsd]). | Right: MPD. Partial: Rockbox. | **LB-5** The T1 shall present external and embedded CUE sheets (including pregaps) and DSDIFF track markers as individual tracks that play gaplessly, and shall tolerate common non-standard sheets. |
| Artwork | Embedded and folder art, the best available resolution, scaled once. | FLAC PICTURE blocks and ID3 APIC carry typed embedded images, several per file ([RFC 9639][rfc9639], [Wikipedia: ID3][wp-id3]). How current DAPs choose between embedded and folder art, and at what resolution: no fetched source (**UNVERIFIED**). NW-A100 shows no art on playlists ([NW-A100 playlists][sony-playlists]). | — | **LB-6** The T1 shall show embedded front-cover art first and folder images (`cover`, `folder`, `front`) as a fallback, scaled to the display without visible artefacts and cached. |
| Lyrics | Synced LRC, embedded lyrics, offset, enhanced word timing. | LRC: `[mm:ss.xx]` stamps, several per line, an `offset` tag, enhanced word-level timing ([Wikipedia: LRC][wp-lrc]). NW-A100 accepts only side-car UTF-8 LRC up to 512 KB with lines of 40 single-byte characters; embedded lyrics are not documented ([NW-A100 LRC][sony-lrc], [NW-A100 lyrics][sony-lyrics]). Synced lyrics have a tag only in ID3 (SYLT) ([Picard tag mapping][picard]). | Partial: NW-A100. | **LB-7** The T1 shall display synced LRC (side-car and embedded), honour the offset tag and enhanced timing, and fall back to unsynced embedded lyrics. |
| Folder browsing vs database | Both, consistent with each other. | Rockbox keeps them separate, and one feature only works from the file browser ([Rockbox playback options][rb-playback]). Users looking for simple DAPs ask for folder browsing and microSD (user reports, [ASR "dumb DAP"][asr-dumbdap]). | Partial: Rockbox. | **LB-8** The T1 shall offer folder browsing and tag browsing over the same files, with every playback feature available from both. |
| Playlists | Portable M3U/M3U8 with relative paths, editable on the device, across internal and card storage. | M3U8 is UTF-8; entries can be relative paths; careless parsing caused vulnerabilities in other players ([Wikipedia: M3U][wp-m3u]). NW-A100: playlists only through Music Center for PC, "You cannot edit playlists with 'Music player'", and one playlist cannot span internal memory and microSD ([NW-A100 playlists][sony-playlists]). FiiO ECHO NANO cleared Favorites stored on microSD when switching drives (V1.3.0, [FiiO ECHO NANO][fiio-echo-nano]). | Wrong: NW-A100, FiiO ECHO NANO (fixed). | **LB-9** The T1 shall read and write M3U and M3U8 with relative paths, edit playlists on the device, allow entries from any storage, and keep favourites and playlists when a card is removed and reinserted. |
| Smart playlists, ratings, play counts | Ratings and counts that survive rescans and can leave the device. | Rockbox records play count, play time, last played, rating and autoscore and exports them to a text file that survives database changes ([Rockbox database][rb-tagcache]). MPD keeps ratings in a separate "sticker" store ([MPD manual (latest)][mpd-user-latest]). | Right: Rockbox. | **LB-10** The T1 shall keep ratings, play counts and last-played dates across rescans, export and import them in an open text format, and build smart playlists from them and from tags. |
| Formats | Every common lossless and lossy format, no file-size or rate surprise. | NW-A100: FLAC, ALAC, WAV and AIFF up to 384 kHz, APE up to 192 kHz, AAC only to 48 kHz, files of 4 GB or more (APE 2 GB) not playable ([NW-A100 formats][sony-formats]). Monkey's Audio at high compression is much more demanding to decode and is BSD-3-Clause since 2023 ([Wikipedia: Monkey's Audio][wp-ape]). DSF has native ID3 tags; DSDIFF does not and may use DST compression ([Wikipedia: DSD][wp-dsd]). Opus and WavPack are listed by Rockbox and MPD ([Rockbox file formats][rb-formats], [MPD plugins (latest)][mpd-plugins-latest]). | Wrong: NW-A100 (4 GB limit). | **LB-11** The T1 shall play FLAC, ALAC, WAV (including > 4 GB and RF64), AIFF, APE (all compression levels), WavPack (including hybrid and DSD), DSF, DFF (including DST), Opus, Vorbis, MP3 and AAC, and shall state for each format the rates it plays bit-perfect on each output. |
| Cards and exFAT | Any card the user formatted, any size, exFAT and FAT32; safe removal. | SDXC (to 2 TB) and SDUC (to 128 TB) default to exFAT ([SD Association][sda-cap]). exFAT is in mainline Linux since 5.4 and is not journaled, so removal during a write risks corruption ([Wikipedia: exFAT][wp-exfat]). NW-A100: "Use a microSD card that has been formatted on the player. Sony does not guarantee the operation of other microSD cards" ([NW-A100 microSD][sony-microsd]). | Wrong: NW-A100. | **LB-12** The T1 shall use exFAT and FAT32 cards formatted on any computer, of any capacity the SD standard defines, shall flush writes promptly and shall recover a card removed during a write without losing the library. |

## 4. Connectivity

| Item | What good looks like | Documented evidence and complaints | Right / wrong | Candidate requirement |
|---|---|---|---|---|
| USB DAC mode (T1 as DAC) | Driverless on every host, all rates the on-board path plays, DSD, volume behaviour defined. | The Linux UAC2 gadget defines rates, sample size, channel mask, volume and mute controls and async feedback bandwidth in configfs ([kernel gadget testing][k-gadget]). Windows' class driver (since 10 1703) supports PCM to 32 bit and float, no DSD format, explicit feedback only for async OUT, one clock source ([Microsoft UAC2][ms-uac2]), so DSD from Windows would need DoP. FiiO ECHO MINI's USB DAC mode is "driver-free, 48kHz sampling" ([FiiO ECHO MINI][fiio-echo-mini]); FiiO JM21 claims 384 kHz/32-bit and native DSD256 ([FiiO JM21 product][fiio-jm21-prod]); FiiO DISC added volume memory and compatibility fixes for USB audio mode ([FiiO DISC][fiio-disc]). | Limited: ECHO MINI (48 kHz). | **CN-1** In USB DAC mode the T1 shall work driverless with Linux, macOS and Windows hosts, accept every PCM rate its on-board path plays bit-perfect and DSD through DoP, apply SYS-012 and remember its own volume. |
| USB transport (T1 as host) | Bit-perfect PCM and native DSD to external DACs, per-device quirks, rate switching without loss. | `snd-usb-audio` provides `dsd_raw` for raw DSD and delay quirks for slow devices ([kernel ALSA config][k-alsa]); MPD's native-DSD / DoP / DSD→PCM order ([MPD manual (latest)][mpd-user-latest]); DoP seek clicks on many USB DACs ([MPD #578][mpd-578]). T1 plan: native DSD over USB, DoP fallback, a compatibility list built from tests ([DSD512 research][os-dsd512]). | — | **CN-2** As a USB host, the T1 shall stream bit-perfect PCM and native DSD up to the external DAC's limits, with DoP as fallback, and shall publish a tested DAC compatibility list. |
| Bluetooth transmit | LDAC, aptX family, AAC, LE Audio; the codec and bitrate in use shown. | PipeWire/WirePlumber lists SBC, SBC-XQ, AAC, LDAC, aptX, aptX HD, aptX LL, FastStream, LC3plus, Opus and LC3, with LDAC quality selectable ([WirePlumber Bluetooth][wp-bt]). LDAC runs at 330/660/990 kbps or adaptive ([Sony LDAC][ldac]). LE Audio's LC3 claims better quality than SBC at half the bitrate, plus Auracast broadcast ([Bluetooth SIG LE Audio][bt-leaudio]). Bluetooth problems recur in changelogs: ECHO MINI popping and reconnection, DISC repeated compatibility fixes ([FiiO ECHO MINI][fiio-echo-mini], [FiiO DISC][fiio-disc]); HiBy R2 II "Bluetooth is really bad" (user report, [ASR "dumb DAP"][asr-dumbdap]). | Wrong: HiBy R2 II (user report); FiiO ECHO MINI, DISC (fixed). | **CN-3** The T1 shall transmit Bluetooth audio with every codec its radio and the standard Linux stack support, show the codec, rate and bitrate in use, let the user pin LDAC quality, reconnect to the last device automatically, and label Bluetooth output as not bit-perfect. |
| Bluetooth receive | The T1 as a high-quality Bluetooth receiver for a phone. | A2DP sink and BAP sink roles exist in the standard stack ([WirePlumber Bluetooth][wp-bt]); an LDAC sink requires a Sony licence ([Sony LDAC][ldac]). FiiO JM21 receives SBC, AAC and LDAC ([FiiO JM21 product][fiio-jm21-prod]); FiiO DISC added power-saving shutdown in receive mode ([FiiO DISC][fiio-disc]). | Right: FiiO JM21. | **CN-4** The T1 should act as a Bluetooth audio receiver with the codecs the standard stack allows without a licence, and LDAC if a licence is obtained (maintainers' decision). |
| Network renderers | Play from the user's own server and phone over Wi-Fi. | UPnP/DLNA renderer support is asked for by IEM users choosing small DAPs; HiBy R1 is noted for its DLNA renderer (user reports, [ASR low-noise DAP][asr-lownoise]). AirPlay 2 receiver: open-source shairport-sync takes 24-bit/48 kHz lossless, not 96/192 kHz, and has no Apple certification ([shairport-sync][shairport], [AirPlay 2 notes][shairport-ap2]). Roon Ready is a closed certification with no public SDK ([Roon Ready][roon]). Spotify Connect hardware needs a partner application, NDA and certification, and the page says new applications are currently limited to voice-assistant integrations ([Spotify hardware][spotify-hw]); the open-source librespot is unofficial ("probably forbidden") ([librespot][librespot]). Qobuz and Tidal Connect: no page fetched (**UNVERIFIED**). | — | **CN-5** The T1 shall act as a UPnP/DLNA (and OpenHome) renderer on its own network; proprietary protocols (AirPlay, Roon Ready, Spotify Connect, Tidal or Qobuz Connect) depend on certification programmes and are a maintainers' decision. |
| Network shares | Play from SMB and NFS shares directly. | MPD has SMB (disabled by default because of libsmbclient bugs), NFS and UPnP storage ([MPD plugins (latest)][mpd-plugins-latest]); slow browsing of NAS folders and full re-adds on update are open issues (titles only, [MPD #2211][mpd-2211], [MPD #1768][mpd-1768]). | — | **CN-6** The T1 shall index and play SMB and NFS shares, with incremental updates and offline caching where storage allows (OS-022). |
| File transfer | Fast, works on macOS without extra software, preserves timestamps. | MTP: one operation at a time, in-place edits need a full copy, no macOS support, modification times replaced by the copy time ([Wikipedia: MTP][wp-mtp]). USB mass storage exports a block device the device itself cannot use meanwhile ([kernel mass storage][k-msc]); Rockbox requires mass-storage mode on some players to install ([Rockbox installation][rb-install]). BRD-034 already lists file transfer over the main USB-C (USB 2.0, BRD-044). | Wrong: MTP-only devices on macOS. | **CN-7** The T1 shall offer file transfer that works on Linux, macOS and Windows without extra software, preserves file timestamps, and keeps playing from the other storage while one is exported. Transfer speed over the main USB-C shall be measured and published. |
| Car use | Starts playing on power, resumes, remembered volume, USB to a car head unit. | No fetched source (**UNVERIFIED**). | — | **CN-8** The T1 should support a car profile: resume on external power, a fixed volume for the head unit, and Bluetooth auto-reconnect (to be researched). |

## 5. Device

| Item | What good looks like | Documented evidence and complaints | Right / wrong | Candidate requirement |
|---|---|---|---|---|
| Battery claims vs reality | Claims with stated conditions, reproducible. | FiiO JM21 claims 12.5 h SE / 9.5 h balanced (3 GB model) without stated test conditions ([FiiO JM21 product][fiio-jm21-prod]); HiBy R4 claims 11 h SE and "over 675 Hours Standby" ([HiBy R4][hiby-r4]). Measured runtimes vs claims: no fetched source (**UNVERIFIED**). FiiO ECHO NANO showed three bars after a full charge (V1.4.0, [FiiO ECHO NANO][fiio-echo-nano]). Rockbox estimates runtime from the configured capacity ([Rockbox system options][rb-system]). T1 defines its own session thresholds (T1-039). | — | **DV-1** Every battery figure the T1 publishes shall state its test conditions (format, output, load, volume, screen, radios) and come from a repeatable measurement; the UI shall show remaining time for the current conditions. |
| Standby drain | Days to weeks in standby, measured. | HiBy R4 standby claim ([HiBy R4][hiby-r4]); real standby drain: no fetched source (**UNVERIFIED**). Deep suspend on the RK3566 depends on mainline TF-A, not verified ([OS requirements][os-req]). | — | **DV-2** The T1's standby drain shall be measured and published, and a low-battery shutdown shall preserve the playback state (GP-5). |
| Charging while playing | No noise, no throttling surprises. | Rockbox lets the user choose whether to charge from a USB connection ([Rockbox system options][rb-system]). Noise during charging on current DAPs: no fetched source (**UNVERIFIED**). | — | **DV-3** The T1 shall play while charging with no audible change in noise, and charge-current limits for heat or battery care shall be user-visible (T1-045). |
| Heat | Cool in a pocket during long sessions. | No fetched source (**UNVERIFIED**). BRD-010 requires temperature monitoring. | — | **DV-4** Thermal limits shall reduce charge current or processing before they affect audio, and any such limit shall be shown to the user. |
| Boot time and responsiveness | Music within seconds of power-on; no UI stalls during scans. | No fetched source for boot times (**UNVERIFIED**). Freezes fixed in changelogs: FiiO DISC (system, UI and playback freezes, V1.86; unexpected restarts, V2.40), ECHO MINI (freeze opening Favorites, V3.6.0) ([FiiO DISC][fiio-disc], [FiiO ECHO MINI][fiio-echo-mini]). | Wrong (fixed): FiiO DISC, ECHO MINI. | **DV-5** Time from power-on to resumed playback shall be measured and published, and library scans and network activity shall never stall the UI or playback. |
| Firmware updates and abandonment | Regular fixes, no regressions, long support. | FiiO M17 firmware went from 1.0.6 (2022-12-28) to 1.0.7 (2026-06-30) with nothing in between, as listed on FiiO's page ([FiiO M17][fiio-m17]). HiBy R3 firmware 1.14 broke microSD recognition ([HiBy downloads][hiby-dl]). A custom-firmware developer writes that Astell&Kern "software issues remain unresolved for years" (developer claim, [ASR AKCFW][asr-akcfw]). Rockbox exists because of manufacturer firmware limits ([Wikipedia: Rockbox][wp-rockbox]). Discontinued FiiO M3K and M3 Pro are mentioned as lost options (user reports, [ASR "dumb DAP"][asr-dumbdap]). The T1 already requires signed releases, A/B rollback and open source (OS-011, OS-017, T1-001). | Wrong: Astell&Kern (developer claim), FiiO M17 cadence. | **DV-6** Every T1 release shall pass an automated regression suite covering this document's playback, signal-path and library items before release, and shall publish a changelog that names fixed user-visible problems. |
| Volume knob vs buttons | Fast, precise, impossible to change by accident in a pocket. | The T1 has Volume + and − buttons with key repeat (T1-014, controls spec §1.3); no knob. Accidental volume changes with the screen off and lock failures appear in FiiO changelogs (see GP-12). | — | Covered by GP-12 and SP-3. |
| Hearing safety | Warnings and a dose limit that do not surprise or nag. | Sony's EU/KR behaviour: a warning above the safe level, repeated every 20 cumulative hours at that level, after which the volume is lowered automatically; volume starts lower after a power cycle ([NW-A100 volume][sony-volume]). The T1 requires a seven-day dose budget, 85 dB(A), per-headphone profiles (SYS-012). | Reference: Sony. | **DV-7** The SYS-012 dose shall be shown as a remaining budget, any automatic reduction shall be announced before it happens and be gradual, and the per-headphone profile shall be linked to the EQ preset (SP-8). |
| Durability and repairability | User-replaceable battery, standard parts, documented repair. | EU Regulation 2023/1542 (portable batteries) could not be fetched; its replaceability article and dates are **UNVERIFIED**. iFixit DAP battery guides: none found (**UNVERIFIED**). The T1 already requires a standard commercial cell (T1-012). | — | **DV-8** The T1's battery shall be replaceable by the user with common tools and an orderable standard cell, with a published repair guide. |

## 6. Top candidate requirements

The twenty with the widest evidence or the strongest tie to T1-040 and T1-045, in priority order. IDs are local to this document.

| # | ID | Candidate |
|---|---|---|
| 1 | SP-1 | Show source format, DAC format and every processing step, with a bit-perfect indicator proven by the capture self-test. |
| 2 | GP-1 | Sample-exact gapless for same-format tracks, honouring encoder delay and padding, tested bit-exact every release. |
| 3 | GP-10 | No click or pop at any start, stop, seek, rate, format, output or gain change, measured on the prototype. |
| 4 | GP-2 | Rate, family or PCM/DSD changes mute and reconfigure without clicks or lost first samples. |
| 5 | SP-2 | Native DSD whenever possible, DoP only as fallback to known-capable DACs, conversions labelled. |
| 6 | SP-3 | Volume inside the DAC, fine steps, identical channels, remembered per output, safe level after power cycle. |
| 7 | GP-5 | Restore queue, shuffle order, repeat, track and position after pause, power-off, crash or battery cut. |
| 8 | GP-12 | Full button control with the screen off and a configurable key lock (the T1 has no hold switch). |
| 9 | GP-11 | Pause on plug removal and never route music to the speaker by surprise. |
| 10 | GP-4 | ReplayGain/R128 with preamp and clipping prevention, applied in the DAC volume to stay bit-perfect. |
| 11 | SP-8 | Parametric EQ importing AutoEq profiles, automatic clip-safe preamp, per-headphone presets, true bypass. |
| 12 | LB-1 | Library of at least 200,000 tracks with bounded memory, measured scan and browse times. |
| 13 | LB-2 | Automatic, incremental, background, resumable scans that never interrupt playback, with unreadable files reported. |
| 14 | LB-3 | Full tag mapping, multi-value fields, sort and album-artist tags, no truncation. |
| 15 | LB-11 | All common formats including > 4 GB WAV, APE, WavPack, DSF and DFF/DST, with bit-perfect rates stated per output. |
| 16 | LB-12 | exFAT/FAT32 cards formatted anywhere, any capacity, safe against removal during writes. |
| 17 | LB-9 | Portable M3U/M3U8 with relative paths, edited on device, across storages, kept when the card is reinserted. |
| 18 | CN-2 | USB host: bit-perfect PCM and native DSD to external DACs, DoP fallback, published compatibility list. |
| 19 | CN-7 | File transfer that works on macOS, Linux and Windows without extra software and preserves timestamps. |
| 20 | DV-6 | Automated regression suite over these behaviours before every release, with a user-facing changelog. |

## 7. Open questions for the maintainers

1. Plug detection on both jacks (GP-11): does the board provide it? (**UNVERIFIED**.)
2. A selectable low-gain mode for sensitive IEMs (SP-5) versus the fixed amplifier gain of ADR 0012.
3. Whether to pursue proprietary network and streaming certifications (CN-5) and an LDAC sink licence (CN-4).
4. Numeric targets: library size (LB-1), control latency (GP-13), boot-to-music time (DV-5).

## 8. Sources

Every source below was fetched on 2026-10-22, except the OpenDAP repository links, which point to the published copies of files read locally in this workspace.

- `wp-rockbox`: <https://en.wikipedia.org/wiki/Rockbox>
- `wp-gapless`: <https://en.wikipedia.org/wiki/Gapless_playback>
- `rfc7845`: <https://www.rfc-editor.org/rfc/rfc7845.html>
- `m3-3425`: <https://github.com/androidx/media/issues/3425>
- `m3-3200`: <https://github.com/androidx/media/pull/3200>
- `m3-2576`: <https://github.com/androidx/media/issues/2576>
- `mpd-1140`: <https://github.com/MusicPlayerDaemon/MPD/issues/1140>
- `mpd-2588`: <https://github.com/MusicPlayerDaemon/MPD/issues/2588>
- `mpd-1599`: <https://github.com/MusicPlayerDaemon/MPD/issues/1599>
- `mpd-user`: <https://mpd.readthedocs.io/en/stable/user.html>
- `mpd-user-latest`: <https://mpd.readthedocs.io/en/latest/user.html>
- `mpd-plugins`: <https://mpd.readthedocs.io/en/stable/plugins.html>
- `mpd-plugins-latest`: <https://mpd.readthedocs.io/en/latest/plugins.html>
- `mpd-2592`: <https://github.com/MusicPlayerDaemon/MPD/pull/2592>
- `mpd-578`: <https://github.com/MusicPlayerDaemon/MPD/issues/578>
- `mpd-262`: <https://github.com/MusicPlayerDaemon/MPD/issues/262>
- `mpd-1096`: <https://github.com/MusicPlayerDaemon/MPD/issues/1096>
- `mpd-813`: <https://github.com/MusicPlayerDaemon/MPD/issues/813>
- `mpd-902`: <https://github.com/MusicPlayerDaemon/MPD/issues/902>
- `mpd-2211`: <https://github.com/MusicPlayerDaemon/MPD/issues/2211>
- `mpd-1768`: <https://github.com/MusicPlayerDaemon/MPD/issues/1768>
- `rb-playback`: <https://raw.githubusercontent.com/Rockbox/rockbox/master/manual/configure_rockbox/playback_options.tex>
- `rb-autoresume`: <https://raw.githubusercontent.com/Rockbox/rockbox/master/manual/configure_rockbox/autoresume.tex>
- `rb-bookmarks`: <https://raw.githubusercontent.com/Rockbox/rockbox/master/manual/configure_rockbox/bookmarking.tex>
- `rb-wps`: <https://raw.githubusercontent.com/Rockbox/rockbox/master/manual/rockbox_interface/wps.tex>
- `rb-startup`: <https://raw.githubusercontent.com/Rockbox/rockbox/master/manual/configure_rockbox/startup_shutdown_options.tex>
- `rb-sound`: <https://raw.githubusercontent.com/Rockbox/rockbox/master/manual/configure_rockbox/sound_settings.tex>
- `rb-system`: <https://raw.githubusercontent.com/Rockbox/rockbox/master/manual/configure_rockbox/system_options.tex>
- `rb-tagcache`: <https://raw.githubusercontent.com/Rockbox/rockbox/master/manual/rockbox_interface/tagcache.tex>
- `rb-formats`: <https://raw.githubusercontent.com/Rockbox/rockbox/master/manual/appendix/file_formats.tex>
- `rb-install`: <https://raw.githubusercontent.com/Rockbox/rockbox/master/manual/getting_started/installation.tex>
- `wp-rg`: <https://en.wikipedia.org/wiki/ReplayGain>
- `ebu-r128`: <https://tech.ebu.ch/docs/r/r128.pdf>
- `fiio-m11`: <https://forum.fiio.com/note/showNoteContent.do?id=202105171708309850783&tid=17>
- `fiio-m11s`: <https://forum.fiio.com/note/showNoteContent.do?id=202212281000205937102&tid=136>
- `fiio-m17`: <https://forum.fiio.com/note/showNoteContent.do?id=202112241051165653057&tid=123>
- `fiio-jm21`: <https://forum.fiio.com/note/showNoteContent.do?id=202412181641298410115&tid=17>
- `fiio-m21`: <https://forum.fiio.com/note/showNoteContent.do?id=202505140958060216556&tid=17>
- `fiio-m27`: <https://forum.fiio.com/note/showNoteContent.do?id=202510291436497507515&tid=17>
- `fiio-echo`: <https://forum.fiio.com/note/showNoteContent.do?id=202602051101498161611>
- `fiio-echo-mini`: <https://forum.fiio.com/note/showNoteContent.do?id=202501210934383987154>
- `fiio-echo-nano`: <https://forum.fiio.com/note/showNoteContent.do?id=202605161533005115232>
- `fiio-disc`: <https://forum.fiio.com/note/showNoteContent.do?id=202601311712087234434>
- `fiio-jm21-prod`: <https://www.fiio.com/jm21>
- `fiio-m23`: <https://www.fiio.com/m23>
- `sony-queue`: <https://helpguide.sony.net/dmp/nwa100/v1/en/contents/TP0002432986.html>
- `sony-hold`: <https://helpguide.sony.net/dmp/nwa100/v1/en/contents/TP0002432977.html>
- `sony-volume`: <https://helpguide.sony.net/dmp/nwa100/v1/en/contents/TP0001694372.html>
- `sony-sound`: <https://helpguide.sony.net/dmp/nwa100/v1/en/contents/TP0002432957.html>
- `sony-capacity`: <https://helpguide.sony.net/dmp/nwa100/v1/en/contents/TP0002432965.html>
- `sony-update`: <https://helpguide.sony.net/dmp/nwa100/v1/en/contents/TP0002433009.html>
- `sony-lrc`: <https://helpguide.sony.net/dmp/nwa100/v1/en/contents/TP0002432945.html>
- `sony-lyrics`: <https://helpguide.sony.net/dmp/nwa100/v1/en/contents/TP0002433031.html>
- `sony-playlists`: <https://helpguide.sony.net/dmp/nwa100/v1/en/contents/TP0002432989.html>
- `sony-formats`: <https://helpguide.sony.net/dmp/nwa100/v1/en/contents/TP0002432963.html>
- `sony-microsd`: <https://helpguide.sony.net/dmp/nwa100/v1/en/contents/TP0002433017.html>
- `android-output`: <https://developer.android.com/media/platform/output>
- `android-src`: <https://source.android.com/docs/core/audio/src>
- `android-bitperfect`: <https://source.android.com/docs/core/audio/preferred-mixer-attr>
- `pw-conf`: <https://docs.pipewire.org/page_man_pipewire_conf_5.html>
- `k-alsa`: <https://docs.kernel.org/sound/alsa-configuration.html>
- `k-cs43130`: <https://raw.githubusercontent.com/torvalds/linux/master/sound/soc/codecs/cs43130.c>
- `k-gadget`: <https://docs.kernel.org/usb/gadget-testing.html>
- `k-msc`: <https://docs.kernel.org/usb/mass-storage.html>
- `dop`: <https://dsd-guide.com/sites/default/files/white-papers/DoP_openStandard_1v1.pdf>
- `wp-dsd`: <https://en.wikipedia.org/wiki/Direct_Stream_Digital>
- `nwavguy`: <https://nwavguy.blogspot.com/2011/02/headphone-amp-impedance.html>
- `asr-lownoise`: <https://www.audiosciencereview.com/forum/index.php?threads/small-dap-with-low-noise-floor-and-dlna-upnp-renderer-support.72102/>
- `asr-dynamics`: <https://www.audiosciencereview.com/forum/index.php?threads/enough-volume-but-not-enough-dynamics.50547/>
- `asr-r6iii`: <https://www.audiosciencereview.com/forum/index.php?threads/hiby-r6iii-native-peq-and-convolution.52776/>
- `asr-akcfw`: <https://www.audiosciencereview.com/forum/index.php?threads/akcfw-astell-kern-custom-firmware-development-releases-discussion.73066/>
- `asr-dumbdap`: <https://www.audiosciencereview.com/forum/index.php?threads/looking-for-a-%E2%80%98dumb%E2%80%99-dap.48423/>
- `autoeq`: <https://github.com/jaakkopasanen/AutoEq>
- `autoeq-results`: <https://raw.githubusercontent.com/jaakkopasanen/AutoEq/master/results/README.md>
- `bs2b`: <http://bs2b.sourceforge.net/>
- `soxr`: <https://github.com/chirlu/soxr>
- `wp-mqa`: <https://en.wikipedia.org/wiki/Master_Quality_Authenticated>
- `xiph-vc`: <https://xiph.org/vorbis/doc/v-comment.html>
- `wp-id3`: <https://en.wikipedia.org/wiki/ID3>
- `picard`: <https://picard-docs.musicbrainz.org/en/latest/appendices/tag_mapping.html>
- `wp-cue`: <https://en.wikipedia.org/wiki/Cue_sheet_(computing)>
- `rfc9639`: <https://www.rfc-editor.org/rfc/rfc9639.html>
- `wp-lrc`: <https://en.wikipedia.org/wiki/LRC_(file_format)>
- `wp-m3u`: <https://en.wikipedia.org/wiki/M3U>
- `wp-ape`: <https://en.wikipedia.org/wiki/Monkey%27s_Audio>
- `sda-cap`: <https://www.sdcard.org/developers/sd-standard-overview/capacity-sd-sdhc-sdxc-sduc/>
- `wp-exfat`: <https://en.wikipedia.org/wiki/ExFAT>
- `hiby-dl`: <https://store.hiby.com/pages/download>
- `hiby-r4`: <https://store.hiby.com/products/hiby-r4>
- `ms-uac2`: <https://learn.microsoft.com/en-us/windows-hardware/drivers/audio/usb-2-0-audio-drivers>
- `ldac`: <https://www.sony.co.jp/en/Products/LDAC/>
- `wp-bt`: <https://pipewire.pages.freedesktop.org/wireplumber/daemon/configuration/bluetooth.html>
- `bt-leaudio`: <https://www.bluetooth.com/learn-about-bluetooth/feature-enhancements/le-audio/>
- `shairport`: <https://github.com/mikebrady/shairport-sync>
- `shairport-ap2`: <https://github.com/mikebrady/shairport-sync/blob/master/AIRPLAY2.md>
- `roon`: <https://help.roonlabs.com/portal/en/kb/articles/roon-ready>
- `spotify-hw`: <https://developer.spotify.com/documentation/commercial-hardware>
- `librespot`: <https://github.com/librespot-org/librespot>
- `wp-mtp`: <https://en.wikipedia.org/wiki/Media_Transfer_Protocol>
- `os-req`: <https://github.com/opendap-project/blackbird-os/blob/main/docs/requirements/README.md>
- `os-dsd512`: <https://github.com/opendap-project/blackbird-os/blob/main/docs/research/2026-10-dsd512.md>
- `brd-0012`: <https://github.com/opendap-project/board/blob/main/docs/adr/0012-headphone-output-stage.md>

[wp-rockbox]: https://en.wikipedia.org/wiki/Rockbox
[wp-gapless]: https://en.wikipedia.org/wiki/Gapless_playback
[rfc7845]: https://www.rfc-editor.org/rfc/rfc7845.html
[m3-3425]: https://github.com/androidx/media/issues/3425
[m3-3200]: https://github.com/androidx/media/pull/3200
[m3-2576]: https://github.com/androidx/media/issues/2576
[mpd-1140]: https://github.com/MusicPlayerDaemon/MPD/issues/1140
[mpd-2588]: https://github.com/MusicPlayerDaemon/MPD/issues/2588
[mpd-1599]: https://github.com/MusicPlayerDaemon/MPD/issues/1599
[mpd-user]: https://mpd.readthedocs.io/en/stable/user.html
[mpd-user-latest]: https://mpd.readthedocs.io/en/latest/user.html
[mpd-plugins]: https://mpd.readthedocs.io/en/stable/plugins.html
[mpd-plugins-latest]: https://mpd.readthedocs.io/en/latest/plugins.html
[mpd-2592]: https://github.com/MusicPlayerDaemon/MPD/pull/2592
[mpd-578]: https://github.com/MusicPlayerDaemon/MPD/issues/578
[mpd-262]: https://github.com/MusicPlayerDaemon/MPD/issues/262
[mpd-1096]: https://github.com/MusicPlayerDaemon/MPD/issues/1096
[mpd-813]: https://github.com/MusicPlayerDaemon/MPD/issues/813
[mpd-902]: https://github.com/MusicPlayerDaemon/MPD/issues/902
[mpd-2211]: https://github.com/MusicPlayerDaemon/MPD/issues/2211
[mpd-1768]: https://github.com/MusicPlayerDaemon/MPD/issues/1768
[rb-playback]: https://raw.githubusercontent.com/Rockbox/rockbox/master/manual/configure_rockbox/playback_options.tex
[rb-autoresume]: https://raw.githubusercontent.com/Rockbox/rockbox/master/manual/configure_rockbox/autoresume.tex
[rb-bookmarks]: https://raw.githubusercontent.com/Rockbox/rockbox/master/manual/configure_rockbox/bookmarking.tex
[rb-wps]: https://raw.githubusercontent.com/Rockbox/rockbox/master/manual/rockbox_interface/wps.tex
[rb-startup]: https://raw.githubusercontent.com/Rockbox/rockbox/master/manual/configure_rockbox/startup_shutdown_options.tex
[rb-sound]: https://raw.githubusercontent.com/Rockbox/rockbox/master/manual/configure_rockbox/sound_settings.tex
[rb-system]: https://raw.githubusercontent.com/Rockbox/rockbox/master/manual/configure_rockbox/system_options.tex
[rb-tagcache]: https://raw.githubusercontent.com/Rockbox/rockbox/master/manual/rockbox_interface/tagcache.tex
[rb-formats]: https://raw.githubusercontent.com/Rockbox/rockbox/master/manual/appendix/file_formats.tex
[rb-install]: https://raw.githubusercontent.com/Rockbox/rockbox/master/manual/getting_started/installation.tex
[wp-rg]: https://en.wikipedia.org/wiki/ReplayGain
[ebu-r128]: https://tech.ebu.ch/docs/r/r128.pdf
[fiio-m11]: https://forum.fiio.com/note/showNoteContent.do?id=202105171708309850783&tid=17
[fiio-m11s]: https://forum.fiio.com/note/showNoteContent.do?id=202212281000205937102&tid=136
[fiio-m17]: https://forum.fiio.com/note/showNoteContent.do?id=202112241051165653057&tid=123
[fiio-jm21]: https://forum.fiio.com/note/showNoteContent.do?id=202412181641298410115&tid=17
[fiio-m21]: https://forum.fiio.com/note/showNoteContent.do?id=202505140958060216556&tid=17
[fiio-m27]: https://forum.fiio.com/note/showNoteContent.do?id=202510291436497507515&tid=17
[fiio-echo]: https://forum.fiio.com/note/showNoteContent.do?id=202602051101498161611
[fiio-echo-mini]: https://forum.fiio.com/note/showNoteContent.do?id=202501210934383987154
[fiio-echo-nano]: https://forum.fiio.com/note/showNoteContent.do?id=202605161533005115232
[fiio-disc]: https://forum.fiio.com/note/showNoteContent.do?id=202601311712087234434
[fiio-jm21-prod]: https://www.fiio.com/jm21
[fiio-m23]: https://www.fiio.com/m23
[sony-queue]: https://helpguide.sony.net/dmp/nwa100/v1/en/contents/TP0002432986.html
[sony-hold]: https://helpguide.sony.net/dmp/nwa100/v1/en/contents/TP0002432977.html
[sony-volume]: https://helpguide.sony.net/dmp/nwa100/v1/en/contents/TP0001694372.html
[sony-sound]: https://helpguide.sony.net/dmp/nwa100/v1/en/contents/TP0002432957.html
[sony-capacity]: https://helpguide.sony.net/dmp/nwa100/v1/en/contents/TP0002432965.html
[sony-update]: https://helpguide.sony.net/dmp/nwa100/v1/en/contents/TP0002433009.html
[sony-lrc]: https://helpguide.sony.net/dmp/nwa100/v1/en/contents/TP0002432945.html
[sony-lyrics]: https://helpguide.sony.net/dmp/nwa100/v1/en/contents/TP0002433031.html
[sony-playlists]: https://helpguide.sony.net/dmp/nwa100/v1/en/contents/TP0002432989.html
[sony-formats]: https://helpguide.sony.net/dmp/nwa100/v1/en/contents/TP0002432963.html
[sony-microsd]: https://helpguide.sony.net/dmp/nwa100/v1/en/contents/TP0002433017.html
[android-output]: https://developer.android.com/media/platform/output
[android-src]: https://source.android.com/docs/core/audio/src
[android-bitperfect]: https://source.android.com/docs/core/audio/preferred-mixer-attr
[pw-conf]: https://docs.pipewire.org/page_man_pipewire_conf_5.html
[k-alsa]: https://docs.kernel.org/sound/alsa-configuration.html
[k-cs43130]: https://raw.githubusercontent.com/torvalds/linux/master/sound/soc/codecs/cs43130.c
[k-gadget]: https://docs.kernel.org/usb/gadget-testing.html
[k-msc]: https://docs.kernel.org/usb/mass-storage.html
[dop]: https://dsd-guide.com/sites/default/files/white-papers/DoP_openStandard_1v1.pdf
[wp-dsd]: https://en.wikipedia.org/wiki/Direct_Stream_Digital
[nwavguy]: https://nwavguy.blogspot.com/2011/02/headphone-amp-impedance.html
[asr-lownoise]: https://www.audiosciencereview.com/forum/index.php?threads/small-dap-with-low-noise-floor-and-dlna-upnp-renderer-support.72102/
[asr-dynamics]: https://www.audiosciencereview.com/forum/index.php?threads/enough-volume-but-not-enough-dynamics.50547/
[asr-r6iii]: https://www.audiosciencereview.com/forum/index.php?threads/hiby-r6iii-native-peq-and-convolution.52776/
[asr-akcfw]: https://www.audiosciencereview.com/forum/index.php?threads/akcfw-astell-kern-custom-firmware-development-releases-discussion.73066/
[asr-dumbdap]: https://www.audiosciencereview.com/forum/index.php?threads/looking-for-a-%E2%80%98dumb%E2%80%99-dap.48423/
[autoeq]: https://github.com/jaakkopasanen/AutoEq
[autoeq-results]: https://raw.githubusercontent.com/jaakkopasanen/AutoEq/master/results/README.md
[bs2b]: http://bs2b.sourceforge.net/
[soxr]: https://github.com/chirlu/soxr
[wp-mqa]: https://en.wikipedia.org/wiki/Master_Quality_Authenticated
[xiph-vc]: https://xiph.org/vorbis/doc/v-comment.html
[wp-id3]: https://en.wikipedia.org/wiki/ID3
[picard]: https://picard-docs.musicbrainz.org/en/latest/appendices/tag_mapping.html
[wp-cue]: https://en.wikipedia.org/wiki/Cue_sheet_(computing)
[rfc9639]: https://www.rfc-editor.org/rfc/rfc9639.html
[wp-lrc]: https://en.wikipedia.org/wiki/LRC_(file_format)
[wp-m3u]: https://en.wikipedia.org/wiki/M3U
[wp-ape]: https://en.wikipedia.org/wiki/Monkey%27s_Audio
[sda-cap]: https://www.sdcard.org/developers/sd-standard-overview/capacity-sd-sdhc-sdxc-sduc/
[wp-exfat]: https://en.wikipedia.org/wiki/ExFAT
[hiby-dl]: https://store.hiby.com/pages/download
[hiby-r4]: https://store.hiby.com/products/hiby-r4
[ms-uac2]: https://learn.microsoft.com/en-us/windows-hardware/drivers/audio/usb-2-0-audio-drivers
[ldac]: https://www.sony.co.jp/en/Products/LDAC/
[wp-bt]: https://pipewire.pages.freedesktop.org/wireplumber/daemon/configuration/bluetooth.html
[bt-leaudio]: https://www.bluetooth.com/learn-about-bluetooth/feature-enhancements/le-audio/
[shairport]: https://github.com/mikebrady/shairport-sync
[shairport-ap2]: https://github.com/mikebrady/shairport-sync/blob/master/AIRPLAY2.md
[roon]: https://help.roonlabs.com/portal/en/kb/articles/roon-ready
[spotify-hw]: https://developer.spotify.com/documentation/commercial-hardware
[librespot]: https://github.com/librespot-org/librespot
[wp-mtp]: https://en.wikipedia.org/wiki/Media_Transfer_Protocol
[os-req]: https://github.com/opendap-project/blackbird-os/blob/main/docs/requirements/README.md
[os-dsd512]: https://github.com/opendap-project/blackbird-os/blob/main/docs/research/2026-10-dsd512.md
[brd-0012]: https://github.com/opendap-project/board/blob/main/docs/adr/0012-headphone-output-stage.md
