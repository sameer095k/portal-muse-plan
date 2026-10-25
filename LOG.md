# LOG

Newest first. Every change made on the device gets an entry: what, the exact
command, and its undo. Sessions where Sameer's hands are on the device and
Muse is remote eyes get dated headers like his TV project logs.

## 2026-10-25 (afternoon) — WhatsApp video FIXED: the AI camera services are on the camera path

- The sideloaded real WhatsApp (companion-linked by Sameer) placed calls, but
  his video was black on both his preview and the far side. Permissions were
  granted and the log showed the camera opening — then dying ~4 s in:
  `Camera3-OutputStream: Error queueing buffer to native window: No such
  device (-19)`, followed by CameraService disconnect. Not the shutter
  (he opened it; no change).
- Cause: the debloat had disabled `com.facebook.portal.aiservice` and
  `com.facebook.alohaservices.presence`, which sit on this firmware's camera
  path. **Fix: `pm enable` both → video works immediately** (his test call).
  Disabled count 12 → **10**. Undo: `pm disable-user` both (video calls in
  the sideloaded app break again).
- Open decision (his): keep both on for video, or trim back and lose video.
  The Phase 7 router firewall remains the network-side mitigation either way.

## 2026-10-25 (afternoon) — v2 dashboard BUILT: side rail + persistent view, Usuals, Photos, app-launch doors

Approved as proposed (his cuts held: no quantity, no store tags, no delete confirmations, no home automation). Shipped as **DashServer v6 + page v2**, all verified on-device:

- **Side rail** (Home / Prayer / Weather / Groceries / Photos) on the left of both twins. Selection persists **server-side** (`/api/ui-state` → `ui-state.json` on the device); the app POSTs on tap, both twins poll every 15 s. **Verified:** state set to Photos → device idled → the *screensaver itself* opened on the Photos slideshow (1 dream start, 0 stops, screenshot). No auto-home, ever.
- **Groceries two-pane:** live list (tap = green, ✕ = instant delete — add/delete round-trip proven on a scratch item, real list untouched) + **Usuals panel**. New endpoints `/api/usuals` + `/api/grocery/delete`; the Usuals sheet tab **auto-created and seeded** by the Portal itself on first read (Pantry/Apple cider vinegar, Personal care/Pads, Household/Metal spatula, Treats/Lindt roll — from his actual list; he/Abeer edit the tab in Sheets). Tiles dim when the item is already on the list; tapping a dim tile does nothing (no duplicates).
- **Photos:** local `photos/` folder; `/api/photos`, `/api/photos/upload-open` (10-minute window, returns the LAN URL `http://192.168.1.117:8321/photo-add`), and a multipart `/photo-add` page for the phone (403 when the window is closed). Upload proven end-to-end with a test image (shown in the slideshow, then removed). Slideshow advances every 30 s in the Photos view; undecodable files (HEIC) are skipped automatically. The iOS Shortcut recipe (Convert to JPEG → POST) is the intended phone path; the web page is the fallback.
- **Phone/Chrome bug — diagnosed and our side fixed.** Two stacked causes: (1) Immortal's home has **no real Chrome tile** — its "Browser" tile is Meta's Portal Browser; (2) every screensaver exit funnels to the Dashboard app, and Immortal's frame-hold (REDREAM → PhotoFramePreviewActivity) re-covers the screen after Meta's force-wakes, so other apps feel unreachable. Fix shipped in our code: the app twin loads with `?app=1` and MainActivity now intercepts `portal://launch/<pkg>` / `portal://apps` — the rail gained a **◎ button that launches the real org.chromium.chrome** (verified: Chrome is the resumed foreground activity after the tap) and a **▦ button that opens Immortal's app grid** from anywhere.
- **What still blocks Chrome visually (his tap, not ours):** the firmware puts Meta's "Share data to improve Portal Browser?" consent sheet in front of browser use (it's what the screenshot kept showing, identical both times). Leave the checkbox **unchecked**, tap Continue once — it never comes back. Chromium's own first-run is behind it (also his tap-through; pairs with the WhatsApp Web QR step).

