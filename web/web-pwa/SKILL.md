---
name: web-pwa
description: Composable modifier that adds offline support (hand-written service worker + Cache API) and installability (web app manifest) to an existing web frontend. Composes on top of `web-static`, `web-sprinkles`, or `web-components`; every rule of the composed skills stays in force — no build step, no generated code, no new dependencies. Use when a site or app should work without a network connection or be installable to the home screen. Triggers on "PWA", "progressive web app", "offline", "offline support", "work offline", "service worker", "installable", "add to home screen", "web app manifest", "precache", "cache the app". Not for push notifications or background sync — those are separate concerns; does nothing on its own without a composed stack skill.
argument-hint: "[site or app to make offline-capable / installable]"
---

Add offline support and installability to $ARGUMENTS. This is a modifier skill — it adds
exactly one capability to the composed stack: the app shell keeps working without a network.

## Purpose & Precedence

Service workers and the Cache API are web standards (Baseline **Widely Available** since 2020,
`service-workers` in the `/web-conventions` bundled snapshot), so offline support fits the
platform-first stacks without any tooling. When composed with `/web-static`, `/web-sprinkles`,
or `/web-components` (and their shared `/web-conventions`), this skill overrides:

- the **Baseline Policy** of `/web-conventions` for exactly one feature family: the web app
  manifest (`manifest`, Baseline **limited**) is permitted as a declared progressive
  enhancement — a manifest is inert in browsers that ignore it, nothing breaks, and the
  limited status is stated honestly in the deliverable
- `/web-static`'s no-JavaScript constraint for exactly one file: `sw.js` is infrastructure
  outside the pages, not a page enhancement — the pages themselves remain JavaScript-free
  and fully functional if the service worker never installs

Everything else in the composed skills remains binding. Explicitly **not** relaxed:

- the no-build rule — no Workbox, no `vite-plugin-pwa`, no generated precache manifests;
  the precache list is hand-maintained source code
- dependency rules — the service worker adds zero dependencies
- `/web-components`' routing rule "reload means reload" — the service worker serves cached
  responses to requests the browser makes; it never suppresses or fakes a reload
- `/web-sprinkles`' enhancement contract and `/web-static`'s verification loop
- accessibility, semantic HTML, and design-token discipline

## Service Worker Rules

One hand-written classic script `sw.js` at the served root, so its scope covers the whole
site. Keep it a classic script, not a module — `js-modules-service-workers` is Baseline
**Newly Available**; revisit when it reaches Widely.

- **Precache list** — a `const` array of root-absolute URLs at the top of `sw.js`, listing
  every file the app shell needs: the HTML document(s), stylesheets, scripts/modules, vendored
  libraries, and the favicon. Hand-maintained: adding a source file to the project means
  adding its URL to the list, and the stack's verification catches omissions
- **Versioned cache name** — a single `const CACHE = "<app>-v<version>"`; bump the version on
  every change to any precached file. In `/web-components` projects co-bump it with
  `appVersion` in `app.config.js`
- **`install`** — open the cache, `cache.addAll(PRECACHE)`, nothing else
- **`activate`** — delete every cache whose name is not the current `CACHE`
- **`fetch`** — cache-first for same-origin GET requests in the precache list; everything
  else goes to the network untouched

```js
const CACHE = "app-v1";
const PRECACHE = ["/", "/index.html", "/style.css", "/app.js"];

self.addEventListener("install", event =>
    event.waitUntil(caches.open(CACHE).then(cache => cache.addAll(PRECACHE))));

self.addEventListener("activate", event =>
    event.waitUntil(caches.keys().then(keys =>
        Promise.all(keys.filter(key => key !== CACHE).map(key => caches.delete(key))))));

self.addEventListener("fetch", event => {
    const { request } = event;
    if (request.method !== "GET" || new URL(request.url).origin !== location.origin) return;
    event.respondWith(caches.match(request, { ignoreSearch: false })
        .then(cached => cached ?? fetch(request)));
});
```

## Stack-Specific Fetch Handling

The stacks differ in exactly one place — how navigation requests are answered:

- **`/web-components` (SPA)** — the deploy target serves `index.html` for unknown paths
  (client-side routing); the service worker must reproduce that fallback or deep links like
  `/edit/17` 404 offline. Answer every `request.mode === "navigate"` with the cached
  `/index.html`. Precache keys are the **import-map-resolved URLs** (e.g. `/libs/lit-html.js`),
  never bare specifiers — the browser requests the mapped URL, and that is the cache key.
  Do not precache vendored files the active import map does not reference
- **`/web-static` and `/web-sprinkles`** — no navigation fallback; every page is a real file
  and belongs in the precache list, and an uncached navigation goes to the network

## Registration

Register from the page as a progressive enhancement — the site must be fully functional when
the registration never runs:

```html
<script>
    if ("serviceWorker" in navigator)
        navigator.serviceWorker.register("/sw.js");
</script>
```

In `/web-components` projects the registration belongs in `index.html` (or the entry module),
never inside components. No update-nagging UI: the versioned cache plus the `activate` cleanup
is the whole update flow — a normal reload after the new worker activates serves the new
version.

## Web App Manifest

`manifest.webmanifest` at the served root, linked from every HTML document:

```html
<link rel="manifest" href="/manifest.webmanifest">
<meta name="theme-color" content="#...">
```

Minimal field set — `name`, `short_name`, `start_url`, `display` (usually `standalone`),
`background_color`, `theme_color`, and `icons` with at least a 192px and a 512px PNG, one of
them `"purpose": "maskable"`. Take colors from the project's design tokens. Restate in the
project README that the manifest is Baseline limited and installed as progressive enhancement.

## Verification Loop Amendment

The composed stack's verification loop runs unchanged, with these additions:

- **Existing specs stay service-worker-free** — set `serviceWorkers: "block"` in the
  Playwright `use` config of existing `/web-system-tests` specs and coverage runs, so cached
  responses never mask a regression and coverage instrumentation is not bypassed
- **One dedicated offline spec** in its own project or config without the block: load the
  site online, wait for the registration (`await page.evaluate(() => navigator.serviceWorker.ready)`),
  set `context.setOffline(true)`, reload, and assert the app renders; with `/web-components`
  additionally open a deep route offline
- **Manual check** — serve with zws, open DevTools → Application: the worker is activated,
  the cache holds exactly the precache list, and toggling the Network "Offline" checkbox and
  reloading still renders the site; the manifest panel shows name and icons without errors

## What NOT to Do

- do not use Workbox, `workbox-cli`, `vite-plugin-pwa`, or any precache-manifest generator —
  the list is hand-written source
- do not cache non-GET or cross-origin requests, and do not intercept requests outside the
  precache list beyond passing them to the network
- do not `importScripts` or import app modules into `sw.js` — the worker is self-contained
- do not add push notifications or background sync — out of scope, separate concern
- do not use an unversioned cache name or skip the `activate` cleanup — stale caches are the
  classic service-worker failure mode
- do not register the worker inside components or feature modules
- do not let the service worker interfere with the dev loop — zws with `--live` reload works
  against the network; when cached responses confuse local development, unregister via
  DevTools rather than adding dev-mode conditionals to `sw.js`
