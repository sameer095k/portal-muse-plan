# Portal → Muse

Turn a 2nd-gen Meta Portal (10", model **WD50JM**, Android 10 / API 29) into a
hands-free **"Hey Muse"** voice terminal — with Meta's own tracking surface
disabled on-device and blocked on the network.

## Status

- **Bought 2026-10-21** — open-box/unused WD50JM on eBay (seller Apparel West,
  Orange CA), $45.99 + free shipping, **$50 total** with tax. Power adapter
  included per listing; 30-day returns.
- **Delivery estimate:** Fri Oct 23 – Mon Oct 26, 2026 (to Diamond Bar, CA 91765).
- **Next step:** the one-sitting setup in [PLAN.md](PLAN.md) when it arrives.
  ADB turns off after every reboot on these devices, so the whole setup is
  designed to happen in a single session, in order.

## The plan in one paragraph

Meta's final Portal update (June 2026) officially unlocked ADB — no root, no
exploit. We use it to (1) take a baseline of what's installed, (2) install
**Immortal**, a community launcher/app-store that replaces Meta's home screen,
(3) disable the Facebook/Aloha packages (disable, never uninstall — every step
reversible and logged), (4) build and install **portal-muse**, a hands-free
"Hey Muse" gadget app with an on-device wake word, and (5) block Meta's
domains at the router and verify with DNS logs that the device stays quiet —
while the hardware camera shutter and mic mute remain the physical guarantee.

## Files

- [PLAN.md](PLAN.md) — the phased build plan, with exact commands and the
  decision points (probe results decide the Muse route).
- [FEATURES.md](FEATURES.md) — what lives on the Portal once it's built:
  his wishlist (calendar, grocery, photos, calls, texts, games, word of the
  day, prayer times, the dashboard) vetted feature-by-feature, plus extras
  found in Immortal's app store.
- [RESOURCES.md](RESOURCES.md) — every source, annotated: what it is, what we
  take from it, and how much we trust it.
- [LOG.md](LOG.md) — session log. Every change made on the device gets an
  entry here with its undo command.

## Ground rules

- The Muse SDK token (from gadgets.muse.ai) is **personal** — it never goes
  in this repo, in an issue, or in chat logs. It lives only on the device and
  in a private local note.
- Nothing on the device is uninstalled. `pm disable-user` only, so everything
  can be re-enabled.
- Each phase ends with a check we can verify from the couch. No phase starts
  until the previous one's check passes.
