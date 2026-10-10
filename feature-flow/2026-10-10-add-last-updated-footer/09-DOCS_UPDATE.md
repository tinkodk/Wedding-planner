# Docs Update

## Changed
No doc changes needed — README.md documents the wedding project's content/status (venues, suppliers, budget, checklist), not the app's internal UI; this feature is a purely cosmetic, static footer in `index.html` with no new user-facing functionality, structure, or convention that README's scope covers.

CLAUDE.md was intentionally left untouched this run per instructions (it already has unrelated, pre-existing uncommitted line-ending changes being excluded from this feature's commit). If it were editable, the only change worth proposing is a single bullet under its existing "UI Conventions" section, to be applied once the file's pre-existing state is sorted out:

> - Static `.app-footer` div (last child of `#main`, after all `.section` blocks) shows a "last updated" date via `renderFooterDate()`, computed once from `new Date()` during init — no `db`/localStorage/sync involvement, and automatically excluded from print output since it lives inside `#main` (already hidden by the existing `@media print` rule).

This is optional/minor — the feature is small enough that CLAUDE.md's existing "Key Functions" list and architecture description don't strictly need updating (the function is trivial, one-off, and not part of any CRUD/render cycle), but the bullet above would help a future agent understand the footer exists and why it has no data impact.
