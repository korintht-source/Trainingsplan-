# CLAUDE.md

Guidance for Claude Code (and other AI assistants) working in this repository.

## What this is

A German-language, installable PWA (Progressive Web App) with two pages:

- **`index.html`** — a 15-minute bodyweight interval-training timer. Two alternating
  programs (A: push/core, B: pull/legs), each with 6 exercises, 3 rounds, 35 s work /
  15 s rest. Renders hand-drawn SVG stick-figure illustrations per exercise, runs a
  work/rest countdown with audio beeps (Web Audio API), keeps the screen awake during a
  session (Screen Wake Lock API), and supports keyboard control (space bar = start/pause).
- **`ernaehrung.html`** ("Nutrition") — a daily calorie/macro target calculator (based on
  hours of cycling and intensity) plus a bodyweight tracker with an inline SVG line chart
  (7-day rolling average) and a trend verdict, persisted to `localStorage`.

There is no backend, no build step, and no package manager — this is deployed as-is via
**GitHub Pages** and installed on iOS via Safari's "Add to Home Screen". See
`ANLEITUNG.md` for the (German) end-user setup instructions — that file is
user-facing documentation, not developer docs, and does not need to be kept in sync with
this file.

## Repository layout

```
index.html              Training timer page (styles + logic inline)
ernaehrung.html          Nutrition/weight-tracking page (styles + logic inline)
manifest.webmanifest      PWA manifest (name, icons, theme colors, start_url)
sw.js                     Service worker: cache-first offline support
apple-touch-icon.png       iOS home-screen icon
icon-192.png / icon-512.png / icon-512-maskable.png   PWA icons (referenced by manifest)
ANLEITUNG.md               German step-by-step guide for non-technical users to deploy
                            this repo to GitHub Pages and install it on an iPhone
```

There is no `src/`, no bundler, no `node_modules`, no test suite, and no CI config.
Each HTML file is fully self-contained: a `<style>` block in the `<head>` and a
`<script>` block before `</body>`. Treat each page as one unit — there's no shared CSS
or JS file to import from.

## Development workflow

- **No install step.** Open the HTML files directly in a browser, or serve the directory
  with any static file server (e.g. `python3 -m http.server`) if you need `fetch`/service
  worker behavior that doesn't work well from `file://`.
- **No build, no bundler, no transpiler.** Write plain ES2017+ JS and modern CSS directly
  in the files; nothing compiles them.
- **No automated tests and no linter are configured.** Validate changes by opening the
  page in a browser and exercising the feature manually (start/pause the timer, switch
  programs, add a weight entry, resize to a narrow viewport, toggle dark/light where
  relevant). There is no headless test to run instead of this.
- **Service worker caching will hide your changes** if you're testing via GitHub Pages
  or a previously-installed PWA — see "Service worker cache versioning" below. When
  developing locally, use a private/incognito window or unregister the service worker
  in DevTools to avoid stale caches.

## Key conventions

### Language and audience
All UI copy, code comments, and commit-adjacent docs (`ANLEITUNG.md`) are in **German**.
Keep new user-facing strings in German and in the existing tone (short, direct,
imperative). Variable/function names in the JS are in English, which is the existing
mixed style — follow it rather than "fixing" it.

### No shared layout — pages are kept in sync by hand
Both pages repeat the same `<nav>` markup:
```html
<nav>
  <a href="index.html" class="on" aria-current="page">Training</a>
  <a href="ernaehrung.html">Ernährung</a>
</nav>
```
with `class="on"`/`aria-current="page"` moved to whichever link matches the current
page. If you add a third page, update the `<nav>` block in **both** existing files and
give the new page the same nav markup. There is no templating system — this is manual.

### CSS
- Each page defines its own CSS custom properties in `:root` (a "petrol" dark color
  scheme: `--petrol-900/800/700`, `--chalk` text, `--signal` yellow accent, `--rest`
  orange for rest/pause states). Reuse these variables rather than hardcoding colors;
  keep the two pages' palettes visually consistent if you touch them.
