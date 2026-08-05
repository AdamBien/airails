---
name: web-sprinkles
description: Build static multipage websites enhanced with small, self-contained ES modules — progressive enhancement, where every page is complete and usable with zero JavaScript and scripts only add behavior to markup that already exists. Composes with `web-static` (CSS-only patterns, structure, verification loop), `web-conventions` (HTML/CSS/Baseline), and `javascript-conventions` (language rules); owns the enhancement contract, the CSS-versus-script cut line, and a no-JavaScript verification pass. Use when a static site needs a theme toggle, a copy button, a client-side filter, `Intl` formatting, or a small `fetch`. Triggers on "sprinkles of JavaScript", "a bit of JavaScript", "small amount of JS", "progressive enhancement", "static site with JavaScript", "vanilla JS page", "enhance this static page", "no framework, just a script". Not for pure HTML/CSS sites — use `web-static`; not for applications with client-side state, routing, or templating — use `web-components`.
argument-hint: "[description of the page and the behavior it needs]"
---

Build or maintain a static site with JavaScript enhancements using $ARGUMENTS. Apply all rules below strictly.

## Composition

This skill is `/web-static` plus a bounded amount of JavaScript. Everything in `/web-static` applies
unchanged — CSS-only interactive patterns, CSS reset, page transitions, performance rules, project
structure, and the verification loop — except its no-JavaScript hard constraint, which this skill
replaces with the Enhancement Contract below.

- `/web-conventions` — semantic HTML, accessibility, modern CSS, design tokens, Baseline policy
- `/javascript-conventions` — modules, syntax, collections, asynchrony, errors, JSDoc types, and the
  Baseline lookup for JavaScript features
- `/web-static` — everything above the JavaScript line
- `/web-latest` may be composed on top for experiments and PoCs

Rules in this skill override all of them.

## Enhancement Contract

- every page is **complete and usable with zero JavaScript** — content readable, navigation
  traversable, forms submittable and validated by the browser
- JavaScript only **adds behavior to markup that already exists**; it never produces the content
  that is the page's purpose
- links keep real `href`s and forms keep `action`/`method` — navigation and submission never depend
  on a script
- a module that fails — throws, 404s, hits an unsupported API — degrades the page to its no-script
  baseline, never to a broken page
- enhancements are independent: one module failing must not disable the others
- nothing renders late — no content flashes in, moves, or is replaced after load

## The Cut Line

If HTML or CSS can express it, a script is a defect, not a choice. Before writing one, ask what the
page looks like with it removed; if the answer is "broken", the design is wrong, not the script.

| Behavior | Verdict |
|---|---|
| Accordion, disclosure | `<details>`/`<summary>` — no script |
| Tabs, hamburger, dropdown | `:checked` + siblings, `:focus-within` — no script |
| Modal | `:target` — no script; `<dialog>` plus a script only when focus trapping and `Esc` matter |
| Form validation | input types and constraint attributes — a script only for rules markup cannot express |
| Page transitions, smooth scroll, carousels | `@view-transition`, `scroll-behavior`, scroll snap — no script |
| Theme toggle that persists | script — CSS cannot read `localStorage` |
| Clipboard | script — `navigator.clipboard` |
| Filter, sort, or search over content already in the DOM | script |
| Locale or relative time formatting | script — `Intl` over a `<time datetime>` already in the HTML |
| Loading supplementary data | script — but content the page exists to show stays in the HTML |

## Modules

- one module per behavior, named after the behavior: `theme-toggle.js`, `copy-button.js`
- each module exports a single `enhance()` — no work at import time (per `/javascript-conventions`)
- one entry point, `js/enhance.js`, imports each module and calls it inside its own `try`/`catch`,
  so a failing enhancement cannot take down the rest:

  ```javascript
  import { enhance as themeToggle } from "./theme-toggle.js";
  import { enhance as copyButton } from "./copy-button.js";

  for (const enhancement of [themeToggle, copyButton]) {
      try {
          enhancement();
      } catch (cause) {
          console.error(new Error(`enhancement failed: ${enhancement.name}`, { cause }));
      }
  }
  ```

