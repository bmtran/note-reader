# Coding Conventions

**Analysis Date:** 2026-04-29

This document describes the coding conventions observed in the `note-reader` codebase. Since the project is a single-file HTML application (`index.html`) with no build step, package manager, or linting toolchain, conventions are entirely informal and embodied in the existing code.

## Naming Patterns

**JavaScript Identifiers:**

| Category | Pattern | Example |
|----------|---------|---------|
| Functions | `camelCase` | `buildDiatonicPath`, `addLabelSVGs`, `drawGrandStaff` |
| Variables | `camelCase` | `clefMode`, `hintLevel`, `activeClef` |
| Constants / lookup tables | `UPPER_SNAKE_CASE` | `TREBLE`, `BASS` |
| State properties | `camelCase` | `state.score`, `state.bestStreak` |
| Element references | `camelCase` | `const area = document.getElementById('staffArea')` |

**CSS Selectors:**

| Category | Pattern | Example |
|----------|---------|---------|
| Classes | `kebab-case` | `.clef-toggle`, `.answer-grid`, `.note-btn` |
| IDs | `camelCase` | `#staffArea`, `#hintPathBtn`, `#scoreVal` |
| Data attributes | `lowercase` | `data-note`, `data-clef` |

**HTML:**
- Tags and attributes: lowercase (standard HTML5 convention).
- No BEM or other formal CSS naming methodology is used; classes are short and scoped to the single file.

## Code Style

**Indentation & Spacing:**
- Indentation width: **2 spaces** (observed in both HTML and CSS).
- CSS blocks: single-line declarations where possible.
- JavaScript: compact single-line object literals and short expressions; multi-line blocks for larger functions.

**Quotes:**
- JavaScript: single quotes `'string'` (predominant).
- HTML attributes: double quotes `"value"`.

**Semicolons:**
- Used but not perfectly consistent. Example at `index.html:503` shows assignment with semicolon, while some short assignments in CSS-like blocks omit them.

**Line Endings:**
- Likely LF (inferred from Git repository context).

## Import Organization

**No module system** is used. The project runs in the browser as a global script.

- VexFlow is loaded via CDN in a `<script>` tag: `index.html:7`
  ```html
  <script src="https://cdn.jsdelivr.net/npm/vexflow@4.2.5/build/cjs/vexflow-bravura.js"></script>
  ```
- Application JavaScript is embedded inline in `index.html:191–505`.
- Dependencies are accessed via the global `Vex.Flow` namespace.

## Error Handling

**Current strategy:** Guard clauses and early returns; **no `try/catch` blocks**.

**Patterns observed:**

- **Null guard** in `addLabelSVGs` (`index.html:241`):
  ```javascript
  const svg = document.querySelector('#staffContainer svg');
  if (!svg) return;
  ```

- **State guard** in `checkAnswer` (`index.html:428`):
  ```javascript
  if (state.answered) return;
  ```

- **No input validation** beyond basic type coercions (e.g., `parseInt(tOctStr)` at `index.html:211`).

**Implication:** The app assumes a controlled environment (trusted HTML) and does not defensively validate DOM querying results or user input beyond the UI layer.

## Logging

- **No logging framework** is used.
- **No `console.log` statements** are present in the source.
- Debugging is likely done via browser DevTools breakpoints or by temporarily adding `console.log` during development.

## Comments

**Comment style:** `//` line comments with decorative section headers.

**Example** (`index.html:192`):
```javascript
// ─── Note pools ─────────────────────────────────────────────────────────────
```

**Guidelines observed:**
- Major sections (Note pools, Hint helpers, Rendering, Game State, Events, Boot) are delimited by long comment lines.
- Inline comments explain *why* a non-obvious workaround is used (e.g., the raw SVG ellipse strategy to avoid the missing `setX` API at `index.html:321`).

**No JSDoc/TSDoc** is used; there are no type annotations (the project is plain JavaScript).

## Function Design

**Size:** Functions are small to medium (mostly 5–30 lines). The largest function, `drawGrandStaff` (`index.html:267–366`), is ~100 lines and handles rendering, geometry calculation, SVG manipulation, and conditional hint drawing.

**Parameters:**
- Prefer explicit named parameters over positional arguments when clarity demands it.
- Example: `drawGrandStaff(activeClef, targetNote)` (`index.html:267`) takes two strings.

**Return values:**
- Functions return plain objects, arrays, primitives, or `undefined`.
- Example: `pickNote()` returns `{ clef, note }` (`index.html:403–415`).

**Arrow functions** are used for simple callbacks (`index.html:465–467`):
```javascript
document.querySelectorAll('.note-btn').forEach(btn =>
  btn.addEventListener('click', () => checkAnswer(btn.dataset.note))
);
```

**Destructuring** is used to extract namespaced APIs (`index.html:271`):
```javascript
const {Renderer, Stave, StaveNote, StaveConnector, Voice, Formatter} = Vex.Flow;
```

## Module / File Design

**No module boundaries.**

Because the entire application lives in a single HTML file, there is no concept of imports, exports, or barrel files. The script executes in the global scope of the browser window.

- Helper functions (`buildDiatonicPath`, `keyToLabel`, `addLabelSVGs`) are hoisted and available everywhere.
- The `state` object (`index.html:369–377`) is a module-level singleton that acts as the global game store.
- DOM elements are queried at event-setup time rather than cached upfront.

## Anti-Patterns Present

### Monolithic File

**What happens:** All HTML, CSS, JavaScript, and assets are in one 500+ line file (`index.html`).
**Why it's problematic:** No separation of concerns makes concurrent editing difficult, complicates code review, prevents tree-shaking, and blocks any meaningful unit testing.
**Do this instead:** Extract styles to `styles.css`, scripts to `app.js` (or modularize into `render.js`, `game.js`, `events.js`), and load them via `<link>` and `<script src="...">`.

### Global Mutable State

**What happens:** The `state` object is a top-level mutable singleton (`index.html:369`).
**Why it's problematic:** Any function can mutate it, making data flow hard to trace and race conditions possible if async logic is added.
**Do this instead:** Encapsulate state in a factory function or class, and expose explicit getters/setters (or use a lightweight state manager).

### Inline Event Listeners

**What happens:** Event listeners are attached inside a script block with anonymous arrow functions (`index.html:465–499`).
**Why it's problematic:** Events cannot be removed easily (no reference to the handler), which complicates cleanup or testing.
**Do this instead:** Name handler functions and attach them with named references so they can be removed later:
```javascript
function onNoteClick(e) { checkAnswer(e.target.dataset.note); }
btn.addEventListener('click', onNoteClick);
// cleanup: btn.removeEventListener('click', onNoteClick);
```

---

*Convention analysis: 2026-04-29*
