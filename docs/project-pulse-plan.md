# Mona's Project Pulse — Implementation Plan

Date: 2026-06-11

This document is an actionable implementation plan for building "Mona's Project Pulse" — a compact, accessible dashboard showing project health, status, and high-level metrics. The plan is organized into ordered phases, maps file ownership to agents (Coder / Designer), lists dependencies, parallelization opportunities, validation steps, edge cases/risks, and open questions for the Orchestrator.

Save this file as: docs/project-pulse-plan.md

---

## 1) Summary

Mona's Project Pulse is a single-page dashboard that provides a quick status overview of projects (progress, risk/health, owners, upcoming milestones). Goals:

- Present critical project signals in a glanceable, accessible, and responsive interface.
- Use a simple static data source (JSON) to enable rapid local preview in Codespaces (no build system required).
- Provide clear visual cues (badges/colors), compact cards/list view, and a top-level KPI strip.
- Be easy to iterate on: Designer provides CSS system and accessibility hooks; Coder provides HTML structure, JSON sample data, and minimal JavaScript to render the data.

Minimum artifacts the Coder must create/modify in the repo (explicitly required by the task):
- app/index.html
- app/styles.css
- app/project-data.json
- .vscode/launch.json

---

## 2) Ordered Implementation Steps (Phases)

Each phase includes purpose, acceptance criteria, file-level assignments, owner, validation, and estimated effort.

Phase 1 — Project skeleton and preview configuration
- Purpose: Create the workspace files and a VS Code launch configuration so teammates can open and preview index.html from Codespace easily.
- Acceptance criteria:
  - app/index.html exists with a minimal placeholder page.
  - app/styles.css exists and is linked from index.html.
  - app/project-data.json exists with example data (schema + 3 sample projects).
  - .vscode/launch.json exists with a "Launch index.html" configuration.
  - Opening the launch config in Codespace opens index.html (or the file opens in the browser/editor reliably).
- File assignments:
  - Create: app/index.html (Coder)
  - Create: app/styles.css (Designer & Coder: Designer will later refine; initial file created by Coder)
  - Create: app/project-data.json (Coder)
  - Create: .vscode/launch.json (Coder)
- Owner: Coder (setup); Designer may contribute to styles.css contents after.
- Validation / manual checks:
  - Open VS Code Run & Debug → select "Launch index.html" → Start Debugging. Index page opens in the browser or preview.
  - Manually open app/index.html in editor to confirm links to styles.css and JSON sample.
- Estimated effort: small

Phase 2 — Data model and sample dataset
- Purpose: Define the project's data shape and populate app/project-data.json with representative sample data for UI development and edge-case testing.
- Acceptance criteria:
  - Clear JSON schema in project-data.json (commented or documented) with fields: id, name, owner, status (enum), progress (0-100), health (enum), nextMilestone {name, date}, tags (array), optional notes.
  - At least 6 sample projects including: on-track, behind schedule, at-risk, completed, and one with missing optional fields and one with very long name/owner.
- File assignments:
  - Modify/create: app/project-data.json (Coder; Designer may request additional fields)
  - Add: docs/project-data-schema.md (Coder) — optional but recommended to reduce ambiguity (small text doc).
- Owner: Coder
- Validation:
  - Manually open app/project-data.json and verify valid JSON (use VSCode JSON validation).
  - Load the JSON via the Coder's rendering script (Phase 3) and confirm all samples render (or fall back gracefully).
- Estimated effort: small

Phase 3 — Static HTML structure & accessibility scaffolding
- Purpose: Add semantic HTML skeleton for header, KPI strip, project list/cards, and placeholders for interactive controls (filter/search).
- Acceptance criteria:
  - app/index.html contains semantic HTML5 structure: header (h1), ARIA-compliant top KPI strip, main region with role="region" and aria-labels, empty container element(s) where JS will inject project cards/list.
  - Designers and accessibility hooks (data-* attributes, CSS classes) are present so Designer can target visuals.
