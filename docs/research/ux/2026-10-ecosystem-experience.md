# UX research: the OpenDAP experience across every device

- **Date:** 2026-10-23
- **Status:** Desk research and design reasoning. Nothing here was tested with users, and nothing is decided; requirements will be drawn from it with the maintainers.
  - Personas extend the archetypes of the [persona, JTBD and journey research](2026-10-persona-jtbd-journey.md) (2026-10-22). They are not survey results.
  - Claims about other products link their sources; what was not checked against a primary source is **UNVERIFIED**.
- **Question:** what is the best complete experience a listener can have with the T1, OpenDAP Desktop, the `opendap` CLI, the other devices they own and any optional online service? And what does that ask of the software and OS, the hardware, the desktop and the protocol between them?
- **Not limited to what others do.** The design starts from principles, theory and research (§2). The market benchmark (§4) is a check on it and a list of failures to avoid, not a template.
- **Companion documents:**
  - [Connecting a user's devices: towards an OpenDAP Protocol](https://github.com/opendap-project/system/blob/main/docs/research/2026-10-device-ecosystem-protocol.md) (`system`): channels, roles, the protocol and the benchmark of standards;
  - [OpenDAP Desktop as the companion](https://github.com/opendap-project/desktop/blob/main/docs/research/2026-10-desktop-companion.md) (`desktop`): library, curation, device sync, providers and the benchmark of companion apps and stores.
- **Inputs read (OpenDAP):**
  - requirements in `t1` (T1-001 to T1-064), `system` (SYS-001 to SYS-041), `blackbird-os` (OS-001 to OS-028), `desktop` (DESK-001 to DESK-009), `toolkit-cli` (TK-001 to TK-011) and `board` (BRD-025 to BRD-039);
  - [system ADR 0002](https://github.com/opendap-project/system/blob/main/docs/adr/0002-system-architecture.md), the [CLI spec](https://github.com/opendap-project/toolkit-cli/blob/main/docs/specs/opendap-cli.md), the [controls spec](../../specs/controls-and-status-led.md), the [audio profiles spec](../../specs/audio-profiles.md) and the [design system](https://github.com/opendap-project/ui-components/blob/main/docs/specs/design-system.md).

## 1. Summary

1. **The T1 is for listening; the desktop is for keeping; the system makes them one.** Each task goes to the surface that fits its attention and form:
   - the T1 for listening and operating audio, often eyes-free and on the move;
   - the desktop for curating, buying, configuring in depth and looking after devices, with a keyboard, a large screen and time;
   - the CLI for automation;
   - every other device through standards.

   No surface depends on another to do its own job.
2. **Music never waits for another device.** The T1 holds a complete, local replica of what its owner chose and plays it with the computer off, no network and no account. Sync is how devices meet, never a condition for listening. This is the local-first stance ([Kleppmann et al. 2019](https://www.inkandswitch.com/local-first/)) applied to a player.
3. **One owner, one library, many replicas.**
   - Playlists, ratings, play history, bookmarks, resume points and audio profiles belong to the owner, not to a device or a provider.
   - They merge across every device they pair, and they export in documented formats.
   - Leaving is as easy as staying.
4. **The listener's music is never changed behind their back.** Tag fixes, deduplication, transcoding and reorganisation are offered, previewed and undoable. Deletion passes through a recoverable state on every device. The research on psychological ownership names control, intimate knowledge and self-investment as what makes something "mine" ([Pierce et al. 2001](https://doi.org/10.5465/amr.2001.4378028)). Silent changes erode all three.
5. **Providers, not a store.**
   - Buying music goes through optional providers the listener chooses.
   - Purchases land as files in the one library.
   - Playlists are keyed by open identifiers (MusicBrainz, ISRC), so they survive any provider leaving.
   - Nothing proprietary is required, and the absence of providers costs nothing.
6. **Show the seams that matter, one line deep.** "Is this album on my T1?", "did my purchase arrive?" and "when did it last sync?" are answered at a glance on every surface, with detail one step down. Seams are shown where they help the listener and hidden where they do not ([Chalmers and MacColl 2003](https://www.alandix.com/academic/conf/ubicomp-models/pdf/maccoll.pdf)).
7. **Calm.** Status lives in the periphery; only what needs a decision reaches the centre. Listening is never interrupted by sync, updates or prompts ([Weiser and Brown 1996](https://calmtech.com/papers/designing-calm-technology.html); [Case 2015](https://calmtech.com/)).
8. **Audiophile first, engineer one step away, on every surface** (SYS-041).
   - On the T1: music first, and the signal path one action away.
   - On the desktop: profiles with their curves, measurements imported, level-matched A/B comparisons.
   - In the CLI: everything scriptable.

## 2. Foundations

The principles below come from theory and research, with the test that turns each one into a check. Each references what it rests on; the full bibliography is in §14.

| # | Principle | Test | Rests on |
|---|---|---|---|
| E1 | **Music never waits for another device.** Playback, local browsing and the buttons work with the desktop off, no network and no cloud | Airplane mode, no desktop, fresh boot: time to first note and full library browse unchanged | Local-first ideals 1 and 3 ([Kleppmann et al. 2019](https://www.inkandswitch.com/local-first/)); calm technology, "work even when it fails" ([Case 2015](https://calmtech.com/)); Kano must-be |
| E2 | **Must-be quality first.** No new feature ships at the cost of gapless, bit-perfect, glitch-free playback, sync integrity or battery | A must-be regression suite gates every release | Kano et al. 1984; Norman 2004 (behavioural level); peak-end rule (Kahneman et al. 1993) |
| E3 | **Each task lives on the surface that fits it.** | Every step of the job map (§6) names its primary surface; no T1 screen needs text entry beyond search and names | Outcome-driven job maps (Ulwick 2005); form-factor fit (Levin 2014); Fitts 1954 |
| E4 | **Choose the C deliberately.** Each cross-device feature is consistent, continuous or complementary | Feature specs carry the label; continuity resumes in one action | Levin's 3C model (*Designing Multi-Device Experiences*, 2014) |
| E5 | **The owner keeps the data and the exit.** | Full export and re-import into a clean install loses nothing; formats are public | Local-first ideals 5 and 7; GDPR Art. 20 |
| E6 | **Never change the collection silently.** | No file or tag changes without a user action in the log; every batch has an undo | Psychological ownership (Pierce et al. 2001, 2003); Shneiderman's golden rules (reversal, control) |
| E7 | **Destruction is recoverable across devices.** | Delete on the T1, recover from the desktop within the retention window, and the reverse | Nielsen's heuristics (user control, error recovery); Abowd and Dix 1992 |
| E8 | **Show the seams that matter, one line deep.** | From the desktop, "is this album on my T1?" is one step | Seamful design (Chalmers and MacColl 2003); "It's on my other computer!" (Dearman and Pierce 2008) |
| E9 | **Conflicts never lost, rarely asked.** | Concurrent offline edits never drop data; fault injection covers reorder, duplicate and partition | Bayou (Terry et al. 1995); CRDTs (Shapiro et al. 2011); calibrated trust (Lee and See 2004) |
| E10 | **Listening is never interrupted.** | Soak tests with sync and indexing running: no dropouts, no pops, no modal while playing | Flow (Csikszentmihalyi 1990); calm technology; T1-046, T1-047, SYS-036 |
| E11 | **Status in the periphery, only decisions in the centre.** | Every interruption in an inventory is justified by a decision the user must take | Weiser and Brown 1996; Case 2015; Simon 1971 |
| E12 | **Listener by default, engineer one step away.** | Signal path one action from Now Playing; A/B level-matched by default, blind on request | Progressive disclosure (Nielsen 2006); Tesler's law; blind vs sighted tests (Toole and Olive 1994); SYS-041 |
| E13 | **Controls answer locally and at once.** Remote control shows its result immediately and reconciles | Input-to-feedback latency on the T1 under load; remote actions give feedback before the round trip | Response-time limits (Miller 1968; Card et al. 1991; Nielsen 1993); T1-057 |
| E14 | **Eyes-free first.** Core playback works by feel | A blindfolded session completes play, pause, track, volume and lock | Inclusive design, situational constraints (Microsoft 2016; Holmes 2018); most listening happens alongside other activities (Sloboda et al. 2001) |
| E15 | **Support curation; give the collection a place.** | On a large test library, the desktop finds and batch-fixes every health issue, with undo | Personal music collections (Cunningham et al. 2004); virtual possessions are "placeless, spaceless, formless" (Odom et al. 2011, 2014) |
| E16 | **Retrieval follows how people choose music.** Mood, memory and context, besides artist and album | Time to choose "music for how I feel now", in a study | Uses and gratifications (Lonsdale and North 2011); functions of listening (Schäfer et al. 2013); episodic memory (Juslin and Västfjäll 2008) |
| E17 | **Privacy is the default.** | Packet capture on a default install shows only what the owner enabled; every endpoint is listed | Privacy by Design (Cavoukian 2009); GDPR Art. 5, 25; autonomy (Deci and Ryan 2000) |
| E18 | **No dark patterns, above all in purchases.** | Every flow reviewed against the five strategies of Gray et al. before release | Gray et al. 2018; Mathur et al. 2019 |
| E19 | **Safety limits visible and owned.** | Dose and mode are visible; no undisclosed limit in the signal path | WHO-ITU H.870 ([WHO](https://www.who.int/publications/i/item/9789241515276)); SYS-012; T1-045 |
| E20 | **Design the ends and the long tail.** | Blueprints for setup, sync, update, repair and leaving, with named fail points | Peak-end rule; service blueprints (Shostack 1984; Bitner et al. 2008); customer journey (Lemon and Verhoef 2016) |

Three readings shape everything that follows:

- **Why people listen.** Mood regulation is the main reason people give ([Lonsdale and North 2011](https://researchportal.hw.ac.uk/en/publications/why-do-we-listen-to-music-a-uses-and-gratifications-analysis/)). Self-awareness and social relatedness are the most important functions ([Schäfer et al. 2013](https://frontiersin.org/articles/10.3389/fpsyg.2013.00511/full)). A personal collection and choice go with contentment and motivation ([Krause, North and Hewitt 2015](https://researchonline.jcu.edu.au/64286/)). The job is emotional first and technical second. Fidelity serves it; it does not replace it.
- **Collections grow unmanaged.** Digital possessions do not force curation as they grow ([Odom, Zimmerman and Forlizzi 2014](https://hcii.cmu.edu/project/virtual-possessions)). So the desktop must make curation light, and the T1 must give the collection a physical place and form.
- **Trust is calibrated, not asked for.** People trust automation in proportion to how observable it is ([Lee and See 2004](https://doi.org/10.1518/hfes.46.1.50_30392)). Sync, updates, auto-tagging and the hearing-safety dose earn trust by showing what they did and why.

## 3. Method

- **Personas.** The four archetypes of the 2026-10-22 research, plus seven derived from them where a different situation changes the cross-device experience (§5).
- **Jobs.**
  - A universal job map (define, locate, prepare, confirm, execute, monitor, modify, conclude; [Bettencourt and Ulwick 2008](https://hbr.org/2008/05/the-customer-centered-innovation-map)), with desired outcomes per step and the surface that serves each step (§6).
  - Job stories ("When…, I want to…, so I can…", Klement 2016) for the cross-device jobs the earlier research did not cover.
- **Journeys.** Fourteen journeys across devices (§7). For each: stages, the surface at each stage, the intended feeling, the principle or UX law applied and the fail points.
- **Emotional design.** Norman's visceral, behavioural and reflective levels per surface; the peak-end rule for the moments to design; and the anxieties to design out (§8).
- **UX laws.** From [Laws of UX](https://lawsofux.com/), applied per surface and across devices (§9).
- **Kano.** Ecosystem attributes as must-be, one-dimensional, attractive, indifferent or reverse (§10).
- **Service blueprint.** For the two flows with the most backstage work: from purchase to playback, and sync (§11).
- **Not covered.** Interviews, usability tests and surveys (next steps, §13), and technology choices (the maintainers have not opened them). The details of any OpenDAP online service are outside this public document.

## 4. What others do, briefly

Detail and sources: [desktop companion research](https://github.com/opendap-project/desktop/blob/main/docs/research/2026-10-desktop-companion.md) (companion apps, library managers, stores) and [protocol research](https://github.com/opendap-project/system/blob/main/docs/research/2026-10-device-ecosystem-protocol.md) (standards).

| Pattern | Who | What it teaches |
|---|---|---|
| **The player is self-contained; the phone is a remote** | Sony, Astell&Kern, FiiO, HiBy, Shanling ([HiBy Link](https://guide.hiby.com/en/docs/guides/hibylink/hibylink)) | The right base. None offers a desktop library with two-way sync of ratings and play counts |
| **Two-way sync with a desktop** | iTunes and the iPod; today Finder plus an Apple Music or iTunes Match subscription for play counts | The reference for "it just arrives"; also for the worst defaults: delete propagates, "Erase and Sync" when changing library ([Apple](https://support.apple.com/en-au/guide/mac-help/mchlbf6a1fab/11.0/mac)) |
| **Server first, offline later** | Plexamp, Roon ARC | Offline as an afterthought fails: download caps, missing artwork offline (user reports) |
| **Statistics as an event log** | Rockbox's playback log; Finamp's queued offline plays | Survives offline periods if each event keeps its own time; Jellyfin's stamping every offline play at reconnection is the failure (user reports) |
| **Unified library across providers** | Roon; Music Assistant (provider mappings, matching by ISRC or MusicBrainz ID first) ([usage](https://www.music-assistant.io/usage)) | Albums bound to the provider they came from break when it leaves: Plex's TIDAL integration ended in October 2024 and left unplayable entries ([Plex forum](https://forums.plex.tv/t/tidal-integration-ending/886456)) |
| **Stores** | Qobuz, Bandcamp, NativeDSD, Presto, HDtracks, ProStudioMasters, iTunes | DRM-free files, but no open purchase or entitlement API for third-party clients; re-downloads are not guaranteed ([Qobuz help](https://help.qobuz.com/en/articles/10166-can-i-re-download-my-purchases)) |
| **Streaming on a third-party device** | Spotify, TIDAL, Qobuz, Apple Music | Offline copies stay inside each service's own app or certified SDK; the "Connect" programmes need partner agreements and certification |
| **File transfer** | Mass storage, MTP, Wi-Fi upload | No classic option is both native on macOS and safe while the device plays ([9to5Google](https://9to5google.com/2024/02/16/android-file-transfer-mac-hidden/)) |
| **Updates** | Over the air, with a file or USB fallback | Sony asks users to back up music before an update that cannot be reverted (UNVERIFIED excerpt). No vendor documents automatic rollback (UNVERIFIED) |

**What nobody offers, and the best experience needs:**

- a player that is complete on its own **and** a desktop library that syncs everything both ways, with no subscription;
- playlists and history that survive any provider;
- a sync state the listener can read at a glance;
- updates that can always be undone;
- an exit with all the data.

## 5. Personas

### 5.1 The four archetypes (2026-10-22)

From the [persona research](2026-10-persona-jtbd-journey.md) §3; only what matters across devices is restated.

| Persona | Devices they live with | Cross-device stake |
|---|---|---|
| **The engineer audiophile** (primary) | T1, a computer, a desktop DAC/amp, measurement gear, several headphones | Wants every stage visible and controllable from the device that is best for it: tune on the desktop, listen on the T1, prove it with measurements |
| **The commuter with IEMs** | T1, phone, IEMs and Bluetooth earphones | The T1 must be complete in the pocket. Sync happens when charging at home, unnoticed |
| **The collector** (large lossless and DSD library) | T1 with a large card, a NAS or computer master library, home system | One master library, and the T1 a faithful subset of it that never drifts |
| **The listener who wants it to just work** | T1, maybe a laptop, Bluetooth speaker | No file management if possible; music arrives; nothing to configure |

### 5.2 Derived personas

Each one is a situation that changes the cross-device experience, derived from an archetype.

| Persona | Derived from | Situation | Jobs that matter most | What the best experience is |
|---|---|---|---|---|
| **The sound engineer** | Engineer audiophile, at the engineer end of SYS-041 | Checks mixes and masters away from the studio; compares against references; needs a known, flat reference and honest meters | Carry a mix to the T1 in seconds; level-matched A/B against a reference; notes that return to the computer | A drop folder on the desktop that reaches the T1 at once; the Monitor profile; A/B and meters one step away; notes and timestamps synced back |
| **The builder** | Engineer audiophile, on a weekend (the tinkerer) | Builds the DIY edition, writes capabilities, designs HATs, scripts the device | Automate, inspect, flash, recover, extend | Everything the desktop does is also in the CLI and the SDK; an agent can drive it (TK-009); recovery is guaranteed |
| **The curator-buyer** | Collector | Buys hi-res and DSD downloads from several stores; supports artists directly; keeps a wish list | Find, compare editions, buy, have it arrive tagged and on the T1 | Providers side by side, the purchase lands as files in the library with tags and art, and it reaches the T1 at the next sync without a step |
| **The streaming switcher** | Just works, moving towards the collector | Has years of playlists and history in a subscription service; wants to own music and leave | Bring playlists and history; see what they already own; build the missing list | Import playlists and history from export files or a provider; match to owned files by open identifiers; a missing list across providers; no data loss when the subscription ends |
| **The home listener** | Collector and engineer audiophile | Listens at home on a hi-fi system; the T1 travels with them | Use the T1 as the source or the remote at home; continue a session when leaving | The T1 plays into the home DAC by USB or the network, serves its library, and picks up a session from the desktop with one action |
| **The household** | Just works | Two or more people, two or more T1s, one shared music collection on a computer or NAS | Share the collection, keep personal state apart | One shared library, each person's own playlists, ratings, history and profiles; a T1 given as a gift works before anyone configures anything |
| **The listener with access needs** | All of them | Permanent, temporary or situational: low vision, limited dexterity, gloves, bright sun, walking | Do every core job without seeing the screen, or with large text and high contrast | The buttons cover core playback; every touch gesture has a visible alternative (SYS-039); the desktop meets WCAG 2.2; screen-reader support on the desktop, and on the T1 as far as the platform allows (system ADR 0002 leaves the level open) |

## 6. Jobs

### 6.1 The core job, mapped

**Core job: listen to music I love, the way I want it, wherever I am.** Mapped on the universal job steps, with desired outcomes and the surface that serves each step (E3). ● primary · ○ supporting · — not involved.

| Step | Desired outcomes (minimise…) | T1 | Desktop | CLI | Others |
|---|---|---|---|---|---|
| **Define**: decide what to listen to and how | …the time to find music that fits how I feel now (E16); …the effort to choose a listening setup for the headphone in hand | ● | ○ (smart selections, profiles) | — | — |
| **Locate**: get the music and the setup | …the steps to acquire a release in the format I want; …the chance that something I own is missing from the T1 | ○ (wish list) | ● (library, providers) | ○ | Providers |
| **Prepare**: put it where I will listen | …the time from "bought" to "on the T1"; …the space wasted; …the chance of a broken or duplicated file | ○ | ● (device sync rules) | ○ | — |
| **Confirm**: know it is ready | …the uncertainty about what is on which device (E8); …the uncertainty about battery and storage | ● (glance) | ● | ○ | — |
| **Execute**: listen | …the time to first note; …any audible artefact; …the attention needed to control it (E14) | ● | — (it may remote) | — | Headphones, car, speakers |
| **Monitor**: keep the experience right | …the effort to know the signal path is untouched (E12); …the risk to hearing (E19); …the attention needed for status (E11) | ● | ○ | ○ | — |
| **Modify**: adjust | …the steps to change the profile, the output or the queue; …the latency of any change (T1-057) | ● | ● (deep edits) | ○ | Remote |
| **Conclude**: end and come back | …the effort to resume exactly where I left off (T1-051); …the loss of history and statistics | ● | ○ (history) | — | — |

### 6.2 The jobs around it

| Job | Main steps | Primary surface |
|---|---|---|
| **Build and keep my collection** | Acquire (buy, rip, import); identify and tag; curate (duplicates, art, editions); back up | Desktop |
| **Own my setup** | Pair headphones and outputs; make and refine profiles; compare; share a profile | T1 for choosing and comparing; desktop for editing in depth |
| **Keep my devices healthy** | Update; back up and restore; diagnose; replace the battery; move to a new device | Desktop and T1 together |
| **Share music with people around me** | Play to a speaker or a car; let a friend queue; give a T1 to someone | T1 |
| **Extend** | Install capabilities; build HATs; script; automate with agents | CLI, SDK and desktop |
| **Leave or migrate** | Export everything; bring data from another player or service | Desktop and CLI |

### 6.3 Cross-device job stories

These complement the device-centred stories of the 2026-10-22 research (§4 there). Type: **F** functional, **E** emotional, **S** social.

| Job story | Type | Persona |
|---|---|---|
| When I buy an album on the computer, I want it on my T1 the next time I pick it up, so I can listen without thinking about transfers. | F, E | Curator-buyer, just works |
| When I leave home in the middle of an album, I want to continue on the T1 at the same second, so I can keep the mood. | F, E | Home listener, commuter |
| When I plug the T1 into the computer, I want it to charge, sync and back up at once, so I can trust it is all in order. | F, E | All |
| When I wonder whether an album is on my T1, I want to see it from the computer at a glance, so I can decide what to carry. | F | Collector |
| When I rate or skip tracks on the T1, I want the desktop to know, so my library learns from my listening. | F, E | Collector, curator-buyer |
| When I tune a profile for a headphone on the desktop, with its curves, I want to hear it on the T1 at once, so I can compare and decide. | F | Engineer audiophile, sound engineer |
| When I finish a mix, I want it on the T1 in seconds and my notes back on the computer, so I can check it on the train. | F | Sound engineer |
| When a store stops selling an album I bought, I want my copy and my playlists untouched, so I can rely on owning it. | F, E | Curator-buyer |
| When I leave a streaming service, I want my playlists and history to come with me, so years of listening are not lost. | F, E | Streaming switcher |
| When someone in my home has their own T1, I want us to share the collection and keep our lists apart, so we each have our own music life. | F, S | Household |
| When I am away with no computer, I want new purchases to reach the T1 over the internet if I chose it, and the T1 to work perfectly if I did not. | F | Commuter, curator-buyer |
| When I get a new T1, or repair mine, I want everything back as it was, so the device is replaceable and my music life is not. | F, E | All |
| When I stop using OpenDAP, I want all my data in formats other software reads, so I never feel trapped. | E | All |
| When I build something, I want every action of the desktop available in the CLI, so I can automate it. | F | Builder |

## 7. Journeys

Each journey shows its stages, the surface at each stage, what the listener should feel, and the principle or law applied. Feelings are targets, not observations. **Fail points** are what the blueprint (§11) must handle visibly.

### 7.1 First day

Persona: every one; the peak is the first sound.

| Stage | Surface | What happens | Feeling | Applied |
|---|---|---|---|---|
| Unbox | T1 | Charged enough to play; a card with music, or the built-in demo track, plays at a safe level | Anticipation → first delight | Peak-end: the first sound is the peak to design (moment 2 of the 2026-10-22 research); SYS-012 safe level |
| First sound | T1 | Music before any account, network or computer | Relief, "it just plays" | E1; aesthetic-usability effect |
| Language, headphone | T1 | Two or three choices, each skippable | Calm | Hick's law; goal-gradient (a short visible path) |
| Get the desktop | Computer | Download OpenDAP Desktop; it finds the T1 on the cable or the network | Confidence | Jakob's law (install like any app) |
| Pair | T1 + desktop | Both screens show the same short code; one confirmation on the T1 | Control | E17; trust on first pairing, confirmed visually |
| Choose what to carry | Desktop | The whole library, or a selection (artists, playlists, smart rules), or "fill the free space", with a preview of what will be copied | Control | E6; the "dry run" pattern praised in beets-alternatives and Apple's fill-free-space |
| First sync | Both | Runs in the background and is resumable; the T1 keeps playing; progress shows on both | Patience without anxiety | E10; Zeigarnik effect: an unfinished task stays on the mind, so show progress and the end clearly |
| End of the day | Both | A summary: what arrived, what is left, what failed and why | Closure | Peak-end: a good end; E20 |

**Fail points:** no computer (the T1 must still be complete with a card, E1); the cable is power-only; the desktop cannot reach the T1 over the network; the first sync is interrupted.

### 7.2 Daily listening on the move

Persona: commuter; no other device takes part, on purpose.

| Stage | Surface | What happens | Feeling | Applied |
|---|---|---|---|---|
| Pick up | T1 | It resumes exactly where it stopped (T1-051) | Continuity | E1; T1-051 |
| Pocket | T1 | Screen off; Action, volume and track by feel (T1-060 to T1-063) | Freedom | E14; Fitts (the edge buttons need no aiming) |
| Glance | T1 | Status bar: time, card, volume, Bluetooth, Wi-Fi, battery (SYS-038) | Calm | E11; periphery |
| Interrupted | T1 | Unplugged headphones pause; never to the speaker (T1-053) | Safety | E2 |
| Home | T1 + charger or desktop | Charging is when sync, backup and updates happen, unnoticed | Nothing to think about | E10; calm |

**Fail points:** none involve another device, and that is the design. Battery, hearing dose and storage warnings come calmly and early (E19, E11).

### 7.3 Buying music

Persona: curator-buyer. The journey crosses the most surfaces and has the most backstage work (§11.1).

| Stage | Surface | What happens | Feeling | Applied |
|---|---|---|---|---|
| Wish | T1 | "Want this" on something heard on the radio, a friend's speaker or a track preview; the item appears in the desktop's wish list | Curiosity kept | E3 (capture on the T1, act on the desktop); complementary (E4) |
| Find | Desktop | Search across the enabled providers, with editions, formats and resolutions side by side, and what is already owned marked | Clarity | Hick's law (grouped choices); Miller (chunks by edition) |
| Choose | Desktop | Compare editions; see the provider, price, format, region; no default provider imposed | Fairness | E18 (no interface interference); E5 |
| Buy | Provider's own checkout | The listener pays on the provider's site or flow; OpenDAP never handles payment for a third party | Trust | E17, E18; the store-side reality of §4 |
| Arrive | Desktop | The download is found and imported, tagged from the file's own tags and, if the listener enabled them, metadata providers (SYS-045); provenance kept (provider, order, date) | "It just arrived" | Tesler's law: the system carries the complexity |
| Reach the T1 | Background | At the next meeting of the two devices (cable, LAN or, if chosen, the internet) | Anticipation | E8 (shown as "on its way to the T1") |
| First play | T1 | "New" on the T1's home; first play at the right profile | Delight | Peak-end |

**Fail points:** the provider changes its site; the download is incomplete or the format is wrong; identification fails (new or niche release); the store withdraws the title later (the file and the playlists stay, E5).

### 7.4 Curating the library

Persona: collector, curator-buyer.

| Stage | Surface | What happens | Feeling | Applied |
|---|---|---|---|---|
| Health | Desktop | One view of collection health: missing art, inconsistent tags, duplicates, editions, unidentified files, files not on any device | Mastery | E15 |
| Fix | Desktop | Batch fixes with a preview of every change; undo per batch | Control | E6, E7 |
| Decide what travels | Desktop | Per-device rules: everything; selections; smart rules (recently added, most played, rated); "fill the free space"; optional transcoding for space, per device, never altering the library | Control | E6; transcoding per device as in iTunes, MusicBee and beets-alternatives |
| See it on the T1 | T1 | Browse as the tags say, with the user's sort rules (2026-10-22 research, §4.3) | Ownership | Intimate knowledge (Pierce et al.) |

**Fail points:** a large library makes the desktop slow (Sony Music Center complaints); FAT timestamps cause spurious re-copies (content hashes needed); indexing on the T1 disturbs playback (Tangara user reports; T1-046).

### 7.5 A new headphone

Persona: engineer audiophile, then commuter.

| Stage | Surface | What happens | Feeling | Applied |
|---|---|---|---|---|
| Plug in | T1 | Direct profile, safe level; if the headphone is known and measured, the T1 offers Calibrated in one tap (audio profiles spec §1) | Safety, then curiosity | SYS-041; E19 |
| Listen | T1 | Direct or Calibrated; Bit-perfect or DSP shown on a quiet line | Trust | E12 |
| Refine | Desktop | The profile with its curves: target, measurement, PEQ, tone macros, crossfeed | Mastery | E3 (graphs need a large screen) |
| Compare | T1 (with the desktop as remote, optionally) | Level-matched A/B between two profiles, blind on request | Certainty | E12; Toole and Olive 1994 |
| Keep | Both | The profile syncs to every device; shareable as a documented file (audio profiles spec §6) | Pride | Socio-pleasure (Jordan 2000) |

**Fail points:** a measurement that does not match the unit; a profile using a block another device lacks (it loads with that block disabled, SYS-040).

### 7.6 Home listening and handoff

Persona: home listener.

| Stage | Surface | What happens | Feeling | Applied |
|---|---|---|---|---|
| At the desk | Desktop | The desktop plays, or remote-controls the T1 that feeds a home DAC over USB | Comfort | Complementary (E4) |
| Hand off | T1 | "Continue here" on the T1: same track, same second, the queue, the profile meant for the headphone in hand | Continuity | Continuous (E4); one action |
| Use the T1 as the source | T1 | USB to the home DAC, bit-perfect; or a network renderer in the room | Pride | T1-054; good citizen of standards |
| Remote | Desktop or a phone | Control with immediate feedback; the T1 never waits for the remote | Directness | E13 |

**Fail points:** the network blocks discovery (guest networks); the queue references music the T1 does not hold (offer to stream it from the desktop or skip it, and say which).

### 7.7 Checking a mix

Persona: sound engineer.

| Stage | Surface | What happens | Feeling | Applied |
|---|---|---|---|---|
| Export | Computer | A drop folder in OpenDAP Desktop | Speed | E3 |
| Carry | T1 | Arrives within seconds over the cable or the LAN; marked "Work" apart from the library | Focus | E8 |
| Check | T1 | Monitor profile; reference tracks level-matched; mono check, loudness and true-peak meters (audio profiles spec §5) | Precision | E12 |
| Note | T1 | Time-stamped notes, by a short text or, optionally, a voice note (the faceplate has a microphone) | Flow kept | E14; flow |
| Back | Desktop | Notes and timestamps appear next to the file | Closure | E20 |

**Fail points:** a mix in a format or rate the T1's path converts (shown clearly); notes edited on both sides (kept both, E9).

### 7.8 Leaving a streaming service

Persona: streaming switcher.

| Stage | Surface | What happens | Feeling | Applied |
|---|---|---|---|---|
| Bring the data | Desktop | Import playlists and history from the service's export file or an optional provider; ListenBrainz and Last.fm history too | Relief | E5 |
| Match | Desktop | Each entry matched to owned files by open identifiers (MusicBrainz, ISRC), then by title, artist and length; uncertain matches shown for review | Trust | Calibrated trust; E9 |
| What is missing | Desktop | A missing list, per album, across the enabled providers | Agency | E18 (no pressure to buy) |
| Day after the subscription | All | Playlists still exist; missing items appear greyed and say why, never vanish | Safety | E5; the Plex and Roon failure modes in §4 |

**Fail points:** an export file format changes; a service restricts its API (Spotify restricted access in 2024–2025, per its [developer blog](https://developer.spotify.com/blog/2024-11-27-changes-to-the-web-api)).

### 7.9 Updates and backups

Persona: every one; the most trust-sensitive journey.

| Stage | Surface | What happens | Feeling | Applied |
|---|---|---|---|---|
| Known | Desktop and T1 | "An update is ready": what users will notice, in plain words (2026-10-22 research, §7.5) | Informed | E11; honesty |
| Staged | Desktop or the T1 itself | Downloaded in the background by whichever device is online | Nothing to do | E10 |
| Applied | T1 | When idle and charging, or when the owner says; never during playback | Calm | E10 |
| Checked | T1 | A/B update with automatic rollback (OS-017); capabilities re-validated | Safety | E7, E2 |
| Undone | T1 or desktop | The previous version is one action away | Control | E7 |
| Backed up | Desktop | The T1's state (library database, playlists, history, profiles, settings) backed up at each sync, with versions | Peace of mind | E5 |

**Fail points:** power loss during an update (A/B covers it); an update fails its checks (automatic rollback, and it says so); an old desktop with a new T1 (versioned protocol, never a version lock, §4).

### 7.10 Repair, replacement, a second device

| Stage | Surface | What happens | Feeling | Applied |
|---|---|---|---|---|
| Battery ageing | T1 | Battery health visible; a guided replacement with common tools (T1-055) | Care | E20; EU battery regulation (Reg. (EU) 2023/1542) |
| Lost or broken | Desktop | The paired device revoked from any surface; the item tracker (BRD-030) helps find it | Control | E17 |
| New T1 | T1 + desktop | Pair; "restore from backup"; everything back: library selection, playlists, profiles, position | "It is mine again" | E5; reflective level |
| Second T1 | Both | One owner's library, each device with its own selection | Freedom | One owner, many replicas |

**Fail points:** no backup exists (offer it early and calmly, E11); the new device lacks a capability of the old one (disabled with the reason, SYS-040).

### 7.11 The household

| Stage | Surface | What happens | Feeling | Applied |
|---|---|---|---|---|
| A gift | T1 | Works on first power-on with a card or the shared library (7.1) | Joy | E1 |
| Shared collection | Desktop or NAS | One collection shared; each person's playlists, ratings, history and profiles stay theirs | Respect | E17 |
| Guest | T1 | A friend's phone may queue to a T1 playing on a speaker, if the owner allows it, with no access to private state | Sociality | Socio-pleasure; E17 |

**Fail points:** personal and shared state mixed (the data model must separate them from the start).

### 7.12 Away from home

| Stage | Surface | What happens | Feeling | Applied |
|---|---|---|---|---|
| No computer | T1 | Complete on its own | Independence | E1 |
| A speaker or a car | T1 | Bluetooth or the network, by standards; the codec in use is shown | Ease | Good citizen |
| New purchases | Online, if chosen | Through a relay the owner chose (their own or a provider's); off by default | Choice | E17; E1 holds when it is off |

### 7.13 Building and automating

Persona: builder.

| Stage | Surface | What happens | Feeling | Applied |
|---|---|---|---|---|
| Discover | CLI | `opendap device list` finds every T1 on USB, the network and MaskROM (CLI spec) | Competence | TK-009 |
| Do anything | CLI | Every desktop action exists in the CLI, from one core (DESK-004) | Freedom | E3 |
| Automate | CLI and agents | Scripted flows, the MCP server of the CLI spec | Leverage | TK-009 |
| Recover | CLI | MaskROM flashing and recovery always available (OS-017) | Safety to experiment | 2026-10-22 research, §4.7 |

### 7.14 Leaving OpenDAP

| Stage | Surface | What happens | Feeling | Applied |
|---|---|---|---|---|
| Export | Desktop or CLI | Everything in documented formats: files untouched, playlists (JSPF and M3U8), history (an open listen format), profiles (AutoEq / Equalizer APO format and the profile file), settings | Respect | E5 |
| Use elsewhere | Any software | Other players read the files and playlists | No regret | Local-first ideal 5; reflective level |

The ease of this journey is what makes staying a choice, and it is the strongest argument for the reflective level (§8).

## 8. Emotional design

### 8.1 The three levels per surface

| Level | T1 | Desktop | The whole system |
|---|---|---|---|
| **Visceral** | The case, the silver keys, the orange Action button, the screen, and above all the sound | A calm, precise interface in the same design system, with the device drawn faithfully | The sense of one family of objects |
| **Behavioural** | Instant response, buttons by feel, never losing the place, never a click or a pop | Fast with any library size; previews before changes; undo | Sync that works without being watched |
| **Reflective** | "I own this, I can open it, repair it, prove it" | "My collection is in order and it is mine" | "I can leave whenever I want, so I stay" |

The reflective level is the T1's strongest asset with its audience (Jordan's ideo-pleasure, the 2026-10-22 personas), but it holds only if the behavioural level holds (Norman 2004; Kano: no delight compensates a failed must-be).

### 8.2 Peaks and ends to design

| Moment | Kind | Design target |
|---|---|---|
| First sound out of the box | Peak | Before any setup, at a safe level |
| A calibrated headphone "opening up" | Peak | One tap, an instant level-matched comparison |
| A purchase that arrives on the T1 by itself | Peak | Marked "New" at first sight |
| The end of a listening session | End | The resume point saved; nothing to do |
| The end of a sync | End | A one-line summary; failures with a reason and a next step |
| The end of setup | End | A short summary of what is ready, nothing nagging after |
| An update | End | What changed, in plain words; how to undo it |
| Years later | Long tail | The battery replaced, the data intact, the device still updated |

### 8.3 Anxieties to design out

| Anxiety | Comes from | Designed out by |
|---|---|---|
| "Did it change my files?" | iTunes' cloud matching changing local files (user reports, 2015) | E6: no silent change; a log; undo |
| "Is it on my T1?" | Invisible sync | E8: one-glance state |
| "Is it really bit-perfect?" | Hidden processing on other players | The signal path shown (audio profiles spec §3); verifiable by measurement (BRD-040) |
| "Will the update brick it?" | Updates that cannot be reverted | A/B and rollback (OS-017); recovery guaranteed |
| "What if the store or service goes away?" | Plex and TIDAL, Qobuz withdrawals | Files and open identifiers (E5) |
| "Am I hurting my hearing?" | Loud, long listening | A calm dose display and a mode the owner chooses (E19) |
| "Who sees what I listen to?" | Cloud players | E17: local by default |
| "I'll break it if I tinker" | Locked devices | Guaranteed recovery (OS-017, MaskROM) |

## 9. UX laws across the ecosystem

From [Laws of UX](https://lawsofux.com/). The T1-only applications are in the [controls and gestures research](2026-10-controls-gestures-small-screen.md) §3.

| Law | On the T1 | On the desktop | Across devices |
|---|---|---|---|
| **Fitts's law** | Large primary target, edge buttons, edge gestures | Bulk actions near the selection | Do on each surface what its pointing allows: no fine editing on 3.5" |
| **Hick's law** | Few choices per screen | Grouped options; advanced settings one level down | The task's surface carries its choices, not every surface |
| **Miller's law, chunking** | Metadata grouped | Editions, health issues and sync changes grouped | Sync summaries chunked: arrived, waiting, failed |
| **Jakob's law** | Phone and headset conventions | Desktop app conventions on each host (macOS first) | Pairing like Bluetooth or a TV code; "continue here" like phone handoff |
| **Doherty threshold** | The T1's own, stricter budgets (T1-057) | Every view within the threshold with any library size | Remote actions give feedback before the round trip (E13) |
| **Tesler's law** | Profiles carry the complexity | The desktop absorbs tagging, matching, formats | The system chooses the link (USB, LAN, internet) |
| **Postel's law and its critique** | Accept messy tags and files | Repair or flag, never reject silently | The own protocol strict and versioned ([RFC 9413](https://www.rfc-editor.org/rfc/rfc9413)) |
| **Peak-end rule** | First sound; session end | Sync and setup summaries | §8.2 |
| **Zeigarnik effect** | A pending sync stays visible until done | Progress and a clear end | Never leave a task "almost done" without saying so |
| **Goal-gradient effect** | A short visible setup path | First-sync progress | Restore from backup shows what is back |
| **Aesthetic-usability effect** | The design system on a small panel | The same system on the desktop | One family; but polish never hides a missing must-be (E2) |
| **Von Restorff effect** | One orange primary action | One primary action per view | — |
| **Law of common region, proximity, similarity** | Segmented capsules (DS-013) | Grouped panels | The same element looks the same on every surface (consistent, E4) |
| **Serial position effect** | First and last items of a list stand out | Health issues ordered by impact | Summaries lead with what needs attention |
| **Paradox of the active user** | People will not read a manual: teach in context | Same | Setup teaches by doing, in two or three steps |
| **Cognitive load** (Sweller 1988) | The T1 keeps per-screen choices few | The desktop takes the complex work where attention is available | Move complexity to where attention is |

## 10. Kano

| Class | Attributes |
|---|---|
| **Must-be** (absence fails, presence unnoticed) | Gapless and bit-perfect playback; playback in sync across every device in real time (SYS-042); no clicks or pops; works with no other device; no data ever lost; files never changed silently; sync that is correct; resume exactly where left; updates that can be undone; private by default |
| **One-dimensional** (more is better) | Battery hours; library size handled without slowing; sync and transfer speed; search speed; how much of the library fits (card capacity, optional transcoding per device) |
| **Attractive** (delight, no penalty when absent) | Calibrated offered for a known headphone; collection health view; a purchase that arrives by itself; a wish list captured on the T1; "on this day" and rediscovery from the owner's own history; level-matched blind A/B; notes on a mix |
| **Indifferent** (for most) | Social feeds; badges; listening "streaks" |
| **Reverse** (presence dissatisfies) | A mandatory account; telemetry by default; engagement notifications; automatic library reorganisation; a default store; ads |

Kano's attributes drift: today's attractive becomes tomorrow's must-be. Handoff and two-way sync are likely to become expected, so they are designed into the architecture now, not added later.

## 11. Service blueprints

Front stage is what the listener sees; back stage is what runs behind the line of visibility (Bitner et al. 2008). Every back-stage step has a fail point with a visible state front stage (E8, E20).

### 11.1 From purchase to playback

| Lane | Find | Buy | Arrive | Reach the T1 | Play |
|---|---|---|---|---|---|
| **Listener** | Searches, compares editions | Pays on the provider's checkout | Sees "Downloading", then "In library" | Sees "On its way to the T1", then "On the T1" | Plays |
| **Front stage** | Desktop: results across providers, owned items marked | Provider's site or flow | Desktop: import progress, tags, art | Desktop and T1: one-line sync state | T1: "New" |
| **Back stage** | Provider adapters: search | Provider: order, entitlement | Download; verify; identify (tags; metadata providers if enabled, SYS-045); write tags only to the library copy; record provenance | Sync service: plan per device rules; transfer resumable and verified; T1 indexer adds it in the background | Audio engine: profile for the output |
| **Support** | Provider availability; regional catalogue | Payment (the provider's) | Metadata services; storage | Protocol; links (USB, LAN, relay) | — |
| **Fail points** | A provider's search fails: show results from the others | A failed payment is the provider's: say so | Incomplete download: retry, resume; unidentified: queue for review | No link: "Waiting for the T1"; no space: say how much and offer choices | A format the path converts: say so |

### 11.2 Sync between the owner's devices

| Lane | Meet | Exchange | Settle | Report |
|---|---|---|---|---|
| **Listener** | Plugs in, or comes home | Keeps listening | — | Glances at the state |
| **Front stage** | "Syncing" in the periphery (quick settings and the desktop) | Progress, never a modal | — | "Synced 2 min ago"; a list of what changed |
| **Back stage** | Discovery and mutual authentication | State changes (playlists, ratings, plays as timestamped events, profiles) and files by device rules; heavy work deferred to idle or charging | Merge; conflicts kept both ways; deletions held recoverable | Summary written; backup versioned |
| **Fail points** | Not found on the network: try other links, then say how to connect | Interrupted: resume; no space: stop cleanly | A real conflict: shown once, plainly | Something failed: the reason and the next step |

## 12. What each part does

The responsibilities that follow from the journeys. They are needs, not designs; how each is built is for the maintainers, and none of them chooses a technology.

### 12.1 Responsibility matrix

● primary · ○ supporting · — not involved. "Online" means optional services the owner chooses, from any provider or the owner's own host.

| Capability | T1 hardware | Blackbird OS and system | T1 UI | Desktop app | Desktop service | CLI | Protocol | Online |
|---|---|---|---|---|---|---|---|---|
| Playback, profiles, signal path | ● | ● | ● | ○ (remote, edit) | — | ○ | ○ | — |
| Library on the device (index, browse, search) | ○ (storage) | ● | ● | ○ | — | ○ | ○ | — |
| Master library, curation, health | — | — | — | ● | ○ | ● | — | ○ (metadata) |
| Acquisition through providers | — | — | ○ (wish list) | ● | ○ (downloads) | ○ | — | ● (providers) |
| Device selection rules and transfer | ○ (USB, radios, card) | ● (receiver) | ○ (state) | ● | ● | ● | ● | ○ (relay) |
| State sync (playlists, ratings, plays, profiles) | — | ● | ○ | ● | ● | ○ | ● | ○ (relay) |
| Handoff and remote control | — | ● | ● | ● | ○ | ○ | ● | ○ |
| Pairing and trust | ○ (key storage) | ● | ● | ● | ○ | ● | ● | — |
| Updates (stage, apply, roll back) | ○ (A/B storage) | ● | ● | ● | ● | ● | ● | ○ (download) |
| Backup and restore | — | ● | ○ | ● | ● | ● | ● | ○ (off-site, opt-in) |
| Standards at the edges (UAC2, A2DP and AVRCP, UPnP, MPRIS, mDNS) | ● | ● | ○ | ○ | — | — | ○ | — |
| Engineering tools (meters, A/B, measurement sessions) | ● (test pads) | ● | ● | ● | — | ● | ○ | — |
| Export and leaving | — | ○ | ○ | ● | — | ● | — | — |
| Extension (capabilities, HATs, automation) | ● (HAT interface) | ● | ○ | ○ | — | ● | ○ | ○ (package endpoint) |

### 12.2 Software and OS (Blackbird OS, `system`)

1. **A device-side library engine with a change log.** It indexes in the background with bounded memory (SYS-033, T1-046). It keeps a replica's state, records changes as events, and accepts changes from peers without blocking playback (SYS-036).
2. **Play statistics as timestamped events,** kept offline and merged without losing when each play happened (the Rockbox and Finamp patterns; the Jellyfin failure).
3. **Owner-scoped pairing and trust:** device identity, the owner's set of devices, revocation, per-peer permissions on the `org.opendap.*` interfaces (protocol research §6.2).
4. **Sync and transfer services** that choose the link, defer heavy work to idle or charging, resume, verify content (not timestamps, given FAT's two-second granularity), and expose their state to every surface.
5. **Recoverable deletion** across devices, with a retention window.
6. **Handoff:** the position, the queue and the profile's intent travel as one state.
7. **Standards at the edges:** UAC2 gadget (T1-054), A2DP source and sink with AVRCP target and controller, UPnP/DLNA renderer and server, an mDNS presence, MPRIS on the device's session (protocol research §6.3). The proprietary ecosystems stay optional (OS-027).
8. **Backup and restore** of the whole device state, in documented formats.
9. **A "Devices" place in Settings:** paired devices, their last sync, revoke, the links in use. The sync state also appears in quick settings (SYS-039), not as a permanent status-bar item, to keep the status bar calm (SYS-038).
10. **Updates applied only when idle and charging** or on request, never during playback, with rollback (OS-017).

### 12.3 Hardware (`t1`, `board`)

1. **Plugging in is meaningful.** The main USB-C port (BRD-034) carries charging, a composite gadget for the desktop (networking, serial, and the USB DAC role when chosen) and host mode for an external DAC. One cable to a computer means "charge, sync, back up" at once.
2. **Sync follows charging.** Charging is when heavy sync, backups and updates run, so battery life (T1-039) is not spent on them. That ties the power path, charging detection and the sync scheduler together.
3. **Presence and wake-up.** Bluetooth LE lets the desktop find a sleeping T1 and wake it for a sync, within the standby budget. The standby cost must be measured; if it is too high, sync waits for the next wake.
4. **Device keys stored safely.** Device identity should live where the SoC offers protected storage (a TrustZone-backed store, if the platform provides it). This is **UNVERIFIED** for the chosen SoC and is an open point for `board`.
5. **The card is a link too.** The SD/TF card (BRD-038) carries a library prepared by the desktop with no other link. Hot-plug detection and incremental indexing make that seamless.
6. **The status LED shows device-to-device states,** as part of its code (T1-044): pairing, transferring and update in progress, visible with the screen off.
7. **Item tracking** (BRD-030) supports the lost-device journey.
8. **Options for the maintainers** (each with a BOM cost to weigh against BRD-053):
   - NFC for tap-to-pair and tap-to-hand-off. A precedent is phones' tap-to-share; it is not needed, because the short code works.
   - A haptic actuator for eyes-free confirmation (E14).

### 12.4 OpenDAP Desktop (`desktop`)

1. **The master library and its health view** (E15): identification, tags, art, editions, duplicates, files not on any device. Batch fixes with preview and undo (E6, E7).
2. **Device sync rules** per device: everything, selections, smart rules, fill the free space, optional transcoding for space. A preview of changes before applying, and nothing destructive by default (the iTunes lessons in §4).
3. **Providers:** optional adapters that declare what they can do (search, hand off to checkout, download, list purchases, metadata, scrobbling, playlist import). Purchases land as files with provenance. No provider is a default and none is privileged (E18).
4. **Unified playlists** keyed by open identifiers (JSPF/XSPF semantics, MusicBrainz and ISRC), resolved to local files first. An entry that cannot be resolved stays, greyed, with the reason.
5. **History and statistics** merged from every device as events, exportable.
6. **Audio profiles in depth:** curves, measurements, targets, tone macros, A/B with the T1. Sharing as documented files.
7. **Device management:** pairing, updates staged for the T1, backups with versions, restore to a new device, diagnostics and logs the owner can read and choose to send.
8. **A drop folder** for work files (the sound engineer's mixes), outside the library.
9. **The background service** (DESK-006) does what must happen with the app closed: find devices, sync, back up, download updates and notify. It stays light (DESK-008).
10. **Speed with any library size** (T1-046 applies to the desktop too): the Sony Music Center complaints are the failure to avoid.

### 12.5 The `opendap` CLI (`toolkit-cli`)

1. **Everything the desktop does**, from the same core (DESK-004): library, sync, providers, profiles, devices, backups and exports.
2. **Structured output for agents** and an MCP server (TK-009, CLI spec), so automation and assistants use the same verbs as people.
3. **Recovery and flashing** paths that work when nothing else does (OS-017).

### 12.6 The protocol (`system`, then an RFC)

As described in the [protocol research](https://github.com/opendap-project/system/blob/main/docs/research/2026-10-device-ecosystem-protocol.md):

- one owner, many devices;
- the device's own interfaces, carried off the device;
- link-independent, never on the critical path;
- visible state, standards at the edges;
- conformance before compatibility.

Prose says *OpenDAP Protocol*; technical identifiers use `odap`.

### 12.7 Optional online services

Public principle only:

- Anything online is optional and replaceable by the owner's own host.
- The device never requires an online service to play, browse, sync with the owner's devices or update from a local copy.
- With every online service off, nothing in the experience degrades except reach beyond the home and off-site copies.

### 12.8 The phone

An OpenDAP phone app will exist, after OpenDAP Desktop (T1-068). Until then, the phone works through standards:

- Bluetooth source and remote;
- UPnP control points;
- OpenSubsonic clients, if the T1 or the desktop serves that API.

The journeys show where a phone remote would add most: the home listener's remote (§7.6), the guest queue (§7.11) and the wish list (§7.3). Its scope is to be defined from these journeys.

## 13. Open questions and next steps

### 13.1 Decided by the maintainers (2026-10-23)

The questions first raised here were answered; the requirements are recorded in each repository.

| Question | Decision | Recorded |
|---|---|---|
| Copying music to the T1 without vendor software | Plain files: over USB the T1 appears by default as a standard storage device with an organised directory tree; OpenDAP Desktop and the CLI are recommended | T1-065 (Draft: the layout is to define) |
| The T1 without the desktop, and purchases | The T1 is fully useful without the desktop: files bought (for example FLAC) and loaded by any means simply play. On the device, a wish list; a provider may offer one-action purchase, opt-in in that provider's settings | T1-066, T1-067 |
| Sync and connected features with a computer | Through OpenDAP Desktop or the CLI, over any link; without them, plain USB storage and standalone use | DESK-011 |
| Transcoding on sync | Offered, per device, optional | DESK-010 |
| A phone app | Yes; OpenDAP Desktop comes first | T1-068 |
| The household | A must: one shared collection with each member's personal state; family members in the manner of streaming services' profiles, also a must | SYS-043, SYS-044 |
| Playback across devices | Real-time sync of playback across all of the listener's devices is a must | SYS-042 |
| Multi-room (the same music on several devices at once) | A should | SYS-046 |
| What the protocol syncs | Everything, down to the library index and its details, so the computer can prepare it; without a desktop client the T1 works on demand | OS-029 |
| Third parties (MusicBrainz, AcoustID, stores, streaming, scrobbling) | All can be integrated, always as opt-in providers, never in the core | SYS-045 |
| NFC and haptics | Yes, when suitable parts are available | BRD-057, BRD-058 |

**What changes in this research:**

- **Identification and enrichment are providers** (SYS-045). The core reads identifiers already in the files' tags (a MusicBrainz ID, an ISRC) and the listener's own data. Lookups against MusicBrainz, AcoustID, Discogs or a store happen only through providers the listener enables. With none enabled, the library works from the tags as they are. This also settles the licence question: each provider follows its service's terms.
- **Real-time playback sync is a must** (SYS-042). Handoff (§7.6) and "now playing" are no longer attractive extras (§10). They become must-be: the queue, the position and the state are live on every device, and any device takes over.
- **The household is in the data model from the start** (SYS-043): the collection and the personal state are separate objects (desktop research §3.1).

### 13.2 Validation with people

1. Interviews with each persona about the cross-device jobs of §6.3: where music lives today, what they fear losing.
2. A usability test of the first day (§7.1) on the virtual T1 (OS-018) with the desktop.
3. A diary study of a month of buying, curating and listening, once prototypes exist.
4. The outcome statements of §6.1 rated for importance and satisfaction, the outcome-driven way, to find what is most under-served.

## 14. Sources

### Theory and research

- Bettencourt, L. and Ulwick, A. (2008). "The Customer-Centered Innovation Map". *Harvard Business Review* 86(5). <https://hbr.org/2008/05/the-customer-centered-innovation-map>
- Bitner, M. J., Ostrom, A. and Morgan, F. (2008). "Service blueprinting". *California Management Review* 50(3).
- Case, A. (2015). *Calm Technology*. O'Reilly. <https://calmtech.com/>
- Cavoukian, A. (2009). *Privacy by Design: the 7 foundational principles*. Information and Privacy Commissioner of Ontario.
- Chalmers, M. and MacColl, I. (2003). "Seamful and seamless design in ubiquitous computing". UbiComp 2003 workshop. <https://www.alandix.com/academic/conf/ubicomp-models/pdf/maccoll.pdf>
- Csikszentmihalyi, M. (1990). *Flow*. Harper & Row.
- Cunningham, S. J., Jones, M. and Jones, S. (2004). "Organizing digital music for use". ISMIR 2004. <https://researchcommons.waikato.ac.nz/handle/10289/66>
- Dearman, D. and Pierce, J. (2008). "It's on my other computer!". CHI 2008.
- Gray, C. M. et al. (2018). "The dark (patterns) side of UX design". CHI 2018.
- Holmes, K. (2018). *Mismatch*. MIT Press; Microsoft (2016). *Inclusive Design Toolkit*.
- Juslin, P. and Västfjäll, D. (2008). "Emotional responses to music". *Behavioral and Brain Sciences* 31(5).
- Kahneman, D. et al. (1993). "When more pain is preferred to less". *Psychological Science* 4(6).
- Kano, N. et al. (1984). "Attractive quality and must-be quality". *Journal of the Japanese Society for Quality Control* 14(2).
- Kleppmann, M., Wiggins, A., van Hardenberg, P. and McGranaghan, M. (2019). "Local-first software". Onward! 2019. <https://www.inkandswitch.com/local-first/>
- Klement, A. (2016). *When Coffee and Kale Compete*.
- Krause, A., North, A. and Hewitt, L. (2015). "Music-listening in everyday life: devices and choice". *Psychology of Music* 43(2). <https://researchonline.jcu.edu.au/64286/>
- Lee, J. and See, K. (2004). "Trust in automation". *Human Factors* 46(1). <https://doi.org/10.1518/hfes.46.1.50_30392>
- Lemon, K. and Verhoef, P. (2016). "Understanding customer experience throughout the customer journey". *Journal of Marketing* 80(6).
- Levin, M. (2014). *Designing Multi-Device Experiences*. O'Reilly.
- Lonsdale, A. and North, A. (2011). "Why do we listen to music?". *British Journal of Psychology* 102(1). <https://researchportal.hw.ac.uk/en/publications/why-do-we-listen-to-music-a-uses-and-gratifications-analysis/>
- Mathur, A. et al. (2019). "Dark patterns at scale". *Proc. ACM HCI* 3 (CSCW).
- Nielsen, J. (1994). "10 usability heuristics"; (2006). "Progressive disclosure". Nielsen Norman Group.
- Norman, D. (2004). *Emotional Design*. Basic Books.
- Odom, W., Zimmerman, J. and Forlizzi, J. (2011). "Teenagers and their virtual possessions". CHI 2011; (2014). "Placelessness, spacelessness, and formlessness". DIS 2014. <https://hcii.cmu.edu/project/virtual-possessions>
- Pierce, J., Kostova, T. and Dirks, K. (2001). "Toward a theory of psychological ownership in organizations". *Academy of Management Review* 26(2). <https://doi.org/10.5465/amr.2001.4378028>
- Schäfer, T., Sedlmeier, P., Städtler, C. and Huron, D. (2013). "The psychological functions of music listening". *Frontiers in Psychology* 4:511. <https://frontiersin.org/articles/10.3389/fpsyg.2013.00511/full>
- Shapiro, M. et al. (2011). "Conflict-free replicated data types". SSS 2011.
- Shostack, G. L. (1984). "Designing services that deliver". *Harvard Business Review* 62(1).
- Sloboda, J., O'Neill, S. and Ivaldi, A. (2001). "Functions of music in everyday life". *Musicae Scientiae* 5(1).
- Sweller, J. (1988). "Cognitive load during problem solving". *Cognitive Science* 12(2).
- Terry, D. et al. (1995). "Managing update conflicts in Bayou". SOSP 1995.
- Toole, F. and Olive, S. (1994). "Hearing is believing vs. believing is hearing". AES 97th Convention.
- Weiser, M. and Brown, J. S. (1996). "Designing calm technology". *PowerGrid Journal* 1.01. <https://calmtech.com/papers/designing-calm-technology.html>
- WHO and ITU (2019, revised 2022). *Safe listening devices and systems*, ITU-T H.870. <https://www.who.int/publications/i/item/9789241515276>
- Laws of UX. <https://lawsofux.com/>
- RFC 9413 (2023). "Maintaining robust protocols". <https://www.rfc-editor.org/rfc/rfc9413>

Bibliographic details of the books and papers without a link were cited from standard references and are to be spot-checked before quoting them outside this research.

### Products

Listed with their findings in the [desktop companion research](https://github.com/opendap-project/desktop/blob/main/docs/research/2026-10-desktop-companion.md) and the [protocol research](https://github.com/opendap-project/system/blob/main/docs/research/2026-10-device-ecosystem-protocol.md).
