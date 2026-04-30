# Phase 1: Fix Rendering Bugs - Context

**Gathered:** 2026-04-29
**Status:** Ready for planning

<domain>
## Phase Boundary

This phase fixes two visual rendering bugs in the Note Reader single-file HTML app:

1. **RENDER-01:** Diatonic path hint notes currently render as raw SVG `<ellipse>` elements with hardcoded Y positions. They should render as proper VexFlow notes (grayed out / de-emphasized) that align correctly on staff lines.
2. **RENDER-02:** The brace connector between treble and bass staves is clipped/cut off on the left side of the SVG.

No new capabilities are added — only existing visual bugs are fixed.
</domain>

<decisions>
## Implementation Decisions

### Hint rendering approach — VexFlow strategy for gray hint notes
- **D-01:** Use a **single shared Voice** containing all diatonic path hint notes plus the target note. The VexFlow `Formatter` spaces all notes automatically in sequence.
- **D-02:** The **target note naturally appears at the right end** of the sequence (not centered). It is the last note in the shared Voice.
- **D-03:** Hint notes use **quarter or half note duration** (`'q'` or `'h'`) to visually distinguish them from the target whole note (`'w'`).

### Hint visual treatment — styling for grayed-out notes
- **D-04:** Hint notes should be **semi-transparent at 30–40% opacity** (strong fade). This applies to note heads, stems, and flags uniformly.
- **D-05:** Note name **labels (C, D, E, etc.) should remain at full opacity** for readability, even when the hint notes themselves are ghost-like.

### the agent's Discretion
- **Brace connector fix strategy:** Not discussed. Researcher/planner should investigate SVG `viewBox`/`overflow` settings, renderer dimensions, or x-offset adjustments to prevent left-edge clipping. The user deferred this decision.
- **Verification criteria:** Not discussed. Standard approach: open `index.html` in a browser, test multiple note positions across both clefs, verify both fixes render correctly without regressions in scoring, streaks, hint labels, or clef toggle.
</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Requirements
- `.planning/REQUIREMENTS.md` — RENDER-01 and RENDER-02 requirements with current vs expected behavior
- `.planning/ROADMAP.md` — Phase 1 goal, success criteria, and technical notes
- `.planning/PROJECT.md` — Constraints (single-file HTML, VexFlow 4, no setX API)

### Codebase
- `index.html` — The entire application (single file). Relevant sections:
  - Lines 207–264: Hint helpers (`buildDiatonicPath`, `addLabelSVGs`)
  - Lines 267–366: `drawGrandStaff` rendering function (target note Voice/Formatter, raw SVG ellipses for hints, brace connector)

### External
- No external specs — requirements fully captured in decisions above
</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- `drawGrandStaff()`: Already creates a `Voice` + `Formatter` for the target note (lines 304–310). This pattern should be extended to include hint notes.
- `buildDiatonicPath()`: Generates the sequence of note keys from C4 to the target. Output feeds directly into a Voice.
- `addLabelSVGs()`: Appends raw SVG `<text>` labels. Should be updated to use the new hint note positions.

### Established Patterns
- VexFlow 4 global namespace (`const {Renderer, Stave, StaveNote, StaveConnector, Voice, Formatter} = Vex.Flow`)
- Raw SVG manipulation as workaround for VF4 limitations (current approach, being replaced)
- `setStyle({ fillStyle, strokeStyle })` used for note styling (can also set opacity via SVG)
- Staff area CSS: `position: relative; overflow: hidden;` — relevant to RENDER-02 if overflow is causing the brace clipping

### Integration Points
- Hints are drawn conditionally based on `state.hintLevel` (lines 313–365)
- `drawGrandStaff()` is called on every question change, hint toggle, and clef change
- Brace connector drawn at lines 288–294 (`StaveConnector` with `type.BRACE`)
</code_context>

<specifics>
## Specific Ideas

- The user specifically noted that raw SVG ellipses cause positioning bugs and hardcoded Y values don't align with actual staff lines. They want to move away from raw SVG entirely for hints.
- The user wants the diatonic path hints to look like a sequence of small notes leading to the target, not just dots.
</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope.
</deferred>

---

*Phase: 1-Fix Rendering Bugs*
*Context gathered: 2026-04-29*