- Layout is hand-rolled CSS Grid/Flexbox, mobile-first, with a single
  `@media (max-width:400px)` breakpoint per page for small phones, and
  `@media (prefers-reduced-motion: reduce)` to kill transitions.
- `env(safe-area-inset-*)` is used for iOS notch/home-indicator safe areas — preserve
  this when touching body/layout padding.

### JavaScript
- No modules, no framework, no dependencies. Everything is inline `<script>`, using
  plain functions and a handful of `const`/`let` globals scoped to that page.
- Common shorthand used throughout: `const $ = id => document.getElementById(id);`
- `index.html`'s exercise figures are hand-authored inline SVG built via a small `fig()`
  helper and a `FIG` lookup object keyed by exercise id (e.g. `pushup`, `lunge`,
  `squat`). When adding an exercise, add a new SVG entry to `FIG` in the same
  minimalist stick-figure style (viewBox `0 0 200 120`, `.fig-body`/`.fig-limb`/
  `.fig-head` classes) and reference its key from the relevant `PROGRAMS.*.ex` entry.
- `ernaehrung.html` persists data through a small `Store` wrapper around `localStorage`
  that falls back to an in-memory object if `localStorage` is unavailable (private
  browsing, quota errors, etc.). Always go through `Store.get/set/del`, not
  `localStorage` directly, when adding persisted state.
- Defensive `try/catch` is used around browser APIs that can throw or be unavailable
  (Web Audio, Wake Lock, `localStorage`) — follow this pattern for any new use of such
  APIs so the app degrades gracefully instead of crashing.

### Accessibility
Existing patterns to keep: `aria-pressed` on toggle buttons, `aria-live="polite"` on the
live-updating timer region, `aria-hidden="true"` on decorative SVGs, `aria-current="page"`
on the active nav link, and visible `:focus-visible` outlines. Match these when adding
interactive elements.

## Service worker cache versioning (critical)

`sw.js` is a cache-first service worker. `SHELL` lists every file that gets pre-cached
on install:

```js
const CACHE = "trainingsplan-v2";
const SHELL = [
  "./", "./index.html", "./ernaehrung.html", "./manifest.webmanifest",
  "./apple-touch-icon.png", "./icon-192.png", "./icon-512.png", "./icon-512-maskable.png"
];
```

Whenever you change the contents of **any** cached file (HTML/CSS/JS/icons/manifest):

1. **Bump the `CACHE` version string** (`trainingsplan-v2` → `trainingsplan-v3`, etc.).
   Without this, installed PWAs (especially on iOS) keep serving the old cached version
   indefinitely — the `activate` handler only evicts caches whose key differs from the
   current `CACHE` constant.
2. If you add a **new file** that should work offline, add it to the `SHELL` array too.

This is the single most important thing to remember when editing this repo — it's easy
to change `index.html` or `ernaehrung.html` and forget the version bump, and the bug
this causes ("my change isn't showing up") is silent and confusing for end users.

## Deployment

There is no CI/CD. Deployment is: push to `main` → GitHub Pages serves the repo root
directly (see `ANLEITUNG.md`, Schritt 4, for the one-time Pages setup). Any file at the
repo root is publicly served as-is at `https://<user>.github.io/<repo>/<file>`. Because
the app is meant to be installed as an offline PWA, always pair a content change with
the cache-version bump described above before considering the change "done".

## Non-goals / things not to introduce without being asked

- Don't add a build tool, bundler, framework, or package.json unless explicitly
  requested — the project's simplicity (open two HTML files, no install) is a deliberate
  property described in `ANLEITUNG.md`.
- Don't split the inline `<style>`/`<script>` into external files unless asked — the
  service worker and Pages deployment model assume a small, flat set of cacheable files,
  and splitting increases the number of things to keep versioned together.
- Don't introduce a backend, analytics, or third-party network calls — the app's offline
  guarantee depends on there being none.
