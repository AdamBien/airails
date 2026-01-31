---
name: web-static
description: Build modern static websites using semantic HTML and CSS without external dependencies or build systems
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

## What NOT to Do

- Do not use tables for layout
- Do not use `<br>` for spacing
- Do not use non-semantic `<div>` when a semantic element exists
- Do not use `onclick` or any inline event handlers
- Do not add JavaScript for interactions achievable with CSS
- Do not use CSS frameworks or preprocessors