- File assignments:
  - Modify: app/index.html (Coder)
  - Update: app/styles.css (Designer owns visual rules but initial layout scaffolding added by Coder)
- Owner: Coder (primary) with Designer input
- Validation:
  - Manual check: use browser DevTools to inspect semantic tags and aria-* attributes.
  - Keyboard-only navigation: Tab through header and top-level controls; focus states should be visible (initial CSS may provide minimal styling).
- Estimated effort: small/medium

Phase 4 — Minimal rendering logic (vanilla JS)
- Purpose: Implement a small, dependency-free renderer to fetch project-data.json and populate the page, with simple client-side filtering (status) and sorting (by progress or nextMilestone).
- Acceptance criteria:
  - app/index.html loads a script (app/scripts/render.js or inline <script>) that fetches app/project-data.json via fetch() and renders project entries into the DOM.
  - Renderer supports: basic error handling (JSON load failure), empty-state view (no projects), and an accessible list of cards.
  - Basic controls: status filter (All / On-track / At-risk / Behind / Completed) and sort selector.
- File assignments:
  - Create: app/scripts/render.js (Coder)
  - Modify: app/index.html to include the script and container elements (Coder)
  - Potentially modify: app/styles.css to add classes used by JS (Designer will finalize).
- Owner: Coder
- Validation:
  - Open the page and verify data renders.
  - Test filter and sort controls; verify UI updates.
  - Simulate JSON load failure by renaming project-data.json temporarily to see error state.
- Estimated effort: medium

Phase 5 — Design: visual system, responsive and accessible styles
- Purpose: Designer produces the visual language (colors, spacing, badges, typography, responsive rules), writes CSS classes in app/styles.css, and provides visual assets (if any).
- Acceptance criteria:
  - app/styles.css contains:
    - CSS variables for color scale (success/warn/error/neutral), spacing tokens, and typography.
    - Classes for .kpi-strip, .project-card, .status-badge, .avatar (owner), .progress (bar), and responsive layout rules (grid for >= 900px, stacked for mobile).
    - Visible focus states and accessible color contrast >= WCAG 2.1 AA for text and at least large-text 3.0 where possible for badges.
    - Utility classes for hidden/accessibility-only content (.sr-only).
  - Provide a small design token comment block at top of styles.css documenting intended use and class hook patterns for Coder.
