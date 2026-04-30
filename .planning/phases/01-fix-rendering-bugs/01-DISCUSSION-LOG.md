# Phase 1: Fix Rendering Bugs - Discussion Log

> **Audit trail only.** Do not use as input to planning, research, or execution agents.
> Decisions are captured in CONTEXT.md — this log preserves the alternatives considered.

**Date:** 2026-04-29
**Phase:** 1-Fix Rendering Bugs
**Areas discussed:** Hint rendering approach, Hint visual treatment

---

## Hint rendering approach — VexFlow strategy for gray hint notes

| Option | Description | Selected |
|--------|-------------|----------|
| Separate Voice + Formatter for hints | Create a dedicated Voice with all hint notes, format to fill available left-side width. Target stays in its own centered Voice. Most VF4-native approach. Requires two voices but cleanest positioning. | |
| Corrected raw SVG positioning | Keep raw SVG but query the stave for correct Y positions (via stave.getYForLine()) rather than hardcoded math. Minimal code change but still raw SVG. | |
| Let researcher investigate options | Researcher investigates available VF4 APIs (Voice+Formatter nuances, setXOffset, ghost notes, etc.) and planner selects best approach. | |

**User's choice:** User asked "Why not just insert the additional notes onto the staff instead of drawing them separately?" — preferring a single shared Voice approach.

**Follow-up:**
- Target at end of sequence vs centered → **Target at end of sequence** (not centered)
- Single shared Voice vs two separate Voices → **Single shared Voice**
- Hint note duration → **Quarter or half notes** (visually distinct from target whole note)

**Notes:** User prefers the simplest VF4-native approach: one Voice containing all notes, letting the Formatter handle spacing. They do not want the target note to remain centered if it complicates implementation.

---

## Hint visual treatment — styling for grayed-out notes

| Option | Description | Selected |
|--------|-------------|----------|
| Light gray solid fill | Light gray fill with light gray stroke. Solid but muted note heads. Simple, clean. | |
| Outlined (gray stroke only) | White/transparent fill with gray stroke. 'Outlined' look, less heavy. | |
| Semi-transparent | Light gray fill with reduced opacity (semi-transparent). May show staff lines through notes. | ✓ |

**User's choice:** Semi-transparent

**Follow-up:**
- Subtle (60-70%) vs Strong (30-40%) → **Strong (30-40%)**
- Labels also semi-transparent vs full opacity → **Labels at full opacity**
- Everything semi-transparent vs only note heads → **Everything semi-transparent** (note heads + stems + flags)

**Notes:** User wants very ghost-like hint notes. Labels must remain readable even if the notes themselves are very faint. Stems should fade too for visual consistency.

---

## the agent's Discretion

- Brace connector fix strategy: Not discussed — deferred to researcher/planner
- Verification criteria: Not discussed — deferred to standard verification approach

## Deferred Ideas

None — discussion stayed within phase scope.
