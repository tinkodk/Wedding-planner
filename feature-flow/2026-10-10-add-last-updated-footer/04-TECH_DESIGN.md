# Tech Design

## Approach
Add one static, non-interactive footer line as the last child of `#main`, sibling to the `.section` divs (after `sec-disco`, the last section before `</main>`). A single small JS function computes "today" in the app's existing `${d}/${m}/${y}` numeric convention and writes it into a `<span>` inside that footer, called once during the existing init sequence. No data, storage, or render-cycle (`renderAll()`/`showSection()`) involvement — this is pure, static, synchronous DOM text, mirroring the dark-theme-appropriate version of the existing `.print-footer` pattern.

## Changes

- **HTML markup** (`index.html`, between the closing `</div>` of `sec-disco` at line 547 and `</main>` at line 549 — currently a blank line 548):
  Insert:
  ```html
  <div class="app-footer">עודכן לאחרונה: <span id="footer-date"></span></div>
  ```
  This sits outside every `.section` div, so it is unaffected by `showSection()`'s `display:none`/`.active` toggling and is present under every tab (overview, budget, vendors, tasks, guests, seating, dj, bar, disco, photography).

- **CSS** (`<style>` block): add a new rule directly after `.section.active{display:block;}` (line 46), so it lives next to the `.section` layout rules it must visually align with:
  ```css
  .app-footer{text-align:center;padding:16px 34px 20px;margin-top:4px;border-top:1px solid var(--border);font-size:11px;color:var(--text-dim);}
  ```
  This is the dark-theme translation of `.print-footer`'s bare-text/divider pattern (`border-top` + muted small text), using `var(--border)`/`var(--text-dim)` instead of the print report's light-theme `#eee`/`#bbb`. Horizontal padding (`34px`) matches `.section{padding:22px 34px}` so the footer's text aligns with section content above it.
  Add a mobile override inside the existing `@media(max-width:700px){...}` rule (line 231), alongside the other section-padding overrides already listed there:
  ```css
  .app-footer{padding:14px 14px 16px;}
  ```
  matching `.section{padding:18px 14px}` at that breakpoint. No `#main`-margin-specific override is needed — `.app-footer`, like every `.section`, has no margin of its own and will correctly span full width once `#main{margin-right:0}` applies at ≤700px.

- **`renderFooterDate()`** (new function): purpose — compute "today" at call time and write it into `#footer-date`. Input: none (reads `new Date()` only, no `db`/localStorage access). Output: side effect only, sets `#footer-date`'s `textContent` to `${d}/${m}/${y}` using the same construction already used for the print report's "generated on" date (`today.getDate()`, `today.getMonth()+1`, `today.getFullYear()`, no zero-padding, same as existing `gen` variable in `buildPrintHTML`). Mirrors the existing `updateCountdown()` style (`document.getElementById(id).textContent = ...`). Place the function definition right after `doPrint()` and before the `// ============================= INIT =============================` comment, optionally under its own `// ============================= FOOTER =============================` comment for consistency with the file's existing section-comment convention.

- **Init wiring**: in the existing init block at the bottom of the script,
  ```js
  updateCountdown();
  renderAll();
  if(syncConfig.lastSync) updateSyncStatus('synced');
  ```
  insert the new call directly after `renderAll();` and before the `syncConfig` line:
  ```js
  updateCountdown();
  renderAll();
  renderFooterDate();
  if(syncConfig.lastSync) updateSyncStatus('synced');
  ```
  Do **not** call `renderFooterDate()` from inside `renderAll()` or `showSection()` — those run on every CRUD action and section switch, and this value has no reason to be recomputed on that cadence (BRD explicitly scopes this to "once on page load"). A single unconditional call in init guarantees the date is present before first paint settles, with no placeholder/flicker state, regardless of whether `db` is freshly seeded or loaded from `localStorage`.

## Data/schema impact
None. `renderFooterDate()` does not read or write `db`, does not call `saveData()`/`loadData()`, does not touch `localStorage['wedding_v3']`, and does not reference `syncConfig`/`SYNC_KEY`. No new field is added to any existing record shape. The only new persistent artifact is the static `<div class="app-footer">` markup itself, which carries no data attributes.

## Failure modes
- **`#footer-date` element missing or ID typo'd**: `document.getElementById('footer-date')` would return `null`; guard with `const el=document.getElementById('footer-date'); if(el) el.textContent=...;` so a markup mismatch fails silently (footer line simply shows the label with no date) rather than throwing and interrupting the rest of the init sequence (`updateCountdown()`/`renderAll()` must still run even if this one call errors — defensive null-check prevents a thrown `TypeError` from a missing element from blocking anything that happens to be sequenced after it).
- **Clock/timezone oddities (`new Date()` returns an unexpected value)**: no validation needed beyond what `updateCountdown()` and the print report already rely on (same `new Date()` API, same lack of timezone handling) — out of scope to harden beyond existing precedent.
- **No storage/network failure surface**: unlike most features in this app, this one never calls `saveData()`/`loadData()`, never hits `localStorage.setItem`, and never calls the Gist sync functions — so "storage full," "corrupt JSON," and "sync failure" failure modes that apply elsewhere in this codebase do not apply here at all. This should be called out to QA explicitly so no time is spent testing storage-quota/corruption scenarios against this feature.
- **Double-render / duplicate footer**: since the element is static markup (not re-injected via `innerHTML` on every `renderAll()`), there is no risk of the footer being duplicated or re-appended on repeated section switches or data edits.
- **Midnight rollover while page stays open**: by design (per BRD edge case), the date is computed once at load and does not update live; this is expected behavior, not a bug, and should not be flagged by QA as a defect.

