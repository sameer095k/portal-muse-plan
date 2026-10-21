# PLAN — Portal → Muse, phased

Device: Meta Portal 10" 2nd gen, **WD50JM**, Android 10 (API 29), arm64,
no Google Play Services. Host computer: Mac (the 2013 MBP runs Big Sur —
fine for adb; the Gradle build wants JDK 21, see Phase 0).

**Why one sitting:** ADB switches off after every reboot on Portals, and the
first-run flow needs a one-time Facebook/WhatsApp login to reach Settings.
Phases 1–6 are sequenced to happen in a single session in front of the
device. Phase 7 (network) can be done from the router any time after.

---

## Phase 0 — Prep (before the device arrives)

- [ ] Install Android platform-tools on the Mac (`brew install android-platform-tools`) and confirm `adb version`.
- [ ] JDK 21 + Android SDK platform 35 on the build machine (portal-muse's build requirement). If the 2013 Mac can't run JDK 21 comfortably, build on any newer machine — only the APK needs to reach the Portal.
- [ ] Clone the upstreams (see RESOURCES.md):
  - `wobsoriano/portal-muse` (the Muse app + its day-one **probe** app)
  - `wobsoriano/immortal-glass` (launcher; we mainly need its `provisioning/` kit)
  - `cromestant/muse_gadget_android` (fallback route B)
  - `facebookincubator/muse-gadget-sdk` (official SDK the ports derive from)
- [ ] Get a Muse **SDK token** from https://gadgets.muse.ai — store it in a private local note only. Never commit it.
- [ ] USB-C data cable ready; Portal power adapter from the box.
- [ ] Decide the device name it will show in the Muse app (e.g. "Kitchen Portal").

## Phase 1 — Unbox + baseline (no changes yet)

- [ ] Power on with the included adapter; complete first-run Wi-Fi.
- [ ] Settings → About: confirm model **WD50JM** and check for the final software update; install it if offered (this is the update that unlocks ADB). After this phase we freeze updates (Phase 4) — no more Meta updates exist anyway.
- [ ] Note the firmware/build string in LOG.md.

## Phase 2 — One-time login + ADB on

- [ ] Complete the one-time Facebook/WhatsApp sign-in to reach the full Settings (expected requirement; the launcher swap in Phase 4 is what frees us from it afterwards).
- [ ] **Settings → Debug → ADB Enabled** (enter PIN if prompted).
- [ ] USB-C to the Mac → tap **Allow** on the Portal → `adb devices` shows the device.
- [ ] **Baseline capture (before touching anything)** — save outputs into this repo under `baseline/`:
  ```
  adb shell pm list packages > baseline/packages-all.txt
  adb shell pm list packages | grep -i -E 'facebook|aloha|meta' > baseline/packages-meta.txt
  adb shell cmd shortcut get-default-shortcut  # informational
  adb shell dumpsys window windows | grep -i mCurrentFocus > baseline/focus.txt
  adb shell settings get global device_name
  ```
- [ ] Record the current default home/launcher component in LOG.md — that's the restore target if we ever roll back.

## Phase 3 — Probe first (decides the Muse route)

portal-muse was built and verified on **Gen 1 (Android 9)**; our Gen 2 runs
Android 10/API 29, which everything else (Immortal, cromestant build) already
targets. The author's own **probe app** exists exactly to answer "what does
this Portal allow" before committing:

```
adb install -g <portal-muse>/android/probe/build/outputs/apk/debug/probe-debug.apk
adb shell am start -n dev.wobsoriano.museportal.probe/.ProbeActivity
adb logcat -s MuseProbe        # the day-one report
```

- [ ] Save the probe report to `baseline/probe-report.txt` and LOG.md.
- [ ] **Decision gate:** mic access, BLE, TTS, foreground-service-at-boot all green → Route A. Anything red → fall back per Phase 6.

## Phase 4 — Immortal launcher (replaces Meta's home)

Immortal (`com.immortal.launcher`) supports API 29 explicitly and ships a
provisioning kit that does this in one double-click, reversibly:

- [ ] From `immortal-glass/provisioning/`: run `Provision-Portal` (macOS) with the Portal connected and ADB on.
- [ ] In the kit options: set Immortal as **home screen** and **screensaver**, and **freeze OS updates** so the setup sticks.
- [ ] Verify: Home shows Immortal's grid + clock/weather header; reboot once and confirm Immortal still comes up as home (the kit's changes persist across reboots; ADB does not — that's fine, we're done with the cable for now).
- [ ] Rollback if needed: the kit's `Restore-Portal` / `--restore`.

What this buys us: Meta's home/login surface is out of the daily path, plus an
on-device app store (curated catalog, F-Droid resolution) for later installs
without a cable.

## Phase 5 — Debloat (disable, never uninstall)

Only packages confirmed present in the Phase 2 baseline get touched, one at a
time, checking the device still behaves after each. Starting list (from the
community guides — verify against our own dump):

