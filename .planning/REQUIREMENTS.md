# Requirements: Note Reader — Bug Fix Milestone

**Defined:** 2026-04-29
**Core Value:** Users can quickly learn to recognize notes on the grand staff through repeated, low-friction practice with visual feedback and optional progressive hints.

## v1 Requirements (Active)

### Bug Fixes — Rendering

- [ ] **RENDER-01**: Diatonic path hint notes render as proper VexFlow notes (grayed out / de-emphasized), not raw SVG ellipses with hardcoded Y positions
  - Current behavior: raw SVG `<ellipse>` elements with manually calculated Y coordinates, which don't align with actual staff line positions
  - Expected behavior: use VexFlow `StaveNote` with styling (e.g., `setStyle({ fillStyle: '#bbbbbb', strokeStyle: '#bbbbbb' })` or similar) so notes appear as proper whole notes but grayed
  - Constraints: VexFlow 4 API only (no `setX`), single-file HTML

- [ ] **RENDER-02**: Brace connector between treble and bass staves renders fully, not cut off on the left side
  - Current behavior: brace is clipped at the left edge of the SVG (or by the container's `overflow: hidden`)
  - Expected behavior: full brace glyph visible, matching the right-side bar connector
  - Likely cause: SVG `viewBox` or `overflow` setting, or x-position of brace starting too close to edge

## v2 Requirements

Deferred to future milestone. Tracked in PROJECT.md but not in current scope.

## Out of Scope

| Feature | Reason |
|---------|--------|
| Audio playback | Requires Web Audio API, not relevant to visual note-reading |
| Alto/tenor clefs | Out of scope for grand staff learning tool |
| User accounts / cloud sync | Single-user local tool, localStorage sufficient |
| Refactoring to multi-file project | Out of scope for this milestone; stays single-file HTML |

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| RENDER-01 | Phase 1 | Pending |
| RENDER-02 | Phase 1 | Pending |

**Coverage:**
- v1 requirements: 2 total
- Mapped to phases: 2
- Unmapped: 0 ✓

---
*Requirements defined: 2026-04-29*
*Last updated: 2026-04-29 after initial definition*
