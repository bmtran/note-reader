# Codebase Structure

**Analysis Date:** 2026-04-29

## Directory Layout

```
note-reader/
├── index.html          # Single-file application (HTML + CSS + JS)
├── SPECS.md            # Functional specification and design notes
└── .planning/
    └── codebase/
        ├── ARCHITECTURE.md   # This codebase architecture analysis
        └── STRUCTURE.md      # This file
```

## Directory Purposes

**Project Root (`note-reader/`):**
- Purpose: Contains the entire application and documentation
- Contains: Single source file, specification markdown, and planning artifacts
- Key files: `index.html`, `SPECS.md`

**.planning/codebase/:**
- Purpose: Stores generated codebase analysis documents consumed by other GSD commands
- Contains: Architecture and structure reference documents
- Key files: `ARCHITECTURE.md`, `STRUCTURE.md`

## Key File Locations

**Entry Points:**
- `index.html`: Sole application entry point — loaded directly by the browser. Serves as HTML skeleton, stylesheet, and JavaScript runtime.

**Configuration:**
- None. There are no config files, no `package.json`, no `tsconfig.json`, and no bundler configuration.

**Core Logic:**
- `index.html` (lines 191–505): All JavaScript game logic and rendering code.

**Documentation:**
- `SPECS.md`: Detailed visual, interaction, and technical specification.

**Testing:**
- None. There are no test files.

## Naming Conventions

**Files:**
- Application file: lowercase with hyphen — `index.html`
- Specification file: UPPERCASE — `SPECS.md`
- Planning docs: UPPERCASE — `ARCHITECTURE.md`, `STRUCTURE.md`

**CSS Classes:**
- kebab-case — `.clef-toggle`, `.staff-area`, `.note-btn`, `.hint-row`, `.score-item`
- State modifier classes: `.correct`, `.wrong`, `.shake`, `.show`, `.used`, `.active`

**JavaScript Identifiers:**
- Top-level constants: `SCREAMING_SNAKE_CASE` — `TREBLE`, `BASS`
- Functions: camelCase — `pickNote()`, `drawGrandStaff()`, `buildDiatonicPath()`, `checkAnswer()`
- State object: lowercase — `state`
- DOM element variables: terse ids — `fb`, `area` (not always descriptive)

**Data Attributes:**
- `data-note` for note buttons — values: C, D, E, F, G, A, B
- `data-clef` for clef toggle buttons — values: both, treble, bass

## Where to Add New Code

Because the entire application lives in a single file, additions must be made to `index.html` in the appropriate section.

**New Feature (e.g., timer, difficulty setting):**
- HTML markup: Add inside `<div class="card">` near relevant section (e.g., below scoreboard or inside hint row)
- CSS: Add inside existing `<style>` block, grouped logically
- JavaScript logic: Add inside `<script>` block; place new state fields near the `state` object (line 368–377), event listeners near existing listener blocks (lines 464–499), and utility functions near existing helpers (lines 207–264)

**New Visual Component:**
- HTML: Add element within `.card`
- CSS: Add class rules within `<style>` block; use CSS custom variables from `:root` for colors
- JS: Query and manipulate in existing event handlers or create new handler functions

**New Game Mode or Setting:**
- Add state field to the `state` object (line 368–377)
- Add UI toggle in HTML near `.clef-toggle` (line 146–150)
- Add click handler in JavaScript near clef toggle handlers (lines 469–476)
- Modify `pickNote()` or `drawGrandStaff()` as needed

**New Persistence Key:**
- Follow the pattern used for `notereader-best` (lines 374, 441): read on boot with `localStorage.getItem(key)`, write on update with `localStorage.setItem(key, value)`

## Special Directories

**.planning/:**
- Purpose: Contains codebase mapping and planning artifacts used by GSD commands (`/gsd-plan-phase`, `/gsd-execute-phase`)
- Generated: Yes (by GSD mapper)
- Committed: Should be committed to allow other agents to reference codebase docs

**.git/:**
- Purpose: Git version control metadata
- Generated: Yes
- Committed: N/A (not tracked)

---

*Structure analysis: 2026-04-29*
