# Green Man 2025 — Festival Companion PWA
## Claude Code Project Handoff Document

---

## What this project is

A personal, offline-first Progressive Web App (PWA) for Android that acts as a festival companion for **Green Man Festival 2025** (Brecon Beacons, Wales, 14–17 August 2025).

The app is built as a **single self-contained HTML file** with all data embedded inline, so it works with zero internet connection once loaded. It's intended to be hosted on GitHub Pages, installed to the Android home screen via Chrome's "Add to Home Screen" (PWA), and used on-site at the festival where WiFi is restricted and mobile signal is poor.

This is a **personal project** — no commercial use, no public audience, no server infrastructure needed.

---

## Current file structure

```
greenman.html     — the entire app (HTML + CSS + JS, all inline)
gm2025.json       — the source lineup data (not loaded by the app, embedded inside greenman.html)
GREENMAN_HANDOFF.md  — this document
```

The app is a single file by design. Keep it that way unless there's a compelling reason to split it — the whole point is simplicity and offline reliability.

---

## Tech decisions already made

| Decision | Choice | Reason |
|----------|--------|--------|
| App format | Single-file HTML PWA | Offline-first, no build step, installable via Chrome |
| Hosting | GitHub Pages | Free, simple, no server needed |
| Data storage | Embedded JSON + localStorage | Works fully offline |
| Font | Playfair Display (display) + DM Sans (body) | Earthy, editorial feel matching GM brand |
| Colour palette | Dark forest green base, amber accent, cream text | Matches Green Man's visual identity |
| Photo source | Last.fm API (`artist.getInfo`) | Free, good coverage, no auth beyond API key |

---

## Data

The full 2025 lineup is embedded directly in `greenman.html` as a JavaScript constant (`FESTIVAL_DATA`). It covers:

- **323 acts** across **10 stages** over **4 days**
- Stages: Mountain Stage, Far Out, Walled Garden, Rising, Chai Wallahs, Round The Twist, Babbling Tongues, Cinedrome, Wishbone, Rough Trade tent signings
- Each act has: `stage`, `day` (YYYY-MM-DD), `start` (full ISO timestamp), `end` (full ISO timestamp), `artist`
- Acts that run past midnight have correct date stamps on their end time (e.g. a Friday night act ending at 2am has end `2025-08-16T02:00:00`)

The source `gm2025.json` file is kept separately for reference and future updates (e.g. swapping in 2026 data).

---

## App structure — three views

### 1. Browse (🎵)
- Shows all acts for the selected day, grouped into hourly time blocks
- Filterable by stage via a horizontal pill row
- Each act card has a **4-button rating widget**: ✕ Skip · ◦ Maybe · ✦ Want to see · ★ Must see (left to right)
- Ratings persist in `localStorage` under key `gm2025_ratings`
- Card background: stage-specific colour gradient by default; swaps to artist photo once one is fetched
- Rating 0 (Skip) fades the card to 45% opacity and does NOT trigger a photo fetch
- Ratings 1, 2, 3 trigger a Last.fm photo fetch for that artist in the background

### 2. Now & Next (🟢)
- **Swipe carousel** showing only the 3 main music stages: Mountain Stage → Far Out → Walled Garden
- Each slide shows **Live now** card and **Up next** card for that stage
- Cards show artist name, time range, countdown ("23 mins left" / "in 45 mins"), and rating pip if rated
- Dot indicators at top show which stage you're on
- Refreshes every 60 seconds automatically

### 3. My Picks (⭐)
- Lists all acts rated 1, 2 or 3 stars, grouped by day, sorted by time
- Shows rating pip colour, artist name, abbreviated stage name, start time
- **Clash detection**: any two picks that overlap in time get a red left border and ⚡ clash label

---

## Key constants / config in the JS (top of `<script>` block)

```javascript
// ── Last.fm API key ──────────────────────────────────────────
// Get a free key at: https://www.last.fm/api/account/create
const LASTFM_API_KEY = '';   // <-- paste key here

// ── Dev mode ─────────────────────────────────────────────────
// Simulates being at the festival so Now & Next works before August
// Set DEV_MODE = false when using at the actual festival
const DEV_MODE = true;
const DEV_TIME = '2025-08-15T19:00:00';  // simulated "now"
```