- load it once per page: `<script type="module" src="/js/enhance.js"></script>` — modules defer by
  default, so it belongs in `<head>`
- a module finds its targets through a dedicated `data-` attribute (`[data-copy]`), never through a
  class used for styling — the hook is explicit, greppable, and its absence means no enhancement
- delegate events on a container rather than binding per element
- feature-detect every Newly Available API with a fallback, per the `/javascript-conventions`
  Baseline lookup — a missing API degrades to the baseline, it does not throw
- **zero runtime dependencies** — no lit-html, no CDN, no npm, no bundler, no TypeScript. A sprinkle
  that needs a templating library is not a sprinkle
- no store, no client-side router, no shadow DOM
- a custom element is permitted only as a wrapper that enhances its existing light-DOM children;
  one that renders its own content belongs to `/web-components`

## Budget

Keep the JavaScript smaller than the CSS. A module past ~100 lines, or more than a handful of
modules on a page, means the site has become an application — escalate rather than grow.

## Project Structure

`/web-static`'s structure, with all enhancement modules under a single `js/` directory:

```
project/
├── index.html
├── css/style.css
├── js/
│   ├── enhance.js        # entry point: imports and calls each enhance()
│   ├── theme-toggle.js
│   └── copy-button.js
├── checks.md
└── images/
```

One directory is a rule, not a convention — the no-JavaScript pass below removes it in a single step.

## Verification Loop

`/web-static`'s loop runs unchanged — serve with `java zws <site-root>`, then console, structure,
responsive, preferences, and Lighthouse checks on every page, plus `checks.md` and the Baseline
check. Never author with `zws --live` during a run; the injected reload script contaminates both
passes. Three amendments:

**Console.** An empty console no longer proves the JavaScript policy — it proves the modules run
clean. Errors and failed requests are red as before.

**Baseline check.** It now covers the JavaScript features the modules use as well as the CSS,
resolved through the same `web-conventions` snapshot.

**No-JavaScript pass — required, every page.** The Chrome DevTools MCP cannot disable JavaScript, so
remove it at the source: copy the site root to a scratch directory, delete its `js/` directory,
serve the copy with a second `zws` instance, and re-run the **structure** check plus every
`checks.md` line marked Baseline. The only console output permitted in this pass is the failed
request for the missing entry point — that failure is the evidence the scripts were absent; anything
else is red. If the copy cannot be served, report the pass as not runnable rather than self-certify.

### checks.md

Two sections, so every line declares which pass it belongs to. Labels stay stable identifiers and
remain the requirement id under `/sbce`.

```markdown
# Checks

## Baseline — holds with js/ removed
- [R1.2] /catalog/ at 1280px: every workshop is an <article> containing a <time>
- [nav-mobile] / at 375px: snapshot contains button "Menu"; nav links hidden until checked

## Enhanced — holds with the modules loaded
- [copy-btn] /snippets/ at 1280px: clicking button "Copy" changes its label to "Copied"
- [theme] / at 1280px: clicking button "Dark" sets data-theme="dark" on <html>; reload keeps it
```

Every enhancement needs a Baseline line stating what stands in for it without JavaScript.

**Green** = `/web-static` green, plus the no-JavaScript pass green, plus every Enhanced check.
Report per label with the observed evidence. The enhanced pass alone never establishes green.

## Escalate to /web-components

Stop and switch stacks at the first of these — converting is cheap early and expensive late:

- state shared between two enhancements, or state that outlives a page view
- views selected by URL, or any client-side routing
- content that exists only after a `fetch`
- a need for templating, or for components that render themselves
- the budget above exceeded

## What NOT to Do

In addition to the `/web-static` and `/web-conventions` prohibitions:

- do not use inline `<script>` blocks or `onclick`-style handlers
- do not render content that the page exists to show
- do not script an interaction the cut line assigns to CSS
- do not intercept a link or form submission that works without JavaScript
- do not add a dependency, a build step, or a polyfill
- do not use a Newly Available API without a detection and a fallback
- do not declare a page green without the no-JavaScript pass
