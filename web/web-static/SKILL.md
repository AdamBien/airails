---
name: web-static
description: Build modern static websites using semantic HTML and CSS without external dependencies or build systems. Also owns the verification loop for such sites — drives the rendered pages through Chrome DevTools MCP (console, accessibility snapshot, viewport resize, dark/reduced-motion emulation, Lighthouse), executes the checks.md manifest, and audits CSS against Google Baseline. Use when creating or editing static HTML/CSS sites, AND whenever a static site must be verified, audited, or declared green — triggers on "verify the site", "run the checks", "is it green", "lighthouse audit", "baseline check", "checks.md".
argument-hint: "[description of the website or page to build]"
---

Build or maintain a static website using $ARGUMENTS. Apply all rules below strictly.

## Hard Constraints

- **No JavaScript** — no `<script>` tags, no inline event handlers, no `.js` files
- **No external dependencies** — no frameworks, libraries, CDNs, or package managers
- **No build systems** — no transpilation, bundling, preprocessing, Sass, Less, or Tailwind
- **No inline styles** — all CSS in separate `.css` files
- **Baseline 2025** — only use CSS features widely available across modern browsers

## HTML Rules

- Semantic elements over generic `<div>`/`<span>` — use `<header>`, `<nav>`, `<main>`, `<article>`, `<section>`, `<aside>`, `<footer>`, `<figure>`, `<time>`, `<address>`
- One `<main>` per page
- Proper heading hierarchy (h1 → h2 → h3, no skipping)
- `lang` attribute on `<html>`
- `alt` on all images (empty `alt=""` for decorative)
- `aria-label` on `<nav>` when multiple navs exist
- `<label>` associated with every form input
- Skip link for keyboard users
- Use `<a>` for navigation, not buttons
- Use `<br>` only for line breaks in content, never for spacing

## CSS Rules

- Use CSS custom properties for theming (`--color-primary`, `--spacing-unit`, etc.)
- Use logical properties (`margin-block`, `padding-inline`, `inline-size`, `block-size`)
- Mobile-first responsive design with `min-width` breakpoints
- Use modern features: CSS nesting, container queries, cascade layers, subgrid, `oklch()` colors
- Use `clamp()` for fluid typography
- Prefer `gap` over margins for spacing in flex/grid layouts
- Include `prefers-reduced-motion: reduce` and `prefers-color-scheme: dark` media queries
- Use `content-visibility: auto` for off-screen sections
- WCAG AA color contrast minimum (4.5:1)
- Always maintain visible `:focus-visible` indicators
- Use `@supports` for feature detection when applying Baseline Newly Available features, providing a graceful fallback for older browsers
- When recommending a newer CSS feature, cite its Baseline status (Widely Available, Newly Available, or Limited) so the reader can judge browser support

## CSS Reset Baseline

Every project starts with:

```css
* { box-sizing: border-box; margin: 0; padding: 0; }
body { font-family: system-ui, -apple-system, sans-serif; line-height: 1.5; }
```

## Interactive Patterns (CSS-Only)

When interactivity is needed, use these approaches — never JavaScript:

- **Accordion**: `<details>`/`<summary>`
- **Modal**: `:target` pseudo-class with anchor links
- **Tabs**: hidden radio buttons with `:checked` + sibling selectors
- **Dropdown menu**: `:hover` + `:focus-within`
- **Mobile hamburger**: hidden checkbox with `:checked` + sibling selectors
- **Form styling**: `appearance: none` to customize native controls

## Performance

- `loading="lazy"` on below-fold images
- `<picture>` with WebP sources for responsive images
- Preload critical CSS and fonts
- `contain: layout style paint` on repeated components

## Project Structure

```
project/
├── index.html
├── css/
│   └── style.css
├── images/
├── fonts/
└── icons/
```

For small sites, a flat structure (HTML + single `style.css` + `images/`) is acceptable.

## Verification Loop (Chrome DevTools MCP)

