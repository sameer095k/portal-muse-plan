# LOG

Newest first. Every change made on the device gets an entry: what, the exact
command, and its undo. Sessions where Sameer's hands are on the device and
Muse is remote eyes get dated headers like his TV project logs.

## 2026-10-26 (afternoon) — Built-in WhatsApp signed out; the real app is the only Portal WhatsApp now

- Sameer saw two linked devices on his phone's WhatsApp and logged the **built-in** Portal client (linked during the Oct 24 takeover) out from iPhone → WhatsApp → Settings → Linked Devices. To tell them apart, Muse opened the sideloaded app on the Portal first so its entry showed active-now (the keeper); he removed the stale one.
- The sideloaded app's companion link (Oct 25) is untouched and remains the Portal's WhatsApp. Portal-native WhatsApp calling via the Calls tile goes with the built-in account — already a dead end, no loss.
- His stated preference, recorded: no historical-message sync needed on the Portal app (new messages only); full history stays on his phone. Nothing to configure — a companion only ever holds a bounded recent window from its link, and unlinking the built-in device never touched the phone's archive.
- **Follow-up the same afternoon:** he kept getting WhatsApp's "Syncing paused" notification on the Portal — the app's standard warning when the OS pauses its background sync (no Play Services on this device, so WhatsApp holds its own connection). The app was already battery-whitelisted from install day; added the explicit background-run allowances (`appops set com.whatsapp RUN_IN_BACKGROUND allow` + RUN_ANY_IN_BACKGROUND) — the same class of fix as the Hey Muse mic allowances. At check time no sync notification was posted (it had cleared when the app last came to the front). If it still nags, the remaining lever is muting the app's "Other notifications" channel; sync on a GMS-free device will never be phone-grade regardless.

## 2026-10-26 (afternoon) — Photo rotation: split settings across faces fixed (DashServer 6.10)

- **Sameer's report:** rotation ticked every ~10 s despite his 1-minute setting, flashing "No photos yet" and landing back on the same photo.
- **Diagnosis (measured, not guessed):** the library and server are healthy — 62 photos (Main 36, Us 26), `/api/photos` payload byte-identical across calls. The app face's settings sheet confirmed his **1 min** was correctly stored — in the APP's localStorage. But the dashboard's two faces (DashServer app vs Immortal screensaver) are different apps with **separate localStorage**, and rotate speed + pin lived in localStorage — so the screensaver face ran its own stale speed and never saw his setting. Second bug, in code: `#phempty` ("No photos yet") was `display:flex` **by default**, only hidden once photos loaded — every face (re)load flashed it before the first fetch landed, and a reloaded face restarts at photo 0 ("just shows the old photo"). The screensaver's own stored value couldn't be read directly (its surface doesn't screencap — black frames), but the split-storage flaw is architectural and sufficient.
- **Fix — DashServer 6.10 (vc 20):** ui-state POST now also merges `rotate` (validated against the chip values) and `pin` (string, or explicit JSON null to clear) alongside view/rail/album.
- **Fix — page:** photo settings post to ui-state on every change (either face); `syncViewState` adopts the server's rotate/pin (20 s guard against racing a just-made local change); the app face seeds the server once from its localStorage if the server has none; localStorage demoted to seed/fallback. `#phempty` is `display:none` by default — it can now only appear when the library is genuinely empty after a load.
- **Verified:** ui-state round-trip via curl (rotate set, pin set, pin cleared to null, other keys untouched); canonical state now `rotate:60000, pin:null`; page deployed (served md5 `37cd2fd2…`), cold start restored the Photos view with a photo up and no empty-state flash. Commits: page `e292d2f4`, APK `f859ef17` (archived `apks/dashserver-6.10.apk`).
- **Diagnosis side effects, disclosed:** the Portal was put to sleep twice and woken again while testing dream triggers; left awake on the Photos view.
- **Undo:** page — restore the `8af8b48d` index.html; server — reinstall `apks/dashserver-6.9.apk` (ui-state just stops accepting the new keys; stored values are inert).

## 2026-10-26 (afternoon) — Word band: audio fixed (6.8), then sentences-only redesign (6.9)

