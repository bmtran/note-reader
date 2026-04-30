# Codebase Concerns

**Analysis Date:** 2026-04-29

## Tech Debt

### Single-File Monolith
- **Issue:** The entire application (~507 lines) — HTML structure, CSS (`<style>` block, lines 8–139), and JavaScript (`<script>` block, lines 191–505) — lives in one file (`index.html`). There are no modules, no imports, and no build step.
- **Files:** `index.html`
- **Impact:** Code review and refactoring are difficult because concerns (markup, presentation, logic) are interleaved. Adding features requires appending to an already-long file, increasing the risk of merge conflicts and regressions. There is no dead-code elimination, no tree-shaking, and no way to unit-test individual functions.
- **Fix approach:** Split into `index.html`, `styles.css`, and `app.js` (or multiple ES modules under `src/`). Introduce a minimal bundler (Vite, Rollup, or Parcel) to support `import`/`export`, CSS extraction, and minification.

### No Development Tooling
- **Issue:** No linter (ESLint), formatter (Prettier), type checker (TypeScript), test runner (Jest/Vitest), or pre-commit hooks exist. `package.json` is absent entirely.
- **Files:** `index.html` (repo root)
- **Impact:** No automated enforcement of coding standards. Typos in variable names, unused variables, or logic errors are caught only by manual review or runtime failure. Formatting is inconsistent (mixed quote styles, ad-hoc spacing).
- **Fix approach:** Initialize an npm project, add ESLint + Prettier with a shared config, and a GitHub Actions workflow that lints on PR.

### Inline Styles and Scripts
- **Issue:** All CSS and JavaScript are embedded directly in `index.html`.
- **Files:** `index.html` (lines 8–139 for CSS, lines 191–505 for JS)
- **Impact:** Browsers cannot cache stylesheets or scripts independently, so every page load re-downloads the full payload. Content Security Policy (CSP) headers requiring external resources are impossible to apply without `'unsafe-inline'` exceptions.
- **Fix approach:** Extract CSS to `styles.css` and JS to `app.js`, reference them with `<link>` and `<script src>` tags. Add CSP headers on the hosting server.

## Known Bugs

### Note Pool Deduplication Omits Higher-Octave Notes
- **Symptoms:** Higher-octave notes (C5–G5 in treble, A3–C4 in bass) never appear as quiz questions.
- **Files:** `index.html` (lines 403–415)
- **Trigger:** Run the app, switch to Treble or Bass mode, and observe that notes above/below middle octaves are absent.
- **Code pattern:**
  ```javascript
  const unique = [];
  const seen = new Set();
  for (const n of pool) {
    if (!seen.has(n.name)) { unique.push(n); seen.add(n.name); }
  }
  const note = unique[Math.floor(Math.random() * unique.length)];
  ```
- **Why it's wrong:** `seen` tracks only the note *name* (e.g., `"C"`), not the full `key` (e.g., `"c/4"` vs `"c/5"`). The `TREBLE` array contains two `"C"` entries (`c/4` and `c/5`), but the second is discarded. This reduces the treble pool from 12 to 7 and the bass pool from 10 to 7.
- **Fix approach:** Deduplicate by `n.key` instead of `n.name`, or remove deduplication entirely if the full range is desired.

### Bass Clef Hint Y-Positioning Is Completely Wrong
- **Symptoms:** When the bass clef is active and Show Path is enabled, gray hint ellipses are drawn far below the visible staff area.
- **Files:** `index.html` (lines 344–348)
- **Trigger:** Select Bass clef, click **Show Path**, and observe hint note placement.
- **Code pattern:**
  ```javascript
  stavePos = 9 - (noteIdx === 0 ? 9 : noteIdx) - (oct - 4) * 7;
  const noteY = 90 + stavePos * 5;  // bass: y increases going up
  ```
