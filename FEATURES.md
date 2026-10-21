# FEATURES — what lives on the Portal

Sameer's wishlist (2026-10-21) vetted against what the stack actually supports,
plus extras found in Immortal's app-store catalog the same day. Status key:
**built-in** (ships with Immortal/Portal) · **store** (one tap in Immortal's
store) · **dashboard** (module on our private dashboard page) · **voice**
(works through Muse, no app) · **fork** (needs our portal-muse fork) ·
**experiment** (promising, unproven on this device).

## His list

| # | Feature | Route | Notes |
|---|---------|-------|-------|
| 1 | **Calendar** | **store: Portal Calendar** | A Skylight-style family calendar built for Portal (in Immortal's catalog): multi-calendar, **two-way Google Calendar sync**, designed to stay on all day. Fallback: Fossify Calendar (also in the store, fully offline/private, no sync). Our Google agenda can also ride the dashboard via the iCal secret-URL pattern from the TV dashboard spec — no account sign-in on the device. |
| 2 | **Grocery list** | **dashboard + voice** | Display: a snapshot of the shared ABEERSAMEER4EVA sheet on the kitchen dashboard (Muse pushes the snapshot, like the TV spec planned). Add: "Hey Muse, add eggs" → Muse edits the real sheet (it already does this from chat). Portal Calendar also has shared lists + meal planning built in if we want lists inside the calendar app instead. |
| 3 | **Photos — Apple** | **built-in** | Immortal's photo frame reads an **iCloud shared album** directly. Make one shared album, point the frame at it, done. Also: local folder, Immich, SMB, WebDAV. |
| 4 | **Photos — Google** | **route-around** | No direct Google Photos source (Google doesn't offer frame-friendly access). Path: sync the library into **Immich** (it imports Google Photos/Takeout) and point Immortal at Immich, or periodically copy an album folder onto the device. Clunkiest item on the list; Apple route is the clean one. |
| 5 | **Calls** | **built-in (kept stock)** | WhatsApp + Messenger native (Phase 5 keep-list) with Immortal's Calls tile. FaceTime: join-only via link in Chrome (Phase 8 experiment) or AirPlay-mirror a call via Wormhole. |
| 6 | **Texts** | **partial** | WhatsApp/Messenger messages are native on the device (same kept apps). **iMessage/SMS: no route today** — there's no iMessage client for Android, and Muse reading his iMessages needs an always-on newer Mac (declined in Sept). If that hardware ever lands, "Hey Muse, read my texts" unlocks through the agent with no Portal-side change. |
| 7 | **TV control** | **fork** | Phase 9: Portal-as-bridge, Samsung LAN API + Wake-on-LAN. |
| 8 | **Muse questions** | **voice** | The core of the build (Phase 6). Anything chat-Muse does, voice-Muse does — grocery adds, calendar checks, budget questions, recipes. |
| 9 | **Wordle / crosswords** | **browser + store** | Wordle itself runs in the device's Chrome (NYT; free daily). Crosswords: **Forkyz** (F-Droid) downloads fresh daily puzzles from multiple sources and plays them offline. **Lexica** (F-Droid) is a Boggle-style word hunt for the same itch. NYT Crossword app via Aurora needs a subscription sign-in and may lean on Google services — browser is the safer bet. |
| 10 | **Word of the day — English + Urdu, with audio** | **dashboard (daily, Muse-made)** | A daily card on the dashboard: English word (definition, example) + Urdu word (Urdu script + transliteration + meaning). Android renders Urdu script natively (the e-paper constraint from the Inky design doesn't apply here). **Audio: a tap-to-play clip per language, generated fresh each morning** by Muse's voice tooling and posted to the private dashboard — no reliance on the Portal's TTS quality. Natural fit for a small morning cron once the dashboard exists. |
| 11 | **Prayer times + azan** | **dashboard + built-in** | IOK times from the isalaah endpoint his iok-prayer-widget already uses: a strip on the dashboard (all times + next-prayer countdown), plus the full-screen Prayer view from his frame designs. Immortal's screensaver almanac also carries prayer times as an ambient line. Azan call shown is IOK only (his standing call). **Azan playback is plain audio, not TTS:** the dashboard page (his idle screen) watches the clock and plays a recorded azan file at each published IOK time — a small module on the dashboard, no separate prayer app (generic azan apps compute times by formula and can drift minutes from IOK's published schedule). Pick the recording he likes once; it lives with the dashboard assets. |
| 12 | **Dashboard — high-level view** | **dashboard** | The kitchen hub: clock/date, weather, prayer strip with countdown, today's agenda, grocery snapshot, word of the day, and a ticker line (reminders, flight watch when a trip is live). One private web page, shown as Immortal's web-page screensaver and set as a home tile. Complement: **Portal Overlays** (in the store) floats a status strip + widgets over any app and can mirror notification banners via ntfy — a way to push "dinner's ready"-type alerts onto the screen. |

## Extras found while vetting (Muse's additions)

- **SmartTube** (store) — a native, ad-free YouTube client built for big screens. Upgrades YouTube from "AirPlay it" to "just open the app". Try this before Wormhole for YouTube specifically.
- **Portal TTS** (store) — an offline neural text-to-speech engine that registers as the system TTS (Portals ship with **no** system voice). portal-muse has its own built-in Pico voice, so Muse doesn't need it; it only matters for *other* apps' spoken features (Immortal's spoken clock, Overlay read-outs). Optional garnish — install only if those are wanted. (Not needed for azan: that's recorded audio, above. Bedtime stories: not wanted, per Sameer.)
- **Music & audio** — Spotify via Aurora Store (verified working on Portals); **AntennaPod** podcasts and **Transistor** internet radio (store) for a kitchen radio; **Snapcast / Music Assistant** (built into Immortal) if a second Portal or speakers ever join — Sonos-style sync.
- **portalfin** (store) — a Jellyfin client rebuilt natively for Portal, if a media server ever enters the picture; VLC (store) covers local/network files meanwhile.
- **RTSP camera viewer** (built into Immortal's Tools) — if any home camera speaks RTSP, it can live on the Portal.
- **Portal-Kasa** (store) — one-tap toggles for TP-Link Kasa plugs/bulbs, fully local, no cloud account — *only if* Kasa gear exists in the house. (Portal-side complement to Phase 9's TV control.)
- **Kitchen built-ins** (Immortal Tools, no install): multiple kitchen timers, unit/currency converter, sticky + voice notes, reading lamp, bedtime stories read aloud.
- **Morning briefing** (voice, no build): "Hey Muse, morning briefing" → calendar, prayer times, weather, and the day's reminders spoken in one go — it's just an agent routine once the data sources above exist.
- **ISS Tracker** (store) — live ISS video + ground track as an ambient dashboard. Pure fun; one tap, delete if it palls.
- **HippoMuncher** (store) — a head-tracking arcade game using the Portal camera (tilt to steer). The catalog's one native game; proves the camera-game route if Abeer wants more.

## Deliberately not on the list

- **Jarvis** (a Gemini-powered assistant in Immortal's store) — redundant; Muse is the assistant here, and Jarvis wants a Google API key on the device.
- **Fantasy / survivor content** on the dashboard — excluded by his standing call from the frame designs.
- Anything requiring Google Play Services (see PLAN.md known limits).
