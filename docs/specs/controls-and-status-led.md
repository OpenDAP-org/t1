# Controls and status LED

- **Status:** Draft. The structure follows T1-043 and T1-044; timings and colours are proposals to confirm on the first prototypes.
- **Date:** 2026-10-22
- **Applies to:** every T1, whatever the faceplate. Faceplates that add controls report them through the [faceplate interface](faceplate-interface.md); this document covers the four body buttons and the status LED.

## 1. Buttons

The T1 has four body buttons (T1-014, T1-015): **Main** (right edge) and **Action**, **Volume +** and **Volume −** (left edge). The combinations follow Android's conventions, so they are familiar, and they never depend on the faceplate. Their physical form is in §1.4.

### 1.1 Press types

| Press | Duration | Reference |
|---|---|---|
| Short | released before 500 ms | Android long-press timeout (`ViewConfiguration`), proposed value |
| Long | held 500 ms or more | as above |
| Hold | held 2 s or more | proposed, to separate deliberate system actions from long presses |
| Forced power-off | held until the PMIC cuts power | RK809 long PWRON press, handled in hardware without the OS ([RK809-5 sheet](https://github.com/opendap-project/parts-lib/blob/main/Rockchip/RK809-5.md)); the time is a PMIC register setting, value to be fixed in bring-up (**UNVERIFIED**) |

### 1.2 Device off

| Combination | Result | Reference |
|---|---|---|
| **Main**, long | Power on, normal boot | Android power-on |
| **Main + Volume −**, held at power-on | **MaskROM** (USB download over the main USB-C) | Android: Power + Volume Down enters the bootloader/download mode. T1: [board ADR 0009](https://github.com/opendap-project/board/blob/main/docs/adr/0009-bom-v1-decisions.md), BRD-035, implemented in hardware. MaskROM through the buttons needs a full power-off first (a warm restart does not reopen the entry window) and an empty or unbootable microSD slot (board BOM v1.6 line 97; software can still reboot into the loader) |
| **Main + Volume +**, held at power-on | **Recovery** (OpenDAP OS recovery; reflash from SD/TF, BRD-043) | Android: Power + Volume Up enters recovery on many devices. BRD-017 (recovery key) |

### 1.3 Device on

| Combination | Result | Reference |
|---|---|---|
| **Main**, short | Screen off / on (playback continues) | Android |
| **Main**, long | Power menu: power off, restart, restart to recovery | Android power menu |
| **Main + Volume −**, short | Screenshot | Android |
| **Main + Volume −**, hold | Power menu (same as Main long), also reachable when the screen is off | maintainers' choice (2026-10-22) |
| **Main**, held until forced power-off | Hardware power-off, works even if the OS is unresponsive | Android forced restart; implemented by the PMIC |
| **Volume + / −**, short | Volume step | Android |
| **Volume + / −**, long | Repeat volume steps | Android key repeat |
| **Action**, short | Assignable; default play / pause | Developer-accessible (BRD-036) |
| **Action**, long | Assignable; default none | as above |
| **Action + Volume +** / **Action + Volume −** | Next / previous track | proposed, so track control works with the screen off |

**Rules:**

- System combinations (with **Main**) cannot be reassigned. **Action** and the Action combinations can.
- Safety-related functions never depend only on software: forced power-off (PMIC) and MaskROM entry (hardware circuit) work with any OS state.
- Volume limits and the hearing-safety dose (SYS-012) apply to button volume changes like any other.

### 1.4 Physical control design

The button design in the [concept art](../concept-art/) governs (T1-059): it is the reference for the form of the controls, and this section describes it. Other details in that concept art are stale (480 × 800 screen, 1.27 mm pins, earlier dimensions) and are not a source. The shared visual language is in the [design system spec](https://github.com/opendap-project/ui-components/blob/main/docs/specs/design-system.md); materials, finishes and manufacturability in the [case industrial-design spec](https://github.com/opendap-project/case/blob/main/docs/specs/industrial-design.md).

Seen from the front:

| Edge | Button | Form | Role |
|---|---|---|---|
| Left, top | **Action** | A round disc, subtly concave: circular outline, shallow comfortable concavity, a defined tactile perimeter with no sharp rim, a stable central contact area. Deeper travel and a clearer click (T1-022). The only control in the orange brand accent. | The main action button; default play / pause (§1.3) |
| Left, below Action | **Volume +** and **Volume −** | Two half-capsules: together they form one capsule split across the middle, each half with a rounded outer end and a flat inner edge at the split, operated as two separate buttons. Not a rocker. Short, predictable travel. | Volume steps |
| Right | **Main (power)** | A taller capsule with horizontal ribs for touch identification. Graphite, visually quieter than Action. | System button (§1.2, §1.3) |

All buttons are graphite except Action.

**Button groups.** The volume pair is the reference instance of the button-group pattern shared by physical and on-screen controls (ui-components DS-013): one capsule split into segments, rounded outer ends, flat inner edges at each split, every segment its own button, with a consistent gap or divider.

**Feedback.** Each button confirms its action through travel, click, shape and position, and through the screen when it is on. A physical action never depends only on a small on-screen animation: with the screen off, volume and playback changes are audible, and the action follows the press within the [experience budgets](experience-budgets.md) (T1-057).

**Distinction by touch.** Disc, split capsule and ribbed capsule differ in shape, position and texture, so each button is found without looking.

## 2. Status LED

The status LED is on the faceplate (FPT-003), RGB where possible (FPT-025), and developer-controllable (FPT-016). The colours follow **IEC 60073** (meanings of colours for indicators), and the on-screen semantic colours use the same families for the same meanings ([design system](https://github.com/opendap-project/ui-components/blob/main/docs/specs/design-system.md) §18.4): red for danger or fault, yellow/amber for an abnormal or warning condition, green for normal, blue for a condition that needs an action, white for neutral information. Patterns follow the same standard's use of flashing to draw attention.

### 2.1 Patterns

| Pattern | Definition (proposed) |
|---|---|
| Solid | On |
| Breathing | Slow fade in and out, 2 s period |
| Slow blink | 1 Hz, 50 % duty |
| Fast blink | 4 Hz, 50 % duty |
| Single pulse | One 100 ms flash |

### 2.2 Base set

Listed by priority: when several states apply, the highest one is shown.

| Priority | State | Colour | Pattern | IEC 60073 meaning |
|---|---|---|---|---|
| 1 | Hardware or thermal fault, unsafe condition | Red | Fast blink | Danger / fault |
| 2 | Battery critically low | Red | Slow blink | Danger, action required |
| 3 | Update or flashing in progress: do not power off | Blue | Breathing | Action required (do not interrupt) |
| 4 | Recovery mode | Blue | Solid | Mandatory action context |
| 5 | Battery low | Amber | Slow blink | Warning |
| 6 | Charging | Amber | Solid | Abnormal (not yet ready) |
| 7 | Charged, still on external power | Green | Solid | Normal |
| 8 | Bluetooth pairing | Blue | Fast blink | Action required (pair now) |
| 9 | Booting | White | Breathing | Neutral information |
| 10 | Developer control (FPT-016) | Any | Any | Defined by the developer |
| — | Normal operation | Off | — | The screen carries normal feedback; the LED stays dark to save power |

**Rules:**

- Priorities 1–4 cannot be overridden by developer control. Below them, a developer may take the LED through the Linux LED class (`/sys/class/leds`, triggers such as `heartbeat` or `timer`).
- With a single-colour LED (if RGB is not fitted), the base set keeps its meaning through patterns only: fast blink = fault, slow blink = low battery, breathing = busy/booting, solid = charging or charged.
- MaskROM cannot drive the LED, because no software runs on the SoC; the host tool reports MaskROM instead (`opendap device wait --mode maskrom`).
- The microphone privacy LED is separate, hardware-tied to the microphone power (BRD-027), and never part of this code.

## 3. Open points

- Press durations and LED pattern timings: confirm on the first prototype for feel and readability.
- The PMIC forced power-off time.
- Whether Action + Volume combinations conflict with any faceplate control.
