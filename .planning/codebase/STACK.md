# Technology Stack

**Analysis Date:** 2026-04-29

## Languages

**Primary:**
- HTML5 — Document structure and markup. Single `index.html` containing inline CSS and JavaScript.
- CSS3 — All styling is inline within `<style>` tags in `index.html`.
- JavaScript (ES2015+) — Application logic is inline within `<script>` tags in `index.html`.

**Secondary:**
- Not applicable — no build-step languages (TypeScript, SCSS, etc.) are used.

## Runtime

**Environment:**
- Browser runtime (client-side only). No server runtime.

**Package Manager:**
- Not applicable. No `package.json`, `npm`, `yarn`, `pnpm`, or other package manager is used.
- No lockfile present.

## Frameworks

**Core:**
- VexFlow 4.2.5 — Music notation engraving library, loaded via CDN. Renders SVG staves, clefs, notes, and connectors.
  - Import: `const { Renderer, Stave, StaveNote, StaveConnector, Voice, Formatter } = Vex.Flow`

**Testing:**
- Not detected.

**Build/Dev:**
- Not applicable. No build tooling, bundler, transpiler, or dev server. File is served directly as static HTML.

## Key Dependencies

**Critical:**
- VexFlow 4.2.5 (`vexflow-bravura.js`) — Bravura font build via jsdelivr CDN. The app uses `Renderer`, `Stave`, `StaveNote`, `StaveConnector`, `Voice`, and `Formatter` to render the grand staff and target note.
- Bravura music font — Bundled inside the VexFlow Bravura build for glyph rendering.

**Infrastructure:**
- Not applicable.

## Configuration

**Environment:**
- No environment configuration files (`.env`, `.env.local`, etc.).
- No build or bundler configuration.
- No `tsconfig.json`, `babel.config.js`, `vite.config.js`, or similar.

**Build:**
- None. The app is a single static file (`index.html`, ~20 KB) served directly.

## Platform Requirements

**Development:**
- Any modern web browser (Chrome, Firefox, Safari, Edge).
- Text editor only. No build step, no install step.
- For live preview, can open `index.html` directly in a browser or serve via any static file server (e.g., `python -m http.server`).

**Production:**
- Deployed as static HTML to GitHub Pages.
- Requires CDN access to `https://cdn.jsdelivr.net/npm/vexflow@4.2.5/build/cjs/vexflow-bravura.js` at runtime.
- No server-side runtime required.

---

*Stack analysis: 2026-04-29*
