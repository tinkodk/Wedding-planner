# Development Pipeline Proposal

## Context
This document captures a discussion about setting up a structured multi-agent development pipeline
for the wedding planning app, to prevent back-and-forth and unverified "done" claims.

---

## The Problem We've Been Hitting

The back-and-forth happens because of two gaps:
1. **No spec** — "add numbers to circles" sounds simple but has hidden complexity (SVG rendering, RTL, hidden sections)
2. **No real test** — declaring "done" without actually opening a browser

---

## Proposed Pipeline (Tiered by Feature Size)

### 🟢 Always (every feature)
| Stage | Agent Role | Output |
|---|---|---|
| **1. Spec** | Product + UI/UX merged | BRD with acceptance criteria + visual description |
| **2. Build** | Developer | Code changes in `index.html` |
| **3. Verify** | QA Tester | Browser test via Chrome DevTools → screenshot proof |

### 🟡 Medium Features (new section, new modal, data schema change)
Add between Build and Verify:
| **2b. Code Review** | Reviewer | Check logic, edge cases, RTL, Hebrew encoding |

### 🔴 Large Features (new data model, external integration, Excel/PPTX parsing)
Add before Build:
| **1b. Tech Design** | Architect | Data schema, function contracts, failure modes |
| **1c. Security** | Reviewer | XSS, input validation, localStorage integrity |

---

## Agent Chain — What Each Stage Gets as Input

```
User request
    → Spec Agent      → BRD.md        (goal, user stories, acceptance criteria, edge cases)
    → UI/UX Agent     → UI_SPEC.md    (layout, colors, interactions, Hebrew/RTL notes)
    → [Arch Agent]    → TECH_DESIGN.md (schema changes, new functions, risks)
    → Dev Agent       → code changes  (references BRD + UI_SPEC)
    → [Review Agent]  → REVIEW.md     (issues found, approved / rejected)
    → QA Agent        → screenshot proof + pass/fail per acceptance criteria
```

---

## Recommended for This App (Single-File, No Build Pipeline)

**4 real stages, every time:**

1. **Spec** — Write a mini-BRD before touching any code (even for small features)
2. **Dev** — Implement against the spec
3. **Self-review** — Re-read the diff; explicitly check: RTL, Hebrew strings, edge cases, hidden-section rendering
4. **Browser test** — Use Chrome DevTools MCP to verify with a screenshot before saying "done"

---

## Implementation Options Discussed

| Option | Description |
|---|---|
| **A** | `PIPELINE.md` in the repo defines the process + templates; followed automatically for every request |
| **B** | Actual Claude subagents triggered via slash commands (`/spec`, `/build`, `/test`) |
| **C** | Both — documentation + lightweight automation |

---

## Next Session: Decide & Build

Pick one of A / B / C above and we'll implement the full pipeline.

Key questions to answer next session:
- Should the BRD/spec be stored as a temp file per feature, or accumulated in a log?
- For the QA stage, should the Chrome DevTools test be mandatory (block merge) or advisory?
- Do we want slash commands (`/feature "description"`) that kick off the whole chain automatically?
- Should the pipeline also auto-commit at each stage with a structured message?
