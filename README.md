# NEXUS

Ibrahim's personal operating system — five specialized apps, one cohesive interface, zero subscriptions, zero frameworks.

Everything runs in the browser. Data lives in `localStorage` first (instant, offline-first), and optionally mirrors to a Supabase cloud database so phone and laptop stay in sync within about a second of each other. No build step, no `npm install`, no bundler. Drop the folder on any static host and it works.

---

## The Five Apps

### 🎯 Goals — `index.html`
The command center and home screen. A circular day-progress energy ring tracks your waking hours — wake and sleep times are user-configurable via a gear button (saved to `day_window_v1`, default 7 AM wake / midnight sleep). Below it, a long-term goals tracker lets you add goals with a title, current value, target value, unit, and direction (going up = savings/reps, going down = weight loss). Progress percentage and bar auto-calculate from the numbers. You can update the current value inline at any time. The old today's goals card, tomorrow's goals card, rollover logic, goal ticker, and streak system have been removed — the dashboard is now focused on long-term progress only.

### 🧘 Wellness Hub — `health.html`
Full replacement for the old supplement tracker. Five tabs covering every daily health dimension:

- **😴 Sleep** — Log bed time, wake time, and quality (1–5 stars). Auto-calculates sleep duration. 7-day bar chart (green ≥ 8h, indigo ≥ 7h, amber below). Stats row shows last night's hours, 7-day average, and current 7h+ streak. Recent log list with delete.
- **✅ Habits** — Daily habit checklist with six defaults (water, stretch, sunlight, no phone before bed, reading, cold shower). Per-habit streak counter fires a 🔥 at 3+ consecutive days. Add and delete custom habits. Completion progress bar and done count reset daily at midnight. Marks `wellness:done:YYYY-MM-DD` as `true` when all habits are checked.
- **⚡ Recovery** — Six muscle soreness sliders (Chest, Back, Shoulders, Arms, Legs, Core — 0 to 5). Energy and stress ratings (1–5 pill selectors). Calculates a 0–100 readiness score weighted 40% energy, 30% soreness, 30% stress. SVG ring animates to score and changes color: 🟢 Full Send (≥ 75), 🟡 Steady (≥ 50), 🔴 Rest Day (below 50). Today's data auto-loads on revisit. 7-day history list.
- **🌙 Dreams** — Log type (Lucid, Normal, Vivid, Nightmare), technique (MILD, WILD, WBTB, SSILD, FILD), title, and description. Stats track total logged, lucid count, and lucid rate %. Reality check panel at top with six daily RC reminders. Full journal list with delete.
- **📓 Journal** — Seven mood selectors (🔥 😊 😐 😔 😤 😴 🧠), freeform textarea with live word count, autosaves 1.8s after you stop typing. Past entries listed below, expandable on tap, deletable. Today's entry loads automatically if already saved.

### 🏋️ Summer Grind Gym — `gym.html`
4-day push/pull/legs/core split (Mon–Thu) with per-exercise checkoffs. Key features:

- **Mark as missed** — a dedicated `✕ Mark missed` button sits alongside `Mark workout done`. They're mutually exclusive: marking one clears the other. Both write to the session history.
- **Consistency chart** — 8-week bar chart (Chart.js) showing weekly workout completion percentage. Green = all scheduled days done, teal = partial, red = missed. Displays this-month % and current streak at the top.
- **Heatmap calendar** — 16-week GitHub-style contribution grid. Each cell is a day: green = done, red = missed, gray = rest, dark = no data. Hover any cell for the date. Stats row shows total done, total missed, best streak.
- **Planche progression** — 4-phase tracker (Lean → Tuck → Advanced Tuck → Full) with Mon/Thu hold-time logging per week, delta tracking, and a progress bar per phase.
- **Weight tracker** — bodyweight log with a trend line chart, 7-day delta, and a progress bar from start (185 lbs) to goal (175 lbs).
- **Daily nutrition log** — calories, protein, steps, energy level, food eaten, notes. Target pills auto-highlight green/red vs. goals (1900–2100 kcal, 130–150g protein, 10k steps).

### ⚡ Grind Log — `grind-log.html`
Gamified productivity tracker. Log tasks across six categories: Code, Study, Fitness, Content, Focus, Other. Each has preset quick-log tasks (e.g. "Built a feature +75 XP", "TryHackMe room +60 XP"). Manual entry with custom XP. Seven-tier rank ladder: Rookie → Grinder → Hustler → Focused → Elite → Ascendant → Legend. 14-day XP bar chart, category breakdown, recent activity log, streak counter, best-day stat. XP is cumulative across all time.

### 🏹 Valorant Command Center — `valorant-cc.html`
Full match-history tracker for ranked grinding. Log agent, map, KDA, headshot %, result (win/loss/draw), RR change, score, rank, and notes per session. Auto-updates the RR tracker when you log a session with an RR change. Stats: win rate, avg KDA, avg HS%, best map, most played agent, session count. Win/loss streak banner (fires at 2+ consecutive). Agent and map win-rate breakdowns with mini bar charts. 14-day W/L bar chart. Manual RR entry with a progress bar toward Immortal.

---

## Architecture

**Zero-dependency core.** Vanilla JavaScript, CSS3, HTML5. Every page opens directly in a browser with no compilation.

**Local-first storage.** All reads and writes hit `localStorage` first. `sync.js` watches a whitelisted set of keys per app and pushes debounced updates to Supabase in the background. On load, it pulls remote state and merges it. Real-time sync is handled via Supabase postgres_changes subscription.

**Shared chrome — `topbar.js`.** A sticky status bar injected on every page showing live counts for all five apps: goals complete, habits done today, gym status (done/missed/rest), XP earned today, and Valorant W/L. Tapping any pill navigates there. Also registers the service worker for PWA offline support. The wellness pill reads `wellness:done:YYYY-MM-DD` (a boolean — `true` when all habits for that day are checked off).

