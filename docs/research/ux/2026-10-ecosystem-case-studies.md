# UX research: case studies for the OpenDAP ecosystem

- **Date:** 2026-10-23
- **Status:** Desk research. It complements the [ecosystem experience research](2026-10-ecosystem-experience.md) with reference cases: what went right, what went wrong, security incidents, quality and update failures, longevity and trust.
- **Method:** each case gives what happened, the root cause where known, the impact, the response and the lesson for OpenDAP. Every factual claim links its source; most are press and Wikipedia, with primary sources (advisories, regulator filings, CVE records) where they could be read. What could not be confirmed is **UNVERIFIED**; no number is added beyond what the sources state.

---

## 1. Library and sync correctness

### 1.1 iTunes 12.3.3 / iCloud Music Library deletions (2016) — error

- **What happened.** In May 2016 user James Pinkstone reported that his local music files (including original WAV recordings) had disappeared from his library and disk after enabling Apple Music / iCloud Music Library, with cloud versions in their place. An Apple support agent reportedly told him this was intended behaviour. ([MacSales/OWC blog](https://eshop.macsales.com/blog/36344-apple-and-the-curious-case-of-the-missing-music/), [New Statesman](https://www.newstatesman.com/science-tech/2016/05/apple-music-really-deleting-users-songs-without-their-consent))
- **Root cause.** Disputed. Macworld argued neither Apple Music nor iCloud Music Library deletes files, and that the cloud service was "an unfortunate scapegoat" for an iTunes 12.3.3 library-database bug; matched tracks are re-delivered as 256 kbps AAC, with DRM when no iTunes Match subscription exists, and the iOS delete dialog was unclear about whether a delete propagates to the cloud library. ([Macworld, 2016-05-06](https://www.macworld.com/article/3066789/streaming-media/apple-music-doesnt-delete-your-music-files.html), [Snopes](https://www.snopes.com/apple-music-deleting-files))
- **Impact.** The user recovered from his own backup ([MacSales](https://eshop.macsales.com/blog/36344-apple-and-the-curious-case-of-the-missing-music/)). The general failure modes: originals replaced by lossy or DRM copies, wrong matches, deletes propagating across devices.
- **Response.** Apple shipped an iTunes update with added safeguards in May 2016 (**UNVERIFIED**: version 12.4 and Apple's exact statement could not be fetched).
- **Lesson.** Sync never deletes or replaces an original file as a side effect. A "matched" or transcoded copy never substitutes for the source. Deletes that propagate are explicit, previewed, reversible (trash with retention), and the wording says which devices are affected. Support staff must not be the place where intended behaviour is defined: the spec is.

### 1.2 Google Play Music shutdown (2020–2021) — mixed

- **What happened.** Streaming stopped in December 2020; Google extended the deadline and deleted all Play Music data, including uploads and purchases, on 2021-02-24. ([9to5Google](https://9to5google.com/2021/02/04/google-play-music-delete/), [Digital Music News](https://www.digitalmusicnews.com/2021/02/05/google-play-music-library-deleted/))
- **Response (win).** A transfer tool to YouTube Music that carried library, uploads and taste profile, plus a download option (Google Takeout). ([The Star](https://www.thestar.co.uk/read-this/your-google-play-music-purchases-and-data-will-be-deleted-on-24-february-heres-what-to-do-3130518))
- **Impact.** Users who missed the deadline lost uploads and library data permanently ([Stuff](https://stuff.co.za/2021/02/25/google-play-music-data-is-gone/)).
- **Lesson.** Any online service in the ecosystem needs a published end-of-life procedure: notice period, export in open formats at any time (not only at shutdown), and a deadline extension policy. Better: the device and desktop hold the canonical library, so a service closing removes a convenience, never data.

### 1.3 Microsoft PlaysForSure / MSN Music DRM (2006–2008) — error

- **What happened.** PlaysForSure (2004) certified devices and stores for Windows Media DRM. Microsoft's own Zune (2006) could not play PlaysForSure music bought from MSN Music. In April 2008 Microsoft announced the MSN Music license servers would stop (2008-08-31); after outcry it extended support (reported to 2011). Yahoo! Music Unlimited and others also shut down. ([Wikipedia: PlaysForSure](https://en.wikipedia.org/wiki/PlaysForSure), [Wikipedia: MSN Music](https://en.wikipedia.org/wiki/MSN_Music), [BetaNews](https://betanews.com/2008/04/23/microsoft-to-end-support-for-msn-music-drm))
- **Root cause.** Playback depended on online re-authorisation; a new PC, a disk replacement or an OS upgrade needed a server that would cease to exist ([MSN Music](https://en.wikipedia.org/wiki/MSN_Music)).
- **Lesson.** Nothing a user has acquired may depend on a server to play. Third-party stores integrated with OpenDAP should deliver DRM-free files (or be clearly marked as streaming, not ownership). A "certified" logo is worthless if the certifier abandons it: OpenDAP compatibility claims must rest on open formats.

### 1.4 Amazon Kindle "1984" remote deletion (July 2009) — error, strong apology

- **What happened.** Around 2009-07-16 Amazon remotely deleted copies of Orwell's *1984* and *Animal Farm* from customers' Kindles, after learning the third-party seller lacked rights; refunds were automatic, without prior notice. ([TechCrunch](https://techcrunch.com/2009/07/23/amazons-bezos-apologizes-for-the-stupid-and-thoughtless-kindle-incident), [NPR](https://www.npr.org/sections/thetwo-way/2009/07/amazon_kindles_lawsuit_for_del.html))
- **Response.** On 2009-07-23 Jeff Bezos called the solution "stupid, thoughtless, and painfully out of line with our principles". Amazon later offered restored copies (with the user's annotations) or a USD 30 gift certificate/check. ([TechCrunch](https://techcrunch.com/2009/07/23/amazons-bezos-apologizes-for-the-stupid-and-thoughtless-kindle-incident), [HNN](https://www.historynewsnetwork.org/article/116446))
- **Lesson.** The device must not offer any vendor a path to delete user content remotely. Revocation by a store applies to the store catalogue, never to files on the device. Annotations, ratings and playlists are user data that survive removal of the item they refer to.

### 1.5 Grooveshark (2015) and Rdio (2015) — service death

- Grooveshark closed 2015-04-30 in a settlement with UMG, Sony and Warner; recovery of playlists required third-party tools ([Wikipedia](https://en.wikipedia.org/wiki/Grooveshark)). Rdio ceased 2015-12-22 after Chapter 11; Pandora bought its assets for USD 75 million ([Wikipedia](https://en.wikipedia.org/wiki/Rdio)). What happened to Rdio users' collections is not documented there (**UNVERIFIED**).
- **Lesson.** Metadata and streaming providers are optional and replaceable. Playlists, play counts and ratings are stored locally, in an exportable format, keyed to stable file identifiers (not to a provider's IDs alone), so a provider disappearing breaks no list.

### 1.7 Kindle Whispersync / Kobo reading position — continuity win

- Whispersync synchronises reading progress, bookmarks and notes across Kindle devices and apps ([Wikipedia](https://en.wikipedia.org/wiki/Whispersync)). Its conflict behaviour (offering to jump to the "furthest page read" instead of silently moving the reader) is from memory: **UNVERIFIED**. Kobo's equivalent was not sourced: **UNVERIFIED**.
- **Lesson.** Position sync conflicts are surfaced, not resolved silently: if the T1 and the desktop disagree on playback position, offer the other one ("continue from 1:23:10 on Desktop?") rather than jumping. Position is per item, and audiobook/long-form position is precious state.

---

## 2. Devices, platforms and end-of-life

### 2.1 Sonos app redesign (May 2024) — error

- **What happened.** In May 2024 Sonos replaced its app to modernise it and support the Ace headphones. It shipped with connectivity problems, lag, accessibility regressions and missing features including alarms and sleep timer. ([Wikipedia: Sonos](https://en.wikipedia.org/wiki/Sonos), [Digital Trends](https://digitaltrends.com/?p=3673544))
- **Response.** CEO Patrick Spence apologised in July 2024 ([Digital Music News](https://www.digitalmusicnews.com/2024/07/26/sonos-ceo-apologizes-for-bump-app-redesign-amid-user-uproar-online/)). Sonos said fixing the app would cost USD 20–30 million and delayed new products until it was fixed ([MacTrast](https://www.mactrast.com/2025/01/sonos-ceo-stepping-down-following-failed-app-redesign/), [Tom's Guide](https://tomsguide.com/audio/speakers/sonos-delays-new-products-until-broken-app-gets-fixed-what-we-know)). Spence resigned in January 2025; Tom Conrad became interim, then permanent CEO in July 2025 ([Wikipedia](https://en.wikipedia.org/wiki/Sonos)). Sonos considered relaunching the old app but did not, because it could not be restored easily (**UNVERIFIED** beyond Wikipedia's mention that it was "considered").
- **Root cause.** Reported as a rewrite shipped before feature parity, driven by a product launch date (**UNVERIFIED** as an internal cause; the missing features themselves are confirmed).
- **Lesson.** A controller rewrite ships only at feature parity, measured against a written list of every existing feature; anything dropped is announced before release. Keep the previous controller installable and able to talk to devices (protocol compatibility both ways), so a bad release is reversible on the user's side. Product launches never set the release date of the software that runs a user's existing system.

### 2.2 Sonos S1/S2 split and Recycle Mode (2020) — error then partial fix

- **What happened.** January 2020: Sonos announced that from May 2020 "legacy" products would get no more software updates. Its trade-up program had a "Recycle Mode" that permanently disabled the old device in exchange for a 30 % discount. ([BetaNews](https://betanews.com/2020/01/24/sonos-updates/), [Consumer Reports](https://www.consumerreports.org/speakers/sonos-ends-recycling-program-that-disabled-older-devices/))
- **Response.** Spence wrote "we did not get this right from the start"; Sonos introduced S2 for new products and renamed the old system S1, allowing split systems, with S1 receiving bug fixes and security patches. Recycle Mode was later dropped. ([Pocket-lint](https://www.pocket-lint.com/speakers/news/sonos/150858-sonos-ceo-we-didn-t-get-this-right-says-legacy-devices-will-work-after-may), [Wikipedia](https://en.wikipedia.org/wiki/Sonos), [Consumer Reports](https://www.consumerreports.org/speakers/sonos-ends-recycling-program-that-disabled-older-devices/))
- **Lesson.** A new generation never fragments the user's system into two apps. The device-to-device protocol must be versioned with capability negotiation so old and new devices coexist (SYS-040: disable what a hardware cannot do, do not split the ecosystem). Never destroy working hardware as a commercial incentive.

### 2.3 Spotify Car Thing bricked (2024-12-09) — error

- **What happened.** Car Thing launched February 2022 and was discontinued later that year with assurances it would keep working; in May 2024 Spotify announced it would stop functioning on 2024-12-09. ([TechCrunch](https://techcrunch.com/2024/05/30/spotify-begins-offering-car-thing-refunds-as-it-faces-lawsuit-over-bricking-the-streaming-device), [9to5Google](https://9to5google.com/2024/05/30/spotify-car-thing-refund/))
- **Response.** After a class action filed 2024-05-28 (S.D.N.Y.), Spotify offered refunds (about USD 90) with proof of purchase. ([TechCrunch](https://techcrunch.com/2024/05/30/spotify-begins-offering-car-thing-refunds-as-it-faces-lawsuit-over-bricking-the-streaming-device), [Engadget](https://engadget.com/spotify-now-says-it-will-refund-car-thing-purchases-193001487.html))
- **Lesson.** No T1 function may require an OpenDAP server to stay alive. If a component ever must be retired, the last update unlocks it (bootloader, open firmware) instead of disabling it. Promises like "will keep working" are written down with conditions.

### 2.4 Revolv hub (Nest, 2016-05-15) — error

- **What happened.** Nest bought Revolv in 2014 and in April 2016 announced the cloud service would end on 2016-05-15, after which the USD 299 hub "would simply cease to function". ([VentureBeat](https://venturebeat.com/technology/alphabets-nest-will-permanently-turn-off-all-revolv-hubs-on-may-15-2016), [The Register](https://www.theregister.com/2016/04/06/nest_kills_revolv/))
- **Response.** Refunds of the purchase price; the FTC examined it and closed without action, citing the small number of units and the full refunds, while noting consumers would not expect a hub to become unusable. ([FTC closing letter](https://www.ftc.gov/system/files/documents/closing_letters/nid/160707nestrevolvletter.pdf))
- **Lesson.** Regulators now look at whether a buyer would expect a device to stop working. Cloud-dependency is a product liability, not only a UX issue. Local function first, always.

### 2.5 Logitech Harmony (discontinued April 2021) — mostly kept promise, then erosion

- Logitech stopped making Harmony remotes in April 2021 and said it would keep servers, apps and the device database running "in perpetuity" while devices are used ([BetaNews](https://betanews.com/2021/04/10/logitech-discontinues-harmony-remotes/), [9to5Mac](https://9to5mac.com/2021/04/11/logitech-harmony-remotes-discontinued/)). Later it dropped support for the oldest models (670, 720, 880, 1100i, Harmony for Xbox 360) ([PCWorld](https://www.pcworld.com/article/2651033/logitech-is-dropping-support-for-its-oldest-harmony-remotes.html)).
- **Lesson.** A device whose configuration lives only in the vendor's cloud has a finite life no matter the promise. The T1 and the desktop hold the configuration; online services cache, never own.

### 2.6 Insteon (April 2022) — failure, rescued by its users

- Insteon shut down abruptly in mid-April 2022 after a sale failed; in June 2022 a group of users bought the company and restored services by October 2022 ([Wikipedia](https://en.wikipedia.org/wiki/Insteon)).
- **Lesson.** Even a rescue needs the servers, keys and code to be transferable. A documented protocol and servers the community can run mean no one has to buy a company to keep devices alive.

### 2.7 Peloton Tread+ "Just Run" paywall (2021) — error, reversed

- After the May 2021 Tread+/Tread recall, a safety update (Tread Lock, a PIN) also made the treadmill unusable without the subscription; Peloton restored "Just Run" for non-subscribers in August 2021 after backlash and gave free months meanwhile. ([TechRadar](https://www.techradar.com/news/peloton-treadmills-lose-one-of-their-most-popular-free-features), [Tom's Guide](https://www.tomsguide.com/news/peloton-will-brick-your-dollar4300-treadmill-if-you-dont-pay-the-dollar40-monthly-fee), [PeloBuddy](https://www.pelobuddy.com/peloton-tread-update-lock-subscription/))
- **Lesson.** A safety change must not smuggle in a feature lock. Matches the OpenDAP rule: limits exist only for safety, documented and visible, never to segment (BRD-049). An account or subscription is never required for a local function.

### 2.8 Garmin Connect ransomware outage (July 2020) — server failure, devices kept working

- **What happened.** WastedLocker ransomware encrypted Garmin systems; Connect, website and support were down for several days until about 2020-07-27 ([BleepingComputer](https://www.bleepingcomputer.com/news/security/garmin-outage-caused-by-confirmed-wastedlocker-ransomware-attack/), [Android Central](https://www.androidcentral.com/garmin-connect-suffers-multi-hour-outage)). Reports that Garmin paid a ransom are **UNVERIFIED** (not confirmed by Garmin).
- **Win inside the failure.** Watches kept recording locally; data synced once services returned, and users could copy `.fit` files over USB and upload elsewhere ([the5krunner](https://the5krunner.com/2020/07/25/garminpocalypse-5-garmin-speaks-out-at-last/), [DC Rainmaker](https://dcrainmaker.com/2020/07/upload-garmin-workout.html), [road.cc](https://road.cc/index.php/news/garmin-connect-down-worldwide-theres-workaround-275767/)).
- **Lesson.** This is the right shape: device is the source of truth, server is a mirror, open file format plus USB mass-storage as the escape hatch. Sync queues must survive days offline without loss or duplication.

### 2.9 Squeezebox → Lyrion Music Server — longevity win via community

- Logitech's last LMS release was 7.7; 7.8 onward came from the community; in 2024 it was renamed Lyrion Music Server (9.0), with 9.1 on 2024-11-29. It still serves Squeezebox hardware. ([Lyrion docs](https://lyrion.org/reference/lyrion-music-server/), [mp3tag community](https://community.mp3tag.de/t/lms-logitech-media-server-now-lyrion-music-server/64369))
- **Lesson.** Open-source server plus documented device protocol let hardware outlive its maker by a decade. Document the OpenDAP device-to-device protocol as a public spec from the start.

### 2.10 Pebble revival (2025) — longevity win

- Google open-sourced PebbleOS on 2025-01-27 at Eric Migicovsky's request; his company Core Devices launched new watches running it, compatible with existing Pebble apps and watchfaces ([Gigazine](https://gigazine.net/gsc_news/en/20250128-pebble-os-open-source), [It's FOSS](https://itsfoss.com/news/pebble-returns-as-open-source/), [The Register, 2025-11-25](https://www.theregister.com/2025/11/25/pebble_eink_smartwatch_open_source/)).
- **Lesson.** Open-sourcing late works, but only after years of loss; open from day one (already OpenDAP policy) avoids the gap.

### 2.11 Winamp — longevity of a brand, failure of a late "open" release

- Winamp shipped 1997; AOL announced shutdown of winamp.com for 2013-12-20; Radionomy bought it in 2014; 5.8 (2018) and 5.9 (2022) followed. On 2024-09-24 Llama Group published source under a "Winamp Collaborative License" forbidding forks; after backlash (including proprietary code found in the repo) the repository was deleted on 2024-10-16. ([Wikipedia](https://en.wikipedia.org/wiki/Winamp))
- **Lesson.** "Source available" with no-fork terms costs trust instead of earning it. Licence and repository hygiene (REUSE, no third-party proprietary code) must be right before publication.

### 2.12 Rockbox — longevity win

- First release 2002-06-02; v4.0 on 2025-04-01; GPLv2+; dozens of stable ports, mostly done by reverse engineering without manufacturer help ([Wikipedia](https://en.wikipedia.org/wiki/Rockbox)).
- **Lesson.** A DAP with an open firmware path stays useful for twenty years. For the T1: unlockable bootloader, documented hardware, mainline kernel make Rockbox-style longevity the default rather than a heroic effort.

---

## 3. Updates

### 3.1 Android A/B (seamless) updates — win

- Updates install while the system runs; if applying fails, the user keeps the old OS; if the new slot fails to boot, the device falls back automatically; dm-verity guarantees an uncorrupted image; updates can stream without free space for the package ([AOSP docs](https://source.android.com/docs/core/ota/ab)).
- **Lesson.** Blackbird OS updates should be A/B (or equivalent) with automatic rollback on failed boot, verified images, and no user data in the system partition.

### 3.2 iOS 8.0.1 (2014-09-24) — update disaster, fast recovery

- 8.0.1 broke Touch ID and cellular service on iPhone 6/6 Plus; Apple pulled it, told users to restore 8.0, and shipped 8.0.2 the next day ([Wikipedia: iOS 8](https://en.wikipedia.org/wiki/IOS_8)).
- **Lesson.** Staged rollout (small cohort first, halt on telemetry-free signals such as update-check failures or user reports), and an always-available rollback path.

### 3.3 Apple "Error 53" (2016) — update vs repair

- iPhone 6 units with third-party home-button repairs were disabled by an iOS update's hardware check; Apple shipped a fix on 2016-02-18 and said the check was meant for factory testing ([Wikipedia](https://en.wikipedia.org/wiki/Error_53)).
- **Lesson.** An update must never disable a device because of a part swap. With interchangeable faceplates and HATs, unknown or third-party parts degrade gracefully with a visible explanation (SYS-040).

### 3.4 CrowdStrike Falcon (2024-07-19) — update disaster

- A faulty content update (Channel File 291) caused an out-of-bounds read and boot loops on about 8.5 million Windows devices; it was reverted 78 minutes later; customers had no way to delay content updates. CrowdStrike then committed to staged deployment and customer control. ([Wikipedia](https://en.wikipedia.org/wiki/2024_CrowdStrike-related_IT_outages))
- **Lesson.** "Data" updates (codec tables, provider plugins, metadata rules) are code in disguise and get the same validation, staging and rollback as the OS. Users can defer updates.

### 3.5 Tesla OTA recall 23V-838 (December 2023) — reference for OTA as a remedy

- Tesla filed a recall covering 2,031,220 vehicles (Autosteer driver-monitoring) whose remedy was an over-the-air software update, after NHTSA's investigation PE21-020 / EA22-002 ([NHTSA Part 573 report](https://static.nhtsa.gov/odi/rcl/2023/RCLRPT-23V838-8276.PDF)).
- **Lesson.** OTA can deliver a fix to a whole fleet within days, which is why the update channel itself is safety-critical: signed, verified, staged and auditable.

### 3.6 Western Digital My Book Live mass wipe (June 2021) — end-of-life plus security

- In June 2021 attackers factory-reset My Book Live devices remotely through an unauthenticated administrator API (CVE-2021-35941, distinct from CVE-2018-18472), destroying stored data ([CVE record](https://cveawg.mitre.org/api/cve/CVE-2021-35941)). The devices had been out of support for years (**UNVERIFIED** date; WD's own notice could not be fetched).
- **Lesson.** Destructive operations (factory reset, wipe, delete library) always require local physical confirmation, never only a network call. End-of-support devices still on networks must fail safe: before support ends, ship a final update that closes remote admin surfaces.

---

## 4. Security

### 4.1 KDE Connect CVE-2025-66270 (2025-12-05) — direct analogue

- Protocol 8 exchanges an unencrypted identity packet, then an encrypted one after TLS; implementations did not check that `deviceId` and `protocolVersion` matched between the two. An attacker on the adjacent network who knows a paired device's ID could impersonate it and skip authentication. CVSS 4.7. Fixed in KDE Connect desktop 25.12, iOS 0.5.4, Android 1.34.4, GSConnect 68, Valent 1.0.0.alpha.49. ([CVE record](https://cveawg.mitre.org/api/cve/CVE-2025-66270), [KDE advisory 20251128-1](https://kde.org/info/security/advisory-20251128-1.txt), [NVD](https://nvd.nist.gov/vuln/detail/CVE-2025-66270))
- **Response (good).** Coordinated disclosure across all five independent implementations, patches linked per implementation, a workaround (disable on untrusted networks or unpair) ([advisory](https://kde.org/info/security/advisory-20251128-1.txt)).
- **Lesson.** For the OpenDAP device-to-device protocol: identity is the authenticated key (certificate fingerprint pinned at pairing), never a claimed ID field; anything said before the secure channel is a hint, re-verified inside it. Write the handshake as a spec with test vectors so every implementation (device, desktop, CLI, third party) is checked against the same cases.

### 4.2 AirDrop sender identification (China, January 2024)

- Beijing Wangshendongjian Forensic Appraisal Institute built rainbow tables of phone numbers and emails to reverse the hashes AirDrop exchanges and identify senders. TU Darmstadt researchers had reported the weakness to Apple in 2019. ([TechTarget](https://www.techtarget.com/cybersecurity/news/366565895/China-claims-it-cracked-Apples-AirDrop-can-track-senders), [Bitdefender](https://www.bitdefender.com/en-gb/blog/hotforsecurity/china-cracks-apples-airdrop-to-extract-phone-numbers-and-emails-from-alleged-threat-actors))
- **Lesson.** Hashes of low-entropy identifiers (phone numbers, emails, short names) are not privacy. Discovery must not broadcast user identifiers, even hashed; use random per-pairing keys and private set intersection or nothing. A reported design flaw left unfixed for years becomes someone's tool.

### 4.3 Bluetooth specification attacks: KNOB (2019), BIAS (2020), BLUFFS (2023)

- **KNOB**, CVE-2019-9506: entropy negotiation forced down to 1 byte, keys brute-forced in real time; all 24 tested devices vulnerable; Bluetooth SIG erratum ([knobattack.com](https://knobattack.com/)).
- **BIAS**, CVE-2020-10135: impersonation of a previously paired device in BR/EDR ([BIAS site](https://francozappa.github.io/about-bias/)).
- **BLUFFS**, CVE-2023-24023: breaks forward and future secrecy of Bluetooth Classic sessions, spec 4.2–5.4, tested on 17 chips; the researcher proposed a hardened key derivation ([EURECOM](https://www.eurecom.fr/en/node/4809210), [SecurityWeek](https://www.securityweek.com/new-bluffs-bluetooth-attacks-have-large-scale-impact-researcher/)).
- **Lesson.** Bluetooth link security is not the security boundary for anything beyond audio. Remote control, file transfer, sync over BLE or BR/EDR run an application-layer authenticated encryption on top, bound to the pairing keys. Enforce minimum key size and Secure Connections Only where the controller allows; track BlueZ and controller firmware updates.

### 4.4 Plex breaches (2022, 2025) and the LastPass chain (2022)

- Plex: August 2022 database intrusion (emails, usernames, bcrypt hashes) and September 2025 a second one with the same data types; both led to password resets ([Security Affairs](https://securityaffairs.com/134814/data-breach/plex-data-breach.html), [The Register, 2025](https://www.theregister.com/2025/09/09/plex_breach/), [Help Net Security](https://www.helpnetsecurity.com/2025/09/09/plex-tells-users-to-change-passwords-due-to-data-breach-pushes-server-owners-to-upgrade/)).
- LastPass: attackers stole encrypted vaults and customer data between 2022-08-12 and 2022-10-26; a DevOps engineer's home computer was compromised through an unpatched Plex Media Server (reported as CVE-2020-5741, a 2020 bug), and a keylogger captured the master password ([9to5Mac](https://9to5mac.com/2023/02/27/lastpass-devops-engineers-hacked/), [The Record](https://therecord.media/plex-unaware-of-vulnerabilities-lastpass-hacked/), [Wladimir Palant](https://palant.de/2023/02/28/lastpass-breach-update-the-few-additional-bits-of-information/)).
- **Lesson.** (a) A home media server is an attack surface on the user's network: OpenDAP Desktop and the T1 must update easily and nag about old versions. (b) Accounts are optional (SYS-047), and the less any server holds, the less a breach leaks: no password storage when device keys suffice. (c) Any off-site backup is end-to-end encrypted, with keys only on the owner's devices, so a server breach yields only ciphertext.

### 4.5 Sonos / UPnP exposure — **partly UNVERIFIED**

- Sonos speakers expose an unauthenticated local control API (UPnP/SOAP); Trend Micro's 2017 research on internet-exposed Sonos and Bose speakers is widely cited, and NCC Group presented Sonos Era 100 vulnerabilities at Black Hat USA 2024. Both source pages failed to load: **UNVERIFIED** (details, CVE numbers, counts).
- **Lesson.** "Trusted LAN" is not an authentication model. Every control endpoint on the T1 requires a paired key; UPnP/DLNA rendering, if offered, is opt-in, scoped to playback and never exposes admin functions; nothing is ever reachable from the internet by default.

### 4.6 Default credentials (Astell&Kern AK File Drop)

- AK File Drop turns the player into an FTP server on the LAN; the documented default login is `admin` / `admin` ([Moon Audio how-to](https://moon-audio.com/how-to-astell-kern-file-drop)).
- **Lesson.** No shared default credentials ever (also a legal requirement under the EU Radio Equipment Directive delegated act and the UK PSTI regime, **UNVERIFIED** here as not re-checked). File transfer services are off by default, per-device credentials or pairing, time-limited when enabled, visibly indicated while active.

### 4.7 Syncthing — positive track record

- Private disclosure via `security@syncthing.net`, PGP-signed releases with SHA-256 checksums ([Syncthing security](https://syncthing.net/security/)). The GitHub repository lists two published advisories, both moderate: a relay-protocol crash (2021-04-06) and a Web GUI XSS (2023-06-06) ([advisories](https://github.com/syncthing/syncthing/security/advisories?state=published)). Device identity is the certificate hash, which is exactly what KDE Connect lacked (Syncthing design: **UNVERIFIED** against its spec).
- **Lesson.** Device ID = public-key fingerprint; relays forward encrypted traffic only; signed releases; a small number of public advisories over a long life is achievable with that model.

### 4.8 Matter security model — reference

- Device attestation certificates with PKI, a Distributed Compliance Ledger publishing certified-device information, local operation without internet for core functions; v1.0 published 2022-10-04 ([Wikipedia: Matter](https://en.wikipedia.org/wiki/Matter_(standard))).
- **Lesson.** Attestation lets a controller know a device is genuine, and local-first operation is a certification requirement, not a promise. For HATs and faceplates: an attestation mechanism can be offered, but per OpenDAP rules it may only inform the user, never lock out a part.

### 4.9 Item trackers and stalking: AirTag, Tile, DULT

- AirTag (2021): reported as "frighteningly easy" to misuse; Apple shortened the separation alarm from three days to 8–24 hours (June 2021), released Tracker Detect for Android (December 2021) and added setup warnings (February 2022); speaker-removed AirTags were sold online ([Wikipedia: AirTag](https://en.wikipedia.org/wiki/AirTag)).
- Apple and Google shipped cross-platform unwanted-tracking alerts on 2024-05-13 (iOS 17.5, Android 6.0+) under the "Detecting Unwanted Location Trackers" specification; Chipolo, eufy, Jio, Motorola and Pebblebee committed to it ([Apple Newsroom](https://www.apple.com/newsroom/2024/05/apple-and-google-deliver-support-for-unwanted-tracking-alerts-in-ios-and-android/)). The IETF DULT accessory draft is co-authored by Apple and Google ([IETF datatracker](https://datatracker.ietf.org/doc/draft-ietf-dult-accessory-protocol/)).
- Tile: Georgia Tech researchers reported in September 2025 that Tile trackers broadcast identifying data unencrypted and servers stored locations and MAC addresses without end-to-end encryption ([Wikipedia: Tile](https://en.wikipedia.org/wiki/Tile_(company))). Tile's "Anti-Theft Mode" (ID verification, a stated USD 1 million penalty for misuse) is **UNVERIFIED**.
- **Lesson.** The T1's BLE item tracking must implement DULT (or its successor) from the first release: rotating identifiers, detectability by phones of both platforms, sound on demand, no plain-text stable identifiers, location reports end-to-end encrypted. Treat anti-stalking as a must, not a should.

---

## 5. Privacy and trust

### 5.1 Signal — privacy as product

- Signal states the only data it can produce under subpoena is the account's registration date and last connection date; it publishes each request (cases from 2016 to 2026) ([signal.org/bigbrother](https://signal.org/bigbrother/)).
- **Lesson.** Collect so little that a breach or a subpoena has nothing to give, and publish the answer: any online service in the ecosystem publishes a data inventory and a transparency log.

### 5.2 Local-first software (Ink & Switch, 2019) — design reference

- Seven ideals: no spinners, not trapped on one device, the network is optional, seamless collaboration, the long now, security and privacy by default, ultimate ownership ([Ink & Switch](https://www.inkandswitch.com/essay/local-first/)).
- Obsidian applies it: notes are plain Markdown files in a folder; any editor can open them; sync via its own service or third parties ([Obsidian help](https://obsidian.md/help/data-storage)).
- **Lesson.** OpenDAP's library database is a cache of facts derivable from files plus a small, documented, exportable user-data store (ratings, playlists, positions) in an open format. Sync between devices works peer-to-peer on the LAN with no account; a relay only adds reach.

---

## 6. Quality and continuity wins

### 6.1 Apple Handoff / Continuity

- Start a task on one device and continue on a nearby one; requires the same Apple Account, Bluetooth and Wi-Fi ([Apple Support](https://support.apple.com/en-us/102426)).
- **Lesson.** Excellent UX, but account-bound. OpenDAP's equivalent ("continue on T1 / on Desktop") must work with paired devices and no account.

### 6.2 Spotify Connect

- Selecting a speaker transfers playback; the phone becomes a remote; the speaker streams directly; playback state is reflected on all Connect devices even when not active ([Spotify developer docs](https://developer.spotify.com/documentation/commercial-hardware/implementation/guides/connect-basics)).
- **Lesson.** The model to copy for real-time playback state: one active renderer, many controllers, the renderer fetches media itself (bit-perfect from the source, not relayed through the controller), state events broadcast to every peer. Do it locally, without the cloud Spotify relies on.

### 6.3 Framework Laptop and Fairphone 5 — repair and support life

- iFixit scored the Framework Laptop 10/10 on 2021-07-28: replaceable mainboard, keyboard, battery, storage; captive screws; QR-labelled parts; included screwdriver ([iFixit](https://www.ifixit.com/News/51614/framework-laptop-teardown-10-10-but-is-it-perfect)).
- Fairphone 5 (announced 2023-08-30) promises five major Android updates and eight years of security patches, enabled by choosing an industrial-lifecycle Qualcomm QCM6490 ([Wikipedia](https://en.wikipedia.org/wiki/Fairphone_5)).
- **Lesson.** Support life is decided at part selection: choose SoCs with long vendor lifecycles and mainline support (already OpenDAP sourcing policy). Publish a dated support commitment for the T1, and design so the battery is replaceable with ordinary tools.

---

## 7. Lessons for OpenDAP

### Security

- **Lesson 1.** **Identity is a key, not a field.** Pin a public-key fingerprint at pairing and authenticate every session with it; re-verify anything exchanged before the secure channel. Spec plus shared test vectors for all implementations. (KDE Connect §4.1, Syncthing §4.7, Matter §4.8)
- **Lesson 2.** **Do not trust the radio or the LAN.** Application-layer authenticated encryption over Bluetooth and Wi-Fi; no unauthenticated control endpoint; nothing exposed to the internet by default. (KNOB/BIAS/BLUFFS §4.3, Sonos UPnP §4.5, My Book Live §3.6)
- **Lesson 3.** **No hashed identifiers in discovery.** Low-entropy identifiers leak through rainbow tables. (AirDrop §4.2, Tile §4.9)
- **Lesson 4.** **No default credentials, services off by default.** File transfer and remote admin are opt-in, paired, visible while active. (Astell&Kern §4.6, My Book Live §3.6)
- **Lesson 5.** **Destructive actions need local physical confirmation.** (My Book Live §3.6, Kindle §1.4)
- **Lesson 6.** **Item tracking implements DULT from day one.** (AirTag, Tile, DULT §4.9)
- **Lesson 7.** **Servers hold as little as possible; backups are end-to-end encrypted.** (Plex, LastPass §4.4, Signal §5.1)
- **Lesson 8.** **Fixes are fast, coordinated and documented**: private disclosure address, advisories with affected versions and workarounds. (KDE Connect §4.1, Syncthing §4.7, AirDrop's unfixed 2019 report §4.2 as the counter-example)

### Quality

- **Lesson 9.** **Rewrites ship at feature parity, checked against a written feature list; launches never set the software date.** (Sonos 2024 §2.1)
- **Lesson 10.** **The previous controller version stays usable** (protocol compatibility both directions) so a bad release is reversible for users. (Sonos 2024 §2.1, S1/S2 §2.2)
- **Lesson 11.** **Accessibility regressions block a release.** (Sonos 2024 §2.1)

### Updates

- **Lesson 12.** **A/B with automatic rollback and verified images** (OS-017). (Android A/B §3.1)
- **Lesson 13.** **Staged rollout and user-deferrable updates, including "data" updates.** (iOS 8.0.1 §3.2, CrowdStrike §3.4)
- **Lesson 14.** **Updates never disable hardware for a part swap or a business reason.** (Error 53 §3.3, Peloton §2.7, Sonos Recycle Mode §2.2)
- **Lesson 15.** **The update channel is safety-critical**: signed, auditable, able to deliver fleet-wide fixes. (Tesla 23V-838 §3.5)

### Longevity

- **Lesson 16.** **No function depends on an OpenDAP server.** Online services add reach, never capability. (Car Thing §2.3, Revolv §2.4, Harmony §2.5, Garmin §2.8)
- **Lesson 17.** **Open protocol spec, open server code, unlockable bootloader** so the community can carry the product after the company. (Lyrion §2.9, Pebble §2.10, Rockbox §2.12, Insteon §2.6)
- **Lesson 18.** **Published, dated support commitment and end-of-life procedure**, including a final update that unlocks rather than disables. (Fairphone §6.3, Google Play Music §1.2, Car Thing §2.3)
- **Lesson 19.** **Open licences done right before publishing.** (Winamp §2.11)

### Trust

- **Lesson 20.** **No remote deletion of user content by any vendor or store.** (Kindle §1.4)
- **Lesson 21.** **Acquired content never depends on a licence server.** (PlaysForSure §1.3)
- **Lesson 22.** **Accounts and subscriptions never gate a local function.** (Peloton §2.7, Handoff's account requirement §6.1)
- **Lesson 23.** **When it goes wrong, say so plainly and fix it**: an apology without a change does not restore trust; Sonos paid USD 20–30 million and a CEO. (Kindle §1.4, Sonos §2.1/§2.2)

### Sync correctness

- **Lesson 24.** **Sync never deletes, replaces or transcodes an original as a side effect**; propagated deletes are explicit, previewed, reversible. (iCloud Music Library §1.1)
- **Lesson 25.** **Device is the source of truth; servers mirror.** Offline queues survive days without loss or duplicates; open formats and USB mass storage are the escape hatch. (Garmin §2.8, local-first §5.2)
- **Lesson 26.** **User data (playlists, ratings, positions, annotations) is local, exportable, keyed to stable identifiers, and survives a provider or item disappearing.** (Grooveshark/Rdio §1.5, Kindle §1.4, Google Play Music §1.2)
- **Lesson 27.** **Conflicts are surfaced, not silently resolved** (e.g. playback position offered, not jumped to). (Whispersync §1.7, UNVERIFIED mechanism)
- **Lesson 28.** **Real-time playback state follows the Connect model, locally**: one renderer, many controllers, renderer pulls media bit-perfect, state broadcast to all peers. (Spotify Connect §6.2, Handoff §6.1)

---

## Not covered or unverified

- Tile Anti-Theft Mode, Sonos/UPnP and NCC Group Sonos research details, Kobo sync, the Whispersync conflict behaviour and the iTunes 12.4 safeguards: **UNVERIFIED** (some pages refused fetching). Spotify's feature removals and Roon's reliability record were not sourced and are left out.
- Peloton's 2021 API data exposure and the Sonos 2024 "seven commitments" were not researched.