- **Audio was never working — owned.** Sameer: the Urdu word audio "doesn't play anymore." Diagnosis: the Portal's TTS engine (Pico, com.facebook.aloha.app.ttsservice) has no Urdu voice; DashServer ≤6.7's synthesizeToFile silently wrote a 6,188-byte, 0.128 s, pure-silence WAV stub and stamped it good. The Oct 25 "verified real clip" was that same stub — that check verified existence/size only, never content. **DashServer 6.8 (vc 18):** `kickAudio` is cloud-first — fetches the spoken line (word ×2 + example) from Google Translate TTS (`translate_tts`, tl=ur, ≤180-char chunks, MP3 frames concatenated) on a worker thread; a clip is stamped good only if it's a real MP3 (ID3/frame-sync header AND ≥8,000 bytes). Pico kept as fallback, and its finalize now rejects ≤8,000-byte stubs instead of stamping them. Proven end-to-end: clip deleted on-device → regenerated 44,928 B / 5.6 s / mean −23.2 dB, log line "word audio fetched (cloud Urdu TTS)".
- **Sentence made visible (page-only, commit `2c403ee1`):** the example sentence was only ever audible; the band now drew it under the word. Sameer: it made the band taller while the left half sat blank — redesign instead.
- **Redesign — his pick from mocks (v1 two-column, v2 sentences-only; he chose v2):** the standalone word is GONE from the band. The band is now just the sentence pair: English translation top-left, Urdu sentence bottom-right, the word of the day highlighted gold in both (Urdu matching is stem-based — غصہ highlights as غصے), play button on the right. Band back to ~original height. If a day ever has no example sentence, the band falls back to the old word + meaning block.
- **DashServer 6.9 (vc 19)** supplies the new field: `/api/word` gains `example_meaning` (the feed's "Sentence meaning", parsed up to the trailing URL). Page renders it via `highlightEnglish` (whole-word match of the primary meaning) + `highlightUrdu` (stem match). Deployed: APK `install -r` over wireless ADB, word cache busted so the day's payload re-parsed, page PUT (served md5 `07612796…`), dashboard cold-started.
- **Verified on-device (screenshot `portal-dashboard/word-band-v2-home.png`):** "Don't get carried away by **anger**." top-left, Urdu sentence bottom-right with **غصے** in gold, audio clip untouched (44,928 B). Page commit `8af8b48d`, APK archived `apks/dashserver-6.9.apk` (commit `fbfd67bd`).
- **Undo:** page — restore the `2c403ee1` index.html via the PUT route; server — reinstall `apks/dashserver-6.8.apk` (payload just loses the extra field).

## 2026-10-26 (morning) — AirPlay added: Wormhole Display installed (receiver live, iPhone test pending)

- **Sameer's call:** "Remember we were gonna try to add airplay" → "Install Wormhole Display now." This is the RESOURCES.md find (pgodlews/wormhole-display, an Android port of UxPlay) finally tried — it makes the Portal an AirPlay receiver for iPhone/iPad/Mac: Screen Mirroring with sound, plus audio-only AirPlay speaker.
- **Fit check first:** our regular Gen 2 (WD50JM) is NOT in the app's tested table (Portal+ Gen 2 / Mini / Go / TV), but the device reports `arm64-v8a` in its ABI list and the APK is arm64-only, targetSdk 29 — install took cleanly, no refusal. Release APK v0.3.0 (23,944,215 B) downloaded from GitHub Releases; sha256 matched the release's own checksum file (`beb18923…`). Local copy archived at portal-dashboard/apks/wormhole-display-0.3.0.apk.
- **Install:** `adb install -r` over wireless ADB (Tailscale forwarder) — Success; package `io.github.pgodlews.wormhole`, firstInstallTime 2026-10-26 10:02.
- **Setup done from here:** launched the dashboard once (status "Ready to Connect"), granted Auto-open via `appops set io.github.pgodlews.wormhole SYSTEM_ALERT_WINDOW allow` — the dashboard now reads Auto-open **Enabled**. Defaults left as shipped: Run in Background ON, Start on Boot ON, Speaker Output ON, orientation Landscape. Advertised Bonjour name: **"Wormhole Display"**. Receiver process is running in the background with the AirPlay port (7000) listening; dashboard returned to the front afterward — receiving doesn't need the app open.
- **Pending Sameer's iPhone:** Control Center → Screen Mirroring → "Wormhole Display" — first real mirror (and a FaceTime-call mirror, the Phase 8 fallback) is his test; discovery from this side can't be proven without a sender. DRM'd apps (Netflix etc.) won't mirror — that's Apple's restriction, expected.
- **Undo:** `adb uninstall io.github.pgodlews.wormhole` — ordinary app, nothing else touched.

## 2026-10-26 (morning) — Show page: pushed content gets a full dashboard view (page-only)

- **Sameer's ask:** the pancake card worked, but the recipe had to scroll inside the little pop-up — "make it like a page instead of just a pop up, so you have more real estate."
- **What:** the pop-up overlay is gone; pushed content is now a proper dashboard view (`shview`, in the view map alongside Prayer/Weather/Groceries). Header: title + subtitle, gold Open (when a URL rides along), Done ✕. Sections render as full-height side-by-side columns (first column narrower — Ingredients beside Steps); the thumbnail is a wide banner under the header for rich content and a large hero for link-only content. Portrait stacks the columns. **Page-only change — no DashServer/APK change:** the /api/show payload and endpoints are untouched.
- **Behaviour carried over, adapted:** a NEW card still switches both faces to the Show page (the screensaver face included — a ui-state sync guard keeps a stale posted view from pulling the dream face off Show while a card stands). Open hands the link to Chrome via portal://open and the page simply stays underneath, so returning lands on it again — the v2 opened-id/reshow machinery is retired. Done ✕ clears server-side and returns Home. If he navigates away mid-card, a rail Show button (play-screen glyph) is visible only while a card stands and brings the page back; it disappears when the card is cleared.
- **Deploy:** page PUT via the Tailscale route; served md5 == local (`ccc5010b…`). Dashboard cold-started once over wireless ADB (page updates land on a cold start).
- **Verified on-device (adb taps, screenshots kept):** the standing pancake card rendered full-page after cold start — all 8 ingredients + all 6 steps on screen at once, zero scrolling. Tap Open → Chrome resumed; card still held server-side (same id); returning to the dashboard landed back on the Show page. Tap Done ✕ → `/api/show` = {"id":null}, back on Home, rail Show button gone. Re-posted the pancake recipe — the dashboard auto-switched from Home to the Show page on the new card, which is the state left standing for Sameer.
- **Undo:** restore the previous index.html (the 6.7 page) via the same PUT route; server and payloads are unchanged, so v2 cards render exactly as before.

## 2026-10-25 (late night) — Show cards v2: Open can't lose the card + recipes on the card (DashServer 6.7)

- **Why:** Sameer's first live tap on the pancake card: Open cleared the card server-side in the same tap whose Chrome launch got swallowed (dream-exit transition) — card gone, nothing opened. His ask on top: "more info than just the video link, like the recipe listed steps ingredients and then link if i wanted."
- **Fix A — Open no longer consumes the card.** Page: Open hides the overlay locally and remembers the opened id for the page session (the 5s poll won't instantly re-show it); only ✕ clears (POST /api/show/clear), and a new card replaces the old. A `visibilitychange` handler re-shows an opened-but-not-dismissed card the moment the dashboard becomes visible again (returning from Chrome/screensaver), and a plain cold start re-shows it too. MainActivity: the portal://open launch is now posted to the view's queue instead of fired synchronously inside shouldOverrideUrlLoading, and the fallback chain is Chrome → any browser → chooser, all FLAG_ACTIVITY_NEW_TASK, in a new openExternal() helper.
- **Fix B — rich cards.** Payload v2 adds optional `sections` ([{heading, lines}], sanitized server-side: ≤6 sections, ≤40 lines, ≤300 chars/line). With sections the page renders a taller scrollable sheet: smaller thumbnail, title, sections (touch scroll), Open pinned at the bottom, ✕ pinned top-right. `url` is optional when sections are present (content-only cards); old payloads render exactly as v1. CLI: `show_on_portal.py --sections <file>` ("## " starts a section), --url optional then.
- **Build/deploy:** DashServer 6.7 (versionCode 17), VM-only raw toolchain (JDK 21 + build-tools 35, debug keystore; `adb install -r` over wireless ADB — install success proves the cert matched), page PUT over Tailscale, served md5 == local (`33bbe198…`). One honest note: DashServer's own version bump needed no data migration; show.json format is backward compatible (v1 cards still parse).
- **Verified on-device:** token 401 / bad-card 400 / sections-only 200 / empty-sections 400; warm face — Open (2nd tap; the 1st bounced off a mid-reload page, a known WebView rawness, card unharmed) → Chrome resumed playing the video, GET /api/show still held the card; cold start re-showed it; ✕ cleared it. Rich recipe card (Ingredients ×8, Steps ×5) rendered + scrolled with title/Open pinned. Required scenario: card up → SLEEP into the Immortal dream (card visible on the dream face) → tap exits dream to the app face → Open → Chrome resumed, card STILL held; back → card standing; ✕ → cleared, final GET {"id":null}, final screenshot clean. No card left up.
- **Undo:** reinstall the 6.6 APK (portal-dashboard/apks/dashserver-6.6.apk); page changes are additive — the v1 page ignores `sections`.

## 2026-10-25 (late night) — Screen pop-up: Muse can throw a card on the dashboard (DashServer 6.6)

- **Sameer's go:** asked for a pancake recipe by voice, got voice-only back, and approved "Build the screen pop-up" — something visual should land on the screen, not just be spoken.
- **What:** DashServer 6.6 (versionCode 16) adds `POST /api/show` (Bearer-token-gated like the uploads; body {title, subtitle?, url, image?} → stored as show.json in the served dir, server-stamped id/created, each POST replaces the card), `GET /api/show` (current card or {"id":null}), and `POST /api/show/clear` (page-side, no token — same trust as the other local /api calls). MainActivity gains a `portal://open?url=<urlencoded>` intercept → ACTION_VIEW targeted at Chrome (org.chromium.chrome), default browser as fallback. The page polls /api/show every 5 s, tracks the card id, and renders a centered overlay (16:9 thumbnail, title, subtitle, gold Open button, ✕) in the existing theme vars, in BOTH faces — visible on the screensaver, tappable in the app; Open and ✕ both clear server-side, so a dismissed card never returns, and a card still up after a page reload shows again.
- **CLI:** `show_on_portal.py` in portal-dashboard (committed) — `--title/--url/--subtitle/--image` posts via the Tailscale proxy route reading portal-push.json + .dash-token; `--clear` clears.
- **Build route:** VM-only (Sameer unplugged the MacBook): JDK 21 + SDK build-tools 35 on the VM (javac → D8 → aapt2 → zipalign → jarsigner), same Android debug cert (SHA1 9F:10:DE…B5:FA:CA:3A) as every DashServer build. Installed with `adb install -r` over wireless ADB; MainActivity relaunched to bring port 8321 back. Page deployed by the Tailscale PUT route; served md5 == local md5 (`f6332b56…`).
- **Verified on-device:** /api/prayer + /api/grocery still 200; POST /api/show without the token → 401, bad payload → 400. End-to-end with a test card (a real YouTube pancake video + its hqdefault thumbnail): card rendered over the dashboard (screenshot), adb tap on Open → Chrome resumed on the exact watch URL and the card auto-cleared (GET /api/show = {"id":null}); back on the dashboard, final screenshot clean — no test card left up. One wrinkle: a warm `am start` resume did NOT pick up the newly deployed page (the WebView kept its old copy); a force-stop + relaunch did — page updates land on a cold start of the app/dream.
- **Undo:** reinstall the 6.5 APK and/or restore the previous index.html; the endpoints are additive and no existing behavior changed.

## 2026-10-25 (night) — Hey Muse permanent fix: mic priming (built, installed, verified)

- **Sameer's go:** "Build the permanent fix", after the diagnosis in the entry below.
- **The specified patch alone did NOT fix it.** v1 = `android:foregroundServiceType="microphone"` on MuseService (manifest) + 3-arg `startForeground(..., FOREGROUND_SERVICE_TYPE_MICROPHONE)` on API 29+. The type verifiably lands in the merged manifest (aapt2: `foregroundServiceType=0x00000080`), but on this Android 10 build the audio policy ignores it for background record creation: with the dashboard in front, post-reopen records still read `silenced:true` and `mic peak 0 over 60 s`. v2 = wake record source VOICE_RECOGNITION -> MIC: identical result (peak 0 after reopen). The policy keys on the app having a resumed activity when the record is **opened**, source-independent — the same fact the diagnosis A/B showed (fronting the app un-silences; a record opened in front stays audible in the background).
- **The fix that works (v4, installed): mic priming.** Before `WakeListener` opens the mic while the app is in the background, it launches the new invisible `MicPrimeActivity` (translucent, own task, `noHistory`, draws nothing), waits for it to resume, opens the record under it, then finishes it — the record is created "in front" and stays audible after the dashboard returns. A 10 s settle window after each open keeps silence-reopen recycling from killing the fresh record: in a quiet room, sparse room ticks pin upstream's reopen limit at its 2 s floor, which without the settle window re-primed and re-opened the mic every ~2.3 s; cadence is now ~12 s. The FGS-type declaration stays (correct hygiene). `FOREGROUND_SERVICE_MICROPHONE` permission NOT added — not required at targetSdk 28 / API 29.
- **Decisive verification (dashboard resumed the whole time, no app-opening):** MuseWake `mic peak` per minute **29943 / 5686 / 18974** across three consecutive quiet minutes (pre-fix: flat 0 after the first reopen). Three settled `dumpsys audio` samples: the wake record reads `silenced:false` with the dashboard in front. (An instantaneous sample can still catch `silenced:true` in the first moment after a recycle; the peak log is the ground truth and it never flatlined.) Dashboard screenshot afterwards: renders normally, no residue from priming. App still shows "Online" — pairing intact.
- **Signing gate (hard rule kept):** installed signer = Android Debug cert, SHA-256 `f7c85f3d144e35562ca72f391d15df5d38c7b97bf773c553c1b146e096723818` = the Mac's `~/.android/debug.keystore` (same key as DashServer). Every build signed with it, every install `adb install -r` — never an uninstall. Data-preservation proof: `firstInstallTime=2026-10-24 22:34:34` unchanged across v1-v4; final `lastUpdateTime=2026-10-25 22:11:20`.
- **Build host note:** the plan was to build on Muse's VM. JDK 21 (Temurin 21.0.13) + SDK platform 35 / build-tools 35 were installed there (`~/toolchain/`) and the harness works (apksigner/aapt2 used for verification), but Gradle itself still can't build in the VM sandbox: the wrapper's JDK downloader dies on the egress proxy's CONNECT (`SSLException: Unsupported or unrecognized SSL message` — the same JDK tunneling quirk that breaks sdkmanager), and with a curl-fetched Gradle 9.8.1 the client-daemon dispatch gets `Connection reset by peer` writing to the daemon's loopback port (a raw Java loopback echo test in the sandbox even returned a wrong byte — arbitrary loopback is intercepted). Same wall recorded 2026-10-24. So the patched source was compiled on the Mac (its JDK 21 + SDK 35; deps cached; ~10-40 s builds) and each APK pulled back to the VM and installed over wireless ADB. Source of record: `~/workspace/portal-muse-fork/` (commit `3f18222` on upstream `f2113da`, includes the 2026-10-24 GattPeripheral pairing fix the installed build already carried); the Mac clone `~/portal-muse` holds the identical tree. The Mac debug keystore was copied to the VM (`~/workspace/portal-muse-fork/signing/`, gitignored, never committed) so a future VM-side signer matches too.
- **Rollback:** the originally installed APK is backed up at `~/workspace/portal-muse-fork/backup/museportal-installed-v0.1.0.apk` (md5 `d239f11536ad527d927f1a9661dc512c`); undo = `adb install -r` that file (same signature, data kept). Final patched APK kept at `~/workspace/portal-muse-fork/app-debug-patched-v4.apk`.
- **Left for Sameer:** one live voice check — dashboard in front, a couple of quiet minutes, say "Hey Muse", no app-opening first. Caveats: priming starts an invisible activity whenever the mic recycles in the background (~12 s cadence in a silent room, also at listener start / after calls / after asks); behavior while the Portal is in its screensaver/dream state is untested (verification ran with the dashboard activity resumed, the normal evening state). Wireless ADB still on; still expected to need one USB `adb tcpip 5555` after a Portal reboot.

## 2026-10-25 (late night) — Hey Muse background wake diagnosed; wireless ADB enabled

- **His report:** "Hey muse activation from dashboard doesnt work" —
  then the A/B that cracked it: it "basically only works when i click
  into the muse app."
- **What was actually wrong (evidence, not vibes):** MuseService was
  never dead — `dumpsys activity services` showed it foreground with
  its notification the whole session, process alive since the
  takeover. The failure is in the audio layer: `dumpsys audio`
  showed the wake listener's record (VOICE_RECOGNITION, session
  10097) as **`silenced:true`** whenever the Muse app wasn't the
  resumed app, and `MuseWake` logged the loop for hours (from
  18:53): "mic peak 0 over 60 s" → "the mic has read exact silence
  for 64 s; reopening it" → Listening, forever. Bringing MainActivity
  to the front flipped the *same* record to `silenced:false` within
  seconds; returning to the dashboard left it heard — until the next
  silence-triggered reopen, whose new record started `silenced:true`
  again (proven twice, riids 10351/10367). Root cause: MuseService
  declares **no `foregroundServiceType="microphone"`** (manifest
  service element is name+exported only; targetSdk 28) and calls the
  2-arg `startForeground`. On this Android 10-based Portal OS a
  generic foreground service does not qualify for background mic —
  records created while another app is in front start silenced, so
  the app's reopen-in-the-background workaround (WakeListener.kt's
  own comment expects a reopened mic to be heard) produces another
  silenced record on this build. A quiet room (>64 s of digital
  silence) is enough to fall into the trap; from then on it can
  never hear anything until the app is fronted once.
