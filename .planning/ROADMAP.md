# Roadmap: Note Reader — Bug Fix Milestone

**Created:** 2026-04-29
**Granularity:** Coarse
**Mode:** YOLO

---

## Phase 1: Fix Rendering Bugs

**Goal:** Resolve both visual rendering issues — diatonic path hints and brace connector.

**Requirements:** RENDER-01, RENDER-02

**Success Criteria:**
1. When "Show Path" is clicked, gray hint notes render as proper VexFlow notes (not raw SVG ellipses) and align correctly on their staff lines
2. The brace connector between treble and bass staves renders fully without being clipped on the left
3. Both fixes verified by opening `index.html` in a browser and testing multiple note positions across both clefs
4. No regressions in existing functionality (scoring, streaks, hint labels, clef toggle)

**Technical Notes:**
- **RENDER-01 approach:** Replace raw SVG `<ellipse>` with VexFlow `StaveNote` instances using `setStyle()` to gray them out. Multiple voices or `Formatter` may be needed since VF4 doesn't support `setX()`.
- **RENDER-02 approach:** Inspect SVG `viewBox`/`overflow` and SVG element positioning. Brace is cut off on the left — likely needs `overflow: visible` or x-offset adjustment.

---

## Requirement → Phase Mapping

| Requirement | Phase | Status |
|-------------|-------|--------|
| RENDER-01 | Phase 1 | Pending |
| RENDER-02 | Phase 1 | Pending |

**Coverage:** 2/2 requirements mapped ✓

---

*Roadmap created: 2026-04-29*
