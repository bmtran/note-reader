# Note Reader

## What This Is

A single-file web app that teaches music note reading on the grand staff through interactive quizzes. The app presents a random note on treble or bass clef, the user identifies it by clicking C–B buttons or pressing keyboard keys, and receives correct/wrong feedback with streak tracking and a progressive hint system.

## Core Value

Users can quickly learn to recognize notes on the grand staff through repeated, low-friction practice with visual feedback and optional progressive hints.

## Requirements

### Validated

- ✓ Render grand staff (treble + bass clef) with VexFlow 4 — existing
- ✓ Present random notes from defined treble/bass ranges — existing
- ✓ User answers via 7 rainbow buttons (C–B) or keyboard (C–B keys) — existing
- ✓ Score, streak, and best streak (localStorage) tracking — existing
- ✓ Clef toggle: Both / Treble / Bass — existing
- ✓ Progressive hint system (Show Path → Show Labels → Reset) — existing
- ✓ Correct/wrong visual feedback with green/red flash — existing

### Active

- [ ] Fix diatonic path hint rendering — hints should render as proper VexFlow notes (grayed out), not raw SVG ellipses with hardcoded Y positions
- [ ] Fix brace connector — brace joining treble and bass staves is cut off on the left side

### Out of Scope

- Multiplayer or leaderboard — single-user learning tool, not competitive
- Audio playback of notes — beyond current VexFlow-only scope
- Mobile-native app — web-first, already works responsively
- Other clefs (alto, tenor) — grand staff is the target domain
- Music theory beyond pitch recognition — scope is note identification only

## Context

- **Technical environment:** Single-file HTML app (~18KB), no build step, uses VexFlow 4 via CDN. Currently bundles all CSS and JS inline.
- **Rendering engine:** VexFlow 4.2.5 (Bravura font) loaded from jsdelivr CDN.
- **Known limitation:** VexFlow 4 doesn't support `setX()` on notes. Current workaround uses raw SVG `<ellipse>` elements for hints, which causes positioning inaccuracies.
- **Hint system design:** Two-stage progressive — Show Path draws diatonic approach notes, Show Labels adds note name labels above/below.
- **Deployment:** GitHub Pages from `main` branch of `bmtran/note-reader` repo.
- **Prior work:** Fixed two VexFlow rendering bugs already (`setX` API issue, broken `VF.Annotation` replaced with raw SVG `<text>`).
- **User:** Building this for personal music education practice, but benefiting from clean code and good UX.

## Constraints

- **No build step:** Must remain a single-file HTML app that can be opened directly in any browser or served statically.
- **VexFlow 4 only:** Upgrading to VexFlow 5 would require significant API changes across the entire rendering pipeline.
- **No external dependencies beyond VexFlow:** Keep self-contained and easy to deploy.
- **Browser compatibility:** Modern browsers with SVG and `backdrop-filter` support (not IE).
- **No server-side:** Everything runs in the browser; `localStorage` is the only persistence.

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| VexFlow 4 + raw SVG for hints | `setX()` doesn't exist in VF4; raw SVG works around it | ⚠️ Revisit — raw SVG causes positioning bugs, hardcoded Y values don't align with actual staff lines |
| Single-file HTML | Zero build step, maximum portability | ✓ Good — easy to open locally and deploy to GitHub Pages |
| Inline all CSS/JS | No HTTP requests needed beyond VexFlow CDN | ✓ Good — fast load, works offline after first render |

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition** (via `/gsd-transition`):
1. Requirements invalidated? → Move to Out of Scope with reason
2. Requirements validated? → Move to Validated with phase reference
3. New requirements emerged? → Add to Active
4. Decisions to log? → Add to Key Decisions
5. "What This Is" still accurate? → Update if drifted

**After each milestone** (via `/gsd-complete-milestone`):
1. Full review of all sections
2. Core Value check — still the right priority?
3. Audit Out of Scope — reasons still valid?
4. Update Context with current state

---
*Last updated: 2026-04-29 after initialization — bug fix milestone*