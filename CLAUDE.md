# Winetemp — Wine Temperature PWA

Client-side PWA (vanilla HTML/CSS/JS, no build step) hosted on GitHub Pages at https://realtj0.github.io/winetemp/. Source repo: https://github.com/realtj0/winetemp.

Files: `index.html`, `manifest.json`, `service-worker.js`, `icon-192.svg`.

## IMPORTANT: Cache busting rule

**Whenever I change `index.html` (or any cached asset), I MUST also bump `CACHE_NAME` in `service-worker.js`.**

```js
const CACHE_NAME = 'winetemp-vN';  // increment N on every asset change
```

Why: the service worker uses a cache-first strategy and the `activate` handler only clears caches whose name differs from `CACHE_NAME`. If the constant isn't bumped, users keep seeing the old cached version indefinitely, even after GitHub Pages reports "deployed".

## Update flow (do not break)

The SW does NOT call `skipWaiting()` in `install` — this is intentional. The flow is:
1. Bumping `CACHE_NAME` triggers a new SW installation on next visit.
2. The new SW stays in "waiting" state.
3. `index.html` detects this via `updatefound` + `statechange === 'installed'` and shows the `#update-banner` ("Nouvelle version disponible — Recharger").
4. User clicks Recharger → page posts `{type: 'SKIP_WAITING'}` → SW calls `skipWaiting()` → `clients.claim()` in `activate` fires `controllerchange` on the page → page reloads with the new version.

If you ever need a silent/forced update without the banner, put `self.skipWaiting()` back in `install` — but don't do that casually, it reloads users' pages without warning.

## Deployment

No build, no dependencies. Edit files → push to `main` branch → GitHub Pages auto-deploys. Remember the cache-busting rule above.