**PWA — `manifest.json` + `sw.js`.** Fully installable on iOS (Safari → Add to Home Screen) and Android. Launches full-screen with no browser chrome. Service worker uses network-first for HTML pages and stale-while-revalidate for static assets.

**Aurora glass aesthetic.** Deep space background (`#02020C`) with layered indigo/violet/cyan radial glows, frosted `backdrop-filter` panels, tabular monospace numbers so stats never jitter on update, consistent CSS variable system across all pages (`--indigo`, `--cyan`, `--violet`, `--surface`, `--border`, etc.).

---

## File Structure

```
nexus/
├── index.html          — Goals + energy ring (home screen)
├── health.html         — Wellness hub (sleep, habits, recovery, dreams, journal)
├── gym.html            — Workout tracker + charts + planche
├── grind-log.html      — XP productivity tracker
├── valorant-cc.html    — Match history + RR tracker
├── sync.js             — Supabase cloud sync helper (shared)
├── topbar.js           — Persistent nav bar + PWA service worker reg (shared)
├── sw.js               — Service worker (offline cache)
├── manifest.json       — PWA manifest
├── favicon-32.png
├── icon-192.png
├── icon-512.png
└── apple-touch-icon-180.png
```

---

## Storage Keys

Each app writes to a specific set of localStorage keys. `sync.js` syncs these per app.

| App | Keys synced |
|---|---|
| Goals | `long_goals_v1`, `day_window_v1` |
| Wellness | `wellness:sleep`, `wellness:habits`, `wellness:recovery`, `wellness:dreams`, `wellness:journal`, `wellness:done:YYYY-MM-DD`, `wellness:tab` |
| Gym | `ibrahim_gym_v1`, `ibrahim_gym_done` |
| Grind | `grind_log_v1` |
| Valorant | `val_cc_v1` |

> **`wellness:done:YYYY-MM-DD`** stores a boolean (`true`/`false`). Set to `true` by `health.html` when all habits for the day are checked. Read by `topbar.js` and the `index.html` dock tile to show the wellness pill status. Format: `wellness:done:2026-06-20`.

---

## Getting Started

### 1. Deploy
Push the folder to GitHub, then import on [Vercel](https://vercel.com) — zero config, it's static files.

### 2. Cloud Sync (optional)
1. Create a free project at [Supabase](https://supabase.com).
2. Create an `app_state` table:
```sql
create table app_state (
  key text primary key,
  data jsonb,
  updated_at timestamptz default now()
);
alter table app_state enable row level security;
create policy "public access" on app_state for all using (true);
```
3. Paste your Supabase project URL and publishable key into `sync.js` and `topbar.js` where indicated.

### 3. Install on iPhone
Open your deployed URL in Safari → Share → **Add to Home Screen**. Launches full-screen, no browser chrome.

---

## Adding a Sixth App

1. Create a new `.html` file using the existing CSS variables (see any page's `:root` block for the full token list).
2. Include the shared scripts:
```html
<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
<script src="sync.js" defer></script>
<script src="topbar.js" defer></script>
```
3. Add an entry to the `TILES` array in `topbar.js` (for the nav pill) and in `index.html` (for the dock tile).
4. Call `initCloudSync({ appKey: 'yourapp', syncedKeys: [...] })` at the bottom of the page.

---

## Tech Stack

| Layer | Choice |
|---|---|
| Language | Vanilla JS (ES2020+), no framework |
| Styling | Pure CSS3, CSS custom properties |
| Charts | Chart.js 4.4 (gym consistency + sleep 7-day charts) |
| Database | Supabase (postgres + realtime) |
| Auth | None — single-user, key-based |
| Hosting | Vercel (static) |
| Offline | Service Worker (cache-first assets, network-first HTML) |

---

## Current Status (June 2026)

Active daily use. Changes in this build:

**Goals / Dashboard (`index.html`)**
- Energy ring wake/sleep times configurable via gear icon (saves to `day_window_v1`); default changed to 7 AM wake
- Fixed modal bug: sleep input was set to `'24:00'` (invalid for browser time inputs), causing the field to go blank and silently reset to midnight on save — now uses `'00:00'`
- "Still sleeping" state now shows the configured wake time in the subtitle so misconfigured times are obvious
- Long-term goals system added (replaces daily goals entirely)
- Daily goals, tomorrow's goals, goal ticker, rollover, streak removed
- Dock tile for Wellness renamed 🌿 and reads `wellness:done:YYYY-MM-DD` (with fallback to old supplement keys during transition)

**Health → Wellness Hub (`health.html`)**
- Old supplement tracker fully scrapped
- Rebuilt as five-tab Wellness Hub: Sleep, Habits, Recovery, Dreams, Journal
- Sleep tracker with 7-day bar chart and streak counter
- Habit tracker with per-habit streaks and custom habit support
- Recovery & Readiness scoring with animated SVG ring (0–100 score)
- Lucid dream journal with type/technique tagging
- Daily journal with mood picker and 1.8s autosave

**Gym (`gym.html`)**
- Missed workout button added (mutually exclusive with done)
- 8-week consistency bar chart added (Chart.js)
- 16-week heatmap calendar added
- Fake muscle rank/LP card removed

---

## Pending / Backlog

- **`topbar.js` wellness pill** — still reads old `stack:taken:YYYY-MM-DD` / `stack:items` supplement keys. Needs updating to read `wellness:done:YYYY-MM-DD` and show habit completion instead of supplement count.
- Decide whether to add a sixth app (candidates: focus timer, finance tracker)

---

## License

Open-source. Fork it, gut it, rebuild it as your own.