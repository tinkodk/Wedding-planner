# BRD

## Goal
Show a small, unobtrusive "last updated" line at the bottom of the app's main content area, displaying today's date, without affecting app data, print output, or existing sections.

## User stories
- As a user, I want to see a "last updated" date at the bottom of the page so that I have a quick visual confirmation of how current the page content is.

## Acceptance criteria
- [ ] A footer element appears at the bottom of `#main`, after the last `.section` block, outside/below all section content.
- [ ] The footer is visible regardless of which section (`overview`, `budget`, `vendors`, `tasks`, `guests`, `seating`, `dj`, `bar`, `disco`, `photography`) is currently active, since it lives outside the per-section `.section` divs that `showSection()` toggles.
- [ ] The footer displays today's date, formatted consistently with the existing date convention used elsewhere in the app (the `${d}/${m}/${y}` numeric pattern used for the print report's "generated on" date, e.g. `10/10/2026`).
- [ ] The date is computed and written into the footer once on page load (alongside the existing `updateCountdown(); renderAll();` init calls), so it reflects the date the page was opened/viewed.
- [ ] The footer's text, color, and spacing are legible against the existing dark theme (uses `--text-dim`/`--border` or equivalent existing CSS variables), matches the Heebo font, and respects `dir="rtl"` alignment.
- [ ] The footer does not alter, read, or write any `db` data, localStorage schema (`wedding_v3`), or `syncConfig`/`SYNC_KEY` state — purely a display element.
- [ ] Adding the footer does not change the layout/scroll behavior of existing sections (no overlap, no clipped content, no new scrollbars introduced on any section).
- [ ] On narrow/mobile widths (≤700px, where the existing `@media(max-width:700px)` rule hides the sidebar and sets `#main{margin-right:0}` and reduces `.section` padding), the footer remains fully visible, readable, and does not overflow or get cut off horizontally.
- [ ] The footer's text/markup, once added, is a static UI element — it does not require any Excel import/export, modal, or CRUD wiring, and does not appear in any `actionBtns`/edit flow.

## Edge cases
- Page left open across midnight (e.g., opened before 00:00, still open after): per the plan's literal "today's date" interpretation, the footer does not auto-update live at midnight — it reflects the date at page load time only. This is expected given the "no live re-render" scope, but should not surprise QA.
- Very first load with no existing localStorage data (fresh `wedding_v3`, i.e. a new/empty `db` via `loadData()`'s default seed): footer must still render the current date correctly, since it depends only on `new Date()`, not on any stored data.
- RTL layout: footer text and any label/value ordering must read correctly right-to-left, consistent with the rest of the page (not just visually centered but actually RTL-correct given mixed Hebrew/numeric date text).
- Footer must be visually distinguishable from — and not duplicate or conflict with — the existing `.print-footer` ("הופק ב־..." line) used only inside the hidden print target (`#print-target`) for the guest-list print/export report; the two are separate elements for separate contexts.
- Sidebar collapsed/hidden at mobile widths: confirm the footer doesn't inherit any sidebar-relative margin that could misplace it once `#main{margin-right:0}` applies.

## Out of scope
- Any true "last saved" / "last edited" timestamp derived from actual data changes (`saveData()` calls) or from `syncConfig.lastSync` (cloud sync) — not part of this request as currently scoped; see Open Questions.
- Live/auto-updating the footer date without a page reload (e.g., via `setInterval` crossing midnight).
- Any change to the existing print/export report's `.print-footer` or its "הופק ב־" generated-date line.
- Any change to `updateCountdown()`'s hardcoded wedding date or the sidebar's "20 מאי 2025" label — unrelated pre-existing issues, not touched by this feature.
- Any new data field, localStorage key, or schema addition.

## Open questions
- Is "today's date" (the literal current/system date on page load) really what's wanted, or is the intent a true "last updated/saved" timestamp reflecting when the data was actually last edited (which would require storing a timestamp on every `saveData()` call — a data-touching, medium-tier change, not a cosmetic one)? Currently the app has no per-edit "last saved" timestamp at all; the only existing timestamp concept (`syncConfig.lastSync`) is for cloud sync, not local edits.
- Should this new footer be suppressed during print/export? Note: `#main` is already fully hidden during print via the existing `@media print{ #sidebar,#main,.modal-overlay{display:none!important;} }` rule, so a footer placed inside `#main` would already be excluded from print output by default, without needing extra CSS — confirm this default (automatic suppression) is the desired behavior, versus wanting the date shown in print too.
