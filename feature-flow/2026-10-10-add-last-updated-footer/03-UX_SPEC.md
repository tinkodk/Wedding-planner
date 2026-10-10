# UX Spec

## Visual language notes
The app is a dark-theme, RTL, Heebo-font single-page dashboard with a fixed right-hand sidebar (`#sidebar`) and a scrollable `#main` content area. Relevant existing patterns this feature should echo:

- **Color palette**: `--bg:#0d0d0f` (page background), `--surface2:#1e1e24` (card backgrounds), `--border:#2a2a35` (hairline dividers), `--text:#e8e8f0` (primary text), `--text-dim:#8888a0` (secondary/meta text), `--gold:#c9a84c` (accent, used for emphasis/active states — not meta text).
- **Low-emphasis meta text precedent**: things like `.page-header p` (subtitle under each `<h1>`), `.stat-card .sub`, `.sidebar-date .sublabel`, and the print report's own `.print-footer` ("הופק ב־…") are all small (11px), `color:var(--text-dim)`, no bold weight. This is the exact register a "last updated" line belongs in — quiet, bottom-of-content, not competing with data.
- **Divider precedent**: `.section-divider{height:1px;background:var(--border);margin:20px 0;}` and the print footer's `border-top:1px solid #eee` show the app's existing convention for separating a trailing meta line from content above it with a thin `--border`-colored rule plus vertical breathing room.
- **Spacing rhythm**: `.section{padding:22px 34px;}` (18px/14px on mobile) is the horizontal margin everything else aligns to. A footer appended after the sections should respect this same horizontal inset so it doesn't look like it belongs to a different grid.
- **RTL**: `dir="rtl"` on `<html>`; numbers inside Hebrew-context strings (e.g. the print "generated on" date) already read correctly right-to-left by default — this is a plain text node, not a flex row needing manual reordering.
- **No existing "card chrome" needed here**: unlike `.stat-card`/`.overview-block`, nothing in the app wraps a single line of ambient status text in a bordered box — the closest precedent (print footer) is bare centered text with a top divider. The new footer should follow that bare-text pattern, not invent a new "card" treatment.

## Per-criterion UX

### Footer element appears at the bottom of #main, after the last `.section` block
- Layout: Single block-level element placed as the last child of `#main`, after `</div>` of the final `.section` (currently `sec-disco`), i.e. a sibling of the `.section` elements, not nested inside any of them. It sits below whichever section is currently visible, as the natural last thing in the scroll flow of `#main`.
- States: Always present and static — there is no empty/loading/error state for this element itself (it has one fixed piece of content: the date).
- Interaction: Non-interactive. No click target, no hover affordance needed — it is read-only ambient text, same tier as the print footer's "הופק ב־" line.
- Mobile: Same placement; it becomes the last visible thing when the user scrolls to the bottom of whichever section is open.

### Footer is visible regardless of which section is active
- Layout: Because it lives outside the `.section` divs (which are the things `showSection()` toggles via `display:none`/`.active`), it never gets hidden by section switching — it's simply always rendered at the bottom of `#main`'s scroll container.
- States: Identical regardless of active section — no section-specific variants.
- Interaction: None — switching sections (clicking sidebar nav items) has no effect on the footer.
- Mobile: Same behavior; sidebar nav collapses to the hidden `#sidebar` at ≤700px but section-switching logic is unaffected, so footer visibility logic is unaffected too.

