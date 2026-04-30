# Architecture

**Analysis Date:** 2026-04-29

## System Overview

Note Reader is a zero-dependency, self-contained single-file web application (excluding the external VexFlow library CDN). It runs entirely in the browser with no build step, no framework, and no server-side component.

```text
┌─────────────────────────────────────────────────────────────┐
│                      Browser (Client-Side)                   │
├──────────────────┬──────────────────┬───────────────────────┤
│   Presentation   │     Game Logic     │     Rendering Engine  │
│  `index.html`    │  `index.html`      │   `index.html` +      │
│  (HTML + CSS)    │  (JavaScript)      │   VexFlow CDN         │
└──────────────────┴──────────────────┴───────────────────────┘
         │                  │                     │
         ▼                  ▼                     ▼
┌─────────────────────────────────────────────────────────────┐
│                    Browser APIs                              │
│  localStorage  │  DOM / SVG  │  Keyboard Events              │
└─────────────────────────────────────────────────────────────┘
```

## Component Responsibilities

| Component | Responsibility | File |
|-----------|----------------|------|
| HTML Structure | UI markup: staff area, buttons, scoreboard, hint controls | `index.html` (lines 141–189) |
| CSS Styling | Layout, animations, color theming, responsive card design | `index.html` (lines 8–139) |
| Game State | Score, streak, clef mode, current question, hint level | `index.html` (lines 368–377) |
| Note Pools | Treble and bass clef note definitions (`TREBLE`, `BASS`) | `index.html` (lines 192–205) |
| Rendering | VexFlow-based grand staff SVG generation + raw SVG overlays | `index.html` (lines 266–366) |
| Event Handling | Click handlers for buttons, keyboard input, hint toggles | `index.html` (lines 464–499) |
| Persistence | Best streak saved/loaded via `localStorage` | `index.html` (lines 374, 441) |

## Pattern Overview

**Overall:** Single-file Vanilla JavaScript (no framework)

**Key Characteristics:**
- All code lives in one HTML file (`index.html`)
- No module system — functions and state are global within the script block
- No build step or bundler — served directly via CDN + static file hosting
- State is managed via a single mutable object (`state`) rather than a formal store
- Rendering is imperative DOM manipulation + VexFlow SVG generation

## Layers

**Presentation Layer:**
- Purpose: Render the user interface and capture input
- Location: `index.html` (HTML/CSS and event listeners)
- Contains: Semantic HTML, CSS custom properties, DOM query selectors, event handlers
- Depends on: Game logic functions, VexFlow library
- Used by: Browser

**Game Logic Layer:**
- Purpose: Manage quiz flow, scoring, note selection, and state transitions
- Location: `index.html` (JavaScript functions in `<script>` block)
- Contains: State object, `pickNote()`, `checkAnswer()`, `newQuestion()`, helper functions
- Depends on: Note pools, rendering functions
- Used by: Event handlers, timers

**Rendering Layer:**
- Purpose: Generate visual staff notation using VexFlow and raw SVG overlays
- Location: `index.html` (lines 266–366)
- Contains: `drawGrandStaff()`, `buildDiatonicPath()`, `addLabelSVGs()`, hint note SVG injection
- Depends on: VexFlow 4 CDN, DOM APIs
- Used by: `newQuestion()`, hint button handlers

**Persistence Layer:**
- Purpose: Store best streak across sessions
- Location: `localStorage` via `index.html`
- Contains: `localStorage.getItem('notereader-best')`, `localStorage.setItem('notereader-best', value)`
- Depends on: Browser `localStorage` API
- Used by: `checkAnswer()` on correct streak updates

## Data Flow

### Primary Quiz Flow

1. **Boot** — `newQuestion()` is called on load (`index.html:504`)
2. **Pick note** — `pickNote()` selects a random note from the active clef pool (`index.html:403–415`)
3. **Render** — `drawGrandStaff()` generates the grand staff SVG with the target note (`index.html:267`)
4. **User input** — Click handler or keyboard event triggers `checkAnswer()` (`index.html:465–467`, `496–498`)
5. **Score update** — `state.score`, `state.streak`, `state.total` are mutated (`index.html:427–456`)
6. **Feedback** — CSS classes (`correct`/`wrong`/`shake`) and feedback text are applied (`index.html:443–451`)
7. **Persistence** — Best streak is written to `localStorage` if exceeded (`index.html:441`)
8. **Next question** — `setTimeout()` delays then calls `newQuestion()` again (`index.html:458–461`)

### Hint System Flow

