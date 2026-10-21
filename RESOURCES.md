# RESOURCES — annotated

Everything the plan draws on, checked 2026-10-21. Ordered by how central
each source is to the build.

## The two projects we're building on

### wobsoriano/portal-muse — the Muse app (Route A)
https://github.com/wobsoriano/portal-muse
- Unofficial Muse gadget app for Portal: hands-free **"Hey Muse"** wake word, no root. Created 2026-10-19, MIT, 64 commits in its first two days, no release APKs — we build it (JDK 21 + Android SDK platform 35).
- Built/verified on **Gen 1 (Android 9)**; our Gen 2 is Android 10. That's why PLAN.md probes first. Its `android/probe` app reports what a given Portal allows (mic, BLE, wake word on Vosk) before we commit.
- Internals: `android/muse` = Kotlin port of the Muse gadget protocol (via the Go core of hey-muse); `android/app` = BLE pairing, mic, wake listener, playback; wake word is **Vosk** `vosk-model-small-en-us-0.15` (41 MB, SHA-256 pinned), detected **on-device**; speech out is bundled **SVOX Pico** TTS. Pairing: paste SDK token from gadgets.muse.ai, then pair from the Muse phone app over Bluetooth.
- Pairing state lives in `muse.json` in the app's files dir and survives rebuilds.

### wobsoriano/immortal-glass — the launcher (replaces Meta home)
https://github.com/wobsoriano/immortal-glass
- "Immortal" — custom launcher + photo-frame screensaver + on-device app store for discontinued Portals. MIT, very active (300+ commits).
- **Explicitly supports Android 10 / API 29 (2019 & 2021 models — that's our WD50JM)** and API 28 Gen 1. Package `com.immortal.launcher`, arm64, no Google services.
- **Provisioning kit** (`provisioning/`): double-click `Provision-Portal` with ADB on — installs Immortal, can set it as home + screensaver, can **freeze OS updates**, fully reversible via `Restore-Portal`. Self-updates over the air afterwards, so later improvements need no cable.
- Extras we may use later: photo-frame screensaver (own photos / Immich / SMB / WebDAV), MQTT → Home Assistant, fleet agent for cable-free management (survives reboots, opt-in).
- Its "honest limitations" list is the source of three facts in PLAN.md: bootloader unlock hard-refused (no root), no Play Services/microG possible, Gen-1's white-on-white installer bug caused by the `com.facebook.aloha.rro.niu.android` overlay (a community find by u/keremimo on r/FacebookPortal) — Gen 2's installer works normally.

## Fallbacks and siblings (same author / same protocol)

### cromestant/muse_gadget_android — Route B
https://github.com/cromestant/muse_gadget_android
- The other Android port of the Muse Gadget protocol. Push-to-talk (no wake word), works end-to-end on a **Portal Mini (DT90GB)**, minSdk 29. BLE pairing v5, Wi-Fi provisioning from the Muse app, Noise XX over WebSocket, spoken replies via Muse's TTS endpoint. JDK 17 + platform 29 build.

### wobsoriano/hey-muse — the Echo sibling
https://github.com/wobsoriano/hey-muse
- Fork of TECHO5 that turns an Echo Show 5 into a Muse voice device; portal-muse's protocol core is ported from its Go implementation. Useful reading for behavior details: after the wake word, up to 60s of speech goes to Muse as a recording kept on the message in your account; with the built-in voice, nothing else leaves the device.

### facebookincubator/muse-gadget-sdk — Meta's official SDK
https://github.com/facebookincubator/muse-gadget-sdk
- Apache 2.0 ESP32 + Linux SDKs (Oct 2, 2026) that both ports derive from. Tokens come from https://gadgets.muse.ai; pairing is via the Muse app (Developer mode → Devices). Also the source of the Muse Gadget Forge skill (Toddoward/muse-gadget-forge) if we ever build an ESP32 gadget.

## Meta's official Portal development docs

### Building apps for Meta Portal devices
https://developers.meta.com/vr/documentation/android-apps/portal-development/ (updated Sep 8, 2026)
- The official line: Settings → Debug → **ADB Enabled**, USB-C, tap Allow. Gen support table (Gen 1 minSdk 28, **Gen 2 minSdk 29**).
- Meta's warning, quoted in PLAN.md: "Removing or modifying system apps via ADB may affect device functionality and may not be reversible." → our disable-only rule.
- Also documents Meta's AI tooling (VR CLI) for building/deploying apps to Portal.

## Debloat + setup research

### SysSeder — "Resurrecting Dead Silicon"
https://www.sysseder.com/2026/06/resurrecting-dead-silicon-how-to-unlock.html
- The concise ADB unlock + debloat walkthrough: `pm list packages | grep facebook`, then `pm disable-user --user 0` on `com.facebook.portal`, `com.facebook.workplace.portal`, `com.facebook.weaver`. Source of the starting package list in Phase 5 (we re-verify against our own device dump).

### XDA — "Anyone been able to do anything with a Facebook Portal?"
https://forum.xda-developers.com/t/anyone-been-able-to-do-anything-with-a-facebook-portal.3878505/
- The long-running thread (2018→2026). Pre-unlock history (fastboot on Portal+, Qualcomm EDL 9008) is obsolete now that ADB is official, but its setup-stage research is gold for Phase 7: a factory-reset Portal needs DNS for `www.facebook.com` during Software Update, and `graph.facebook.com` + `graph-portal.facebook.com` for the Name stage — i.e., block Meta domains only **after** setup completes.

### Kiosk Satellite / presence-service notes (from earlier research, Oct 20)
- Portal OS runs background person detection (`com.facebook.alohaservices.presence` / `com.facebook.portal.aiservice`) that does **not** light the camera LED — the strongest argument for disabling presence packages and for the hardware shutter as the real guarantee.

## Other things a Portal can become (parking lot)

- **TRMNL** — https://trmnl.com/blog/meta-portal — official step-by-step to turn a Portal into a TRMNL e-ink-style dashboard via `adb install` of the TRMNL Display APK (usetrmnl/trmnl-android releases). A good "second life" if the Muse build stalls.
- **PortalKids** (davidedicillo/PortalKids) — kids' routine/chore board with points; mentioned on HN's Portal-ADB thread. Not our use case, good proof the hardware takes normal Android apps well.
- **Wormhole Display** (pgodlews, covered by Hackaday) — Portal as an AirPlay extended display for Mac/iPhone/iPad (Android port of UxPlay, uses the Portal's H.264 decoder). Tempting second app to sideload.
- **"Run Your PC From Another Room: Muse + Tailscale + Meta Portal"** (YouTube) — Route C's recipe: Tailscale on PC + Portal, Muse app sideloaded via Portal Toolkit, talk to your PC from the couch. Troubleshooting notes included (sleeping PC can't answer, re-approve Tailscale in the Muse app).

## Community

- **r/FacebookPortal** — where most of this surfaced (the portal-muse announcement thread Sameer sent, the u/keremimo installer-overlay discovery). Check before Phase 3 for any Gen-2-specific portal-muse reports.
- **Hacker News** — "Meta enables ADB on deprecated Portal devices" thread (Jun 2026) for the official blog link (developers.meta.com/horizon/blog) and early adopter reports.
