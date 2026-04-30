# AGENTS.md — Note Reader

## Project

- **Name:** Note Reader
- **Type:** Brownfield — fixing rendering bugs in existing single-file HTML app
- **Core Value:** Users learn to recognize notes on the grand staff through interactive quizzes
- **Repo:** https://github.com/bmtran/note-reader

## Architecture

- Single-file HTML (`index.html`) — no build step
- VexFlow 4.2.5 loaded from CDN for SVG music notation rendering
- All CSS and JavaScript inline
- State persisted via `localStorage`
- Deployed to GitHub Pages

## Key Constraints

1. **Must remain single-file HTML** — zero build step, open directly in any browser
2. **VexFlow 4 only** — upgrading to VF5 would require huge API changes
3. **No additional dependencies** — keep self-contained
4. **VexFlow 4 limitation:** No `setX()` method on notes

## Current Focus

**Phase 1: Fix Rendering Bugs**
- RENDER-01: Diatonic path hints should render as proper VexFlow notes (grayed out), not raw SVG ellipses
- RENDER-02: Brace connector between staves is cut off on the left side

## Decisions

| Decision | Status |
|----------|--------|
| VexFlow 4 + raw SVG for hints (current) | ⚠️ Revisit in Phase 1 |
| Single-file HTML | ✓ Locked |
| Inline CSS/JS | ✓ Locked |

## GSD Context

- Planning docs: `.planning/`
- Codebase map: `.planning/codebase/`
- Requirements: `.planning/REQUIREMENTS.md`
- Roadmap: `.planning/ROADMAP.md`
- State: `.planning/STATE.md`

