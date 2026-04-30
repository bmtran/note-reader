# Testing Patterns

**Analysis Date:** 2026-04-29

## Test Framework

**Current status:** **None detected.**

The `note-reader` project has no testing infrastructure whatsoever:

- No test runner (Jest, Vitest, Mocha, Playwright, Cypress, etc.)
- No test files (`*.test.*`, `*.spec.*`)
- No test configuration files (`jest.config.*`, `vitest.config.*`, `playwright.config.*`, etc.)
- No CI/CD pipeline that would execute automated tests

## Why Testing Is Absent

This project is intentionally a **zero-build, single-file HTML application** (`index.html`). The author optimized for:

- Zero dependencies (except the VexFlow CDN script)
- Zero build steps
- Direct deployment to GitHub Pages

Consequently, there is no `package.json`, no `node_modules`, and no toolchain into which a test framework could be introduced.

## How to Add Testing (Recommended Path)

Because the application logic is tightly coupled to DOM rendering and the global `Vex.Flow` API, testing requires either:

1. **Modularize the code first**, then add unit tests.
2. **Use an E2E / browser automation framework** that can run the existing single file.

### Option A: Browser-Level E2E Testing (Immediate)

A test framework like **Playwright** or **Puppeteer** can run the file directly in a headless browser and perform user-level assertions.

**Suggested dependency:**
```bash
npm init -y
npm install --save-dev @playwright/test
npx playwright install
```

**Sample test** (`tests/note-reader.spec.js`):
```javascript
import { test, expect } from '@playwright/test';

test('renders a note and accepts a correct answer', async ({ page }) => {
  await page.goto('file://' + __dirname + '/../index.html');

  // Wait for staff to render
  await page.waitForSelector('#staffContainer svg');

  // Read the clef mode
  const clef = await page.evaluate(() => window.state.current.clef);
  const note = await page.evaluate(() => window.state.current.note.name);

  // Click the correct note button
  await page.locator(`.note-btn[data-note="${note}"]`).click();

  // Assert positive feedback
  await expect(page.locator('#feedback')).toHaveClass(/correct/);
  await expect(page.locator('#scoreVal')).toHaveText('1 / 1');
});
```

**Why this works:** The app exposes `state` on the global `window` object (`state` is declared at the top level of the inline script), so tests can inspect internal state via `page.evaluate`.

### Option B: Modularize for Unit Testing (Recommended Long Term)

Extract pure logic into ES modules and unit-test them with **Vitest** (fast, native ESM support, no DOM required for pure functions).

**Suggested file split:**

```
index.html          # Shell + CSS
src/
  main.js           # Entry point: DOM setup, event wiring
  state.js          # Game state factory + rules
  render.js         # VexFlow rendering logic
  helpers.js        # Pure functions: buildDiatonicPath, keyToLabel, pickNote
```

**Example: extracting `helpers.js` and testing it:**

`src/helpers.js`:
```javascript
const SCALE = ['c', 'd', 'e', 'f', 'g', 'a', 'b'];

export function buildDiatonicPath(targetKey) {
  const [tNote, tOctStr] = targetKey.split('/');
  const tOct = parseInt(tOctStr);
  const tIdx = SCALE.indexOf(tNote);
  const keys = [];
  if (tOct > 4 || (tOct === 4 && tIdx >= 0)) {
    let ci = 0, co = 4;
    while (true) {
      keys.push(`${SCALE[ci]}/${co}`);
      if (ci === tIdx && co === tOct) break;
      ci++; if (ci >= 7) { ci = 0; co++; }
    }
  } else {
    let ci = 0, co = 4;
    while (true) {
      keys.push(`${SCALE[ci]}/${co}`);
      if (ci === tIdx && co === tOct) break;
      ci--; if (ci < 0) { ci = 6; co--; }
    }
  }
  return keys;
}

export function pickNote(clefMode, randomFn = Math.random) {
  const TREBLE = [
    { name:'C', key:'c/4' }, /* ... */ { name:'G', key:'g/5' }
  ];
  const BASS = [
    { name:'A', key:'a/2' }, /* ... */ { name:'C', key:'c/4' }
  ];
  const clef = clefMode === 'both'
    ? (randomFn() < 0.5 ? 'treble' : 'bass')
    : clefMode;
  const pool = clef === 'treble' ? TREBLE : BASS;
  const unique = [];
  const seen = new Set();
  for (const n of pool) {
    if (!seen.has(n.name)) { unique.push(n); seen.add(n.name); }
  }
  const note = unique[Math.floor(randomFn() * unique.length)];
  return { clef, note };
}
```

`tests/helpers.test.js`:
```javascript
import { describe, it, expect } from 'vitest';
import { buildDiatonicPath, pickNote } from '../src/helpers';

describe('buildDiatonicPath', () => {
  it('ascends from c/4 to g/5', () => {
    expect(buildDiatonicPath('g/5')).toEqual([
      'c/4','d/4','e/4','f/4','g/4','a/4','b/4','c/5','d/5','e/5','f/5','g/5'
    ]);
  });

  it('descends from c/4 to a/2', () => {
    expect(buildDiatonicPath('a/2')).toEqual([
      'c/4','b/3','a/3','g/3','f/3','e/3','d/3','c/3','b/2','a/2'
    ]);
  });
});

describe('pickNote', () => {
  it('returns treble when random < 0.5', () => {
    const result = pickNote('both', () => 0.3);
    expect(result.clef).toBe('treble');
  });

  it('returns bass when random >= 0.5', () => {
    const result = pickNote('both', () => 0.7);
    expect(result.clef).toBe('bass');
  });
});
```

**Benefits:** Fast tests, no browser needed, deterministic via injected `randomFn`.

## Mocking Patterns

**Current status:** N/A — no tests exist, therefore no mocking.

**If testing the existing single file:**
- Mock `localStorage` globally to avoid real persistence during tests:
  ```javascript
  const storage = {};
  global.localStorage = {
    getItem: (k) => storage[k] ?? null,
    setItem: (k, v) => { storage[k] = String(v); },
  };
  ```
- Mock `Math.random` to make `pickNote` deterministic (see `randomFn` parameter suggestion above).

## Coverage

**Current status:** Not applicable.

**If testing is introduced:**
- Aim for **unit coverage of pure helper functions** (`buildDiatonicPath`, `keyToLabel`, `pickNote`) first.
- **E2E coverage** of the core user flow: start app → see note → click correct answer → score increments → next note appears.

## Test Types Recommended

| Type | Framework | Scope |
|------|-----------|-------|
| Unit | Vitest | Pure logic: `buildDiatonicPath`, `pickNote`, `keyToLabel` |
| Component | Playwright | `drawGrandStaff` output (SVG structure assertions) |
| E2E | Playwright | Full user journey: clef toggle, hints, keyboard input, scoring |

## CI/CD Testing

**Current status:** None.

**If a CI pipeline is added** (e.g., GitHub Actions), recommended workflow:

```yaml
# .github/workflows/test.yml
name: Test
on: [push, pull_request]
jobs:
  e2e:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20 }
      - run: npm ci
      - run: npx playwright install --with-deps
      - run: npx playwright test
```

---

*Testing analysis: 2026-04-29*
