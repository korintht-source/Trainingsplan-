# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A static, offline-capable PWA (Progressive Web App) — no build system, no package manager, no dependencies. Two self-contained HTML files, each with inline `<style>` and `<script>`, deployed as-is via GitHub Pages. Everything (markup, CSS, JS, SVG figures, workout/nutrition data) lives inline in the HTML; there is no bundler, transpiler, or framework.

- `index.html` — 15-minute bodyweight workout timer/player (German UI). Contains the exercise programs (`PROGRAMS.A` / `PROGRAMS.B`), the interval sequencer (`buildSeq`), the countdown/render loop (`step`/`render`), Web Audio beeps (`beep`), and inline stick-figure SVGs (`FIG`).
- `ernaehrung.html` — nutrition/calorie calculator and weight-tracking chart. Persists weight entries to `localStorage` (key `tp_weights`, with an in-memory fallback via the `Store` object if storage is unavailable), computes a rolling 7-day average, and renders an SVG line chart by hand (no charting library).
- `sw.js` — service worker, cache-first strategy. **`CACHE` version string must be bumped (`trainingsplan-v2` → `v3`, ...) whenever any cached file changes**, otherwise installed clients keep serving stale cached files.
- `manifest.webmanifest` — PWA manifest (icons, theme colors, standalone display).
- `ANLEITUNG.md` — German end-user setup guide (GitHub repo → GitHub Pages → iOS "Add to Home Screen"). Not needed for the app to run but explains the deployment target: GitHub Pages serving the repo root, accessed on iOS Safari.

## Working in this codebase

- There is no build/lint/test tooling and none should be added unless explicitly requested — open the HTML files directly in a browser (or serve the repo root with any static file server) to check changes.
- Both HTML files duplicate the same design tokens (CSS custom properties like `--petrol-900`, `--signal`, `--muted`) and the same nav/layout markup. When changing shared visual style, apply the change in both `index.html` and `ernaehrung.html`.
- `sw.js`'s `SHELL` array lists every file that must be cached for offline use. If a new asset is added to the app shell, add it to `SHELL` **and** bump `CACHE`.
- Keep new UI copy in German, consistent with the existing content.
- The stick-figure exercise illustrations (`FIG` object in `index.html`) are hand-authored inline SVG in a consistent side-view style (`fig-body`, `fig-limb`, `fig-head`, `fig-prop`, `fig-ground` classes control stroke/fill via CSS). Follow the same viewBox (`0 0 200 120`) and class conventions when adding a new figure.
