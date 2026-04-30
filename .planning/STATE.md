# State: Note Reader

**Project:** Note Reader (`NOTE-READER`)
**Status:** Active — Phase 1 context gathered, ready for planning
**Date:** 2026-04-29

---

## Project Reference

See: `.planning/PROJECT.md` (updated 2026-04-29)

**Core value:** Users can quickly learn to recognize notes on the grand staff through repeated, low-friction practice with visual feedback and optional progressive hints.
**Current focus:** Phase 1 — Fix Rendering Bugs

---

## Milestone

**Active:** Bug Fix Milestone
- **Start:** 2026-04-29
- **Goal:** Fix two rendering issues (diatonic path hints, brace connector)
- **Requirements:** RENDER-01, RENDER-02

---

## Phase Progress

| Phase | Name | Status | Requirements |
|-------|------|--------|--------------|
| 1 | Fix Rendering Bugs | Context gathered — Ready for planning | RENDER-01, RENDER-02 |

---

## Key Decisions Logged

| Decision | Date | Outcome |
|----------|------|---------|
| VexFlow 4 + raw SVG for hints | original | ⚠️ Revisit — under investigation in Phase 1 |
| Single-file HTML | original | ✓ Good |
| Inline all CSS/JS | original | ✓ Good |

## Phase 1 Decisions

**Hint rendering approach (2026-04-29):**
- Use single shared Voice with all hints + target (Formatter handles spacing)
- Target note at right end of sequence (not centered)
- Hints as quarter/half notes, target as whole note

**Hint visual treatment (2026-04-29):**
- Hints semi-transparent at 30–40% opacity
- Labels remain at full opacity
- Stems + note heads + flags all share the same opacity

---

## Next Action

Run `/gsd-plan-phase 1` to create execution plans for both rendering fixes.