- **Why it's wrong:** The special-casing of `noteIdx === 0` (C notes) and the arithmetic produce nonsensical results. For C3 (`noteIdx=0, oct=3`), `stavePos = 9 - 9 - (-1)*7 = 7`, yielding `noteY = 90 + 35 = 125`. The bass comment on line 343 claims `c/3` should be at `y=70`, but the formula emits `125`. The formula diverges further for other notes.
- **Fix approach:** Replace the mathematical derivation with a verified lookup table (`const BASS_NOTE_Y = {'a/2': 130, 'b/2': 120, ...}`), or derive Y positions from VexFlow's own `getBoundingClientRect` of rendered noteheads.

### Treble Clef Hint Y-Positioning Uses Fragile Magic Numbers
- **Symptoms:** Treble hint notes may be vertically misaligned, especially ledger-line notes like C4.
- **Files:** `index.html` (lines 339–342)
- **Trigger:** Select Treble clef, show path to C4 or G5.
- **Code pattern:**
  ```javascript
  stavePos = 11 - noteIdx - (oct - 4) * 7;
  const noteY = 110 - stavePos * 5;
  ```
- **Why it's wrong:** Hardcoded constants (`11`, `7`, `5`, `110`) depend on VexFlow's internal SVG coordinate system, which can shift with DPI, font metrics, or minor version changes. There is no single source of truth.
- **Fix approach:** Same as bass clef — use a lookup table or dynamically measure rendered note positions.

### Keyboard Input Not Disabled During Feedback Delay
- **Symptoms:** While the correct/wrong feedback is visible and before the next question appears, keyboard presses can queue up and immediately answer the upcoming question.
- **Files:** `index.html` (lines 496–499)
- **Trigger:** Answer any question, then rapidly press C–B keys during the 900–1500 ms delay.
- **Code pattern:**
  ```javascript
  document.addEventListener('keydown', e => {
    const m = { c:'C', d:'D', e:'E', f:'F', g:'G', a:'A', b:'B' };
    if (m[e.key.toLowerCase()]) checkAnswer(m[e.key.toLowerCase()]);
  });
  ```
- **Why it's wrong:** Although `checkAnswer` has `if (state.answered) return;`, the `keydown` handler itself does not check `state.answered`. When `setTimeout` fires and `newQuestion()` resets `answered = false`, any queued key events fire `checkAnswer` on the fresh question, potentially registering an unintended (or even multiple) answers in rapid succession.
- **Fix approach:** Gate the keydown handler on `state.answered === false`, or remove/re-add the listener during the inter-question delay, or disable input buttons immediately in `checkAnswer`.

### Stale `setTimeout` Callbacks
- **Symptoms:** If the user toggles the clef (triggering `newQuestion()`) while a feedback `setTimeout` from a previous answer is still pending, the old timeout fires and unexpectedly replaces the newly-generated question.
- **Files:** `index.html` (lines 458–461)
- **Trigger:** Answer a question, then immediately click a different clef button before the 900–1500 ms feedback delay ends.
- **Why it's wrong:** `setTimeout` timers are not cancelled. The clef-toggle click calls `newQuestion()` immediately, but the old timer still fires later and invokes `newQuestion()` again, causing a second rapid switch.
- **Fix approach:** Store the timeout ID (`const timerId = setTimeout(...)`), clear it (`clearTimeout(timerId)`) at the start of `newQuestion()`.

## Security Considerations

### No Content Security Policy (CSP)
- **Risk:** Without CSP headers or a `<meta http-equiv="Content-Security-Policy">` tag, any injected script or style executes unrestricted. If the CDN is compromised or a reflected XSS vector exists, the browser will not block execution.
- **Files:** `index.html`
- **Current mitigation:** None.
- **Recommendations:** Add a strict CSP meta tag. Example:
  ```html
  <meta http-equiv="Content-Security-Policy"
        content="default-src 'self'; script-src 'self' https://cdn.jsdelivr.net; style-src 'self' 'unsafe-inline'; img-src 'self' data:;">
  ```
  (Remove `'unsafe-inline'` once styles are extracted to an external file.)

