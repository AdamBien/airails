# web-system-tests

An [AIrails.dev](https://airails.dev) skill with the **web realization of the [system-tests](../../bce/system-tests) contract** — browser-driven system tests with [Playwright](https://playwright.dev) against a running frontend.

## Scope

- Project layout: a `tests/` directory outside the served root, one spec per business component
- Configuration: `baseURL` plus `webServer` with `reuseExistingServer`, so the same suite runs against the dev server you are already using and against a CI-started one, unchanged
- Selector policy: role, label, and text locators — the accessibility tree is the public surface of a web UI, which makes the policy a consequence of the black-box contract rather than a preference
- Auto-waiting assertions instead of sleeps; arrangement through the UI instead of the store
- Coverage as an opt-in single-engine run over the same spec files, via a fixture override — never a forked suite
- Green: every test in every configured engine, plus the after-every-change loop

## Composition

- [`system-tests`](../../bce/system-tests) — the stack-neutral contract this skill realizes
- [`ears-tests`](../../bce/ears-tests) — spec-derived expectations: `test.describe` per requirement group, the `Rn.m` id in the test title
- the stack skill owning the application: [`web-components`](../web-components), [`web-sprinkles`](../web-sprinkles), or [`web-static`](../web-static)
- [`bce`](../../bce/bce) — one spec file per business component

Distinct from the Chrome DevTools verification loop in [`web-static`](../web-static): that loop inspects the page as it is right now — rendering, accessibility tree, Lighthouse, Baseline — and leaves no artifact. This skill produces a committed suite that re-runs on every change, in three engines. Where a project has both, green means both.

## Usage

Activated when a web frontend needs end-to-end, browser, or system tests written, run, reviewed, or repaired. [SKILL.md](SKILL.md) contains the full ruleset, the Playwright configuration, and the green oracle. It can be used as a testing guide for developers or as a skill file for AI coding assistants.
