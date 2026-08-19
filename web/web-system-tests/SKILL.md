---
name: web-system-tests
description: The web realization of the system-tests contract — browser-driven system tests with Playwright against a running frontend. Owns the project layout (`tests/`), the Playwright configuration (`baseURL` plus `webServer`, cross-engine projects), the role- and label-based selector policy, code coverage as an opt-in second run, and the green oracle plus its after-every-change loop. Composes with `system-tests` (the black-box contract), `ears-tests` (spec-derived expectations), and the stack skill that owns the application (`web-components`, `web-sprinkles`, `web-static`). Use whenever a web frontend needs end-to-end, browser, or system tests written, run, reviewed, or repaired — triggers on "Playwright", "e2e test", "end-to-end test", "browser test", "system test for the frontend", "test the UI", "cross-browser test", "test coverage for the frontend", "run the e2e tests", "why is this test flaky". Not for auditing a rendered page's markup, accessibility, or Baseline compliance — that is the Chrome DevTools loop in `web-static`.
argument-hint: "[app, page, or business component to test]"
---

Write, run, and maintain browser-driven system tests for $ARGUMENTS. Apply all rules below strictly.

## Composition

- `/system-tests` owns the contract — black box, running system, coordinates as configuration,
  isolation, total verdict reporting. This skill is its **web realization**: layout, configuration,
  selector policy, test syntax, and the green oracle.
- The stack skill owns the application under test — `/web-components` (SPA), `/web-sprinkles`,
  or `/web-static`.
- `/ears-tests` owns spec-derived expectations: one `test.describe` per requirement group, the
  literal `Rn.m` id in the test title.
- `/bce` owns where things live: one spec file per business component, named after it.

### Versus the Chrome DevTools verification loop

`/web-static` drives the rendered page through Chrome DevTools MCP — console, accessibility
snapshot, viewport, Lighthouse, Baseline. That loop inspects **the page as it is right now**:
agent-driven, single-shot, leaving no artifact behind. This skill produces a **committed suite**
re-run by a machine on every change, in three engines, indefinitely.

Neither replaces the other. Rendering, accessibility tree, contrast, and Baseline compliance belong
to the DevTools loop; user-visible **behavior, and its survival across changes**, belongs here.
Where a project has both, green means both.

## The public surface of a web UI is its accessibility tree

`/system-tests` permits assertions through the public surface only. For an HTTP service that surface
is the endpoint; for a web UI it is everything the user can perceive and operate — which is exactly
what the accessibility tree exposes. So the selector policy is a consequence of the contract, not a
style preference:

- Locate by role, label, text, or placeholder: `getByRole`, `getByLabel`, `getByText`, `getByPlaceholder`.
- CSS selectors, ids, class names, `nth-child`, DOM shape, shadow-root piercing, and store internals
  are implementation. A test bound to them fails on a refactor that changed nothing a user can see —
  which is a false alarm, and false alarms are what kill suites.

The payoff shows up when a locator finds nothing: an element with no accessible name is a defect in
the application — a missing `<label>`, a button whose text is an icon, a heading that is a styled
`<div>`. Fix the markup per `/web-conventions`. Reaching for a CSS selector instead hides a real
accessibility bug behind a passing test.

## Project layout

```
project/
├── app/src/                     # the application — stack skill's concern
└── tests/                       # the suite; never shipped with the app
    ├── package.json
    ├── playwright.config.js
    ├── fixtures.js              # only when coverage is enabled
    └── tests/
        └── <business-component>.spec.js
```

Setup, once:

```
npm install -D @playwright/test
npx playwright install          # add --with-deps on CI
```

Tests live outside the served root so the suite is never deployed, and the application keeps its
zero-dependency, no-build-system property — the harness is not the artifact.

## Configuration

```js
// @ts-check
const { defineConfig, devices } = require('@playwright/test');

module.exports = defineConfig({
  testDir: './tests',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    baseURL: process.env.BASE_URL ?? 'http://localhost:3000',
    trace: 'on-first-retry',
  },
  webServer: {
    command: 'java zws ../app/src --single',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
    { name: 'webkit', use: { ...devices['Desktop Safari'] } },
  ],
});
```

- **`baseURL`, and `page.goto('/')` in tests.** A URL literal repeated in every test is the
  hardcoded coordinate `/system-tests` forbids: the suite then runs against one machine only.
  Port 3000 is the default because that is where the dev server already listens; `BASE_URL`
  retargets the whole suite at a deployed preview or a migration target without touching a test.
- **`webServer` with `reuseExistingServer`.** While authoring, your `zws` is already up on 3000 and
  Playwright attaches to it — the suite exercises the same server you are clicking through, with
  live reload intact. On CI nothing is listening, so Playwright starts it. One configuration, no
  flag to remember, no second port.
- **`--single`** serves `index.html` for extension-less unknown paths, which client-side routing
  needs for deep links (`/bookmarks/edit/3`). Drop it for `/web-static` and `/web-sprinkles` sites:
  there a real file backs every URL, and the fallback would turn a broken link into a false 200.
