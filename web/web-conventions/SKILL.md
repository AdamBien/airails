---
name: web-conventions
description: Generic, composable web platform conventions — semantic HTML, accessibility, modern CSS, custom-property design tokens, and Baseline browser-support policy that apply across all web frontends (static sites, web component SPAs). Technology-neutral within the web platform; meant to be composed with context-specific skills (e.g. `web-static`, `web-components`). Use when writing, generating, or reviewing HTML or CSS anywhere the composed skill does not already specify a rule. Triggers on "web conventions", "semantic HTML", "accessibility review", "modern CSS", "design tokens", "Baseline status", or any request to write or review HTML/CSS where context-specific skills do not already cover it.
---

Apply all rules below strictly to any HTML and CSS you write, generate, or review.

## Scope

- Platform-level rules for HTML and CSS only — semantics, accessibility, styling, theming, browser-support policy.
- JavaScript policy, architecture, state management, routing, dependencies, project structure, and verification loops are **not** in this skill — see the context-specific skills `web-static` (no-JavaScript static sites) and `web-components` (web component SPAs).
- Responsive strategy (media queries vs container queries) is stack-specific — the composed skill decides.
- When a composed skill specifies a rule, the composed skill wins; this skill is the fallback baseline.

## Guiding Principles

- web standards and web platform first
- minimal external dependencies — every dependency must justify its existence
- progressive enhancement over JavaScript-first design

## HTML Rules

- semantic elements over generic `<div>`/`<span>` — use `<header>`, `<nav>`, `<main>`, `<article>`, `<section>`, `<aside>`, `<footer>`, `<figure>`, `<time>`, `<address>`
- one `<main>` per page
- proper heading hierarchy (h1 → h2 → h3, no skipping)
- `lang` attribute on `<html>`
- `alt` on all images (empty `alt=""` for decorative)
- `aria-label` on `<nav>` when multiple navs exist
- `<label>` associated with every form input
- skip link for keyboard users
- use `<a>` for navigation, not buttons
- use `<br>` only for line breaks in content, never for spacing

## Accessibility Rules

- WCAG AA color contrast minimum (4.5:1)
- always maintain visible `:focus-visible` indicators
- every interactive element reachable and operable by keyboard
- include `prefers-reduced-motion: reduce` and `prefers-color-scheme: dark` media queries

## CSS Rules

- all CSS in separate `.css` files — no inline styles
- use logical properties (`margin-block`, `padding-inline`, `inline-size`, `block-size`)
- use modern features: CSS nesting, container queries, cascade layers, subgrid, `oklch()` colors — subject to the Baseline policy below
- use `clamp()` for fluid typography
- prefer `gap` over margins for spacing in flex/grid layouts

## Design Tokens

- define design tokens as CSS custom properties on `:root` — colors, spacing, typography, radii
- component and page rules reference tokens, never raw values
- CSS custom properties are the token source of truth — do not adopt the DTCG token JSON format or token-translation tooling (Style Dictionary, Terrazzo, etc.)

## Baseline Policy

- **Widely Available** features may be used freely
- **Newly Available** features require `@supports` feature detection with a graceful fallback for older browsers
- **Limited availability** features must not be used
- when recommending a newer CSS feature, cite its Baseline status (Widely Available, Newly Available, or Limited) so the reader can judge browser support

## What NOT to Do

- do not use tables for layout
- do not use `<br>` for spacing — use CSS spacing
- do not use non-semantic `<div>`/`<span>` when a semantic element exists
- do not use inline styles
