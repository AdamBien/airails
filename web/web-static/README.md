# web-static

Rules for building modern static websites with semantic HTML and CSS — no JavaScript, no external dependencies, no build systems. Also owns the verification loop that declares a site green.

## Constraints

- No JavaScript, no frameworks, no CDNs, no inline styles, no build step
- Interactivity via CSS-only patterns (`<details>`, `:target`, `:checked`, `:focus-within`)
- CSS features per the Baseline browser-support policy

## Verification Loop

Drives the rendered pages through Chrome DevTools MCP: console messages, accessibility snapshot, responsive resize, dark-mode emulation, Lighthouse, the site-specific `checks.md` manifest, and a Baseline audit of the stylesheets. Green = every check passes on every page; composed skills (`sbce`, `continuous-testing`) delegate their "are you green?" to this loop.

## Composition

Composes with [web-conventions](../web-conventions) — the shared baseline for semantic HTML, accessibility, design tokens, and the Baseline browser-support policy. Rules in this skill override it. For applications that need client-side state, routing, or JavaScript, use [web-components](../web-components).

## Usage

`SKILL.md` contains the full ruleset and verification protocol. It can be used as a coding guide for developers or as a skill file for AI coding assistants.
