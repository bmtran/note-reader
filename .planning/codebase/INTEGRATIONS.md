# External Integrations

**Analysis Date:** 2026-04-29

## APIs & External Services

**Music Notation Rendering:**
- VexFlow 4.2.5 via jsdelivr CDN
  - What it's used for: Rendering SVG grand staff (treble + bass), clefs, note heads, stave connectors, and voice/formatter layout.
  - SDK/Client: `https://cdn.jsdelivr.net/npm/vexflow@4.2.5/build/cjs/vexflow-bravura.js` loaded via `<script>` tag in `index.html`.
  - Auth: Not applicable (public CDN).

**No other external APIs or services.** No backend, no analytics, no telemetry, no auth provider.

## Data Storage

**Databases:**
- Not applicable.

**File Storage:**
- Local filesystem only. Single static file (`index.html`).

**Caching:**
- Browser localStorage for persisting best streak score between sessions.
  - Key: `notereader-best`
  - Value: integer (best streak length)

**Session Storage:**
- Not used.

## Authentication & Identity

**Auth Provider:**
- Not applicable. No user accounts, login, sign-up, or identity service.

## Monitoring & Observability

**Error Tracking:**
- Not detected. No Sentry, Bugsnag, Rollbar, or similar service.

**Logs:**
- Not detected. No structured logging framework. Console output is not used for application events.

## CI/CD & Deployment

**Hosting:**
- GitHub Pages (`https://bmtran.github.io/note-reader/`)

**CI Pipeline:**
- Not detected. No GitHub Actions, Travis CI, or other CI configuration files present.

## Environment Configuration

**Required env vars:**
- Not applicable. No environment variables or secret configuration needed.

**Secrets location:**
- Not applicable. No secrets, API keys, or tokens are used.

## Webhooks & Callbacks

**Incoming:**
- Not applicable. No server-side endpoints.

**Outgoing:**
- Not applicable. No outbound webhooks.

## Browser APIs Used

**DOM & Events:**
- `document.querySelector` / `document.querySelectorAll`
- `document.getElementById`
- `Element.addEventListener('click', ...)`
- `Element.addEventListener('keydown', ...)`
- `document.createElementNS('http://www.w3.org/2000/svg', ...)` — raw SVG injection for hint notes and labels

**Storage:**
- `localStorage.getItem('notereader-best')`
- `localStorage.setItem('notereader-best', value)`

**Timing:**
- `setTimeout(...)` for delayed question transitions (900ms on correct, 1500ms on wrong)

---

*Integration audit: 2026-04-29*
