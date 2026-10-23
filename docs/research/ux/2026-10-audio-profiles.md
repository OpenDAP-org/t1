# Audio profiles: benchmark

- **Status:** Research, 2026-10-23. The resulting decisions are in the [audio profiles spec](../../specs/audio-profiles.md) (T1-064).
- **Question:** what audio profiles should the T1 offer, from neutral reference listening for sound engineers to personal audiophile sessions with their own colour and texture?

## 1. What the market offers

| Product | What it offers | Source |
|---|---|---|
| **HiBy** (HiBy OS DAPs) | MSEB, "MageSound Eight Ball": tone macros built on a parametric EQ core: overall temperature (warm/dark to cool/bright), bass texture (fast to thumpy), note thickness (crisp to thick), female overtones (3 kHz), sibilance (5.8 and 9.2 kHz), air (10 kHz upward) | [HiBy guide, MSEB](https://guide.hiby.com/en/docs/guides/audioprocess/mseb) |
| **FiiO** (M17, M15S) | 10-band parametric EQ (frequency, gain, Q) set from the FiiO Music app | [FiiO M17 specifications](https://fiio.com/m17_parameters), [M15S](https://fiio.com/m15s_parameters) |
| **Astell&Kern** | Parametric EQ up to 20 bands with 0.01 dB steps and Q; crossfeed; gain levels (Low/Mid/High/Super); up to seven DAC filter choices on some models | [Moon Audio, A&K guide](https://www.moon-audio.com/blogs/expert-advice/astell-kern-user-interface-guide) |
| **Sony Walkman** (NW-WM1ZM2) | "Direct Source" bypasses all processing; DSEE Ultimate (upscaling); DC Phase Linearizer (six low-frequency phase characters, "analog amplifier" behaviour); Vinyl Processor; Dynamic Normalizer (level between tracks). Processing is disabled while Direct Source is on | [Sony help guide](https://helpguide.sony.net/dmp/nwwm1m2/v1/en/contents/TP1000464232.html) |
| **Roon** (MUSE engine) | Fixed headroom management and sample-rate conversion; parametric and procedural EQ, crossfeed (Bauer), convolution and presets, which can be repeated and reordered | [Roon DSP engine](https://help.roonlabs.com/portal/en/kb/roon-labs-llc/audio/dsp-engine), [crossfeed](https://help.roonlabs.com/portal/en/kb/articles/dsp-engine-crossfeed) |
| **Sonarworks SoundID Reference** (studios) | Headphone calibration to a flat "studio monitor in a treated room" target, 500+ headphone profiles, zero-latency / mixed / linear-phase filters, custom target adjustment, translation check against other devices | [Sonarworks headphone calibration FAQ](https://www.sonarworks.com/blog/learn/headphone-calibration-faq), [product](https://www.hhb.co.uk/prod/sonarworks/soundid-reference-for-sparks-headphones/) |
| **Apple** (Headphone Accommodations) | Tuning from the listener's own audiogram, from an in-app hearing test or an entered audiogram | [Hearing Review](https://hearingreview.com/inside-hearing/research/evaluating-apple-airpods-pro-with-headphone-accommodations-as-hearing-devices), [Mimi](https://mimi.io/personalize-apple-headphones-using-the-mimi-hearing-test-app) |

## 2. The open building blocks

| Block | What it is | Licence | Source |
|---|---|---|---|
| **AutoEq** | Parametric EQ generated from measurements of thousands of headphones (oratory1990, crinacle, Rtings and others), corrected to a target such as Harman | MIT | [AutoEq](https://github.com/rezzeted/AutoEq), [PyPI](https://pypi.org/project/autoeq/2.2.0/) |
| **bs2b** | Bauer stereophonic-to-binaural crossfeed, with default, Chu Moy and Jan Meier presets and custom levels; a Rust crate exists | MIT | [bs2b crate](https://docs.rs/crate/bs2b/0.2.0), [Fedora](https://packages.fedoraproject.org/pkgs/libbs2b/libbs2b) |
| **ReplayGain 2.0** | Loudness normalisation from EBU R128 / ITU BS.1770, −18 LUFS reference for music on portable devices, track and album gain, true peak at 4× oversampling; tags only, the audio is not changed | (standard; loudgain is a reference tool) | [loudgain](https://packages.fedoraproject.org/pkgs/loudgain/loudgain) |
| **Harman targets** | Over-ear and in-ear target curves from Olive and Welti's listening tests: 64 % of listeners preferred the target, 21 % the target with 2–3 dB less bass, 15 % with 4–6 dB more | (research) | [Acoustics Today](https://acousticstoday.org/issues/2022AT/Spring2022/files/basic-html/page66.html), [AES](https://aes2.org/publications/elibrary-page/?id=19436) |

## 3. Hardware on the T1

- The **CS43198** has five selectable digital interpolation filter responses, among them fast and slow roll-off and a low-group-delay pseudo-linear-phase option ([Cirrus Logic](https://cirrus.com/products/cs43198)). The exact list and their names are **UNVERIFIED** until checked against datasheet DS1156F2. A filter choice changes how the DAC reconstructs the signal, not the samples it receives, so it keeps the data path bit-perfect.
- The DAC's own volume is hardware too and keeps the samples untouched upstream of the DAC. The output stage has a fixed gain of about 1.6 (board ADR 0012): no switchable gain levels, unlike Astell&Kern's Low/Mid/High/Super.

## 4. Engineering points

- **Headroom.** Boosting EQ clips at full scale unless the signal is attenuated first; a common rule is the largest boost plus about 2 dB, and inter-sample peaks can still clip after processing ([Roon community, headroom](https://community.roonlabs.com/t/post-dsp-volume-limiter-auto-headroom-management/307812), [miniDSP](https://www.minidsp.com/community/threads/clipping-peq-gain-level.1758/post-3167)).
- **Word length.** Processing runs at a higher precision than the output; going back to 24 or 32 bits needs dither.
- **What is not processing.** Tags (ReplayGain) and meters (loudness, true peak) do not change the samples; only applying a gain does.

## 5. Reading

1. Every serious product separates **a direct, untouched path** (Sony's Direct Source, Roon with no DSP) from **processing**, and turns processing off while direct is on.
2. **Engineers** want a known, flat reference: an untouched path, or a calibrated one that corrects the headphone to a stated target (Sonarworks), plus honest meters and quick checks (mono, level-matched A/B).
3. **Audiophiles** want to shape the sound in words, not only in bands: HiBy's eight macros (temperature, bass texture, note thickness, sibilance, air) are the clearest example of "colour and texture" on top of a parametric EQ.
4. The **open blocks** cover most of it with permissive licences: AutoEq for headphone correction, bs2b for crossfeed, ReplayGain 2.0 for level.
5. **Per-headphone** profiles matter (correction depends on the model) and connect to hearing safety (SYS-012 uses per-headphone profiles).
6. Things that **invent** content (upscaling such as DSEE, vinyl emulation) are colour, not correction: if offered, they belong to personal profiles and are labelled as such, never to the reference.
