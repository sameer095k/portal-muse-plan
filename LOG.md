# LOG

Newest first. Every change made on the device gets an entry: what, the exact
command, and its undo. Sessions where Sameer's hands are on the device and
Muse is remote eyes get dated headers like his TV project logs.

## 2026-10-24 (evening) — Setup + takeover sitting (Sameer's hands, Muse remote)

- **Setup fight, then through:** the out-of-box firmware failed every Meta server call — device-name save, WhatsApp QR fetch, and Facebook login (which falsely reported "incorrect password"; his credentials verified good directly). A factory reset (hold Vol+ + Vol− while plugging in power) + the final software update fixed the name save. An account lockout (seen on screen, with a dead-end QR) kept Facebook login rejecting afterward; **WhatsApp login succeeded after a second factory reset** and is the device's account. Facebook stays unlinked — fine, WhatsApp is the keeper.
- **ADB live:** Settings → Debug → ADB Enabled (toggle UI looks unresponsive; it works), USB-C (port on the back, behind the speaker) to the 2013 MacBook (USB-A end; first cable was charge-only — no device on the bus at all). Device `omni`, Android 10 / API 29, build QKQ1.210213.001.3051355900018050. Baseline captured before any change: 153 packages (list in `baseline/packages-all.txt`), full getprop held privately (contains the serial).
- **Builds run on the Mac, not Muse's VM** (the VM's sandbox breaks Gradle's client↔daemon dispatch; the Mac is clean): Temurin JDK 21 + Android SDK platform 35 installed on the Mac 2026-10-24. portal-muse full build (`:muse:test :wake:test :app:testDebugUnitTest :app:assembleDebug :probe:assembleDebug`) = **BUILD SUCCESSFUL**, all tests green.
- **Probe (Phase 3) — day-one report all green on Gen 2:** RECORD_AUDIO granted, BLE adapter + advertising + GATT server + direct-GATT bind all OK (pairing path works), foreground service starts, TTS init OK, api.muse.ai resolves, 2.7 GB RAM. The separate Vosk-only live check still needs Sameer in the room saying "Hey Muse" (first attempt failed on a bad model push path — model files scattered instead of the named folder; fixed and verified; live retest pending).
- **Immortal provisioned (Phase 4)** with our config edits (`DISABLE_OTA=true`, `RESTORE_ALEXA=false`, Shizuku skipped — not in plan): Immortal installed; home = `com.immortal.launcher/.HomeActivity`; screensaver = `com.immortal.launcher/.PhotoDreamService`; `com.facebook.appverifier` + OTA packages disabled by the kit; stock launcher/screensaver snapshot saved on-device for restore. Verified via adb after the fact, not just the script's output. Undo: `provision.sh --restore`.
- **Debloat (Phase 5), `pm disable-user --user 0`** (undo per package: `pm enable <pkg>`): batch 1 — `com.facebook.alohaservices.presence`, `com.facebook.portal.aiservice`, `com.facebook.aloha.analytics`; batch 2 — `com.facebook.aloha.app.portalfeed`, `com.facebook.aloha.app.storytime`, `com.facebook.alohaapps.personaluser`, `com.facebook.alohaservices.alohausers`, `com.facebook.aloha.deviceidentity`, `com.facebook.alohaapps.bugreporter`, `com.facebook.aloha.cnshealthmonitor`, `com.facebook.aloha.platformmobileconfig`. (Kit had also disabled `com.facebook.alohaapps.devicesetup`.) Verified still enabled: WhatsApp, Messenger, Immortal, stock launcher (kept as recovery), Settings, TTS. Record also in `baseline/disabled-2026-10-24.txt`.
- **portal-muse installed (Phase 6, in progress):** `adb install -g app-debug.apk` = Success; Vosk model pre-pushed into the app's private files via `run-as` (verified) so first Hey Muse enable needs no download. **Waiting on Sameer:** paste SDK token on-device (gadgets.muse.ai), name the gadget, pair from the Muse phone app, enable Hey Muse.
- **Still open:** live Vosk/acceptance tests (WhatsApp call, Hey Muse question), Phase 7 router firewall (needs his router login), Phase 9 fork + TV control.

## 2026-10-21 (evening) — Features list vetted

- Sameer's wishlist (calendar, grocery, Google/Apple photos, calls, texts, TV, Muse questions, Wordle/crosswords, English+Urdu word of the day with audio, prayer times, high-level dashboard) researched against Immortal's live app-store catalog + F-Droid and written up in **FEATURES.md** — every item has a route; the only partial is iMessage/SMS texts (no Android client; Muse-read iMessages need an always-on newer Mac he declined). Notable finds in Immortal's catalog: Portal Calendar (Skylight-style, two-way Google sync), SmartTube (native ad-free YouTube), Portal TTS (system voice — Portals ship with none), Portal Overlays (floating HUD + ntfy banners), Forkyz + Lexica for word games.

## 2026-10-21 (evening) — Scope decisions from chat

- **Calling scope:** WhatsApp/Messenger calling apps are a keep-list in Phase 5 — debloat targets home/login/presence, not the calling stack.
- **FaceTime:** join-only via an iPhone-created FaceTime link in sideloaded Chrome; a standing link bookmarked on the Portal is the delivery answer (WhatsApp-the-link as backup). Experiment added to Phase 8.
- **Fork confirmed:** our portal-muse fork adds command handlers (launch apps, open the stored FaceTime link, TV commands) — upstream is conversation-only.
- **TV control:** Sameer picked **Portal-as-bridge** (Phase 9): the fork sends Samsung TV commands directly over the LAN. Mac bridge rejected as the daily path (the Mac lives off in a drawer); Home Link stays the cleaner alternative if it ever ships.

## 2026-10-21 — Plan repo created (no device yet)

- Portal bought (see 2026-10-21 entry below). Device in transit, ETA Oct 23–26.
- Researched and wrote PLAN.md / RESOURCES.md. Key finds of the day:
  - `wobsoriano/portal-muse` — hands-free "Hey Muse" app; built for Gen 1, probe-first on our Gen 2.
  - `wobsoriano/immortal-glass` — launcher + provisioning kit, explicitly supports API 29 (our model).
  - Bootloader unlock is hard-refused → no root; plan is disable + firewall only.

## 2026-10-21 — Bought the device

- eBay itm/168808167061, seller Apparel West (Orange, CA): open-box/unused WD50JM, $45.99 + free shipping, **$50 total** with tax. Power adapter included per listing; 30-day returns (buyer pays).
- Backstory: his $100 sealed-new OfferUp find was the ceiling; the $20 Ontario OfferUp WD50JM seller never replied; a $35 Chino listing turned out to be first gen (seller confirmed — its photo was a Gen 2 stock image). First gen was ruled out for the main build: minSdk 28 narrows app compatibility and it's the weaker fit all around.
