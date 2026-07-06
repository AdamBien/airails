# web-conventions

Generic, composable web platform conventions: semantic HTML, accessibility, modern CSS, custom-property design tokens, and the Baseline browser-support policy.

## Composition

Shared baseline for the stack-specific web skills — both compose this skill:

- [web-static](../web-static) — no-JavaScript static sites; adds hard constraints, CSS-only interactivity, and the Chrome DevTools verification loop
- [web-components](../web-components) — web component SPAs; adds BCE architecture, lit-html, Redux Toolkit, Vaadin Router, and Playwright testing

When a composed skill specifies a rule, the composed skill wins; this skill is the fallback baseline. Stack-specific concerns (JavaScript policy, dependencies, responsive strategy, verification) are deliberately out of scope.

## Baseline Policy

Widely Available features may be used freely; Newly Available features require `@supports` with a graceful fallback; Limited availability features are prohibited. Design tokens are CSS custom properties on `:root` — no DTCG token files or translation tooling.

## Usage

`SKILL.md` contains the full ruleset. Use it standalone for HTML/CSS reviews, or composed with `web-static` / `web-components`. It can serve as a coding guide for developers or as a skill file for AI coding assistants.
