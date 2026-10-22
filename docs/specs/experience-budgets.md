# Experience budgets

- **Status:** Draft. The structure is confirmed (T1-048); the values are proposals to confirm and to calibrate on the first prototype. Where the hardware allows better, the target moves up (T1-045).
- **Date:** 2026-10-22
- **Rule:** each metric has an **ideal** value (the target), a **warning** value (investigate before release) and a **hard limit** (the release does not ship). Every metric is measured by an automated test on the virtual T1 and, where it depends on hardware, on the device (TK-016, TK-017).
- **Sources:** the [UX research](../research/ux/), the T1 requirements and the board power tree. Values without a source are engineering proposals and say so.

## Start-up and responsiveness

| Metric | Ideal | Warning | Hard limit | Basis |
|---|---|---|---|---|
| Cold boot to first sound (resume last track) | ≤ 10 s | > 15 s | > 25 s | Proposal; time to first track is a moment of truth (UX research) |
| Screen on (from off) to interactive | ≤ 150 ms | > 300 ms | > 600 ms | Proposal |
| Button press to action (play, pause, volume) | ≤ 50 ms | > 100 ms | > 200 ms | Proposal; about 100 ms is where delay becomes noticeable |
| Touch to visual response | ≤ 50 ms | > 100 ms | > 150 ms | Proposal |
| UI frame rate while scrolling | 60 fps | < 50 fps | < 30 fps | Display refresh |
| Track change gap (same format) | 0 samples | — | > 0 samples | Gapless is exact or it fails (UX research) |

## Library scale

| Metric | Ideal | Warning | Hard limit | Basis |
|---|---|---|---|---|
| Library size handled without degradation | no fixed limit, tested at 1,000,000 tracks | tested below 500,000 | any fixed cap, or tested below 200,000 | T1-046; competitors cap at 50,000–200,000 tracks (UX research) |
| Open a list or view (any library size) | ≤ 200 ms | > 400 ms | > 1 s | Proposal; SYS-033 |
| Search results (500,000 tracks) | ≤ 100 ms | > 300 ms | > 1 s | Proposal |
| Incremental rescan after adding 1,000 files | ≤ 10 s, in the background | > 30 s | blocks playback or the UI | Proposal; LB requirements |
| Start-up cost of the library (any size) | none (no rebuild at boot) | > 1 s | > 5 s | A flagship rebuilt its database for about 20 s at every power-on (UX research) |

## Audio output

| Metric | Ideal | Warning | Hard limit | Basis |
|---|---|---|---|---|
| Output impedance | ≤ 0.5 Ω | > 1 Ω | > 2 Ω | Proposal; one-eighth of a 16 Ω IEM is 2 Ω |
| Volume step | ≤ 0.5 dB | > 1 dB | > 2 dB | DAC volume resolution; UX research |
| Channel balance at low volume | ≤ 0.1 dB | > 0.3 dB | > 1 dB | Proposal; volume in the DAC |
| Audible pops and clicks at any transition | none | — | any | UX research |
| Bit-perfect on every declared path | 100 % | — | any failure | OS-020 |

Noise floor into sensitive IEMs, distortion and output power are set from the first prototype's measurements against ADR 0012's design figures.

## Power

| Metric | Ideal | Warning | Hard limit | Basis |
|---|---|---|---|---|
| Playback autonomy (core session) | > 12 h | < 8 h | < 6 h | T1-039 |
| Standby drain | ≤ 1 % per day | > 3 % per day | > 5 % per day | Proposal |

## Endurance

| Metric | Ideal | Warning | Hard limit | Basis |
|---|---|---|---|---|
| Memory, file descriptors and response times over 72 h of use | flat | any upward trend | growth that would exhaust a resource in 30 days | T1-047, SYS-034, TK-017 |
