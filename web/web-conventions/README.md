# web-conventions

Generic, composable web platform conventions: semantic HTML, accessibility, modern CSS, custom-property design tokens, and Baseline browser-support policy.

## Role

This skill is the shared baseline for the stack-specific web skills:

- **web-static** — no-JavaScript static sites (adds hard constraints, CSS-only interactivity, and the Chrome DevTools verification loop)
- **web-components** — web component SPAs (adds BCE architecture, lit-html, Redux Toolkit, Vaadin Router, and Playwright testing)

Both compose this skill. When a composed skill specifies a rule, the composed skill wins; this skill is the fallback baseline.

## Contents

- **HTML rules** — semantic elements, heading hierarchy, labels, skip links
- **Accessibility rules** — WCAG AA contrast, focus indicators, keyboard operability, user-preference media queries
- **CSS rules** — logical properties, modern features, fluid typography
- **Design tokens** — CSS custom properties on `:root` as the single token source of truth (no DTCG tooling)
- **Baseline policy** — Widely Available freely, Newly Available behind `@supports`, Limited never

## Usage

`SKILL.md` can be used as a coding guide for developers or as a skill file for AI coding assistants — standalone for HTML/CSS reviews, or composed with `web-static` / `web-components`.
