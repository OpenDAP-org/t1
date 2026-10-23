# Buttons, gestures and touch on a small screen

- **Status:** Research, 2026-10-23. The decisions it led to are recorded in T1-060 to T1-063 (buttons), SYS-039 (gestures), DS-014 and the design system §8.3 and §8.5 (targets and gestures).
- **Scope:** how the four body buttons and the touch screen of the T1 should behave, measured against the conventions people already know and against what a 3.5" panel allows.

## 1. The panel is the original iPhone's

The touch faceplate's panel is 3.5", 320 × 480, about 166 ppi, active area 48.96 × 73.44 mm ([faceplate-touch ADR 0001](https://github.com/opendap-project/faceplate-touch/blob/main/docs/adr/0001-focus-lcds-e35gh-display.md)). The original iPhone was 3.5", 320 × 480, 163 ppi, and Apple's 44 × 44 point minimum comes from that screen, where one point was one pixel: about 6.9 mm ([Big Medium](https://bigmedium.com/ideas/iphone-tap-target-44.html)). On the T1, 44 px is 6.7 mm. A modern phone's 48 dp target (Material) is about 9 mm ([Android accessibility help](https://support.google.com/accessibility/android/answer/7101858)), which is 59 px here: the T1 cannot take a phone layout and shrink it, because a phone's targets are larger in millimetres than the T1's whole budget allows for more than a few per row.

| Source | Target | On the T1 (px) |
|---|---|---|
| Apple HIG, from the original iPhone | 44 × 44 pt ≈ 6.9 mm | 45 |
| WCAG 2.2, 2.5.8 (AA) / 2.5.5 (AAA) | 24 / 44 CSS px, or 24 px spacing ([2.5.8](https://www.disabilityworld.org/toolkit/standards/wcag/2-5-8-target-size-minimum/), [2.5.5](https://www.disabilityworld.org/toolkit/standards/wcag/2-5-5-target-size-enhanced/)) | not physical units; 44 is the AAA reference |
| Microsoft touch guidance | 9 mm ideal, 7 mm minimum ([guidelines for targeting](https://learn.microsoft.com/en-my/windows/apps/develop/input/guidelines-for-targeting)) | 59 / 46 |
| Material | 48 dp ≈ 9 mm, 8 dp apart | 59, with 10 px between |
| Parhi, Karlson and Bederson (MobileHCI 2006), one-handed thumb | 9.2 mm discrete, 9.6 mm serial ([paper](https://www.microsoft.com/en-us/research/wp-content/uploads/2008/01/p1399-karlson.pdf)) | 60 / 63 |

**Reading.** 9 mm is what the thumb needs without errors; 7 mm is the floor where space forces it. The T1 is held in one hand more often than a phone is (it is a player, usually used while doing something else), so the thumb figures apply. The screen fits five 52 px targets across its 272 px content width (24 px margins) with 3 px between them, or four 60 px targets with 11 px: a row of five transport controls is the limit, which is also what Hick's law asks for.

## 2. Size, spacing and placement

- **Minimum target: 48 px (7.4 mm)**, above Microsoft's 7 mm floor and on the 4 px grid; the hit area may extend past the drawn control. 44 px stays only for the documented on-screen keyboard exception, and anything denser than 48 px (the status bar's icons) is not a target of its own.
- **Frequent actions: 52 px (7.9 mm). Primary action: 64 px (9.8 mm)**, above the thumb study's 9.2 mm, because it is the one tapped most and without looking.
- **Spacing: at least 8 px (1.2 mm) between hit areas**, so targets never overlap; **16 px (2.5 mm)** between destructive or opposite actions (delete next to keep, previous next to next is fine). Material's 8 dp would be 10 px; 8 px is the 4 px grid's nearest step with the hit areas already at 48 px.
- **Placement.** Hoober observed 1,333 people: 49 % hold a phone in one hand, 75 % of interactions are by thumb, and on a phone only about a third of the screen is effortless ([UXmatters](https://uxmatters.com/mt/archives/2013/02/how-do-users-really-hold-mobile-devices.php)). The T1's screen is 73 mm tall, so the thumb reaches all of it, but the lower half is still easier: primary and frequent actions go there (the transport row already does), and the top keeps what is used least (status, minimise, like, queue), each also reachable by a gesture (§4).

## 3. Laws of UX applied

From [Laws of UX](https://lawsofux.com/):

| Law | On the T1 |
|---|---|
| **Fitts's law**: acquisition time grows with distance and shrinks with size | The primary action is the largest target and sits where the thumb rests; the Action button is its physical twin. Edge swipes are fast because the edge needs no aiming. |
| **Hick's law**: decision time grows with the number of choices | At most five controls in a row and one primary action per screen; settings in short grouped lists, technical detail on demand. |
| **Miller's law / chunking** | Track metadata grouped (title, artist, format line); lists in sections. |
| **Jakob's law**: people expect what they already know | Edge-swipe back, swipe-down from the top for notifications, swipe-up for more, from Android and streaming apps; headset-style buttons (§5). |
| **Doherty threshold**: under 400 ms neither side waits | The T1's own budget is stricter (≤ 50 ms ideal, T1-057): nothing may wait to disambiguate a press (§5). |
| **Von Restorff effect** | One orange primary action on screen, and one orange button on the body. |
| **Law of proximity / common region** | Button groups as one segmented capsule (DS-013); related controls share a surface. |
| **Tesler's law**: complexity moves, it does not disappear | Profiles and defaults carry the complexity (audio profiles, T1-064), so the main screens stay simple. |

## 4. Touch gestures

The reference is Android's gesture navigation (back from the side edges, home from the bottom, recents by swiping up and holding, assistant from the corners) and its gesture-exclusion API for controls that must be dragged near an edge ([Android Developers blog](https://android-developers.googleblog.com/2019/08/gesture-navigation-backstory.html), [codelab](https://codelabs.developers.google.com/codelabs/gesture-navigation)). The T1 has no phone home screen, so the bottom swipe is free for something more useful.

| Gesture | Action | Why |
|---|---|---|
| Swipe in from the leading edge (left; right in RTL) | Back | Android and iOS convention |
| Swipe in from the trailing edge (right; left in RTL) | The screen's companion screen, sliding in from that side: from Now Playing, the queue; from an album, Now Playing | A forward-in-history gesture has nothing to go to most of the time. Android gives both edges to back for one-handed reach, but a 49 mm wide screen does not need it, so the second edge is free; the panel enters from the side where its button (the queue icon, top right) already is |
| Swipe up from the bottom | "More": details for the current screen; on Now Playing, the signal path and technical details, lyrics, credits | As streaming apps scroll the player; no home to go to |
| Swipe down from the status bar | Notifications and quick settings | Phone convention |
| Swipe down on Now Playing | Minimise the player | Same as its chevron |

- **Edge zones** are 20 px (3 mm) wide. Controls that are dragged (the seek bar, sliders, horizontal lists) exclude their area from the side zones, as Android's gesture exclusion does; exclusions stay small so the gesture keeps working on most of the edge.
- **Alternatives.** WCAG 2.2 requires a single-pointer, non-path alternative for path gestures (2.5.1) and dragging (2.5.7) ([2.5.1](https://www.w3.org/WAI/WCAG22/Understanding/pointer-gestures), [2.5.7](https://www.w3.org/WAI/WCAG22/Understanding/dragging.html)): every gesture above has a visible control (back arrow, chevron, queue icon, a "more" control, a tap on the status bar).
- **Non-touch faceplates** send HID navigation (design system §13), so gestures are never the only way.

## 5. Physical buttons

### 5.1 Press patterns people know

| Pattern | Standard timing | Common use | Note |
|---|---|---|---|
| Single press | released before 500 ms | The basic action | |
| Double press | second press within 300 ms | Next track on Android headsets ([headset spec](https://source.android.com/accessories/headset-spec.html)) and AirPods ([SoundGuys](https://www.soundguys.com/how-to-skip-songs-with-airpods-110891)) | **Delays the single press by up to 300 ms**, while the device waits for a second press |
| Triple press | same window | Previous track on AirPods | Same delay, longer |
| Long press | held 500 ms or more | A second action; the assistant on headsets | No cost to the single press |
| Hold | about 2 s | Deliberate or system actions | |
| Auto-repeat | 250 ms delay, then every 33 ms (Linux defaults, [input core](https://lab.nexedi.com/kirr/linux/-/commit/027c71bbae3a6eeff00c11d1b708593a5c790314)) | Volume, scrolling | Conflicts with giving the long press its own action |
| Chord (two buttons together) | simultaneous | Screenshot, recovery, download mode on Android | Reserved for the system |

Timings: Android's `ViewConfiguration` long-press timeout is 500 ms and its double-tap timeout 300 ms ([reference](https://developer.android.com/reference/android/view/ViewConfiguration), [source](https://code.yawk.at/android/android-9.0.0_r35/android/view/ViewConfiguration.java)).

### 5.2 Volume keys for tracks

Long-pressing volume to change track is an established convention on Android, through music apps such as Poweramp, and only with the screen off there ([Poweramp forum](https://forum.powerampapp.com/topic/2002-long-press-volume-keys-to-skip-tracks-when-screen-off/?page=3), [Appuals](https://appuals.com/use-volume-keys-skip-tracks-android-phones/)). Doing it always, on and off, makes one press mean one thing everywhere, at the cost of the volume auto-repeat; volume still changes step by step, and continuously on screen.

### 5.3 Resulting map (decided 2026-10-23)

| Button | Single press | Long press (≥ 500 ms) | Hold (≥ 2 s) |
|---|---|---|---|
| Action | The current screen's primary action; screen off: play / pause | A direct action the user chooses | — |
| Volume + / − | One volume step | Next / previous track | — |
| Main | Screen off / on | Power menu | Forced power-off (PMIC) |
| Chords with Main | System only: screenshot, recovery, MaskROM | | |

No double or triple press on Action and volume, so they answer at once (T1-057). The previous Action + Volume chord for track change is no longer needed.