- **Ruled out / no-ops:** battery and standby — the service never
  died, `am get-inactive` was already false. Added anyway as
  prevention, all reversible: `cmd deviceidle whitelist
  +dev.wobsoriano.museportal`, `cmd appops set dev.wobsoriano.museportal
  RUN_IN_BACKGROUND allow` and `RUN_ANY_IN_BACKGROUND allow` (undo:
  same commands with `-` / `default`). Proven not to affect
  silencing: the record created right after still started silenced.
  UID `RECORD_AUDIO` appop was already `allow`; re-setting it
  changed nothing.
- **Durable fix (not built):** fork patch — manifest
  `android:foregroundServiceType="microphone"` on MuseService +
  the 3-arg `startForeground(id, notification,
  FOREGROUND_SERVICE_TYPE_MICROPHONE)` on API 29+. Small code
  change, but the toolchain exists on neither machine right now:
  the Mac has JDK 8/11 only and an old SDK; the VM has no Java at
  all. Project notes call for JDK 21 + Android SDK 35 — that's a
  setup step before any build; not started.
- **Interim state left on the device:** one app-front cycle
  (MainActivity front ~6 s, then DashServer front) so the current
  record (session 10145) sits `silenced:false` with the dashboard in
  front and MuseService foreground; `logcat -c` run last so the
  A/B window is clean. This lasts until the next silence-triggered
  reopen — if a wake attempt fails, fronting the Muse app once and
  returning to the dashboard re-arms it the same way.
