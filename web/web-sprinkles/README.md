# web-sprinkles

Rules for static multipage websites enhanced with small, self-contained ES modules. Every page is complete and usable with zero JavaScript; scripts only add behavior to markup that already exists. Zero runtime dependencies, no build step.

## Constraints

- Enhancement contract: links, forms, content, and navigation work with the scripts removed
- A cut line deciding what stays CSS-only (`<details>`, `:checked`, `:target`, `@view-transition`) and what justifies a script (clipboard, persisted theme, client-side filter, `Intl` formatting)
- One module per behavior under `js/`, each exporting `enhance()`, called from a single entry point that isolates failures
- No dependency, no bundler, no store, no router, no shadow DOM
- A budget: past a handful of modules the site is an application — escalate to [web-components](../web-components)

## Verification Loop

The [web-static](../web-static) loop, plus a required no-JavaScript pass: the site is copied without its `js/` directory and re-checked, so the baseline is verified rather than assumed. `checks.md` splits into Baseline and Enhanced sections; green requires both passes.

## Composition

Composes with [web-static](../web-static) — everything above the JavaScript line — and its [web-conventions](../web-conventions) baseline, plus [javascript-conventions](../javascript-conventions) for the language rules and the Baseline lookup for JavaScript features. Rules in this skill override all three. For sites needing no JavaScript at all, use [web-static](../web-static); for client-side state, routing, or templating, use [web-components](../web-components).

## Usage

`SKILL.md` contains the full ruleset and verification protocol. It can be used as a coding guide for developers or as a skill file for AI coding assistants.