- File assignments:
  - Modify: app/styles.css (Designer)
  - Optionally add images: app/assets/*.svg (Designer)
- Owner: Designer
- Validation:
  - Manual visual checks at different viewport widths (mobile/tablet/desktop).
  - Use browser Accessibility tools and color contrast checker (or axe) to verify contrast.
  - Keyboard navigation: ensure focus outlines are present and meaningful.
- Estimated effort: medium

Phase 6 — Interactivity and progressive enhancement
- Purpose: Improve JS to support deeper interactive features (expand/collapse card details, keyboard shortcuts, accessible modals/tooltips), and ensure progressive enhancement (works without JS with sensible fallback content).
- Acceptance criteria:
  - Expandable project details available; expand/collapse is keyboard operable (Enter/Space) and uses aria-expanded.
  - Tooltips or inline help accessible (aria-describedby).
  - When JS disabled, page still displays a readable list or links to JSON data.
- File assignments:
  - Modify: app/scripts/render.js (Coder)
  - Modify: app/index.html (Coder)
  - Modify: app/styles.css (Designer for states)
- Owner: Coder + Designer collaboration
- Validation:
  - Keyboard-only expansion test.
  - Disable JS in the browser and verify fallback is reasonable.
- Estimated effort: medium/large

Phase 7 — Accessibility audit, performance checks, and polish
- Purpose: Run accessibility checks and performance smoke tests; address issues; finalize copy and microcopy for statuses and KPIs.
- Acceptance criteria:
  - Run a11y tooling (e.g., axe devtools) and resolve any critical failures (aria roles, contrast, focus traps).
  - Performance is acceptable for small datasets; UI renders within <300ms on Codespace local preview for sample data.
  - Final copy for headline metrics and status descriptions approved.
- File assignments:
  - Update: app/styles.css and app/index.html as needed (Designer + Coder)
  - Update: app/project-data.json for final sample content (Coder)
  - Add: docs/QA-checklist.md (Coder) — optional
- Owner: Designer + Coder
- Validation:
  - Run axe or use browser devtools accessibility panel; manually test with a screen reader (NVDA/VoiceOver) if available.
  - Resize to mobile and test again.
- Estimated effort: medium

Phase 8 — Documentation and handoff
- Purpose: Document the design system hooks, JSON schema, how to run locally in Codespace, and next steps for extending to a real API.
- Acceptance criteria:
  - docs/project-pulse-plan.md (this document) saved to docs/.
  - docs/project-data-schema.md (if not created earlier).
  - README update with instructions: how to preview locally in Codespace and accepted JSON fields.
- File assignments:
  - Modify: README.md or create docs/README-pulse.md (Coder)
  - Modify: docs/project-data-schema.md (Coder)
- Owner: Coder (with Designer review)
- Validation:
  - Follow the README steps in a fresh Codespace environment to confirm they are complete and accurate.
- Estimated effort: small

---

## 3) Designer Responsibilities (detailed)

Deliverables:
- app/styles.css (primary file)
- Optional assets: app/assets/*.svg (icons for status, avatars)
- Design tokens documented at top of styles.css (CSS variables for color, spacing, font sizes)
- Accessibility helper classes (.sr-only, .focus-ring)

Specific tasks:
- Define color palette with semantic variables:
  - --color-success, --color-warn, --color-error, --color-neutral, --text-primary, --bg
- Provide CSS classes and hooks:
  - .kpi-strip (layout for KPI)
  - .kpi (individual metric)
  - .project-list (grid container)
  - .project-card (card)
  - .status-badge (accepts status modifiers: .status--success, .status--warn, .status--error)
  - .progress (progress bar), with accessible text alternative
  - .filters (controls area) and .control (filter/select)
- Responsive rules:
  - Mobile-first: stacked cards; breakpoint at 640px for two-column, 900px+ grid for 3+ columns.
- Accessibility requirements:
  - Ensure focus states for interactive elements are visible and large enough contrast.
  - Ensure color-only is not the only indicator of status (add icons or text).
  - Ensure adequate contrast: text >= 4.5:1 for regular text, badges at least 3:1 for larger/glyphs.
- Expected deliverables include clear comments showing class purposes and where Coder should attach data-* attributes.

Designer ownership (to avoid merge conflicts):
- Primary modifications to: app/styles.css and any app/assets/*
- Provide style tokens/comments — keep any non-critical changes small to avoid interfering with HTML/Coder changes.

---

## 4) Coder Responsibilities (detailed)

Deliverables:
- app/index.html — semantic HTML structure and script/link tags
- app/project-data.json — sample data and schema
- app/scripts/render.js — small, dependency-free renderer
- .vscode/launch.json — preview config for Codespace
- docs/project-data-schema.md and docs/QA-checklist.md (optional, recommended)
- Minimal accessibility attributes and ARIA hooks in HTML (aria-* attributes)

Specific tasks:
- Build a semantic HTML base:
  - Header with H1 "Mona's Project Pulse"
  - Top KPI strip container with placeholders (data will be injected)
  - Main content container (<main> with role="main") with an element like <div id="project-list" aria-live="polite"></div>
- Implement data fetch:
  - Use fetch('/app/project-data.json') or relative path; include try/catch and timeout/backoff messaging for failure.
- Render logic:
  - Map JSON to DOM nodes with proper roles, data-status attributes, aria-labels, aria-describedby for additional details
  - Implement simple filtering and sorting UI wiring (no heavy frameworks).
- Add minimal JS file structure:
  - app/scripts/render.js (ES module or IIFE) that can be extended later.
- Provide launch config:
  - .vscode/launch.json config named "Launch index.html"

Coder ownership (to avoid conflicts):
- Primary modifications to: app/index.html, app/project-data.json, app/scripts/*
- Coordinate with Designer for any class names or data attributes; use the contract in styles.css comments.

---

## 5) Dependencies

Required:
- Codespaces or VS Code (recommend using GitHub Codespaces for preview)
- Modern browser (Chrome, Edge, Firefox, Safari) for manual testing
- VS Code Run/Debug (built-in)
- No Node/npm required for the minimal implementation (we will use static files + fetch)

Recommended:
- Live Server extension (ritwickdey.liveserver) — simplifies local preview
- VS Code "Simple Browser" or "Open in Browser" extension to preview HTML
- Accessibility tools: axe DevTools (browser extension) or Lighthouse in DevTools
- Screen reader for manual accessibility testing (NVDA for Windows, VoiceOver for macOS)
- Optional: prettier / stylelint for consistent CSS and formatting

Codespace/CLI requirements:
- The Orchestrator will run tasks in a Codespace using GitHub Copilot CLI. No git commits from agents — human will run code changes.
- Ensure .vscode/launch.json is compatible with Codespaces debug environment (the launch config provided will be best-effort).

---

## 6) Parallel Work Decisions

Work that can run in parallel (no overlap/conflict if team follows file ownership above):
- Designer: Build and refine app/styles.css and assets (Designer owns styles file)
- Coder: Create app/project-data.json, app/index.html skeleton (only minimal CSS references), and .vscode/launch.json
- Coder: Start render.js to fetch JSON and render placeholder content while Designer works on styles

Work that must be sequential:
- Final integration of JS rendering with final styles — must wait until Designer finalizes class names and tokens (or both agree on a contract early).
- Accessibility audit (Phase 7) must run after the HTML/CSS/JS are integrated.

To avoid merge conflicts:
- Designers modify only app/styles.css and app/assets/*
- Coders modify only app/index.html, app/scripts/*, app/project-data.json, and .vscode/*
- Agree on class name conventions early (a short comment block at top of styles.css listing class hooks).

---

## 7) Validation Expectations (how to verify each step)

Previewing the app in Codespace:
- Use Run & Debug → select "Launch index.html" → Start. This should open the page in a browser preview that loads app/index.html.
- If "Launch index.html" doesn't open in the browser, use Live Server: right-click app/index.html → "Open with Live Server" (if extension installed) or open the file directly in the browser via the Codespaces forwarded port.

Smoke tests per phase:
- Phase 1: Confirm index.html loads and shows placeholder text.
- Phase 2: Validate JSON syntax (VSCode JSON lint).
- Phase 3: Verify semantic tags and aria attributes exist.
- Phase 4: Confirm projects appear; try filters and sorting; debug console for fetch errors.
- Phase 5: Visual checks across breakpoints:
  - Desktop: grid layout with 3 columns.
  - Tablet: 2 columns.
  - Mobile: stacked cards.
- Phase 6: Keyboard-only navigation and expand/collapse using Enter/Space.
- Phase 7: Run axe DevTools and resolve any critical a11y violations.

Sample data validations:
- project-data.json must parse as JSON and contain fields required by renderer.
- Validate date strings are ISO 8601 (YYYY-MM-DD) for consistent parsing.

Quick smoke test steps:
1. Open Codespace and Run -> Launch "Launch index.html".
2. Confirm the KPI strip shows sample metrics.
3. Verify at least three project cards render.
4. Click a status filter and confirm list reduces to matching projects.
5. Resize browser to mobile width and confirm layout stacks.
6. Tab through interactive elements — focus outline visible and operable.

Automation: No CI required for this MVP, but later a simple HTML validation or lint step could be added.

---

## 8) Edge Cases and Risks to Handle

Data issues:
- Missing fields: projects may lack owner, progress, or nextMilestone — render fallback text (e.g., "Unassigned", "—", "No upcoming milestone").
- Null or malformed dates: treat as "TBD" and avoid crash during parsing.
- progress >100 or <0: clamp values to 0-100.
- status values outside known enums: map to "unknown" and use neutral styling.

UI issues:
- Very long project names or owner names:
  - CSS should truncate with ellipsis and reveal full text on hover via title attribute or accessible tooltips.
- Many projects (hundreds):
  - Performance: client-side rendering of many nodes may be slow. For MVP render up to 200 entries; later implement virtualization if needed.
- Multiple statuses per project:
  - If there are conflicting signals, show primary status and list additional tags.

Accessibility pitfalls:
- Relying on color alone — add text labels and icons.
- Focus traps when showing modals: ensure focus is returned to triggering element.
- Keyboard access to all interactive controls.

Security:
- Using static JSON eliminates remote fetch risk; if later connecting to remote API, enforce CORS and secure endpoints.

Dependencies & environment:
- Launch config may behave differently across environments — include fallback instructions to open file manually.

---

## 9) Open Questions for the Orchestrator / Mona

Design & content:
1. Preferred brand colors or palette for status (success/warn/error/neutral)? If none, Designer will propose a neutral accessible palette.
2. Do you want avatars for owners or just initials? If avatars, will images be provided or generated from initials?
3. Which KPIs should appear in the KPI strip? (e.g., total projects, % on-track, % at-risk, average progress)

Data & interactivity:
4. Is the data source static JSON acceptable for the short term, or should we plan for a backend API earlier?
5. Do you want real-time refresh (polling) or manual refresh button?
6. Scope of interactivity: filters only, or also inline editing of project status/progress?

Accessibility:
7. Required accessibility standard baseline (WCAG 2.1 AA is recommended) — is this acceptable?

Performance:
8. Expected scale (how many projects should the UI handle)? This affects plan for virtualization/pagination.

Handoff & deployment:
9. Are there requirements to deploy this to GH Pages or an internal site? If so, we should add a deploy phase and possible build step.

---

## File Assignment Summary (to reduce merge conflicts)

- Coder (primary):
  - app/index.html
  - app/project-data.json
  - app/scripts/render.js
  - .vscode/launch.json
  - docs/project-data-schema.md (optional)
  - docs/QA-checklist.md (optional)
- Designer (primary):
  - app/styles.css
  - app/assets/* (icons, svgs)
- Joint edits:
  - README.md / docs/README-pulse.md (Coder writes, Designer reviews copy)

Notes:
- Keep changes small and announce class-name choices early to avoid conflicts.
- Prefer prefixed classes (pulse-*) to avoid style collisions with other repo components.

---

## Quick Implementation Timeline & Estimated Effort (suggested)

- Day 0 (small): Phase 1 & Phase 2 — skeleton + JSON (Coder) — small
- Day 1 (small/medium): Phase 3 & initial render (Coder) — small/medium
- Day 2 (medium): Designer finalizes styles.css (Designer) — medium
- Day 3 (medium): Coder wires JS to final styles + interactivity (Coder) — medium
- Day 4 (medium): Accessibility audit and polish (Designer + Coder) — medium
- Day 5 (small): Documentation and handoff (Coder) — small

Actual timing depends on availability and clarifications from Orchestrator for questions above.

---

## Final Notes

- Do not commit changes from agents; the Orchestrator will run edits locally in Codespaces and commit via Copilot CLI.
- Use the file ownership guidance to avoid merge conflicts — communicate class names and data schema early.
- Keep the implementation dependency-free for rapid previews; add tooling later if a build or API integration is required.

If you want, I can:
- Produce a starter app/index.html skeleton and a sample project-data.json schema (as a code artifact in this plan) for the Coder to copy into the workspace.
- Draft a suggested color palette and a minimal set of CSS variables to seed app/styles.css for the Designer.

Which of those would you like to receive next?