- **Wireless ADB enabled (Sameer approved: "Yes, leave wireless ADB
  on"):** over USB, `/Users/User/platform-tools/adb tcpip 5555`
  (bare `adb` is not on the Mac's SSH PATH — full path always).
  Verified from the Mac: `adb connect 192.168.1.117:5555` (LAN) and
  `adb connect 100.115.140.112:5555` (tailnet) both list the device
  and answer `shell echo ok`. MuseService state re-verified after
  the adbd restart (foreground; see the reopen note above).
  **VM side (Mac-free route):** official platform-tools at
  `~/workspace/platform-tools/` + forwarder
  `~/workspace/tools/adb-tailscale-forward.py` (listens on
  127.0.0.1:15555, CONNECTs to the Portal's tailnet address :5555
  through the runtime tunnel proxy on port 3130; start it with
  nohup before VM adb use, then `adb connect 127.0.0.1:15555`).
  First VM connect reached the Portal and the device shows
  `unauthorized` — the VM's RSA key still needs Sameer's "Allow"
  tap on the Portal (with "always allow"). **Persistence:**
  `service.adb.tcp.port` reads 5555, `persist.adb.tcp.port` is
  empty — expect wireless ADB to need re-enabling after a Portal
  reboot (one USB moment). **Undo:** `adb usb` while USB-attached,
  or reboot the Portal. Exposure note: adbd is now reachable on the
  LAN and the tailnet; the RSA key trust still gates shell access —
  only the Mac's key is authorized so far.

## 2026-10-25 (night) — Everything on-device: DashServer 6.5, snapshot retired (Phases 1 + 2)

- **Sameer's directives:** "No let's move everything onto the portal
  itself not this snapshot stuff." Then, Phase 2: "The family calendar
  is the only one i care to share and i already did." — no new calendar
  sharing; Family stays the only calendar the service account can read,
  and the VM snapshot is fully retired rather than kept for the other
  calendars.
- **DashServer 6.5 (versionCode 15), new class `OnDevice.java`:**
  - `GET /api/prayer` — the Portal fetches isalaah (IOK + CHESS) itself;
    parsing/shaping ported exactly from build_data.py (meridiem
    inference, Sunrise row). Per-day on-device cache
    (`prayer-cache.json` in the dashboard root); stale cache served on
    fetch failure, 502 only when no cache exists at all.
  - `GET /api/word` — the Portal fetches the Transparent Language Urdu
    WOTD RSS itself (daily_word.py's parser ported; Urdu only, the
    English line stays dead per his earlier call). Per-day cache
    (`word-cache.json`).
  - **Word audio is generated ON the Portal** with Android
    TextToSpeech `synthesizeToFile` → `audio/wotd-ur.mp3`, lazily, with
    a date stamp (`audio/wotd-ur.stamp`). Verdict: the engine exists
    (`com.facebook.aloha.app.ttsservice`) and produced a real 6,188-byte
    clip on-device. **Caveat:** Urdu is NOT a supported language in that
    engine (availability −2, LANG_NOT_SUPPORTED), so the clip is spoken
    in the engine's default voice — pronunciation is approximate.
    Nothing was faked; the text path is exact.
  - **Queue retry moved on-device:** `GET /api/calendar` now flushes
    the on-device queue first (direct Family insert via PortalApi),
    then reads — replacing build_data.py's host-side drain. Proven
    live: a planted queued event was inserted by a single calendar
    read (Google id `p218gm4sfc7s3o21seh5vs6ihs`), the queue file
    returned to `[]`, and the test event was then deleted with zero
    residue.
- **Page rewrite (deployed md5 `7c38e432e020a556997375659cc821af`,
  via the Tailscale PUT route):** the snapshot is GONE from the page —
  grep-proof zero matches for `data.json|calweeks|loadSnap|SNAP|
  calTombstone`. Prayer reads `/api/prayer`; the word card reads
  `/api/word`; home agenda and grocery cards read the live endpoints;
  weather persists the FULL Open-Meteo object to localStorage on every
  successful fetch and falls back to that cache. All last-good caches
  live on-device (`pr/wx/ag/hg/word_cache_v1`), stale-while-revalidate
  style. Calendar merge is **live Family events + the on-device queue
  only**; the snapshot merge and its tombstone machinery were removed.
- **Prayer match evidence:** `/api/prayer` rows equal the last
  snapshot's rows field-for-field — Fajr 5:56 AM (IOK 6:20 / CHESS
  6:33), Sunrise 7:03, Dhuhr 12:36 PM (1:15 / 1:45), Asr 4:28 (4:45 /
  4:45), Maghrib 6:07 (6:14 / 6:14), Isha 7:16 (8:00 / 7:45).
  `/api/word` returned نیپکن ("napkin", noun) with example — identical
  to an independent parse of the RSS.
- **The rename proof:** with `data.json` temporarily renamed away on
  the device (absence ls-confirmed), the home face rendered fully —
  clock, weather "67° Clear H 81° · L 56°", the full prayer strip,
  live agenda, the live grocery list (incl. Garlic), and the word band
  with its play button. The file was restored afterward; it remains on
  disk, unused.
- **Latent bug found + fixed by this refactor:** the page sends
  calendar `timeMin/timeMax` with fractional seconds
  (`Date.toISOString()`); PortalApi's date parser rejected them, so
  **every page-originated `/api/calendar` read had returned 502 since
  6.4 shipped** — silently, because the snapshot merge covered the
  week view. (The 6.4 live proofs used curl without millis.) Fixed
  with a `parseRfc()` helper that strips fractional seconds; page-
  format queries now return 200 with events. Side effect: the 6.4
  event detail sheet now works from the page for the first time —
  verified on-device by opening the Dec 19 Nikkah (Family, with
  Edit + Delete present, location intact).
- **Phase 2 consequence (his decision):** the retired snapshot's
  calweeks carried 27 events — Family 15, US Holidays 7, primary
  calendar 5. Those non-Family events no longer appear on the Portal:
  Thanksgiving week now shows only the Family event ("Abeer off") —
  Thanksgiving Day and Black Friday are gone from the Portal, as is
  Halloween on Oct 31. The events themselves are untouched in Google;
  they only ever reached the Portal via the snapshot, and the primary
  calendar stays deliberately unshared.
- **VM machinery retired:** crons `portal-dashboard-snapshot-push`
  (30-min) and `portal-dashboard-daily-word` (daily) are DISABLED
  (verified `enabled=false`, bodies preserved). `build_data.py`,
  `push_snapshot.py`, and `daily_word.py` remain in the
  portal-dashboard repo but are no longer part of any live path.
  Nothing in the page reads data.json anymore — there are no remaining
  snapshot dependencies.
- **Verification:** local harness (stub server + headless Chrome,
  fixtures = real device responses) — good scenario 10/10, forced-500
  scenario 5/5 with all last-good caches surviving on screen, and zero
  `data.json` requests in either run. (Harness note: Chrome 152's
  Local Network Access checks block loopback navigations in the VM
  sandbox, so the harness loads the page from file:// with a small
  fetch-rewrite shim; page logic under test is byte-identical.)
  Regression smoke after deploy: prayer, word, agenda, grocery,
  usuals, and calendar endpoints all 200.
- **Commits:** portal-dashboard `820966f` (page + DashServer 6.5
  sources; the upload token stays redacted in the committed
  DashService.java, as before). This LOG entry in portal-muse-plan.
- **Undo:** re-enable the two crons and PUT the previous page
  (md5 `3dd1f86fcad6af539aa60ef06d5ee013`, in git history) and reinstall
  the 6.4 APK; the on-device data.json was never deleted.

## 2026-10-25 (night) — Tailscale installed on the Portal (not signed in)

- **Sameer's ask:** "Sideload the app for now" — install only. The
  sign-in stays his on-device step, and the VM's tailnet join approval
  is still outstanding, so the snapshot PUSH pipe remains unarmed
  (`push_snapshot.py` still no-ops without portal-push.json).
- **APK, official source only:** `tailscale-android-universal-1.104.2.apk`
  from https://pkgs.tailscale.com/stable/ (the only Android APK in the
  stable track listing). 105,587,943 bytes, md5
  `c4ef8f5b8072c96432ce81309df280f8`. Package verified from the APK
  itself before installing (aapt on the Mac): `com.tailscale.ipn`,
  versionName `1.104.2-tc3430f0d2-gb01fa89d5`, minSdk 26.
