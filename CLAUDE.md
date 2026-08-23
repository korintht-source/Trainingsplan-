# Trainingsplan

A static, offline-first PWA: a 15-minute bodyweight home-workout program with an
interval timer, plus a nutrition page. No build step, no dependencies, no
package.json — everything runs directly from static files served over HTTPS
(GitHub Pages).

## Files

- `index.html` — the workout page: interval timer, Programm A/B switch, all CSS
  and JS inline in a single file.
- `ernaehrung.html` — the nutrition page, same single-file style.
- `manifest.webmanifest` — PWA manifest (name, icons, `display: standalone`).
- `sw.js` — service worker, cache-first. `SHELL` lists every file that must be
  cached for offline use.
- `apple-touch-icon.png`, `icon-192.png`, `icon-512.png`, `icon-512-maskable.png`
  — app icons; must stay in the repo root (not a subfolder) or the iOS
  home-screen icon breaks.
- `ANLEITUNG.md` — German end-user setup guide (GitHub Pages + "Add to Home
  Screen" on iPhone). Not needed by the app itself.

## Architecture notes

- Everything is inline (no separate `.css`/`.js` files) — keep it that way
  unless there's a real reason to split, since the service worker cache list
  and the single-file simplicity both depend on the current file set.
- German-language UI (`lang="de"`); keep user-facing text in German.
- No framework, no bundler. Edit HTML/CSS/JS directly in the two `.html` files.

## Cache-busting (important)

`sw.js` uses cache-first: once a file is cached, the app won't re-fetch it
until the cache name changes. **Any change to a cached file requires bumping
the `CACHE` constant** in `sw.js` (e.g. `trainingsplan-v2` → `trainingsplan-v3`),
otherwise installed users keep seeing the old version indefinitely. If you add
or rename a file, also update the `SHELL` array in `sw.js`.

## Deployment

No CI/build pipeline. Deployment is: push to `main` → GitHub Pages serves the
repo root directly. There is no staging environment.

## Testing changes

There's no test suite. Verify by opening the HTML files in a browser
(or a local static server) and exercising the timer / program switch by hand;
for PWA-specific behavior (install, offline cache, service worker update) test
via a real GitHub Pages deploy, since `file://` and service workers don't mix
well.
