# ADR 0002: Faceplate controls use HID over I2C

- **Status:** Accepted
- **Date:** 2026-10-21

## Context

Faceplate controls (buttons, switches, dials) must reach Blackbird OS as standardized binary commands, not raw signals, with a well-standardized core set that is not minimal and room for new commands (T1-023, T1-024). Software must prefer the most standard Linux option (T1-011), and faceplate identification should rely on the interface pinout rather than an EEPROM (T1-020).

## Decision

- Each faceplate exposes its controls as an **HID-over-I2C** device, read by the mainline `i2c-hid` driver. No OpenDAP kernel driver is needed.
- Commands are **usages from the USB-IF HID Usage Tables**. The core set is defined in the [faceplate interface](../specs/faceplate-interface.md#control-commands).
- New commands go in an **OpenDAP vendor-defined usage page**, read by Blackbird OS services through `hidraw`.
- The faceplate's HID descriptor (vendor ID, product ID, version) identifies the faceplate and its controls.