## Test strategy outline
Maps to BRD acceptance criteria; use chrome-devtools (Elements, Application, Rendering, device-toolbar) plus manual interaction in a live reload of `index.html`.

1. **Placement/structure** (AC1, AC2): In Elements panel, confirm `.app-footer` is a direct child of `#main` and a *sibling* of every `#sec-*` `.section` div (not nested inside any of them). Click through every sidebar nav item (overview → photography) and confirm the footer remains visible at the bottom of `#main` regardless of which `.section.active` is current — this should require no additional interaction since it is outside the toggle logic entirely.
2. **Date format/value** (AC3): On fresh load, read `#footer-date`'s text and confirm it equals today's date rendered as `${getDate()}/${getMonth()+1}/${getFullYear()}` (e.g. `10/10/2026`), matching the format used by the print report's "הופק ב־" line — cross-check by opening the print preview modal and comparing the two date strings' format (not value, since they're independent calls).
3. **Load-time computation, no flicker** (AC4): Reload the page with the Elements panel open on `#footer-date`; confirm it is never empty/placeholder after `renderAll()`/`updateCountdown()` run — should be populated in the same paint as the rest of initial content, no visible blank state.
4. **Styling/theme/RTL** (AC5): Via Computed styles, confirm `.app-footer` uses `color: var(--text-dim)` (`#8888a0`), `font-family` inherits Heebo, font-size 11-12px, and a 1px `var(--border)` top divider. Confirm visually that the Hebrew label + numeric date read correctly right-to-left (label first, then number, no manual reordering needed — same as the print footer's precedent).
5. **No data/storage impact** (AC6): In Application → Local Storage, snapshot `wedding_v3` and (if present) the sync-config key before and after load/reload; confirm byte-identical. Confirm no new localStorage key is created. Confirm no network calls fire related to this change (Network panel).
6. **No layout/scroll regression** (AC7): For each section, scroll `#main` to the bottom and confirm the footer appears once, directly after that section's own trailing content, with no overlapping text, no doubled divider lines, and no new unexpected scrollbar on any section. Compare `#main` scrollHeight before/after the change only grows by the footer's own height (~40-50px).
7. **Mobile ≤700px** (AC8): Using the device toolbar, set viewport width to 375px/320px and ≤700px breakpoint; confirm `#sidebar` hides, `#main{margin-right:0}` applies, and `.app-footer` spans full width with no horizontal overflow (check `document.documentElement.scrollWidth` vs `innerWidth` — should match, no horizontal scrollbar). Confirm the date text does not clip or require horizontal scrolling.
8. **Non-interactivity / no CRUD wiring** (AC9): Confirm in Elements that `.app-footer` has no `onclick`, no `tabindex`, is not reachable via Tab-key keyboard navigation, has no button/icon chrome, and does not appear in any modal, `actionBtns()` output, or Excel import/export flow.
9. **Print suppression (edge case)**: Use Rendering panel → "Emulate CSS media type: print" (or actually trigger print preview) and confirm `#main` (and therefore `.app-footer`) is hidden per the existing `@media print{#sidebar,#main,.modal-overlay{display:none!important;}}` rule, and that the unrelated `#print-target`'s own `.print-footer` ("הופק ב־…") is untouched and still renders normally.
5b. **Fresh/first-load edge case**: Clear `localStorage['wedding_v3']` via Application panel, reload, confirm `db` seeds to defaults as before and `.app-footer`/`#footer-date` still render today's date correctly (proving no dependency on `db` contents).

## Non-goals
- No true "last saved/edited" timestamp tied to actual `saveData()` calls or `syncConfig.lastSync` — this footer reflects page-load time only, per BRD's explicit scoping (see BRD Open Questions).
- No live/auto-updating of the date across a midnight rollover while the page stays open — computed once at load, matching the BRD's literal "today's date" interpretation.
- No change to the existing print/export report's `.print-footer` element, its "הופק ב־" generated-date line, or `buildPrintHTML()`/`openPrintPreview()` logic — those are a separate, already-correct implementation for a separate context (the hidden print target).
- No change to `updateCountdown()`'s hardcoded `new Date('2025-05-20')` or the sidebar's "20 מאי 2025" label — unrelated pre-existing data issues, out of scope for this feature.
- No new data field, localStorage key, or `db` schema addition of any kind.
- No suppression logic is needed to hide the footer during print — it is already excluded automatically because it lives inside `#main`, which the existing `@media print` rule already sets to `display:none!important`.
- No CRUD affordance, modal trigger, Excel import/export wiring, or `actionBtns()` usage — purely static, read-only text.
