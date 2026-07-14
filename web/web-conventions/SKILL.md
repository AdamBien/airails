---
name: web-conventions
description: Generic, composable web platform conventions — semantic HTML, accessibility, modern CSS, custom-property design tokens, and Baseline browser-support policy that apply across all web frontends (static sites, web component SPAs). Technology-neutral within the web platform; meant to be composed with context-specific skills (e.g. `web-static`, `web-components`). Use when writing, generating, or reviewing HTML or CSS anywhere the composed skill does not already specify a rule. Triggers on "web conventions", "semantic HTML", "web forms", "form validation", "input types", "accessibility review", "modern CSS", "design tokens", "DESIGN.md", "Baseline status", "serve the site", "dev server", "zws", or any request to write or review HTML/CSS where context-specific skills do not already cover it. Also use when a project contains a DESIGN.md that should guide HTML/CSS generation, when a Baseline status must be looked up — this skill bundles a dated snapshot of the complete webstatus.dev feature set in `references/baseline-snapshot.md` — or when a static site root needs to be served locally — this skill bundles zws, the zero-dependency Java dev server, in `scripts/zws`.
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

## Form Rules

- use the semantically correct input type — `email`, `url`, `tel`, `password`, `search`, `number`, `date`, `time`, `range`, `file`, `checkbox`, `radio` — never `type="text"` plus JavaScript re-implementation; the type alone provides the right mobile keyboard, native widget, and built-in validation (all Widely Available)
- exception: `<input type="color">` is Baseline Limited — do not use it; fall back to a text input with a `pattern` for hex colors
- declare constraints in markup: `required`, `pattern`, `min`/`max`/`step`, `minlength`/`maxlength` — built-in validation is the first line of defense, JavaScript only for rules markup cannot express (Constraint Validation API is Widely Available)
- `autocomplete` on every field with a well-known meaning (`name`, `email`, `street-address`, `current-password`, …) — autofill is both UX and accessibility
- `inputmode` only when no dedicated input type fits (Widely Available)
- `placeholder` is a hint, never a substitute for a `<label>`
- group related controls with `<fieldset>`/`<legend>`
- give every `<button>` inside a form an explicit `type` — the default is `submit`, which makes stray buttons submit the form
- style validation states with `:user-valid`/`:user-invalid` (Widely Available since 2026-05), not `:valid`/`:invalid` — the `:user-*` pseudo-classes wait for user interaction instead of flagging pristine fields

## Accessibility Rules

- WCAG AA color contrast minimum (4.5:1)
- always maintain visible `:focus-visible` indicators
- every interactive element reachable and operable by keyboard
- include `prefers-reduced-motion: reduce` and `prefers-color-scheme: dark` media queries

## CSS Rules

- all CSS in separate `.css` files — no inline styles
- use logical properties (`margin-block`, `padding-inline`, `inline-size`, `block-size`)
- use modern features: CSS nesting, container queries, cascade layers, subgrid, `oklch()` colors, the `:has()` selector — subject to the Baseline policy below
- use `clamp()` for fluid typography
- prefer `gap` over margins for spacing in flex/grid layouts

## Scrolling

- use CSS scroll snap (`scroll-snap-type`, `scroll-snap-align`) for carousels, galleries, and section-based scrolling — never JavaScript scroll hijacking (scroll snap is Baseline Widely Available)
- pair scroll snap with `scroll-padding` on the container so snapped content clears sticky headers
- apply `scroll-behavior: smooth` only inside `@media (prefers-reduced-motion: no-preference)`

## Design Tokens

- define design tokens as CSS custom properties on `:root` — colors, spacing, typography, radii
- component and page rules reference tokens, never raw values
- CSS custom properties are the token source of truth — do not adopt the DTCG token JSON format or token-translation tooling (Style Dictionary, Terrazzo, etc.)
- when a `DESIGN.md` exists at the project root, read it before writing or reviewing any CSS — it declares the design intent: palette, typography, spacing scale, and component rules, with the reasoning behind them
- `DESIGN.md` is context, not a build input: never generate CSS from it mechanically, and never edit it to match the CSS — it is human/tool-owned
- when `:root` tokens contradict `DESIGN.md`, surface the drift and let the user decide which side changes — never silently pick one

## Baseline Policy

- **Widely Available** features may be used freely
- **Newly Available** features require `@supports` feature detection with a graceful fallback for older browsers
- **Limited availability** features must not be used
- when recommending a newer CSS feature, cite its Baseline status (Widely Available, Newly Available, or Limited) so the reader can judge browser support
- determine a feature's status by looking it up by feature id in `references/baseline-snapshot.md` — a bundled, dated snapshot of the complete [webstatus.dev](https://api.webstatus.dev/v1/features) feature set — never from model memory; statuses change over time (Newly crosses into Widely 30 months after the low date)
- a feature missing from the snapshot, or newer than the snapshot's header date, is uncertain: verify against webstatus.dev before recommending it, or treat it as Limited
- the snapshot is generated by [zbaseline](https://github.com/AdamBien/zbaseline) — never edit it in place; re-run zbaseline to refresh, and replace the whole file
- for experiments and PoCs, the `/web-latest` modifier overrides this policy — any feature status becomes usable, with a declared support floor instead of fallbacks

## Bundled Tooling

- `scripts/zws` — a copy of [zws](https://github.com/AdamBien/zws), the zero-dependency single-file Java development server. Serves any static site root on http://localhost:3000 (loopback only, caching disabled, opens the browser). Requires JDK 25+.
- invocation: `java <this skill's directory>/scripts/zws <site-root>` — prefer a `zws` already on the PATH (the maintained install); the bundled copy is the fallback so composed skills (`web-static` verification loop, `web-components` dev serving) work after a zip-only install.
- flags: `--live` reloads connected browsers on file changes (injects a one-line SSE script into served HTML — authoring only, never under a verification run, where the injected script would contaminate the evidence); `--single` serves `index.html` for extension-less unknown paths (SPA fallback for `web-components` routing — wrong for multi-page static sites, where it masks broken links). The flags compose.
- the copy mirrors [AdamBien/zws](https://github.com/AdamBien/zws) — never edit it in place; when upstream changes, re-copy.
- never serve with `python3 -m http.server` or Node-based dev servers.

## What NOT to Do

- do not use tables for layout
- do not use `<br>` for spacing — use CSS spacing
- do not use non-semantic `<div>`/`<span>` when a semantic element exists
- do not use inline styles
- do not re-implement in JavaScript what standard input types and constraint attributes already validate
- do not add `novalidate` to a form unless the Constraint Validation API takes over the checks it disables