The site ships with zero tooling, so verification happens from the outside: drive a real browser
through the Chrome DevTools MCP tools (`chrome-devtools`) and assert against what it renders.
This loop defines **green** for this stack — when a composed skill (`/sbce`, `/continuous-testing`)
asks "are you green?", it means exactly this. Never declare a page done from reading its source;
the rendered page is the oracle. If the `chrome-devtools` MCP tools are unavailable, report the
loop as not runnable — do not silently skip it or self-certify.

### Serve

Serve the site root with [zws](https://github.com/AdamBien/zws), the zero-dependency single-file
Java development server: `java zws <site-root>` (serves http://localhost:3000, caching disabled).
Open pages via `new_page`/`navigate_page`. Verification tooling never ships with the site, so this
violates no constraint — the no-JS/no-dependency rules govern the artifact, not the harness.
`file://` URLs work for a single page but break root-relative links; prefer the server.
`evaluate_script` is permitted for read-only inspection (e.g. `getComputedStyle`) — it runs in
the test browser, not in the site.

### Standard checks — every page, every run

1. **Console** — `list_console_messages`: no errors, no failed requests (also proves the no-JavaScript constraint holds).
2. **Structure** — `take_snapshot` (accessibility tree): exactly one `main`, no skipped heading levels, a label on every form input, `aria-label` on each nav when several exist, skip link first in tab order, alt text on every image.
3. **Responsive** — `resize_page` to 375×667 and 1280×800: the layout adapts, and mobile patterns (hamburger, stacked columns) appear only at the narrow width.
4. **Preferences** — `emulate` with `colorScheme: dark`: the page restyles. The MCP cannot emulate reduced motion, so check it statically: read-only `evaluate_script` over `document.styleSheets` (or a source grep) confirms the `prefers-reduced-motion: reduce` query exists and neutralizes animations/transitions.
5. **Lighthouse** — `lighthouse_audit` (covers accessibility, best-practices, SEO — not performance): accessibility = 100 (includes WCAG AA contrast), best-practices and SEO ≥ 90. When performance is in question, run `performance_start_trace` and judge Core Web Vitals — on request, not part of standard green.

### Project checks — checks.md

Site-specific behavior lives in `checks.md` at the site root (one per BC directory when composed
with `/bce`): one labeled line per check, mechanically executable — exact URL, viewport, and
expected observation, resolved with the same MCP tools.

```markdown
# Checks
- [nav-mobile] / at 375px: snapshot contains button "Menu"; nav links hidden until checked
- [R1.2] /catalog/ at 1280px: every workshop is an <article> containing a <time>
```

Labels are stable identifiers — never renumber, retire instead of reuse. When composed with
`/sbce`, the label **is** the requirement id (`R1.2`), which makes the spec↔check trace
grep-visible — all `/sbce` needs. Write each check as an observation the browser can contradict
("snapshot contains button 'Menu'"), never as a judgment ("menu looks right") — the next run must
be able to disagree.

### Baseline check

Classify each CSS feature the stylesheets use by its Baseline status from knowledge — no external
validation services. A **Widely Available** feature passes; a **Newly Available** feature passes
only when wrapped in `@supports` with a graceful fallback (per the CSS rules above); **Limited
availability** fails the loop. One caveat keeps this honest: the DevTools browser is Chrome, so a
page rendering correctly proves Chrome support, never Baseline — when a feature's status is
uncertain, report the uncertainty instead of guessing.

### Green

Green = standard checks pass on every page, every `checks.md` line passes, and the Baseline check
passes. Report per check — label → pass/fail with the observed evidence — then fix and re-run.
Anything less than all-green is red; there is no "mostly green".

## What NOT to Do

- Do not use tables for layout
- Do not use `<br>` for spacing
- Do not use non-semantic `<div>` when a semantic element exists
- Do not use `onclick` or any inline event handlers
- Do not add JavaScript for interactions achievable with CSS
- Do not use CSS frameworks or preprocessors
