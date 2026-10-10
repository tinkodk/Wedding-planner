# Plan

## Tier
small — purely cosmetic, static-text footer addition with no effect on `db` data shape, localStorage schema, or app logic.

## UI-facing
yes

## Stages to run
- UI/UX Designer: yes
- Security Architect: no

## Task breakdown
1. (UI/UX Designer) Decide exact placement, styling, and copy for the footer — it must sit at the bottom of the scrollable page content (end of `#main`, after the last `.section`), not be confused with the existing print-only `.print-footer` (used only inside the hidden print target around line 2521 of `index.html`), and must fit the existing dark theme / gold-accent / RTL conventions (`--text-dim`, `--border`, Heebo font, `dir="rtl"`).
2. (Developer) Add a small `<footer>`/`<div>` element with a static label plus a span with a stable id (e.g. `id="lastUpdated"`) at the end of `#main`, after all existing `.section` blocks and before `</div>` that closes `#main`.
3. (Developer) Add minimal CSS for the new footer (font size, color, padding, border-top) consistent with the existing style block near the top of `index.html`, scoped so it does not appear inside `.print-footer`/print output (reuse the existing `@media print{ #sidebar,#main,.modal-overlay{display:none!important;} }` rule's pattern — explicitly hide the new footer in print, or place it so it's naturally excluded, to avoid duplicating date info already shown in the print report).
4. (Developer) Add a small JS snippet that computes today's date, formats it using the same convention already used elsewhere in the file (the `` `${d}/${m}/${y}` `` pattern used for the print report's `gen` date), and writes it into the footer span on load — call this alongside the existing `// ============================= INIT =============================` block (`updateCountdown(); renderAll();`).
5. (Developer) Manually verify in-browser: footer renders at the bottom of every section/page view, text is legible against the dark background, RTL alignment looks correct, and the print preview/print output is unaffected (no duplicate or misplaced date).

## Open questions
- The request says "today's date," which is what will be implemented literally (footer always shows the current date on page load, every day). This is not the same as a true "last modified/saved" timestamp derived from `saveData()`/localStorage activity. If the user actually wants a real last-edited timestamp, that is a different (medium-tier, data-touching) feature — confirm literal "today's date" is really what's wanted before Developer builds it.
- Should this new footer be suppressed in the print/export view (to avoid duplicating the date already shown via `.print-footer`'s "הופק ב־" line), or shown there too? Recommend suppressing it in print unless told otherwise.