- **Transfer:** the proxy truncates single-shot sends at ~224 KB, so
  the APK crossed VM → Mac in 196,608-byte chunks with per-chunk size
  verification (538 chunks; truncated chunks retried); reassembled on
  the Mac and md5-matched byte-exact before install.
- **Install:** `adb install ~/tailscale-1.104.2.apk` over the USB
  bridge → Success. Verified on-device: `pm list packages` shows
  `com.tailscale.ipn`; dumpsys confirms the versionName above. The
  app was **not launched** (no pid), not signed in, not configured;
  nothing else on the device was touched.
- **Undo:** `adb uninstall com.tailscale.ipn`.
- **What's left (unchanged from the Oct 24 plan):** Sameer signs in
  on the Portal; approves the VM's tailnet join; the first connection
  to the Portal's tailnet address is approved; then portal-push.json
  gets written and one push is watched landing.

## 2026-10-25 (night) — Calendar events tappable: detail sheet, Family edit/delete

- **Tap any event** — landscape grid block, all-day chip, or portrait day
  row — and a detail sheet opens (the add form's overlay-card language):
  title, When, Where (when present), Calendar (when the data names one).
  **Family events get Edit + Delete**; snapshot events are view-only —
  no controls, by their absence; queued events get Remove (the existing
  `/api/calendar/dequeue`).
- **DashServer 6.4** (versionCode 14, same cert; after `install -r` the
  service needed one MainActivity relaunch to bring port 8321 back, as
  with 6.3): the calendar read now carries **`gid`** — the Google event
  id — on Family-calendar events only, plus `cal` (source calendar
  name); `build_data.py` writes `cal` into calweeks too. New endpoints
  **POST /api/calendar/update** and **POST /api/calendar/delete**; the
  Google path hard-codes the Family calendar id, so an id minted by
  any other calendar simply does not resolve there. Update is a PATCH
  (GoogleClient rides a POST + `X-HTTP-Method-Override: PATCH` —
  HttpURLConnection has no PATCH) touching summary/start/end only, so
  an event's location and description survive an edit; a multi-day
  all-day event keeps its span when edited. Delete arms in place —
  first tap turns the button to "Tap again to delete", second fires.
  A failed save/delete keeps the sheet open with the failure on it.
- **Merge precedence** (was implicit, now explicit): live → snapshot →
  queue, first copy of an instant wins, so the rendered copy of a
  Family event is always the live one carrying its id. After an edit
  or delete the page tombstones the pre-mutation key in localStorage
  (48 h) and filters snapshot copies against it — a stale snapshot
  twin can't resurrect the old time, or the deleted event, before the
  next snapshot build lands.
- **Proven live** on the real Family calendar: throwaway event added
  via the existing add path (Google id `sup782o9hs11j6gvrq1ge60sl0`,
  Oct 27 9:00–9:30 AM PT), moved to 10:00–10:30 and retitled via
  /api/calendar/update, read back independently via gws at
  17:00–17:30Z with the new title, deleted via /api/calendar/delete,
  gws confirms the day empty. Queue `[]` throughout; /api/grocery,
  /api/photos, /api/usuals, /api/agenda all 200 after the install.
  The Dec 18–21 read now returns all 5 Family events with gid —
  the wedding weekend is editable from the couch.
- **Also approved and built (separate items):** queued-event
  visibility — a queued add must never masquerade as a saved event:
  dashed gold outline on grid blocks/chips and portrait rows, a
  "Queued" tag in portrait rows, and a Status: Queued line in the
  detail sheet. Live and snapshot events carry no marker. (The
  landscape blocks keep just the outline — the tag text clipped at
  column width.)
- **Also approved and built (separate items):** the last emoji rail
  buttons (Home ⌂, Prayer ☪, Weather ☀, All apps ▦) are now inline
  SVG line icons in the calendar/photos/grocery style; only Chrome ◎
  and the rail «/» remain glyphs. A proposed home-screen "updated"
  timestamp was **declined** by Sameer — not built.
- Undo: page → re-PUT the previous index.html from git history;
  server → reinstall the previous APK (6.3 source is the prior
  portal-dashboard commit). The live test event left no residue.

## 2026-10-25 (night) — Grocery: line-icon rail button + stale-while-revalidate, no loading state

- **Rail icon:** the grocery button's 🛒 emoji is now an inline SVG
  shopping-cart outline in the same line-icon style as the calendar
  and photos buttons (24×24, stroke currentColor, 1.7 width) — it
  takes the rail's soft/accent tints like its neighbors.
- **No more loading feel (his ask):** the grocery view used to sit
  blank until the live sheet read landed. It now opens instantly on
  the last fetched list + usuals, cached in localStorage
  (`gr_cache_v1`) on every successful fetch, and refreshes silently
  in the background; fresh data swaps in when it arrives. A failed
  refresh changes nothing on screen — no error, no blanking. Every
  mutation already re-fetches, so the cache always reflects the
  latest known state. The data.json snapshot still seeds the very
  first open when no cache exists yet.
- **Check-off semantics changed (his call: "it should just clear
  value basically means we got it that's it"):** check-off = clear
  the cell; the green-toggle is retired. A tap on an item now calls
  POST /api/grocery/checkoff, and DashServer clears the item's cell
  on the sheet — value and any fill — so the item simply leaves the
  list. The read no longer computes a "bought" flag, the calibrated-
  green machinery and the page's bought styling/toggle path are all
  deleted, and there is no un-check because there is no marked state
  to restore. DashServer 6.3 (versionCode 13), built via the Mac
  flow, same signing cert, `adb install -r`.
- **Live-sheet proof:** a throwaway item was added via the API,
  painted with the old green fill via gws, then checked off from
  the API. Read-back: the cell's value is gone and its background is
  plain {1,1,1} — no green. The sheet was left exactly as found.
- **Photos, small:** the photo bar's pin button is now a two-state
  toggle — "Show only this" with no pin, "Shuffle" while a pin is
  active (tap clears the pin and rotation resumes at the current
  speed). The label is set in refreshPinBtn, so the settings-sheet
  Clear and the pinned-file-gone auto-clear flip it too.
- Page deployed via the Mac PUT route, served md5
  f5de14b60ad8d20f60956d1a1fdcd032 = local. Undo: re-PUT the previous
  index.html from git; DashServer undo: reinstall the 6.2 APK.

## 2026-10-25 (night) — Photos immersive: drawer hidden = photo options hidden

- **His ask:** "hide all the photo options if we hide the drawer on that
  page so its an immersive picture only." Done — on the Photos page,
  hiding the rail now takes every piece of photo chrome with it: the
  bottom photo bar (Prev / Next / Add photos / Edit / Show only this /
  filename / ⚙), the album chips with their counts, and the new-album
  row. The picture fills the page; nothing else is on screen.
- **How:** no new state — one CSS rule keyed off the rail's own
  `body.railoff` (app mode, viewing state only):
  `body.app.railoff #phview.open:not(.editing)` hides `#phbar`,
  `#phchips`, `#phnewrow`. The › edge tab that restores the drawer lives
  outside the view and stays put, so the way back is the same tap as
  always — and restoring the drawer brings the photo options back.
- **Exemptions, deliberate:** edit mode is untouched (chips, edit grid
  and edit bar all stay — he's mid-task there); an open settings sheet
  stays up and functional; the screensaver is unchanged (the rule
  requires app mode). The Photos view never had a ✕ of its own —
  leaving it is via the rail.
- **Verified** in the local harness (fixture albums, app mode):
  computed-display asserted for shown / hidden / restored in landscape,
  hidden in portrait, plus both exemption states; console clean.
  Deployed via the Mac PUT route; served md5 `647f1b44ca25ef7cf5fc0b04ddb80e02`
  matches local. Undo: re-PUT the previous index.html from the
  portal-dashboard repo.

## 2026-10-25 (evening, later) — Weather view partial-render fix; Photos rotate speed + single-photo pin

- **Weather view rendered a partial forecast as if it were the whole thing.**
  Sameer's photo of the Portal in landscape: current temp + H/L and the
  radar rendered, but the Hourly and 10-Day cards were empty and the
  detail tiles (feels-like / UV / wind / humidity) never appeared.
  Root cause: the snapshot's weather object is home-face-only by design —
  `now_f / label / hi / lo`, no hours/days — and every section of the
  view render is guarded, so a partial object produced silently blank
  cards instead of an error. The view only ever looked right when the
  live Open-Meteo fetch landed first.
- **Fix (index.html + build_data.py):** `wxFull()` shape gate — the view
  renders a forecast only when hours AND days are present; a partial
  object gets a one-line status in the view instead (loading, or
  unavailable after a failed fetch), and the view re-renders itself when
  the live fetch lands. `build_data.py` now writes the FULL weather
  object into the snapshot (same Open-Meteo transform the page runs),
  so even the fallback render is complete. **Learning: a fallback cache
  must carry the full shape the renderer expects, or be shape-checked
  before render — guarded rendering turns missing data into a silent
  blank.**
- **Photos settings (his ask):** a ⚙ gear at the right end of the photo
  bar opens a settings sheet inside the Photos view. **Rotate chips —
  10s · 30s · 1 min · 5 min · Off** — replacing the hard-coded 30 s
  timer; persisted in localStorage (`ph_rotate_ms`, default 30s), and a
  tap re-arms the timer on the spot. Off stops the timer entirely:
  photos then change only by hand or album switch, and the library
  still refreshes on view open. The tick still calls loadPhotos(), so
  new photos keep arriving at any speed.
- **Show only this (the pin):** a "Show only this" button in the photo
  bar pins the displayed photo — its path, `?v=` stamp stripped — in
  localStorage (`ph_pin`). The pin gates only the AUTOMATIC advance:
  the timer path (`stepPhoto(d, auto)`) snaps back to the pinned photo
  every tick, while manual Prev/Next browsing is never gated. One photo
  state machine drives both faces (the screensaver restores the same
  view + album via ui-state), so the pin holds on the screensaver too.
  The settings sheet shows the pinned photo (thumbnail + file name)
  with a **Clear** button; the bar button turns gold while the pinned
  photo is on screen. If the pinned file disappears from the library,
  the next /api/photos load clears the pin by itself.
- Verified locally before deploy (stubbed /api/photos, fixture albums):
  timer armed at the default and re-armed per chip, Off leaves no
  timer, pin set → auto-step returns to it → manual step leaves it →
  auto-step returns, pin auto-clears when its file vanishes; panel
  screenshotted in both orientations. Deployed by the Mac PUT route;
  served md5 matched the local file (`5e1acbcb`). Page + build_data.py
  committed to portal-dashboard as `dc849e1`.
- Undo: re-PUT the previous index.html from the portal-dashboard repo's
  history. The settings themselves live in the Portal browser's
  localStorage — clearing site data resets rotate to 30s and drops the
  pin.

## 2026-10-25 (evening) — Calendar week view + add-from-screen, multi-album photos, groceries portrait fix

Evening build session (design locked with Sameer in chat first, per the design-first rule), shipped as **DashServer 6.1 → 6.2 + page updates**, all verified live on-device or against Google directly:

- **Groceries portrait fix.** The two panes stayed side-by-side in portrait, which crushed both add rows (the "New usual" field clipped mid-word). Portrait now stacks them: list on top with a full-width "Add an item" row, Usuals below. Page-only change, pushed via DashServer `PUT /upload/index.html`.
- **Calendar 📅 view** built to his locked spec — Skylight-inspired week view: header reads **"Sameer&Abeer"**, week ‹ › navigation, **no ✕** (the rail is the way out), **no clock, no weather** in the header, and **no person split** — no initials, no per-person colors; the data can't attribute events to a person yet, so every event renders in one gold tint. Landscape: hourly day columns (grid 6 AM–11 PM), all-day strip under the day headers, events under 45 min show title only so nothing clips; today column tinted, its number gold. Portrait: stacked day cards, today outlined. Gold **+** button opens the add form (Title / Date / Start / End / All-day).
- **Calendar data = a union of three sources**, deduped by *instant* (title + epoch-ms start/end): the snapshot `calweeks` (build_data.py, all his calendars, ±5 weeks), the live Family-calendar read, and the on-device add queue. Dedupe by instant, not by string — the same event legitimately appears on two calendars with different offset spellings (`09:30-07:00` vs `16:30Z`).
- **Photos multi-album.** Albums are folders on the device and "move" was exclusive by construction. New `POST /api/photos/copy`, and the edit-mode action is now **"Add to"**: the photo is copied into the chosen album and stays where it was, so one photo can live in several albums; delete still removes only that album's copy. `/api/photos/move` is unchanged (round-trip tested: create album → copy in → move back → clean up; library left exactly as found).
- **Rail icons.** The 📅 emoji (renders as somebody else's fixed date) and the ▣ dingbat are gone — both are now inline SVG line icons in the rail's own stroke style. The calendar icon shows **today's date number**, set by JS from the device clock and refreshed every 30 min.
- **DashServer trail.** 6.0 (v2 dashboard) → **6.1**: `GET /api/calendar`, `POST /api/calendar/add` (queue to `cal-queue.json`) + `POST /api/calendar/dequeue`, `POST /api/photos/copy`, `/api/ui-state` whitelist gains `"calendar"` → **6.2**: native adds (below). Same signing cert throughout; built on the Mac (~/dashbuild/build.sh) and installed with `adb install -r`.
- **Native adds (6.2).** `/api/calendar/add` now inserts **directly into the Family calendar**; the queue + build_data.py drain remain only as the fallback if a direct insert fails (logcat prints a `DIRECT` line vs a fallback line, so the path taken is never a guess). **Proven live:** test event posted to the Portal API → Google returned id `a6mveae504s3goj1kbfmd74dpg` → independently confirmed in Google Calendar at 2026-10-26 9:00–9:30 AM PT → queue file untouched → test event deleted. The Dec 18–21 window now reads back all 5 Family events live, matching ground truth.

**Learnings (durable):**

- **A calendar shared with a service account may never appear in the SA's `calendarList`** — Family never did, despite the share existing since setup. That, not missing event data, is why live reads returned `[]` for a day. Fix shipped: the read path appends Family as a guaranteed entry and queries events **by calendar ID directly**; list membership is not required. `calendarList.insert` can't cure it from the SA side — it needs the full `calendar` scope, broader than granted.
- **Scope arithmetic.** `calendarList.list` rejects a token holding only `calendar.events` (403 "insufficient authentication scopes"). The SA's shipped scope is a *superset*, not a swap: `spreadsheets` + `calendar.readonly` + `calendar.events`.
- **Share boundary (Sameer's call, 2026-10-25).** Family calendar → the SA is **writer** (his manual share had landed reader-only on Family and nowhere on primary; the bump to writer was applied via ACL update through his own OAuth). His **primary calendar is deliberately NOT shared** with the SA — every server-side calendar operation targets Family or the SA's own list, and that boundary stays.
- **Deploy gotcha.** `curl --data-binary @~/file` does **not** tilde-expand after the `@` — it silently sends a 0-byte body, and DashServer faithfully wrote empty `index.html`/`data.json` once during the 6.1 deploy (recovered by re-PUT). Rule: **absolute paths in `--data-binary`, and md5-verify the served file after every PUT.**
- **Design rule, confirmed twice today:** kill chrome that duplicates a system affordance (the view ✕ next to the rail), and never render a split (people, colors) that the data can't back up.

**Undo:** page → re-PUT the previous `index.html` (now in git history in the portal-dashboard repo); DashServer → reinstall the previous APK; Family share → drop the SA's role back in Google Calendar → Family → Share with specific people.

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
- **DECISION (2026-10-25 evening, his): keep both services ENABLED** — video
  wins over the fuller debloat. The Phase 7 router firewall remains the
  planned network-side mitigation.

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
