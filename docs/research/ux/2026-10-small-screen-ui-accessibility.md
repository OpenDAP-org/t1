# UX research: interface and accessibility on the T1's screen

- **Date:** 2026-10-23
- **Status:** Desk research, with the decisions it led to recorded in the [design system](https://github.com/opendap-project/ui-components/blob/main/docs/specs/design-system.md) (Engineered Matter) and in DS-015 to DS-020 and SYS-061 to SYS-066. Nothing was tested with users or on the real panel. Values to confirm on the virtual T1 (OS-018) and on the first prototype are marked as such.
- **Question:** how should text, icons, layout, controls, gestures and accessibility work within the T1's real screen? The screen is a 3.5" panel, 320 × 480 px at about 166 ppi (1 px = 0.153 mm, active area 48.96 × 73.44 mm), read at arm's length, often while walking, in sunlight, held in one hand, and often used without looking.
- **Builds on:**
  - the [controls and gestures research](2026-10-controls-gestures-small-screen.md) (targets, gestures, buttons);
  - the [ecosystem experience research](2026-10-ecosystem-experience.md) (principles E1 to E20);
  - the [interface mockups](../../../renders/web/ui-mockups.html), which apply this document.
- **Method:**
  - three benches with primary sources fetched on 2026-10-23: small-screen platforms and dedicated devices; legibility and accessibility standards and research; icons, text and localisation;
  - measurements on the IBM Plex font files;
  - ICU 78.2 and CLDR 48 output.

  What could not be read from a primary source is **UNVERIFIED**.

## 1. Summary

1. **The T1 is a dedicated instrument, not a small phone** (SYS-017).
   - The screen and the four buttons are one interface, and what a button does is drawn beside it.
   - Now Playing is home; the library lies below it and the queue to its right.
   - Structure comes from type, artwork and hairlines, not from cards and tiles.

   Familiar mobile patterns are used only after asking what they do on this device, and are shaped for it (§2).
2. **Text is sized for 35 cm, not 25 cm.**
   - The 14 px body of the first type scale has a cap height of about 1.5 mm. That is below the 16-minute-of-arc minimum for legibility-critical text (ANSI/HFS 100, quoted in the [FAA Human Factors Design Standard](https://hf.tc.faa.gov/hfds/)) beyond about 33 cm.
   - The scale moves up one step: reading text 17 px, secondary 15, labels 13, the smallest 12 (§3).
3. **Space is deliberate.**
   - 16 px side margins (2.4 mm), because 24 px gave up 15 % of a 49 mm-wide screen.
   - Spacing follows proximity, three steps from "one unit" to "another group", never one gap for everything (§4).
4. **Controls are not interchangeable** (§5).
   - Tabs navigate.
   - Button groups act or choose a value.
   - Toggles switch a state.
   - Radio stops choose one of a few options on a scale.
5. **Icons come with words.**
   - IBM Carbon icons, drawn for IBM Plex, with whole-pixel strokes.
   - Visible labels except for transport, volume and status glyphs, which are standardised and learned (§6).
6. **Long text is cut, not scrolled.**
   - An end ellipsis, with no automatic marquee.
   - The Now Playing title wraps to two lines first.
   - Per-script minimum sizes and line heights for Chinese, Devanagari and Arabic (§7).
7. **Accessibility is built in, not added** (§8).
   - Voice guidance driven from the buttons, because Linux has no touch screen reader.
   - Text size up to about 200 % with reflow, and a large-text mode above it.
   - Mono and balance; configurable timings.
   - A light, high-contrast appearance for sunlight and for low vision.

## 2. A dedicated instrument

The rule (SYS-017): the UI must feel like the purpose-built interface of a consumer electronic product, not an operating system. The first mockups followed phone structures: cards inside cards, a quick-settings tile grid, generic settings lists. The current design starts from what the T1 is instead.

| Principle | Grounded in | On the T1 |
|---|---|---|
| **One instrument** | Natural mapping (Norman, *The Design of Everyday Things*): a control and its effect share a place | The physical controls sit at fixed heights beside the panel: Action at about 38 px from the top, Volume + at 144, Volume − at 226 (left edge), Main at 90 (right edge), from the concept art geometry. Volume is drawn on the left edge between its keys; the power choices appear next to Main, and a second press of Main confirms power off; an edge mark beside Action shows when Action does something other than play/pause (A/B switching) |
| **Music is the home** | Continuity (T1-051); the dedicated device's first job (design system §8.1) | The T1 wakes in Now Playing. One spatial model: the library below (swipe down, or the chevron), the queue to the right (trailing edge, or its control), details below the player (swipe up). No bottom tab bar, no launcher |
| **Content, not chrome** | Tesler (the system carries complexity); aesthetic-usability; Garmin's and watchOS's "content first" | Type, artwork and hairlines carry structure. No card inside a card, no floating action button, no hamburger. The playing strip is a full-width band, not a floating card |
| **Engineered clarity** | Engineered Matter identity (technical drawings are part of it); psychological ownership through intimate knowledge (Pierce et al. 2001) | Technical values in IBM Plex Mono, right-aligned like a readout. The signal path is a chain diagram; profiles sit on the SYS-041 axis from "enjoy" to "analyse" |
| **Eyes-free first** | Most listening happens alongside other activities (Sloboda et al. 2001); glance limits from driving (NHTSA guidelines, 2 s mean glance, [78 FR 24818](https://www.govinfo.gov/content/pkg/FR-2013-04-26/pdf/2013-09883.pdf)) | What matters while walking is on a button; the screen confirms. Now Playing's essentials (title, artist, state) read in one glance |
| **Calm** | Calm technology (Weiser and Brown 1996) | The status bar unfolds into its own detail, item by item in the same order, instead of a tile grid. No colour fills for "on" |

**What this changes, concretely:**

- Settings is an index of the device's parts ("This T1": Sound, Library, Buttons and screen, Devices, Storage, Accessibility…), reached from the status panel, not from a gear in every header.
- The library's header carries one action, search.
- The album page puts Play at the bottom, in the thumb zone, not under the title.
- Profiles are positions on an axis, not rows in a list.

## 3. Text

### 3.1 How big text must be

Character height by visual angle, from ANSI/HFS 100-1988 as quoted in the FAA Human Factors Design Standard:

- 16′ (minutes of arc) is the minimum where legibility matters;
- 20–22′ is preferred;
- 10′ is acceptable only where reading is not time-critical.

Converted to the T1 (h = 2·D·tan(θ/2), 0.153 mm per px; cap height of IBM Plex Sans 0.698 em):

| Font size | Cap height | At 25 cm | At 35 cm | At 40 cm |
|---|---|---|---|---|
| 11 px | 1.17 mm | 16.1′ | 11.5′ | 10.1′ |
| 12 px | 1.28 mm | 17.6′ | 12.6′ | 11.0′ |
| 13 px | 1.39 mm | 19.1′ | 13.6′ | 11.9′ |
| 14 px | 1.50 mm | 20.6′ | 14.7′ | 12.9′ |
| 15 px | 1.60 mm | 22.0′ | 15.7′ | 13.8′ |
| 16 px | 1.71 mm | 23.5′ | 16.8′ | 14.7′ |
| 17 px | 1.82 mm | 25.0′ | 17.8′ | 15.6′ |
| 20 px | 2.14 mm | 29.4′ | 21.0′ | 18.4′ |

The design distance is **35 cm**: a handheld held in front of the chest while walking, between the 25 cm close reading and the 40 cm that EN 301 549 assumes for handhelds ([ETSI](https://www.etsi.org/deliver/etsi_en/301500_301599/301549/)). At 35 cm:

- reading text needs **≥ 16 px** (17 px gives margin);
- secondary text sits near 15–16′ at 15 px;
- anything smaller is for identification that is not time-critical.

The first scale's 14 px body and 11 px caption fall below those lines. The reading research agrees: at 35–40 cm, a 14 px body is at or below the critical print size of normally sighted readers ([Legge and Bigelow 2011](https://pmc.ncbi.nlm.nih.gov/articles/PMC3428264/)), so anyone with reduced vision reads slowly.

### 3.2 The type scale

Decided under the maintainers' delegation (DS-015); values to confirm on the panel:

| Role | Was | Now (size / line height, px) | Use |
|---|---|---|---|
| `display` | 32 / 38 | **32 / 40** | Large numbers: volume overlay, readouts |
| `headline` | 24 / 30 | **24 / 32** | Screen titles that carry the screen (Library) |
| `title` | 20 / 26 | **20 / 28** | Track title, screen titles |
| `body-large` | 16 / 22 | **17 / 24** | Primary list rows, reading text |
| `body` | 14 / 20 | **15 / 22** | Secondary lines, sentences |
| `label` | 12 / 16 | **13 / 18** | Control labels, metadata, section labels |
| `caption` | 11 / 14 | **12 / 16** | Status bar, timestamps, technical values; the minimum |

- **Nothing below 12 px.** It matches the watchOS minimum (12 pt; one point on the original iPhone panel is about one T1 px) and Wear OS's Body small (12 sp).
- **Line heights on a 2 px grid**, set explicitly: never "normal", because Plex Sans SC's 1.0 em line gap would double them.
- **Larger text** scales these roles nonlinearly (§8.2).

### 3.3 Scripts

From the font measurements and W3C's layout requirements for [Chinese](https://www.w3.org/TR/clreq/), [Indic](https://www.w3.org/TR/ilreq/) and [Arabic](https://www.w3.org/TR/alreq/) (DS-016):

| Script | Minimum for captions and metadata | Line height | Notes |
|---|---|---|---|
| Latin | 12 px | 1.3–1.4 em in rows, 1.45 in sentences | — |
| Han (Plex Sans SC) | 12 px | 1.4–1.5 em | Digits from the Latin face: SC's digits are 630 units wide, Plex Sans's 600 |
| Devanagari | 13 px | 1.5–1.6 em | Ink reaches about 1.25 em on common words, 1.43 em across the block |
| Arabic | 13 px | 1.5–1.6 em | About 1.34 em with vowel marks; layout mirrors (design system §9.5) |

These become per-locale size and line-height tokens, never per-string overrides. Devanagari and Arabic renders on the T1's own renderer are still to be checked (Slint's Skia renderer uses subpixel positioning; hinting on the device is **UNVERIFIED**).

### 3.4 Long text

- **End ellipsis** for titles, artists and albums; **middle ellipsis** for file and device names; the ellipsis at the logical end in Arabic.
- **No automatic marquee.**
  - Driving guidelines treat automatically scrolling text as unacceptable for eyes-busy use (NHTSA).
  - WCAG 2.2.2 requires moving text to stop within 5 s or be stoppable ([W3C](https://www.w3.org/WAI/WCAG22/Understanding/pause-stop-hide.html)).
  - Scrolling only the focused row after a delay is a setting, off by default (the Rockbox model: 1 s delay, only the selected line).
- **The Now Playing title wraps to two lines** before it is cut; the full text is always reachable (the More view, and the accessible name).

### 3.5 Translation and formats

- **Expansion budgets:**
  - 300 % for buttons and tabs of up to 10 characters (or wrap to two lines);
  - 200 % for labels of 11–20 characters;
  - 160–180 % for sentences ([W3C, text size in translation](https://www.w3.org/International/articles/article-text-size)).

  Side-by-side buttons are allowed only where the longest translation fits; otherwise they stack.
- **Formats from CLDR**, never concatenated:
  - Hindi groups as 12,34,567;
  - Spain's Spanish writes "44,1 kHz";
  - Arabic for Egypt and Saudi Arabia uses Arabic-Indic digits;
  - Arabic has six plural forms.

  Technical values keep Western digits in Plex Mono everywhere (design system §9.4).
- **Sentence case everywhere**, never all caps.

## 4. Space

### 4.1 Margins

**16 px (2.4 mm)** on both sides, leaving 288 px for content.

- The first mockups used 24 px, inherited from an assumed 272 px content width. On a 49 mm-wide screen that gives away 7 mm of a line that is already short.
- 16 px keeps text clear of the bezel and the case lip, and matches the phone convention people know (16 dp).
- The 20 px edge-gesture zones overlap the margin; that is fine, because they only react to swipes that start at the edge and move inward.

### 4.2 Proximity

The Gestalt law of proximity says spacing tells what belongs together. Three steps, chosen per relationship, never applied as one automatic gap (DS-017):

| Relationship | Space | Example |
|---|---|---|
| Within one unit | 0–4 px (`space.unit`) | Title and artist; a label and its value; a value and the slider it sets |
| Within a group | 8–12 px (`space.group`) | Items of one section; an image and its caption; the format line and the seek bar |
| Between groups | 16–24 px (`space.section`) | Sections of a screen; a readout and its explanation from the list below it |

Lists separate their lines with hairlines (`border`, 1 px), not with gaps or cards. Section labels sit 12 px after the previous group and 4 px before their first line.

### 4.3 Rows and density

| Row | Height | Fits on a screen (status bar 28 + header 48 + playing strip 64) |
|---|---|---|
| One line (17/24) | 52 px | 6–7 |
| Two lines (17/24 + 13/18) | 60 px | 5–6 |
| Hit area minimum | 48 px | — |

This density matches the platforms that work at this size:

- Rockbox's touchscreen default of one third of an inch, which is 55 px on the T1 (Rockbox `apps/gui/list.c`);
- Garmin's guidance of about seven items per menu.

Long lists keep their whole content (T1-046) and get an index on the trailing edge, not a cut.

### 4.4 Navigation depth

- Any track is at most three selections from the library: Library → Artists → album → track.
- No path is deeper than five levels (the limit CarPlay sets for audio apps).
- Now Playing is one action away from anywhere.
- Back always behaves the same way: the leading edge, or the back control, which never carries another function.

## 5. Which control for which job

The first mockups used the segmented capsule (design system §15.1) for the library's browse switch. That switch is navigation, and navigation is a tab. The kinds of control are kept apart (DS-018):

| Control | Job | Looks like | Example on the T1 |
|---|---|---|---|
| **Tabs** | Switch between views of the same level; the content below changes | Text labels in a row with an underline under the current one; no container; at least 48 px per tab | Library: Albums, Artists, Playlists, More |
| **Button group** (segmented capsule) | Several actions that belong together, or one value chosen from two to five options that act immediately | One capsule split into segments, rounded outer ends | Pause / Sync now; a DAC filter or output selector |
| **Toggle** | One setting on or off | The setting's name with its state in words or a switch | Bluetooth on; Blind comparison off |
| **Radio stops on an axis** | One of a few options placed on a meaningful scale | Stops on a line with the selected one enlarged | Profiles from "enjoy" to "analyse" |
| **List line** | Go to a thing or a sub-screen | A full-width line with a chevron | Settings parts, artists |
| **Primary action** | The screen's one main action, also on the Action button | The orange capsule or disc, 64 px | Play, Eject, Pair, Continue |

## 6. Icons

- **Set: IBM Carbon icons** (Apache-2.0).
  - Drawn to match IBM Plex: 32 px grid, 2 px stroke, square caps ([Carbon](https://carbondesignsystem.com/elements/icons/usage/), [IBM](https://www.ibm.com/design/language/iconography/ui-icons/design/)).
  - They cover play, pause, skip, shuffle, repeat, volume, Bluetooth, Wi-Fi, battery, headphones, playlist, equaliser and favourite.
  - Missing glyphs (album, lyrics) are drawn in the same grammar.
  - [Material Symbols](https://developers.google.com/fonts/docs/material_symbols) (Apache-2.0) is the fallback.
  - The licence is kept per file under REUSE.
- **Sizes:**

  | Context | Size | Physical |
  |---|---|---|
  | Status bar | 15–16 px | 2.3–2.4 mm |
  | Beside text in rows | 16–20 px | 2.4–3.1 mm |
  | Header actions, secondary transport | 24 px | 3.7 mm |
  | Primary transport | 28–32 px | 4.3–4.9 mm |

- **Strokes:** whole pixels only (2 px at 24 and 32). A 32 px master scaled to 24 px gives 1.5 px strokes that render soft, so redrawn sizes are used where Carbon ships them, or the icon is redrawn.
- **Filled vs outline:** filled glyphs for learned, stateful controls (play/pause, favourite on), because filled icons are found faster when unfamiliar and a fill change is a strong state cue.
- **Labels:**
  - Visible text on every icon except the transport set, volume and the status-bar glyphs.
  - Only home, print and search are close to universal ([NN/g](https://www.nngroup.com/articles/icon-usability/)), and hidden navigation loses more than 20 % discoverability ([NN/g](https://www.nngroup.com/articles/hamburger-menus/)).
  - Every icon-only control has an accessible name naming its function; toggles expose their state separately ([APG](https://www.w3.org/WAI/ARIA/apg/practices/names-and-descriptions/)).
  - Long press is never the only way to learn what a control does.

## 7. Gestures

The gesture set is in SYS-039 and the design system §8.5. This research adds three things:

- every drag (seek, sliders, reorder) has a tap or button alternative, as WCAG 2.5.7 requires;
- activation happens on release, so a touch can be abandoned (WCAG 2.5.2);
- vertical movement is preferred to horizontal paging, which maps badly to buttons and to edge gestures (Garmin, watchOS).

## 8. Accessibility

### 8.1 Scope

The T1 follows WCAG 2.2 AA as WCAG2ICT maps it to non-web software ([W3C](https://www.w3.org/TR/wcag2ict-22/)) and EN 301 549 for software and hardware (SYS-061), whether or not a law requires it. EN 301 549 V4.1.1 (2026-09) is aligned with WCAG 2.2.

### 8.2 What the T1 provides

| Feature | Priority | Why | Notes |
|---|---|---|---|
| **Voice guidance**, driven from the buttons and by touch (explore by touch, double tap to activate) | must (SYS-062) | Eyes-free use is core; Linux has no touch screen reader (GNOME Orca's touch request has been open since 2021, [issue 207](https://gitlab.gnome.org/GNOME/orca/-/issues/207)), so the T1 is closed functionality under EN 301 549 | Speaks now playing, focus and changes; interruptible and repeatable; earcons with visual equivalents; built on AT-SPI so Orca and assistive devices work too |
| **Turning it on without sight** | must (SYS-062) | EN 301 549 5.2 | A documented button sequence with audio confirmation, at first run and at any time |
| **Text size with reflow** | must (SYS-063) | WCAG 1.4.4 and 1.4.10; Android's 200 % nonlinear curve; watchOS's ceiling around 140–150 % | Five steps to about 200 % for reading text and less for titles; layouts stack and wrap instead of cutting |
| **Large-text mode** | must (SYS-063) | EN 301 549 V4.1.1 5.1.4: x-height ≥ 16 CSS px at 400 mm, ≥ 2.38 mm, about a 30 px font | One item per line, about 10–12 characters per line: the layout changes, it does not scale |
| **AT-SPI semantics** for every element (role, name, state, value, actions) | must (SYS-061) | EN 301 549 11.5.2; the foundation of voice guidance | Also a documented accessibility API for third-party capabilities |
| **Increase contrast** (text ≥ 7:1), bold text, reduce motion, reduce transparency and effects | must / should (SYS-064) | WCAG 1.4.6 rationale; design system §12.4 | — |
| **A light, high-contrast appearance** | should (SYS-064) | Dark-on-light is more legible for small type and for older readers ([Piepenbrock et al. 2013](https://doi.org/10.1080/00140139.2013.790485)); in direct sun no colour pair survives on its own | An accessibility appearance, not a second brand theme (SYS-030) |
| **Colour filters** for colour vision deficiencies, grayscale | should (SYS-064) | Parity with platforms people know | Colour is never the only cue anyway (design system §13) |
| **Mono audio and left/right balance**, bit-transparent when off | must (SYS-065) | EN 301 549 4.2.5; the most direct hearing feature of an audio player | Balance is already an audio block (audio profiles spec §2) |
| **Hearing aids** (LE Audio hearing access, ASHA) | should (SYS-065) | — | Where the radio supports LE Audio; BlueZ has no hearing access module yet, so it is carried until upstream (OS-023) |
| **Configurable timings**: long press, hold, key repeat, double-press rejection | must (SYS-066) | WCAG 2.2.1, EN 301 549 5.7–5.8; iOS Touch Accommodations as the reference | Including the 500 ms long press of T1-061 |
| **Haptic confirmation** of actions | should | BRD-058 | With voice guidance and the buttons |
| **External keyboard and switch access**, driving the same focus as the buttons | should | EN 301 549 8.1.2 | — |
| **Accessible documentation** | must (SYS-061) | EN 301 549 | Plain-text manual, a description of the features and how to turn them on |

### 8.3 Sunlight

In direct sun, reflected light swamps the panel's own contrast. With assumed values (500–1000 nit, 2–4 % reflectance, about 100,000 lux), even white on black falls to about 1.4–2.6:1 (**UNVERIFIED**: the T1 panel is not measured). Only four things help:

- panel luminance;
- the anti-reflection treatment;
- larger and bolder type;
- the light appearance.

The faceplate should measure and publish its luminance and reflectance (a point for `faceplate-touch`).

### 8.4 Older users and motor access

- **Targets.** The 48–64 px targets (7.3–9.8 mm) are above WCAG's enhanced target at 40 cm and match the thumb study's 9.2–9.6 mm (Parhi et al. 2006).
- **A two-column large-target layout** for older users, with targets of about 15–20 mm, is a mode (should), not the default: a 49 mm-wide screen cannot carry it everywhere.

## 9. Open points

- Renders of every script at the new sizes on the T1's renderer (Slint, Skia), on the virtual T1 and on the panel.
- Panel luminance and reflectance, for the sunlight case.
- The voice guidance's own design (speech engine, earcons, button grammar): a specification of its own.
- The layouts at each text-size step, drawn in the design system.

## 10. Sources

- ANSI/HFS 100-1988, as quoted in the FAA Human Factors Design Standard, chapter 8: <https://hf.tc.faa.gov/hfds/>
- ETSI EN 301 549 V3.2.1 (2021-03) and V4.1.1 (2026-09): <https://www.etsi.org/deliver/etsi_en/301500_301599/301549/>
- W3C, WCAG 2.2: <https://www.w3.org/TR/WCAG22/>; WCAG2ICT: <https://www.w3.org/TR/wcag2ict-22/>
- Legge, G. and Bigelow, C. (2011). "Does print size matter for reading?". *Journal of Vision* 11(5):8: <https://pmc.ncbi.nlm.nih.gov/articles/PMC3428264/>
- Piepenbrock, C., Mayr, S., Mund, I. and Buchner, A. (2013). "Positive display polarity is advantageous for both younger and older adults". *Ergonomics*: <https://doi.org/10.1080/00140139.2013.790485>
- Parhi, P., Karlson, A. and Bederson, B. (2006). "Target size study for one-handed thumb use on small touchscreen devices". MobileHCI 2006.
- NHTSA, Visual-Manual Driver Distraction Guidelines, 78 FR 24818 (2013): <https://www.govinfo.gov/content/pkg/FR-2013-04-26/pdf/2013-09883.pdf>
- Apple Human Interface Guidelines (typography, accessibility, watchOS), and the iPhone Human Interface Guidelines of 2009.
- Android and Wear OS design documentation (typography, buttons, lists); Garmin Connect IQ user experience guidelines.
- Rockbox source, `apps/gui/list.c` and the manual's display options: <https://github.com/Rockbox/rockbox>
- GNOME Orca issue 207: <https://gitlab.gnome.org/GNOME/orca/-/issues/207>
- IBM Carbon icons: <https://carbondesignsystem.com/elements/icons/usage/>; IBM UI icon design: <https://www.ibm.com/design/language/iconography/ui-icons/design/>
- Nielsen Norman Group: [icon usability](https://www.nngroup.com/articles/icon-usability/), [hamburger menus](https://www.nngroup.com/articles/hamburger-menus/)
- W3C: [text size in translation](https://www.w3.org/International/articles/article-text-size), [Chinese](https://www.w3.org/TR/clreq/), [Indic](https://www.w3.org/TR/ilreq/) and [Arabic](https://www.w3.org/TR/alreq/) layout requirements, [pause, stop, hide](https://www.w3.org/WAI/WCAG22/Understanding/pause-stop-hide.html), [names and descriptions](https://www.w3.org/WAI/ARIA/apg/practices/names-and-descriptions/)
- IBM Plex font files: <https://github.com/IBM/plex>