### Footer displays today's date in the existing `${d}/${m}/${y}` numeric pattern
- Layout: A single short line of text, e.g. `עודכן לאחרונה: 10/10/2026` (exact label wording is the implementer's call, but it should read naturally RTL and match the terse, label-first phrasing style used elsewhere, e.g. "ימים לחתונה" above its value). Centered horizontally to match the print footer's centered convention, or left simply as a quiet left-aligned/right-aligned line consistent with `.page-header p` — centered is preferred since this is a standalone closing line, not a paired label/value row.
- States: One static state — the computed date string, always present once JS runs. No "loading" state is meaningful since this is synchronous on page load.
- Interaction: Plain text, not a button or link — no hover/focus/active states, no cursor change.
- Mobile: Text must wrap or shrink gracefully if the viewport is very narrow, but a short date string like `עודכן לאחרונה: 10/10/2026` fits comfortably on one line even at 320–375px widths given 11-13px font sizing used elsewhere; no truncation/ellipsis needed.

### Date computed and written on page load
- Layout/States: N/A visually — from a UX standpoint the only observable effect is that the footer always shows a value immediately on load, never a placeholder/blank flash. There should be no visible "flicker" where the footer is briefly empty before the date populates (same expectation users already have of `#daysLeft` and the rest of `renderAll()`'s output appearing instantly on load).
- Interaction: None (no re-render triggers tied to user actions).
- Mobile: Same — appears immediately, no layout shift once populated.

### Footer legible against dark theme, uses existing CSS variables, matches Heebo, respects RTL
- Layout: Centered (or consistently aligned) single line within the same horizontal padding as `.section` content (34px desktop / 14-18px mobile), with enough top margin/padding to visually separate it from the last section's content — mirror the print footer's `padding-top:12px;margin-top:12px` plus a `border-top:1px solid var(--border)` rule scaled to the dark theme.
- States: Text color should be `var(--text-dim)` against `var(--bg)`/`#0d0d0f` — this pairing is already used throughout the app (e.g. nav labels, stat card subs) and is the established "quiet meta text on dark background" contrast, so no new contrast check is needed beyond matching that existing precedent.
- Interaction: N/A (non-interactive, see above).
- Mobile: Font-size should stay in the 11-12px range already used for meta text elsewhere (not smaller), so it remains legible on phone screens without zooming; the divider line above it should span the same width as `.section` content at mobile padding (14px), not bleed edge-to-edge.

### Footer does not touch `db`, localStorage, `syncConfig`/`SYNC_KEY`
- Layout/States/Interaction: Purely visual — no checkbox, no sync icon, no data-driven badge. It must not visually imply "this reflects saved/synced data" (e.g. no cloud icon, no green "synced" badge) since it reflects page-load time only, not data state. Wording should avoid implying data freshness (e.g. avoid phrasing that reads like "last saved" if it might mislead users into thinking it tracks edits — see BRD's open question/edge case about this; a neutral "last viewed/opened" or simple date-only framing side-steps that ambiguity without blocking implementation).
- Mobile: No change in behavior.

### No layout/scroll regressions in existing sections
- Layout: The footer must not be `position:fixed`/`sticky` (that would overlay section content or require extra scroll-padding accounting) — it should be a normal static block flowing after the last `.section`, so each section's own internal scroll height and the overall `#main` scrollbar just grow by the footer's own compact height (roughly one line + divider + padding, ~40-50px), with no visual overlap.
- States: Verify visually that on every section (overview through disco) the footer appears once, at the true bottom, with no double borders/dividers stacking against a section's own trailing content.
- Interaction: Scrolling to the bottom of any section should reveal the footer with no "dead space" gap larger than the existing `.section{padding:22px 34px}` bottom padding, and no clipped/overlapping text.
- Mobile: Confirm no new horizontal scrollbar appears — the footer's text and padding must respect the same `.section{padding:18px 14px}` mobile inset so it never exceeds viewport width.

### Mobile width ≤700px: footer fully visible, no overflow
- Layout: At ≤700px, `#sidebar` is hidden and `#main{margin-right:0}` applies — the footer, having no sidebar-relative margin of its own, should simply span the full width of `#main` the same as every section does, so it inherits correct full-width placement automatically without a dedicated mobile override (confirm, don't assume, once built).
- States: Same single static state as desktop.
- Interaction: None.
- Mobile: Text must not overflow horizontally (no `white-space:nowrap` without a wrap fallback) — given the string is short, default wrapping behavior is a safe guard. The divider/line should sit within the same 14-18px side padding as mobile `.section` content, not touch the viewport edges.

### Static UI element — no CRUD/modal/Excel/actionBtns wiring
- Layout/States/Interaction: Deliberately has none of the patterns used by editable rows — no `actionBtns()`-style icon buttons, no modal trigger, no table row chrome, no edit/delete affordance. Visually it should read unambiguously as "informational line," not as a list item or data row (i.e. it must not visually resemble a `.table-card` row or `.stat-card`, which would wrongly imply it's interactive/data-backed).
- Mobile: Same — no tap target at all, so no minimum 44px tap-target concern applies here (nothing to tap).

## Accessibility notes
- Contrast: `var(--text-dim)` (#8888a0) on `var(--bg)` (#0d0d0f) is the same pairing already relied on elsewhere in the app for meta text; keep this exact pairing rather than introducing a dimmer custom color, to stay at least as legible as existing precedent.
- Font size: keep to the existing meta-text floor of 11-12px, not smaller — this app already treats that as its minimum readable size for secondary text.
- Reading/focus order: since the element is non-interactive, it should not receive a `tabindex` or appear in keyboard-focus order at all; it's read-only ambient content, so screen readers will encounter it in normal DOM/reading order at the end of `#main`'s content, which is the correct place (not tucked away with `aria-hidden`, since it is meaningful content, just not interactive).
- No tap-target sizing concerns apply (purely text, not a control) — but ensure line-height/padding give it enough visual breathing room that it isn't mistaken for a mis-clipped section edge on mobile.