- **Three engines.** A stack built on web standards claims three engines, so it is tested in three.
  A test that passes only in Chromium is evidence of a feature below Widely Available; the fix is
  the `/web-conventions` Baseline policy, never the removal of a project.
- **`trace: 'on-first-retry'`** makes a CI failure debuggable without a reproduction attempt —
  `npx playwright show-trace` replays the run.

## Writing tests

- One `test` per user-visible outcome, named for the outcome (`'edits a bookmark label'`), not for
  the mechanics.
- **Arrange through the UI.** Reaching into `localStorage`, the store, or a backend to plant state
  is exactly the internal access the black-box contract excludes — and it silently stops testing the
  path a user takes to create that state. When arranging through the UI is too slow to bear,
  declare the seed data explicitly rather than assuming it.
- **Assert with web-first assertions** — `await expect(locator).toBeVisible()`,
  `.toHaveText()`, `.toHaveValue()`. They retry until they pass or time out, so the wait is bounded
  by the actual event, not by a guess.
- **No sleeps.** `waitForTimeout` is both slower than an auto-waiting assertion on a fast machine
  and shorter than the truth on a loaded one — it is the standard source of flakiness. If a test
  needs a wait that assertions cannot express, wait for the observable event (`waitForURL`,
  `expect(locator).toBeVisible()`), never for a duration.
- **`fill()` by default; `pressSequentially` only when keystrokes matter.** `fill()` sets the value
  and emits one `input` event, which is all a plain store-bound handler needs. Per-keystroke entry
  is required when the behavior under test depends on the intermediate states — typeahead,
  debounce, as-you-type validation, character counters — and there the delay is part of the test,
  not decoration.
- **Isolation comes from the default fixture.** Each test gets a fresh browser context, so cookies,
  `localStorage`, and IndexedDB start empty; the `/system-tests` isolation rule holds as long as
  tests do not share a `page` or a context between them. A suite that needs shared state between
  tests is a suite whose tests are steps of one test.
- **Stubbing marks the boundary.** `page.route` may stand in for a backend outside the system under
  test. It redraws the black-box boundary, so state which service is stubbed in the test name or a
  comment — an undeclared stub reads as end-to-end coverage that does not exist.
- Never commit `test.only` — `forbidOnly` fails the CI run rather than quietly testing one case.

## Spec-derived tests

When the expectations come from an `/sbce` capability spec, `/ears-tests` owns the transform: one
`test.describe` per requirement group `Rn`, one `test` per statement `Rn.m`, with the literal id
leading the test title so it appears in reports and greps against the spec. The concrete shape is
in that skill's `references/realizations.md` (Playwright section). This skill still governs how
those tests locate, assert, and run.

## Coverage — one spec source, opt-in run

Coverage answers "what did the suite never touch". It informs; it never gates. Green is the suite
passing.

`page.coverage` is Chromium-only, so coverage is a separate, single-engine run over **the same
spec files** — never a forked copy of them. A duplicated suite drifts from its original within a
few commits, and the copy that silently stopped matching is the one still reporting green.

Keep the specs importing from a local module that decides what `test` means:

```js
// tests/fixtures.js
module.exports = process.env.COVERAGE
  ? require('./coverage-fixtures')
  : require('@playwright/test');
```

```js
// tests/tests/bookmarks.spec.js
const { test, expect } = require('../fixtures');
```

`coverage-fixtures.js` extends the `page` fixture — start JS and CSS coverage with
`resetOnNavigation: false` before handing the page to the test, collect and add it to a
module-level Monocart report afterwards, and generate the report in `globalTeardown`. Consult the
`monocart-coverage-reports` documentation for the current API shape before writing it.

Overriding the fixture rather than the tests is what keeps the single spec source: the test bodies
never learn that coverage exists, and the isolation rule survives — no shared page, no serial mode.

Run it with `COVERAGE=1 npx playwright test --project=chromium`.

## Running, and green

- `npx playwright test` — the suite, all engines.
- `npx playwright test --ui` — while authoring: time-travel, locator picker, watch mode.
- `npx playwright show-report` — the last HTML report.

**Green** = every test passes in every configured project. Report per test, engine included; a skip
is a verdict that must be stated with its reason, never an omission. Anything less than all-green
is red.

Run the suite after every change to application code and fix before moving on — a suite consulted
only before a release stops being an oracle and becomes archaeology. When the stack skill also
defines a Chrome DevTools loop, both must be green.

## What NOT to do

- Do not locate by CSS selector, id, class, DOM position, or through a shadow root
- Do not read or write the store, `localStorage`, or a database to arrange or assert
- Do not use `waitForTimeout` or any sleep
- Do not repeat a URL literal in tests — `baseURL` plus `page.goto('/')`
- Do not fork the spec files for a second configuration
- Do not gate green on a coverage number
- Do not drive unit-level concerns through the browser — a pure function's edge cases are not a
  system test, they are ten seconds of browser startup buying nothing