```
adb shell pm disable-user --user 0 com.facebook.portal
adb shell pm disable-user --user 0 com.facebook.workplace.portal
adb shell pm disable-user --user 0 com.facebook.weaver
adb shell pm disable-user --user 0 com.facebook.alohaservices.presence
adb shell pm disable-user --user 0 com.facebook.portal.aiservice
```

- The **presence** service matters most: Portal OS runs background person
  detection that does not light the camera LED.
- Never disable `com.android.systemui` or anything outside the facebook/aloha set without a specific reason.
- Every disable goes in LOG.md with its undo: `adb shell pm enable <package>`.
- Meta's own docs warn modifying system apps via ADB "may not be reversible" — `disable-user` is the reversible form; we do not use `pm uninstall --user 0` on system apps.

## Phase 6 — Muse on the device

### Route A (primary): portal-muse — hands-free "Hey Muse"

- Build (JDK 21, Android SDK platform 35):
  ```
  cd <portal-muse>/android
  ./gradlew -p . :muse:test :wake:test :app:testDebugUnitTest :app:assembleDebug :probe:assembleDebug
  adb install -g app/build/outputs/apk/debug/app-debug.apk
  ```
- On-device setup: paste the **SDK token** (gadgets.muse.ai) → name the device → pair from the Muse phone app over Bluetooth → turn on **Hey Muse**.
- Wake word is **on-device** (Vosk small en-us model, 41 MB, SHA-256-pinned download into the app's files) — nothing is sent while it waits for the wake word. After it, the question audio goes to Muse and is kept on the message in the Muse account (same as hey-muse documents).
- Speech out: built-in SVOX Pico TTS on-device. Optional later: a cloud voice endpoint key for a more natural voice (text of answers leaves the device if enabled — off by default in our setup).
- Starts at boot via its foreground service when Hey Muse is on; yields the mic while another app records. If `portal-wake` is installed it delegates wake-word listening to it.
- Over Immortal's home screen, a Muse bar can appear over other apps when invoked.

### Route B (fallback): cromestant/muse_gadget_android

Push-to-talk instead of a wake word; verified end-to-end on Portal Mini
(minSdk 29 = our API level). BLE pairing v5, Wi-Fi provisioning from the app,
Noise XX over WebSocket to Muse's cloud, Muse TTS endpoint for spoken
replies. No Google Play Services, no Meta apps required. JDK 17 + platform 29
build: `./gradlew assembleDebug`.

### Route C (fallback): official Muse app + Tailscale

Sideload the Muse Android app (Portal Toolkit-style install or straight
`adb install`), install Tailscale on the Portal and the home computer, enable
the Tailscale option in the Muse app. Gives the "run your PC from another
room" voice-terminal setup from the YouTube walkthrough. Heavier Meta-app
footprint, so it's the last resort, not the goal.

## Phase 7 — Network firewall + verification

- [ ] Router/Pi-hole: block `facebook.com`, `graph.facebook.com`, `graph-portal.facebook.com`, `*.fbcdn.net`, and the rest of Meta's domains for the Portal's IP/MAC only (don't break the rest of the house).
- [ ] Keep allowed: Muse's cloud endpoints (the gadget link), Tailscale (if Route C bits are in use), Immortal's update/catalog hosts, Open-Meteo (Immortal weather).
- [ ] Note from XDA's setup research: Portal setup stages ask DNS for `www.facebook.com` / `graph.facebook.com` / `graph-portal.facebook.com` — those needs end once setup is complete, which is why the block goes on **after** Phases 1–6.
- [ ] **Verification:** watch the Portal's DNS queries for 48h. Expected after lockdown: no facebook/graph/fbcdn lookups at all (packages disabled + domains blocked). Log the result in LOG.md.
- [ ] Physical layer stays the guarantee: camera shutter closed when idle, mic mute switch works, both tested once.

## Phase 8 — Acceptance checks

- [ ] "Hey Muse" from across the room → answer on screen + spoken aloud.
- [ ] Ask from another app (Muse bar overlays Immortal home).
- [ ] Reboot: Immortal home returns, Hey Muse service resumes, no Meta login demanded.
- [ ] 48h DNS log clean of Meta domains.
- [ ] Household note: anyone who speaks to it is recorded into the Muse account, like any voice note — Abeer knows it's there and what it does.

## Known limits (accepted up front)

- **No root, ever.** The bootloader hard-refuses unlocking (`Flashing Unlock is not allowed`) — confirmed in Immortal's honest-limitations list. Everything here is disable-and-firewall, not purge.
- **No Google Play Services / microG** — apps depending on Google sign-in, push, or DRM won't fully work. Aurora Store (via Immortal's catalog) covers a lot of the rest.
- **ADB is off after every reboot** — re-enable in Settings when we need the cable again.
- **portal-muse on Gen 2 is untested upstream** — hence probe-first (Phase 3) and two fallback routes.
- Meta's warning stands: sideloading and ADB changes are at our own risk; the device is a $50 experiment, and every step above is reversible except a hypothetical bad disable — which is why we disable one package at a time and log the undo.
