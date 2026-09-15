# web-pwa

Composable modifier that adds offline support and installability to an existing web frontend: a hand-written service worker with a versioned Cache API precache, plus a minimal web app manifest. No build step, no generated code, no new dependencies — the precache list is hand-maintained source.

## Composition

Composes on top of [web-static](../web-static), [web-sprinkles](../web-sprinkles), or [web-components](../web-components) and their shared [web-conventions](../web-conventions). It adds exactly one capability — the app shell keeps working without a network — and relaxes exactly two rules:

- the [web-conventions](../web-conventions) Baseline Policy for the web app manifest (Baseline limited), permitted as a declared progressive enhancement — inert where unsupported
- the `web-static` no-JavaScript constraint for the single infrastructure file `sw.js` — the pages themselves stay JavaScript-free and fully functional without the worker

Everything else stays binding: no build system, no dependencies (no Workbox, no `vite-plugin-pwa`), the `web-components` routing rules, the `web-sprinkles` enhancement contract, accessibility, and design-token discipline.

## Shape

- `sw.js` at the served root: hand-maintained precache array of root-absolute URLs, versioned cache name, `install` → precache, `activate` → delete old caches, `fetch` → cache-first for precached same-origin GETs
- with [web-components](../web-components), navigation requests answered with the cached `/index.html` — reproducing the SPA fallback so deep links work offline; cache keys are the import-map-resolved URLs
- feature-detected registration from the page; the versioned cache is the whole update flow
- `manifest.webmanifest` with the minimal field set and a maskable icon, colors from the design tokens

## Verification

The composed stack's loop runs unchanged. Existing [web-system-tests](../web-system-tests) specs and coverage runs block service workers (`serviceWorkers: "block"`); one dedicated spec verifies offline: load online, go offline, reload, assert the app renders — for SPAs also a deep route.

## Usage

`SKILL.md` contains the full modifier rules. Apply it together with a stack skill — it does nothing on its own.