---

## localStorage keys

| Key | Contents |
|-----|----------|
| `gm2025_ratings` | `{ "Stage\|\|2025-08-15T19:05:00": 3, ... }` — rating per act |
| `gm2025_photos` | `{ "John Grant": "https://...lastfm...jpg", ... }` — cached artist photo URLs |

---

## Last.fm photo fetch — how it works

1. User taps a 1/2/3 star rating on an act card in Browse
2. `setRating()` calls `fetchArtistPhoto(artist)` in the background
3. `fetchArtistPhoto()` hits: `https://ws.audioscrobbler.com/2.0/?method=artist.getinfo&artist=ARTIST&api_key=KEY&format=json`
4. Extracts the `extralarge` (or `large`) image URL from the response
5. Saves to `photoCache` (in memory) and `localStorage` (`gm2025_photos`)
6. Calls `updateCardPhoto(artist)` to update any visible browse cards immediately
7. If Last.fm returns an empty string (artist not found), stores `null` so we don't retry

The fetch is skipped if:
- `LASTFM_API_KEY` is empty
- The artist is already in the cache (even if cached as `null`)
- The rating is 0 (Skip)

---

## Visual design language

- **Dark, earthy, festival-atmospheric** — not corporate, not generic
- Each stage has a unique dark gradient placeholder (see `STAGE_GRADIENTS` constant)
- Cards use a bottom-up vignette so text is always readable over photos
- Rating states: Must see = amber `#e8a020`, Want = green `#7ab648`, Maybe = blue `#5b8fc4`, Skip = grey
- Rated cards get a 2px coloured border glow matching their rating colour
- Noise texture overlay on `body::before` for tactile feel

---

## What's NOT done yet (suggested next steps)

### High priority
- [ ] **PWA manifest + service worker** — needed to make "Add to Home Screen" work properly on Android and enable true offline caching. Without this it's just a webpage, not an installable PWA.
- [ ] **Last.fm API key** — user needs to get a free key from last.fm/api and paste it into the `LASTFM_API_KEY` constant. Once done, photo fetching is live.
- [ ] **GitHub Pages setup** — repo needs creating, file pushing, Pages enabling. This is the hosting step before PWA install can be tested properly.

### Medium priority  
- [ ] **Stage placeholder photos** — currently using CSS gradients. The plan is to find 10 atmospheric photos (one per stage/area) to use as the default card background before artist photos load. These should be earthy, moody, festival-site photos.
- [ ] **Photo display in Now & Next** — slot cards already support photos technically (they call `getPhoto()`), but they'll only show once the user has rated an act and the cache is populated. Consider whether stage photos would help here too.
- [ ] **DEV_MODE switch** — before the festival, change `DEV_MODE = false` and remove `DEV_TIME`. Or add a simple in-app toggle for testing.

### Nice to have
- [ ] **Search / filter by artist name** — useful for quickly finding a specific act across all stages/days
- [ ] **"Now & Next" rating pip tap** — tapping a card in Now & Next could jump to that act in Browse so you can rate it on the spot
- [ ] **Haptic feedback on rating** — `navigator.vibrate(10)` on rating tap for a tactile response
- [ ] **2026 data swap** — when next year's lineup is announced, scrape Clashfinder again and update the embedded JSON. The Clashfinder URL pattern is `https://clashfinder.com/m/gm2026/`

---

## Data source

Original lineup scraped from Clashfinder (community-maintained):
`https://clashfinder.com/m/gm2025/?user=1ndc04.ai`

The Green Man website itself (`greenman.net`) may publish primary data closer to the festival — worth checking for any corrections or additions.

---

## Project context

- **Developer**: Mike (personal project, unrelated to work)
- **Target device**: Android phone, Chrome browser
- **Working environment**: Raspberry Pi dev server, accessed via SSH (Terminus app on Android, or Claude Code remote session). Tailscale configured for access outside home network.
- **Deployment**: GitHub Pages → Chrome "Add to Home Screen" on Android
- **Previous work done in**: Claude.ai chat (claude.ai), not Claude Code

---

*Good luck, and enjoy Green Man! 🌿*