1. **User clicks Show Path** — `state.hintLevel` set to `1` (`index.html:479`)
2. **Re-render** — `drawGrandStaff()` redraws with `buildDiatonicPath()` result as gray ellipses (`index.html:313–360`)
3. **User clicks Show Labels** — `state.hintLevel` set to `2` (`index.html:485`)
4. **Label injection** — `addLabelSVGs()` appends raw SVG `<text>` elements to the VexFlow-generated SVG (`index.html:239–264`)

## Key Abstractions

**Note Pool:**
- Purpose: Defines the set of notes available per clef
- Examples: `index.html` lines 192–205 (`TREBLE`, `BASS` arrays)
- Pattern: Array of objects with `name` (display letter) and `key` (VexFlow key string)

**State Object:**
- Purpose: Central mutable state container for the entire app
- Example: `index.html` lines 368–377
- Pattern: Plain JavaScript object mutated directly by functions

**VexFlow + Raw SVG Hybrid:**
- Purpose: Work around VexFlow 4 API limitations (`setX` missing)
- Example: Target note uses `Voice` + `Formatter` (lines 304–310); hints use raw `document.createElementNS('...', 'ellipse')` (lines 351–359)
- Pattern: Library-rendered base + imperative DOM injection for overlays

## Entry Points

**Page Load:**
- Location: `index.html` line 501–504
- Triggers: Browser `DOMContentLoaded` (implied by script at end of `<body>`)
- Responsibilities: Initialize `bestStreak` from `localStorage`, set up event listeners, call `newQuestion()`

**User Interaction:**
- Note buttons: `index.html` lines 465–467
- Clef toggle buttons: `index.html` lines 469–476
- Hint buttons: `index.html` lines 478–494
- Keyboard: `index.html` lines 496–499

## Architectural Constraints

- **Single-file architecture:** All code (HTML, CSS, JS) lives in `index.html`. There is no module splitting, so changes to any concern require editing the same file.
- **No build step:** Direct browser execution means no TypeScript, no transpilation, no tree-shaking. All code must be plain ES5/ES6 compatible JavaScript.
- **Global state:** The `state` object and all functions are in the global script scope. There are no closures or modules encapsulating state.
- **CDN dependency:** The app fails entirely if `cdn.jsdelivr.net/npm/vexflow@4.2.5/build/cjs/vexflow-bravura.js` is unavailable.
- **Blocking script:** The `<script>` tag is not `async` or `defer`, but placed at the end of `<body>`, so it executes after DOM is parsed.
- **SVG mutation after render:** The rendering layer queries the VexFlow-generated SVG via `document.querySelector('#staffContainer svg')` and mutates it directly. This is fragile if VexFlow changes its internal SVG structure.

## Anti-Patterns

### Global Mutable State

**What happens:** A single top-level `const state = {...}` object is mutated directly by multiple functions (`pickNote`, `checkAnswer`, `newQuestion`, hint handlers).
**Why it's wrong:** No change tracking, no encapsulation, race conditions possible if async logic ever expands, and debugging is harder because state mutations are scattered across the file.
**Do this instead:** Encapsulate state behind accessor functions or a minimal store pattern (e.g., `getState()`, `setState(updates)`), or split into ES modules importing a state module.

### Render Function With Side Effects

**What happens:** `drawGrandStaff()` reads from the global `state.hintLevel` to decide whether to draw hints, rather than receiving hint configuration as a parameter.
**Why it's wrong:** The function is not pure; its output depends on implicit global state, making unit testing and reasoning about behavior harder.
**Do this instead:** Pass `hintLevel` as an explicit parameter: `drawGrandStaff(activeClef, targetNote, hintLevel)`.

### Direct DOM Scraping of Library Output

**What happens:** The code queries `document.querySelector('#staffContainer svg')` and appends raw SVG children to the VexFlow-rendered DOM.
**Why it's wrong:** Tight coupling to VexFlow's internal SVG structure. If the library changes IDs, namespaces, or element ordering, hint rendering breaks silently.
**Do this instead:** Render hints onto a separate overlay SVG or canvas layer positioned on top of the VexFlow output.

## Error Handling

**Strategy:** Minimal — there is no explicit error handling in the application logic.

**Patterns:**
- No `try/catch` blocks anywhere in the script
- No fallback if `localStorage` is disabled or unavailable in private browsing
- No fallback if VexFlow fails to load from CDN
- If a note is not found in the pool, the code defaults to `'c/4'` (`index.html:298`)

## Cross-Cutting Concerns

**Logging:** None. There are no `console.log` statements or logging framework.

**Validation:** None. User input is constrained to button clicks and mapped keyboard keys; there is no input validation beyond the `m[e.key.toLowerCase()]` mapping.

**Authentication:** Not applicable — there is no user system or server.

---

*Architecture analysis: 2026-04-29*