### No Subresource Integrity (SRI) on CDN Script
- **Risk:** VexFlow is loaded from `https://cdn.jsdelivr.net/npm/vexflow@4.2.5/build/cjs/vexflow-bravura.js` with no `integrity` attribute. A CDN compromise or unexpected file change injects arbitrary code into the app's execution context.
- **Files:** `index.html` (line 7)
- **Current mitigation:** None.
- **Recommendations:** Generate an SRI hash for the exact file and add:
  ```html
  <script src="..."
          integrity="sha384-..."
          crossorigin="anonymous"></script>
  ```

### Unprotected localStorage Access
- **Risk:** `localStorage.setItem` and `localStorage.getItem` can throw under Safari Private Browsing (`QuotaExceededError`) or when storage APIs are disabled. An unhandled exception crashes the script.
- **Files:** `index.html` (lines 374, 441)
- **Current mitigation:** Direct unwrapped calls.
- **Recommendations:** Wrap in `try/catch`. Example:
  ```javascript
  function saveBestStreak(val) {
    try { localStorage.setItem('notereader-best', val); } catch (_) { /* degrade silently */ }
  }
  ```

## Performance Bottlenecks

### Full DOM Rebuild on Every Interaction
- **Problem:** `drawGrandStaff` sets `container.innerHTML = ''` (line 269) and reconstructs the entire SVG — staves, clefs, brace, bar line, target note, and all hints — on every question, hint toggle, and clef change.
- **Files:** `index.html` (lines 267–366)
- **Cause:** No DOM diffing or layered updates. The base staff (clefs, brace) is static across questions but is redrawn every time.
- **Improvement path:** Cache the base SVG structure after the first render. In subsequent calls, clear only the note/hint layers, or use separate `<g>` groups for notes and hints and update only those groups.

### Repeated DOM Queries Without Caching
- **Problem:** `document.querySelector('#staffContainer svg')` is executed multiple times per render cycle (lines 242, 322) and `document.getElementById` is called repeatedly for the same elements.
- **Files:** `index.html` (lines 242, 322, and throughout the event handlers)
- **Cause:** No caching of DOM references in module-scope variables.
- **Improvement path:** Cache references once at init:
  ```javascript
  const els = {
    staffContainer: document.getElementById('staffContainer'),
    staffArea: document.getElementById('staffArea'),
    // ...etc
  };
  ```

## Fragile Areas

### Scattered Hardcoded Geometry Constants
- **Files:** `index.html` (lines 273–275, 277–278, 302, 317–318, 339–348)
- **Why fragile:** Staff dimensions (`320×230`), margins (`x=10`, `width=280`), note spacing logic (magic numbers `11`, `5`, `9`, `7`), and connector placements are hardcoded and interleaved with rendering logic. Changing the SVG canvas size or adding responsive scaling requires touching disparate values in both CSS and JS.
- **Safe modification:** Extract all geometric constants into a single `CONFIG` object at the top of the script:
  ```javascript
  const CONFIG = {
    svgWidth: 320, svgHeight: 230,
    staffX: 10, staffWidth: 280,
    trebleY: 10, bassY: 90,
    leftMargin: 60,
    staffLineSpacing: 10, // verify against VexFlow
  };
  ```
- **Test coverage:** No automated tests verify rendered SVG geometry. All positioning changes rely on manual visual inspection.

### CDN-Dependent Rendering Without Fallback
- **Files:** `index.html` (line 7)
- **Why fragile:** If jsdelivr.net is unreachable, blocked, or the file path changes, `Vex` is `undefined`. Line 271 (`const {Renderer, ...} = Vex.Flow;`) throws a `TypeError`, halting the entire script and leaving a blank page.
- **Safe modification:** Add a `window.Vex` guard and a user-friendly fallback message. Consider vendoring the VexFlow build artifact into the repo or using a bundler.

### No Accessibility (A11y)
- **Files:** `index.html` (entire file)
- **Why fragile:** Buttons have no `aria-label` or `aria-pressed` states. The SVG staff has no `role="img"` or `<title>` element. Keyboard-only users and screen-reader users have no way to perceive the quiz content.
- **Safe modification:** Add ARIA attributes to all interactive elements and provide an alternative text or audio representation of the target note.

