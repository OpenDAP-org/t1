# Faceplate interface

How a faceplate attaches to the core. Every faceplate (touch, cassette, classic, …) meets this interface; its own mechanics live in its repository, and the case provides the mount.

Faceplates are not meant to be swapped regularly, so this interface is kept to the minimum below. Requirements: CASE-008 to T1-019. Decision: [ADR 0001](../adr/0001-faceplate-interface-lives-in-t1.md).

## Flat-cable connector, interface v0

Defined by function ([RFC 0001](https://github.com/opendap-project/rfcs/blob/main/text/0001-faceplate-interface-v0-pinout.md)). The board maps its SoC pins onto these groups; no SoC pin is part of the interface. Connector: Hirose FH35C-51S, 0.3 mm pitch, 51 contacts: 46 used, 5 reserved (board ADR 0006, "about 10 %" spare). Contact positions are fixed in pinout v1.

| Group | Lines | Contacts | Notes |
|---|---|---|---|
| Display, high speed | MIPI DSI clock ±, data lanes 0–3 ± (or LVDS on the same lines) | 10 | A faceplate uses the lanes its panel needs; a single-lane panel leaves lanes 1–3 unconnected |
| I2C | SCL, SDA | 2 | Touch, ambient light sensor, HID controls (ADR 0002), panel |
| SPI/DBI | SCLK, MOSI, MISO, CS, D/C | 5 | Panels and displays driven over SPI or MIPI DBI |
| Touch | INT, RST | 2 | |
| Panel | RST, TE | 2 | |
| Interrupts | HID_INT, ALS_INT | 2 | Open-drain on the faceplate side; wake sources |
| Status LED | LED_R, LED_G, LED_B | 3 | Anodes, driven by constant-current sources on the board (≤ 20 mA each); cathodes to faceplate ground (FPT-016, FPT-025) |
| Microphone | PDM_CLK, PDM_DATA | 2 | |
| Detect and spare GPIO | FP_DET#, GPIO ×2 | 3 | Slow lines |
| Power | FP_3V3 ×2, FP_1V8, FP_VCI, MIC_1V8, BL_LEDA, BL_LEDK | 7 | Backlight driver on the board; the LED string returns through BL_LEDK |
| Ground | Between DSI pairs, group edges, returns | 8 | |
| Reserved | — | 5 | For later revisions |

Logic levels: 3.3 V for the low-speed groups, 1.8 V for PDM; a faceplate whose parts need other levels translates on its side.

## Control commands

Physical controls on a faceplate (buttons, switches, dials) reach Blackbird OS as standardized binary commands, not raw signals (T1-023, T1-024), through **HID over I2C** ([ADR 0002](../adr/0002-faceplate-controls-use-hid-over-i2c.md)). The faceplate's controller presents one HID device; Linux's mainline `i2c-hid` driver turns its reports into standard input events.

### Core set (Draft)

USB-IF HID Usage Tables. The Linux event is what mainline `drivers/hid/hid-input.c` produces for each usage (checked 2026-10-21). A faceplate implements the subset that matches its controls.

| Function | Usage page | Usage ID | Usage name | Linux event |
|---|---|---|---|---|
| Play / pause | Consumer (0x0C) | 0xCD | Play/Pause | `KEY_PLAYPAUSE` |
| Play | Consumer | 0xB0 | Play | `KEY_PLAY` |
| Pause | Consumer | 0xB1 | Pause | `KEY_PAUSE` |
| Stop | Consumer | 0xB7 | Stop | `KEY_STOPCD` |
| Next track | Consumer | 0xB5 | Scan Next Track | `KEY_NEXTSONG` |
| Previous track | Consumer | 0xB6 | Scan Previous Track | `KEY_PREVIOUSSONG` |
| Fast forward | Consumer | 0xB3 | Fast Forward | `KEY_FASTFORWARD` |
| Rewind | Consumer | 0xB4 | Rewind | `KEY_REWIND` |
| Record | Consumer | 0xB2 | Record | `KEY_RECORD` |
| Shuffle | Consumer | 0xB9 | Random Play | `KEY_SHUFFLE` |
| Repeat | Consumer | 0xBC | Repeat | `KEY_MEDIA_REPEAT` |
| Volume up | Consumer | 0xE9 | Volume Increment | `KEY_VOLUMEUP` |
| Volume down | Consumer | 0xEA | Volume Decrement | `KEY_VOLUMEDOWN` |
| Mute | Consumer | 0xE2 | Mute | `KEY_MUTE` |
| Volume (absolute knob) | Consumer | 0xE0 | Volume | `ABS_VOLUME` |
| Menu | Consumer | 0x40 | Menu | `KEY_MENU` |
| Select | Consumer | 0x41 | Menu Pick | `KEY_SELECT` |
| Up / down / left / right | Consumer | 0x42–0x45 | Menu Up / Down / Left / Right | `KEY_UP` / `KEY_DOWN` / `KEY_LEFT` / `KEY_RIGHT` |
| Escape | Consumer | 0x46 | Menu Escape | `KEY_ESC` |
| Home | Consumer | 0x223 | AC Home | `KEY_HOMEPAGE` |
| Back | Consumer | 0x224 | AC Back | `KEY_BACK` |
| Forward | Consumer | 0x225 | AC Forward | `KEY_FORWARD` |
| Search | Consumer | 0x221 | AC Search | `KEY_SEARCH` |
| Favorites | Consumer | 0x22A | AC Bookmarks | `KEY_BOOKMARKS` |
| Brightness up / down | Consumer | 0x6F / 0x70 | Display Brightness Increment / Decrement | `KEY_BRIGHTNESSUP` / `KEY_BRIGHTNESSDOWN` |
| Scroll wheel (relative) | Generic Desktop (0x01) | 0x38 | Wheel | `REL_WHEEL` |
| Rotary dial (relative) | Generic Desktop | 0x37 | Dial | `REL_DIAL` |
| Horizontal scroll | Consumer | 0x238 | AC Pan | `REL_HWHEEL` |

### Extensions

New commands go in an **OpenDAP vendor-defined usage page** (0xFF00–0xFFFF; the exact page is not assigned yet), read by Blackbird OS through `hidraw`. A command moves to the core set only through an RFC. Candidates that are not decided: hold / lock switch, output select, gain, DAC filter, A–B repeat, like.

### Identification

The HID descriptor's vendor ID, product ID and version identify the faceplate and its controls (T1-020).

### Not defined yet

I2C address and HID descriptor register, interrupt line, report descriptor layout, and power behavior during suspend.

## Dimensions

Not defined yet.

## Anchoring

Screws. Count, size and positions: not defined yet.
