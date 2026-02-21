# Wedding Planning App — CLAUDE.md

## Project Overview

Hebrew-language wedding planning web app for Tinko & Andrea's wedding on **May 20, 2026** (originally 2025, sidebar not yet updated). ~100–120 guests, outdoor garden event in central Israel (Sharon region), no kashrut restrictions.

## Architecture

**Single-file app**: Everything lives in `index.html` — HTML structure, all CSS styles, and all JavaScript in one file. There are no build tools, no npm, no bundler. Open the file directly in a browser.

**External dependencies** (CDN only):
- `Heebo` font from Google Fonts
- `xlsx.full.min.js` from cdnjs (Excel import/export)

**Data persistence**: `localStorage` under the key `wedding_v3`. All CRUD operations call `saveData()` which serializes `db` to localStorage. The `loadData()` function provides default seed data if localStorage is empty.

## Sections & Features

| Section ID | Hebrew Title | Description |
|---|---|---|
| `overview` | סקירה כללית | Dashboard with stats, upcoming tasks, pending vendors, budget summary, unconfirmed guests |
| `budget` | תקציב | Budget items with planned/actual/paid amounts and payment status |
| `vendors` | ספקים | General vendor management with status tracking |
| `tasks` | משימות | Task checklist with priorities and due dates |
| `guests` | אורחים | Guest list with group, source (Andrea/Tinko), count, notification status (הודעה), RSVP (אישור), table assignment, notes |
| `seating` | סידורי ישיבה | Table definitions and guest-to-table assignment |
| `dj` | מוזיקה / DJ | DJ/music vendor comparison with curfew tracking |
| `bar` | שירותי בר | Bar vendor management (passive vs. active bar) |
| `disco` | דיסקו שקט | Silent disco vendor comparison (after 23:00 noise curfew) |
| `photography` | צילום ווידאו | Placeholder section (not yet implemented) |

## Data Schema (`db` object)

```js
db = {
  vendors: [{id, name, cat, contact, phone, price, paid, status, notes}],
  budget:  [{id, cat, desc, planned, actual, paid, status}],
  guests:  [{id, name, group, source, count, notified, rsvp, table, notes}],
  tasks:   [{id, name, cat, date, priority, done}],
  tables:  [{id, name, seats}],
  bar:     [{id, name, type, phone, price, hours, shots, status, notes}],
  disco:   [{id, name, phone, price, qty, channels, hours, status, notes}],
  dj:      [{id, name, type, phone, price, hours, curfew, sound, status, notes}],
  djSettings:   {budget, startTime, curfewTime, guestCount},
  barSettings:  {budget, startTime, endTime, guestCount},
  discoSettings:{budget, curfew, guestCount}
}
```

## UI Conventions

- **RTL layout** (`dir="rtl"`, `lang="he"`) — Hebrew is the primary language, some data in Spanish/English
- **Dark theme** with gold accent (`--gold: #c9a84c`) on dark backgrounds (`--bg: #0d0d0f`)
- **CSS variables** defined in `:root` at the top of `<style>`
- **Badge colors**: green=confirmed/paid, yellow=pending/partial, red=high-priority/cancelled, gray=not paid/default, blue=low-priority
- **Modals** use `.modal-overlay` + `.modal` pattern, toggled with `openModal(id)` / `closeModal(id)`
- **Action buttons** are rendered inline via `actionBtns(editFn, collection, id)` helper
- Editing uses a shared `editingId` global — `null` means new record, a number means editing that ID

## Key Functions

- `renderAll()` — re-renders all sections (called on every data change and section switch)
- `saveData()` / `loadData()` — localStorage persistence
- `showSection(name, el)` — nav routing
- `deleteItem(collection, id)` — generic delete with `confirm()` prompt
- `triggerImport(section)` / `exportSection(section)` — Excel import/export per section
- `downloadTemplate()` — generates an empty Excel template for the active import target

## Wedding Details

- **Date**: May 20, 2026
- **Guests**: 100–120
- **Venue type**: Outdoor garden (חיצוני בגן), not a closed hall
- **Noise curfew**: 23:00 → Silent disco takes over after that
- **Bar**: Passive bar + active service on the dance floor
- **Couple**: Tinko (Tinko) & Andrea

## Working in This Repo

- All changes go into `index.html` — edit HTML, CSS, and JS all in that one file
- No tests, no CI — verify by opening `index.html` in a browser
- The Photography section is intentionally a stub — clicking "Add vendor" shows an alert
- The sidebar date label still shows "20 מאי 2025" (needs updating to 2026)
- `updateCountdown()` is hardcoded to `new Date('2025-05-20')` — also needs updating to 2026