## 2026-10-25 (midday+) — Later section removed from grocery view; Home HQ (joelgaff) reviewed as the grocery/rail reference

- His call: kill the Later section in the Portal grocery view. Done — the full-screen view is now a single "Groceries" list (Now items) + Add row, screenshot-verified on device. The sheet's column B still exists; the Portal ignores it. (One test tap during the change marked "metal spatula" green in the sheet; the Portal-side un-mark won't clear that one cell — flagged to him, fixable from the Sheets side if he wants.)
- Reference review (his X link): **Home HQ** by Joel Gaff — github.com/joelgaff/home-hq (MIT, Rails 8 + Hotwire + SQLite) — literally the productionized build of Wes Bos's Home HQ prototype (the same prototype behind his pinned photo-frame project). What he liked, concretely: (1) the grocery screen = list on the left + a **catalog side panel** of tappable item tiles (grouped by category, tap adds with a toast), items carrying a store tag + quantity stepper, filter by store; (2) the slim **left icon rail** for switching views. Full mapping of borrowable features vs our stack delivered to him as a report; build waits on his pick (design-first rule).

## 2026-10-25 (midday) — Google key LIVE: Portal reads/writes the grocery sheet directly, agenda live

- Key found where he dropped it: `/Users/Sameer/portal-*.json` on the Mac (his Finder home — reachable; `~/Downloads` is TCC-walled to SSH). His Mac has two home folders: `/Users/User` (SSH working home) and `/Users/Sameer` (his GUI home) — files for Muse go in either home folder.
- First load attempt hit `run-as: package not debuggable` (DashServer is a release-style raw-toolchain build). Fixed properly instead of making the app debuggable: **DashServer v5** adds a token-guarded `PUT /upload/sa-key.json` that writes into the app's *private* files dir. Key imported that way; the /sdcard temp copy was deleted in the same run.
- First calls returned Google's "API has not been used in project … or it is disabled" for both Sheets and Calendar (auth itself worked — Google accepted the SA token). He enabled both APIs in project portal-512718; retest went green immediately.
- **Verified end-to-end on-device:** `GET /api/grocery` returns the real sheet (Now: Lindt roll r1, Apple cider vinegar r10, Pads r11; Later: metal spatula). **Write proof:** toggled Pads bought → read back `bought:true` (the actual Sheet cell went green) → toggled off → white again. `GET /api/agenda` returns 200 (today genuinely empty; first real event will be the content proof of the calendar share). The grocery view on the Portal flipped to live mode (tappable rows + "Add an item" row) — screenshot-verified.
- SA identity: `portal@portal-512718.iam.gserviceaccount.com`; key lives ONLY in the app's private storage on the Portal + his spare copies (Mac home, phone). Never in chat, this repo, or memory.
- WhatsApp Web thread (his find, people online are right that it's the chat route): real Chromium (`org.chromium.chrome`) is installed and is the default browser — WhatsApp Web's requirement. It sits at Chrome's **first-run wizard** (never completed). Remaining gates are his hands: tap through first-run (no sign-in), then scan the WhatsApp QR with his phone. Window-stacking on this firmware made remote screencaps unreliable during the attempt (dashboard kept compositing over Chrome) — finish it in the room.

## 2026-10-25 (late morning) — Calls-tile glitch loop: diagnosed, unwedged, dream stable again

- His report: tapping Immortal's phone (Calls) tile opened nothing and started a flicker loop between the stock photo screensaver and our dashboard. Root cause, from Immortal's own source + device logs: the Calls tile *bridges to Meta's stock home* (launchStockHome → markBridge); on this firmware the stock home process starts but never surfaces a window — and its background process then churns dream state (dream start → system wake ~0.6 s → restart, endlessly; the composite screenshot showed Immortal's clock-face frame layered over our dashboard). Meta's Superframe also still registers a DreamService, which muddies the water further.
- Fixes applied: re-enabled `alohaapps.personaluser`, `alohaservices.alohausers`, `aloha.deviceidentity` (user/contacts identity the stock home needs — calling wins over debloat; disabled count 15 → 12), then **force-stopped the wedged stock-launcher process**. Controlled retest: screen timeout 15 s, 40 s window — **dream starts 1, stops 0**; the dashboard dream is stable again (screenshot-verified).
- **Still open (his call):** the Calls tile itself remains a dead end on this firmware — the handoff doesn't complete. Incoming WhatsApp/Messenger calls still ring the Portal (wrapper-level, home-app-independent). Outgoing calls need either the stock home as default (big trade) or the Phase 9 voice fork. Recommendation on the table: hide the Calls tile so it can't re-trigger the wedge.
- Key file: he has the SA key on his iPhone ("saved to download", then "moved") — not yet located on the Mac (Downloads is TCC-walled to SSH; the home-folder drag didn't surface it). Location TBD with him.

## 2026-10-25 (midday+) — Google client built and staged; waiting on his service-account key

- His call: a scoped Google key on the Portal is fine. Route: service account shared only the grocery sheet (Editor) + his calendar (See all event details). Steps given to him (Cloud console: project → enable Sheets + Calendar APIs → service account → JSON key on the Mac → share both) — key lands via cable into the app's private files, never chat.
- **DashServer v4 built + installed**: GoogleClient (SA JWT→OAuth token, RS256 via java.security, org.json — no new deps) + PortalApi behind /api/grocery (read values + cell fills; green = bought, and the write color is *calibrated from the first green cell the read finds*, matching his sheet marks), /api/grocery/toggle (batchUpdate fill), /api/grocery/add (values append), /api/agenda (calendarList + today's events, same shape as the snapshot agenda). No key on device → all endpoints 501 (verified live via adb forward).
- **Page**: Groceries card now opens a full-screen grocery view (Now + Later, big tappable rows; in live mode a tap greens/ungreens the sheet cell and an Add row appears). Verified in snapshot-fallback mode on-device. The agenda also goes live from the key automatically (loadLiveAgenda every 5 min) once the key exists.
- Sidebar findings (his question): WhatsApp/Messenger on Portal OS are `com.facebook.aloha.app.whatsapp` / `.messenger` — system services with NO launcher activity; installed + enabled (debloat-safe); his number linked at setup; usage is the Immortal Calls tile; relink lives in Portal Settings → Accounts. FaceTime remains join-only via an iPhone-created link in the Browser (not yet created).

## 2026-10-25 (midday) — Next-prayer block killed; weather + prayer are now full-screen tap views

- **Next-prayer section removed** (his call: redundant with the highlighted card). The countdown logic still runs internally (card highlight + azan firing); it just isn't displayed on the home page anymore.
- **Tap weather → full-screen weather view** (in-page, fed by the same Open-Meteo pull, extended to hourly + 8-day): current conditions, a 12-hour strip, and the week (verified on-device: Sat shows rain). **Tap any prayer card → full-screen prayer view**: big "Dhuhr 12:36 PM · in 1h 37m" header, all six prayers at reading size with IOK/CHESS iqamah, next one outlined; rows tightened so all six fit without scrolling. ✕ or a backdrop tap closes either view. Both verified by remote taps + screencaps.
- **Grocery app: parked on his thinking** — he saw a design on Twitter he liked; the dashboard card stays read-only from the sheet snapshot meanwhile. When he surfaces the post, match its design; a real app version needs the write path back into the sheet (the read path exists via the snapshot).

## 2026-10-25 (later) — Word source switched to Transparent Language; seconds killed; tabular figures

- **Word of the day now comes from Transparent Language** (his link): the daily job reads their RSS feeds (FeedBlitz: urdu-word-of-the-day + english-word-of-the-day-for-spanish). Quirks found: feed UA matters (browser UA → subscribe page, plain UA → XML); their English feed is for Spanish speakers, so the English slot takes the word + its example sentence as the sub-line. The feeds carry no audio, so clips are still generated (EN local TTS; UR Google TTS speaking word ×2 + the feed's Urdu example sentence). Transliteration comes from the feed's wotd field when present (today: none — نیپکن is a loanword). Fallback: the local 20-pair bank if a feed ever fails — never wordless. First Transparent pair on-device: **leash / نیپکن (napkin)**.
- **Seconds killed** (his ask): clock shows HH:MM + AM/PM; countdown shows "in 1h 43m". **`font-variant-numeric: tabular-nums`** applied page-wide (the technique he was reaching for — digits keep fixed width and update in place without jitter). Also shrank the next-prayer line to 40px so "Dhuhr 12:36 PM" stops kissing the card edge.

## 2026-10-25 — Dashboard feedback fixes + it is now interactive

Sameer's morning notes (photo of the day theme — auto light/dark works): (1) grocery **Later list killed** (Now only); (2) **"Iqamah" relabeled "IOK"** (CHESS line unchanged beneath); (3) **Isha card was clipped** at the right edge — layout rebalanced (370px left column, tighter gaps) and verified fully on-screen; (4) date now always from the device clock (was the snapshot's, a day stale); (5) next-prayer line no longer wraps "PM".

- **Interactivity (his big one):** taps inside the screensaver can never reach the page — a tap dismisses the Android dream by design, which is why the Urdu button "went home". Fix: DashServer **v3** — its launcher activity ("Dashboard") is now a **full-screen WebView** of the same page (JS on, autoplay allowed, keep-screen-on), and Immortal's Screensaver setting **"Open when you tap to exit" → Dashboard**. Flow verified end-to-end by remote taps + device logs: idle → dream shows the dashboard; **tap → the interactive Dashboard opens**; **play button → the Urdu clip actually played** (audio HAL start_output_stream at the tap instant). The dream stays the glanceable twin; the app is the tappable twin — same files, same server. Upload allowlist also gained /index.html so page fixes can ride the push pipe.
- Pushed today's fresh snapshot by cable while ADB was still up: the word had already rotated at 05:48 (serene / سکون sukoon) with new audio — delivery was the only missing piece, exactly what the Tailscale pipe closes.

## 2026-10-24 (midnight) — Dashboard LIVE on the Portal, fully local; Tailscale push prepped

- **Immortal's web-page field rejects `file://`** (verified on screen: "Start the address with http:// or https://", button stays greyed). Muse drove the Portal remotely via adb input + screencaps (also unstuck Sameer from a settings page — Portal has no visible Back in Immortal settings; KEYCODE_BACK works).
- **DashServer built** (source `~/workspace/dashserver-src/`, built on the Mac with javac/d8/aapt2 — no Gradle): `com.sameer.dashserver`, a foreground service serving `/sdcard/portal-dashboard` on port **8321, all interfaces** (localhost for Immortal + tailnet for pushes), autostart on boot, wake-lock held. Immortal's Photo source → Web page set to **http://127.0.0.1:8321/index.html** (selected + saved, verified on screen). The screensaver engaged and rendered the dashboard on-device — screenshot verified (clock/countdown/prayer/groceries/word band all live).
- **Screen timeout note:** temporarily set to 15 s to trigger the screensaver test, restored to 600000 (10 min); the original value was lost to a garbled command — disclosed to Sameer.
- **Push endpoint (v2):** `PUT /upload/data.json|audio/wotd-en.mp3|audio/wotd-ur.mp3`, Bearer-token guarded (token baked into the APK; copy in `~/workspace/portal-dashboard/.dash-token`, chmod 600). Verified: 401 no token, 403 wrong path, 200 + byte-exact file for good pushes. Two build bugs found and fixed on the way: missing WRITE_EXTERNAL_STORAGE (writes silently failed) and 100-continue handling.
- **Direction of the pipe (his pick: Tailscale, setup tomorrow with him):** Muse's VM is a tailnet *client only* (accepts no inbound), so the snapshot flows as a PUSH from the VM to the Portal. Prepped tonight: `push_snapshot.py` (rebuilds data.json, PUTs the 3 files via the tailnet proxy) + cron `portal-dashboard-snapshot-push` every 30 min — silent no-op until `portal-push.json` exists. **Tomorrow:** install Tailscale on the Portal (his sign-in), `tailscale up` the VM (his approval), approve the Portal address, write `portal-push.json`, watch one push land.
- **Leftover:** Muse Probe app still on the Immortal home screen (his to delete).

## 2026-10-24 (late) — Pairing fixed, Hey Muse live, dashboard v1 built

- **Pairing timeout root-caused to an upstream bug:** `GattPeripheral.kt:85` required `SDK_INT == Build.VERSION_CODES.P` (exactly Android 9) — Gen 2 runs API 29, so pairing refused to start and the UI called it a timeout. **Our fork's first patch:** `!=` → `<` (at least Android 9). Rebuilt on the Mac, reinstalled with `adb install -r -g` (token + name survived). Pairing then succeeded first try — logs: "Noise session established", "registered with Muse" 22:52; `muse.json` present in app files. Device LAN IP **192.168.1.117** (TV is .118).
- **Hey Muse enabled by Sameer; wake listener live.** Logcat shows the wake state machine running and two near-misses: "hey" scored 1.0, "muse" 0.78 vs the 0.85 trigger threshold. If it stays stubborn, the threshold is a constant in the wake module — lowering toward ~0.75 is a known lever (cost: more false wakes from TV audio). His call.
- **Dashboard v1 built** (`~/workspace/portal-dashboard/`): one page at 1280×800 — clock/weather, next-prayer countdown, six prayer cards (IOK azan + iqamah, CHESS iqamah), Today agenda, Groceries (shared sheet snapshot), word-of-the-day band (EN + Urdu Nastaliq + transliteration + audio buttons). Auto light (Warm Paper, 06:00–19:00) / dark theme. Verified by rendering at exact size in headless Chrome on the Mac (two passes; fixed a clock wrap and a word-band overflow).
  - **Data architecture (his local-first steer):** the page fetches prayer (isalaah IOK+CHESS) and weather (Open-Meteo) **itself** — both endpoints verified to send `Access-Control-Allow-Origin: *`, so it works even served from a local file. `data.json` (built by `build_data.py`) carries only groceries + agenda + word of the day, and is the prayer/weather fallback.
  - **Urdu audio solved:** Muse's speech engine returns empty audio for Urdu; Google Translate's TTS endpoint produces a valid MP3 (39 KB for day 1) — `daily_word.py` uses it for the Urdu clip and the local engine for English. Day 1 pair: luminous / روشن (roshan).
  - **Daily job:** cron `portal-dashboard-daily-word` (~05:48 PT) rotates a 20-pair bank (`words.json`), regenerates both clips, rebuilds data.json; silent unless it fails.
  - **Pushed to the device:** `adb push` → `/sdcard/portal-dashboard/` (index.html, data.json, word.json, audio/wotd-en.mp3, audio/wotd-ur.mp3 — verified on-device). Immortal's web view sets `allowFileAccess` + `allowFileAccessFromFileURLs` (checked `FlipWebClockFaceView.kt`), so the web clock face / screensaver can load `file:///sdcard/portal-dashboard/index.html` — **Sameer sets that URL on-device**; he is exploring fully-local hosting.
  - **Azan:** page is wired to play `azan.mp3` at each IOK azan time (with a card flash); no file yet — Sameer picks the recording.
- **Still open:** WhatsApp call test (acceptance), router firewall (Phase 7, needs his router login; target 192.168.1.117 + a DHCP reservation), Phase 9 fork handlers + TV control.

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