### Bass Clef Diatonic Path Mixes Clefs
- **Files:** `index.html` (lines 207–233)
- **Why fragile:** `buildDiatonicPath` always starts from `c/4` (middle C) regardless of clef. For bass targets like `a/2`, the path descends from treble territory (`c/4`) through `b/3`, `a/3`, etc. This pedagogically confuses the two clefs for learners.
- **Safe modification:** Generate paths confined to the active clef's own range, starting from the nearest octave boundary within that clef.

### Tautological Conditional in `updateHintButtons`
- **Files:** `index.html` (lines 384–386)
- **Why fragile:**
  ```javascript
  if (state.hintLevel >= 1) {
    pathBtn.textContent = state.hintLevel >= 1 ? 'Path Shown' : 'Show Path';
  }
  ```
  The inner ternary is dead code — it is inside an `if` that already guarantees the condition.
- **Safe modification:** Replace with `pathBtn.textContent = 'Path Shown';` inside the block.

## Scaling Limits

### File Size and Maintainability Ceiling
- **Current capacity:** `index.html` is 507 lines / ~20 KB.
- **Limit:** Adding features (accidentals, rhythm exercises, MIDI input, leaderboard) requires appending to this file. Without modules, the file becomes unmaintainable and untestable.
- **Scaling path:** Modularize into layered modules (`state.js`, `render.js`, `audio.js`, `config.js`) before adding new game modes.

### No Progressive Web App (PWA) Support
- **Current capacity:** Served as a static HTML file with a CDN dependency.
- **Limit:** No service worker, no `manifest.json`, no offline caching. Users in low-connectivity environments (e.g., music classrooms) cannot use the app if the CDN is unavailable.
- **Scaling path:** Add a `manifest.json`, register a service worker that caches `index.html` and a vendored VexFlow copy, and provide offline fallbacks.

## Dependencies at Risk

### VexFlow 4.2.5 via CDN
- **Risk:** No lockfile or reproducible build. CDN patch releases or availability problems can break the app.
- **Impact:** Complete rendering failure. User sees a blank staff area.
- **Migration plan:** Pin the exact version with SRI (see Security). Long-term: vendor the build artifact or manage VexFlow via npm and bundle it.

## Missing Critical Features

### No Automated Tests
- **Problem:** Zero unit, integration, or visual regression tests exist. The codebase is entirely untested.
- **Blocks:** Safe refactoring of `pickNote`, `drawGrandStaff`, or the Y-position math depends entirely on manual QA.
- **Priority:** High.

### No Error Boundaries or Try/Catch
- **Problem:** Any exception — VexFlow rejecting an invalid note key, `localStorage` throwing, or a DOM query returning `null` — crashes the app with no recovery UI.
- **Blocks:** Resilient user experience.
- **Priority:** Medium.

### No Multi-Language / Accessibility Alternatives
- **Problem:** Note names are English-only (C–B). No solfège (Do–Re–Mi) or other notation systems are supported. No screen-reader descriptions.
- **Blocks:** International and visually impaired users.
- **Priority:** Medium.

## Test Coverage Gaps

### Rendering Logic
- **What's not tested:** Correct SVG output for each note in treble and bass clefs, hint ellipse positioning, and visual feedback CSS class toggling.
- **Files:** `index.html` (lines 267–366)
- **Risk:** Visual regressions in note placement go unnoticed until users report them.
- **Priority:** High.

### Game State Logic
- **What's not tested:** Score increments, streak logic, `bestStreak` persistence, `pickNote` deduplication behavior, and rapid-input edge cases.
- **Files:** `index.html` (lines 368–462)
- **Risk:** Score corruption, streak resetting incorrectly, or certain notes never appearing.
- **Priority:** High.

### Hint System
- **What's not tested:** Ascending vs. descending path generation, hint level transitions, and DOM updates after hint button clicks.
- **Files:** `index.html` (lines 207–265, 478–494)
- **Risk:** Hint notes render off-canvas, with wrong labels, or not at all.
- **Priority:** Medium.

---

*Concerns audit: 2026-04-29*
