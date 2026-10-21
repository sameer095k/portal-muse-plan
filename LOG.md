# LOG

Newest first. Every change made on the device gets an entry: what, the exact
command, and its undo. Sessions where Sameer's hands are on the device and
Muse is remote eyes get dated headers like his TV project logs.

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
