# Mona's Project Pulse Dashboard — Implementation Plan

## Summary

Build a small static, runnable dashboard that renders project status cards from a local JSON data file. The app is served from `app/` via a VS Code launch configuration (`python3 -m http.server 5500`) that opens `index.html` directly (never a directory listing). Four files are in scope:

- `app/index.html` — semantic structure, data wiring, card rendering (Coder)
- `app/styles.css` — visual system, `.dashboard` + `.project-card` hooks, responsive/a11y polish (Designer)
- `app/project-data.json` — canonical `projects` array data source (Coder, informed by Designer's IA)
- `.vscode/launch.json` — deterministic launch config with `cwd: ${workspaceFolder}/app`, opens `index.html` (Coder)

The Orchestrator delegates: **Designer** owns UI/UX, IA, accessibility, visual design, and deterministic CSS hooks; **Coder** owns app structure, data schema, JS data wiring, and the launch config. The data schema is defined first so both HTML rendering and CSS hooks stay consistent.

---

## Ordered Implementation Steps

### Step 1 — Define the data schema (foundation)
Establish the canonical shape used by both markup and styling. Top-level `projects` array. Each project object:

- `name` (string, required)
- `owner` (string, required)
- `status` (enum-like string: `"On Track" | "At Risk" | "Blocked" | "Complete"`)
- `recentActivity` (short string, ≤120 chars recommended)
- `priority` (enum-like string: `"High" | "Medium" | "Low"`)

Seed with 4–6 realistic sample projects covering every status and priority value so visual states are exercised.

**Owner:** Coder (schema + seed data), Designer consulted on status/priority vocabularies to ensure they map cleanly to badge variants.
**File:** `app/project-data.json`

### Step 2 — Define information architecture and visual language (design spec)
Designer produces a short internal spec (can live as comments/section headers in `styles.css`) covering:

- Page regions: header (title "Project Pulse", subtitle/last-updated), main `.dashboard` grid, footer optional.
- Card anatomy: title (`name`), owner meta, status badge, priority indicator, recent activity line.
- Status badge color mapping (e.g., green/amber/red/gray).
- Priority treatment (e.g., left border accent or pill).
- Type scale, spacing scale, radius, shadow tokens (as CSS custom properties on `:root`).
- Responsive breakpoints (single column <640px, 2-col ≥640px, 3-col ≥1024px).
- Accessibility rules: color contrast ≥4.5:1, focus-visible outlines, non-color-only status cues (icon or text).

**Owner:** Designer
**File:** `app/styles.css` (spec captured as CSS custom properties + section comments)

### Step 3 — Build semantic HTML skeleton
Create `app/index.html` with:

- `<!doctype html>`, `<html lang="en">`, meta viewport, charset.
- `<title>Project Pulse</title>` and visible `<h1>Project Pulse</h1>`.
- `<link rel="stylesheet" href="styles.css">`.
- `<main class="dashboard">` container with an empty `<ul>` or `<section>` list for cards, `aria-label="Projects"`.
- A `<template id="project-card-template">` or a `<script>` block that fetches `project-data.json` and renders cards.
- Reference to `project-data.json` (via `fetch('./project-data.json')` in an inline `<script type="module">` or plain script).
- Empty-state and error-state DOM regions (hidden by default) to be toggled by JS.

Each rendered card must:

- Have class `project-card`.
- Include `name`, `owner`, `status` (as a `.status-badge` with a modifier class like `status-badge--on-track`), `recentActivity`, and `priority` (as `.priority` with modifier).
- Use semantic elements (`<article>`, `<h2>`, `<dl>` or `<p>`).
- Include `aria-label` or visually hidden text so status/priority are conveyed to screen readers without relying on color.

**Owner:** Coder (structure + data-fetch/render logic), Designer specifies exact class hooks and required child elements.
**File:** `app/index.html`

### Step 4 — Style the dashboard (visual polish)
Designer implements CSS against the structure from Step 3:

- `:root` design tokens (colors, spacing, radius, shadow, font-stack).
- Base/reset styles, typography, focus-visible.
- `.dashboard` — responsive CSS grid (`grid-template-columns: repeat(auto-fill, minmax(280px, 1fr))`), gap using spacing token.
- `.project-card` — background, `border-radius`, `box-shadow`, padding, hover/focus elevation.
- `.status-badge` and modifier variants (`--on-track`, `--at-risk`, `--blocked`, `--complete`).
- `.priority` treatments (`--high`, `--medium`, `--low`) — e.g., left border accent + label.
- Responsive breakpoints and print-safe fallback.
- `prefers-reduced-motion` respected on any transitions.
- `prefers-color-scheme: dark` optional but recommended.

**Owner:** Designer
**File:** `app/styles.css`

### Step 5 — Create the VS Code launch configuration
Coder writes `.vscode/launch.json` as strict JSON (no comments, no trailing commas):

- `version: "0.2.0"`.
- One configuration named exactly **`Run Project Pulse Dashboard`**.
- `type: "node"` or use the built-in Node debugger with `runtimeExecutable: "python3"` — the exercise brief and Step 3 prompt specify `python3 -m http.server 5500`.
- `cwd: "${workspaceFolder}/app"` (required).
- `program`/`args` to run `python3 -m http.server 5500` from `cwd`.
- `serverReadyAction` with `pattern` matching the http.server startup log and `uriFormat: "http://localhost:%s/index.html"` and `action: "openExternally"` (or `debugWithChrome` — pick `openExternally` for reliability in Codespaces).
- Ensures the browser opens `index.html`, not a directory listing.

**Owner:** Coder
**File:** `.vscode/launch.json`

### Step 6 — Integration verification
Coder + Designer together sanity-check that:

- JSON loads without CORS/file-protocol errors (must be served, not opened via `file://`).
- Every schema field renders in the card.
- Every status and priority value has a matching CSS variant.
- Launch config actually opens `index.html`.

**Owner:** Coder drives, Designer reviews.
**Files:** all four.

---

## File Assignments (consolidated)

| File | Primary Owner | Sections / Responsibilities |
|---|---|---|
| `app/project-data.json` | **Coder** | Top-level `projects` array; each item has `name`, `owner`, `status`, `recentActivity`, `priority`. 4–6 seed projects covering all status/priority values. Strict JSON. Designer consulted on enum vocabulary. |
| `app/index.html` | **Coder** | Doctype, `<title>Project Pulse</title>`, viewport meta, link to `styles.css`, `<main class="dashboard">`, fetch of `./project-data.json`, render loop producing `<article class="project-card">` elements with name/owner/status badge/recentActivity/priority. Empty + error states. Semantic + a11y attributes per Designer spec. |
| `app/styles.css` | **Designer** | `:root` tokens; base/reset; `.dashboard` responsive grid; `.project-card` with `border-radius` + `box-shadow`; `.status-badge` variants; `.priority` treatments; focus-visible; `prefers-reduced-motion`; optional dark mode; breakpoints. Must contain literal selectors `.dashboard` and `.project-card`. |
| `.vscode/launch.json` | **Coder** | Strict JSON, no comments. Config name `Run Project Pulse Dashboard`. `cwd: ${workspaceFolder}/app`. Command `python3 -m http.server 5500`. `serverReadyAction` opens `http://localhost:%s/index.html`. |

---

## Designer Responsibilities (explicit)

- **Information architecture:** page regions, card anatomy, ordering of fields for scanability (name → status → priority → owner → recentActivity).
- **Visual design:** color palette, typography scale, spacing scale, radius, shadow, badge and priority variants, hover/focus states.
- **Deterministic CSS hooks:** `.dashboard`, `.project-card`, `.status-badge` (+ modifiers), `.priority` (+ modifiers). Class names must be stable so Coder's render loop and the workflow keyphrase checks pass.
- **Accessibility:** WCAG AA color contrast; non-color status cues (icon/text); semantic landmarks; visible focus rings; screen-reader labels for badges; respects `prefers-reduced-motion`.
- **Responsive layout:** CSS grid `auto-fill` with `minmax`, works from ~320px up; no horizontal scroll.
- **Files owned:** `app/styles.css` (primary), advisory input to `app/index.html` class/attribute contract and `app/project-data.json` enum vocabularies.

## Coder Responsibilities (explicit)

- **App structure:** semantic HTML5, valid doctype, meta viewport, link to `styles.css`.
- **Data wiring:** `fetch('./project-data.json')` → parse → render loop → inject into `.dashboard`. Handle load failures with a visible error state. Escape text content (use `textContent`, not `innerHTML`).
- **Deterministic rendering:** exact class hooks (`project-card`, status/priority modifiers) matching Designer's contract.
- **`.vscode/launch.json`:** strict JSON, no comments; `cwd = ${workspaceFolder}/app`; runs `python3 -m http.server 5500`; `serverReadyAction` opens `http://localhost:%s/index.html`; configuration name is exactly `Run Project Pulse Dashboard`.
- **Files owned:** `app/index.html`, `app/project-data.json`, `.vscode/launch.json`.

---

## Dependencies Between Steps

- Step 2 (design spec) and Step 3 (HTML) both depend on Step 1 (schema).
- Step 4 (CSS implementation) depends on Step 3 (HTML class hooks exist) AND Step 2 (design tokens/spec defined).
- Step 5 (`launch.json`) depends only on `app/index.html` existing (so the URL target resolves). It does NOT depend on styling or data content.
- Step 6 (integration) depends on Steps 3, 4, 5.

---

## Work That Can Run in Parallel

Once Step 1 (schema) is agreed:

- **Parallel track A — Coder:** Step 3 (`app/index.html`) + Step 5 (`.vscode/launch.json`). Non-overlapping files with `styles.css`.
- **Parallel track B — Designer:** Step 2 spec + begin Step 4 (`app/styles.css`) using the agreed class contract from Step 3's spec. Designer can draft tokens and `.dashboard`/`.project-card` skeletons before HTML is finalized as long as the class contract is frozen.
- `app/project-data.json` seed data can be authored in parallel with HTML/CSS work once the schema is frozen.

## Work That Must Run Sequentially

- Step 1 (schema + class contract freeze) → everything else.
- Step 3 HTML class hooks must be finalized before Step 4 CSS can finish; if Designer starts CSS early, any hook change forces a CSS revision.
- Step 6 integration verification must be last.
- Git operations (staging/commit/push) are performed by the learner via Copilot CLI **after** Step 6, never by agents.

---

## Edge Cases to Handle

- **Empty `projects` array:** show an accessible empty state (`"No projects to display yet."`), not a blank page.
- **Fetch failure / malformed JSON:** show a visible error state with the error message; don't leave the UI silently empty.
- **Unknown `status` or `priority` value:** fall back to a neutral badge variant; don't crash the render.
- **Very long `name` or `recentActivity`:** wrap gracefully; consider `overflow-wrap: anywhere` and a max line count.
- **Missing optional field:** render the card without the missing row; never inject `undefined` as text.
- **XSS safety:** always use `textContent` when injecting values from JSON.
- **`file://` protocol:** `fetch` will fail; the launch config MUST serve via HTTP. Document this in a code comment or README note.
- **Port 5500 in use:** http.server will fail loudly; learner may need to change the port — keep the port in a single place in `launch.json`.
- **Directory listing exposure:** without `serverReadyAction` opening `index.html`, http.server would show a directory listing. The `uriFormat` must include `/index.html`.
- **Codespaces port forwarding:** `openExternally` is more reliable than `debugWithChrome` in Codespaces.
- **JSON strictness:** no trailing commas, no comments in `.vscode/launch.json` or `project-data.json` (validated by `python3 -m json.tool` in the exercise workflow).
- **Reduced motion / dark mode:** respect user prefs; do not force animations.
- **Accessibility:** status conveyed by color alone will fail a11y — include text label and/or icon.

---

## Validation Expectations

**Static checks (match `.github/workflows/3-step.yml`):**

- `app/index.html` contains the literal string `Project Pulse`, references `styles.css` and `project-data.json`, includes `project-card`, and shows `name`, `recentActivity`, `priority`.
- `app/styles.css` contains literal selectors `.dashboard` and `.project-card`, plus `border-radius` and `box-shadow`.
- `app/project-data.json` parses as JSON and has a top-level `projects` key with objects containing `name`, `owner`, `status`, `recentActivity`, `priority`.
- `.vscode/launch.json` parses as strict JSON (`python3 -m json.tool`), contains the exact configuration name `Run Project Pulse Dashboard`, includes `index.html` in the launch URL.

**Runtime checks (manual):**

- Open **Run and Debug** → **Run Project Pulse Dashboard** → green play.
- Browser opens `http://localhost:5500/index.html` (NOT a directory listing).
- Dashboard shows the `Project Pulse` title and all seed project cards.
- Each card shows name, owner, a colored status badge, recent activity, and a priority indicator.
- Resize window: layout reflows from 1 → 2 → 3 columns without overflow.
- Keyboard `Tab` reveals visible focus rings on interactive elements.
- DevTools console has no errors (no fetch failure, no JSON parse error).
- Screen reader announces status and priority via text, not color alone.
- Stop the server cleanly from VS Code.

**Accessibility spot-check:**

- Lighthouse/axe: no critical a11y violations.
- Color contrast on badges meets AA.

---

## Open Questions

1. **Runtime for the launch config:** the exercise brief and Step 3 prompt hard-code `python3 -m http.server 5500`. Confirm Python 3 is guaranteed in the Codespace (devcontainer). If not, the plan needs a Node-based fallback (`npx http-server`).
2. **Port conflict policy:** should the plan pick an alternate port if 5500 is busy, or fail loudly? Recommendation: fail loudly and document the fix.
3. **Dark mode:** in scope for this exercise or defer? Recommendation: include as progressive enhancement via `prefers-color-scheme`.
4. **Icons for status badges:** inline SVG vs. emoji vs. text-only. Recommendation: text-only + colored dot to avoid asset dependencies and keep the file scope to the four required files.
5. **JS module vs. inline script:** `type="module"` gives cleaner scoping but requires HTTP (which we already have via launch). Recommendation: `type="module"`.
6. **Last-updated timestamp:** brief mentions "recent activity" per project but not a global timestamp. Include a subtle "Last refreshed: <date>" in the header? Recommendation: optional, low priority.
7. **Enum canonicalization:** should `status` and `priority` values in JSON be lowercase-with-dashes (mapping to CSS modifier suffixes) or human-readable Title Case (mapped in JS)? Recommendation: store human-readable strings in JSON, normalize to modifier suffix in JS to keep JSON contributor-friendly.
