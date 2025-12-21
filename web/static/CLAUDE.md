# Static Web Pages

Static web pages using only W3C and WHATWG standards. No dependencies or build tools.

## Constraints
- no JavaScript unless explicitly requested; ask before adding any JS
- no external dependencies, frameworks, libraries, or CDN links
- no inline styles; all CSS in dedicated `.css` files in `styles/`
- no CSS frameworks or preprocessors
- no `<div>` or `<span>` when semantic elements exist
- no ARIA when native HTML provides the semantics
- no `px` units; use `rem`, `em`, or viewport units
- no physical properties (`left`, `right`, `top`, `bottom`); use logical properties (`inline-start`, `inline-end`, `block-start`, `block-end`)
- no hex or rgb colors; use `oklch()` or `oklab()`
- no media queries for component layout; use container queries

## HTML Preferences
- `<dialog>` with `showModal()` for modals; `popover` attribute for non-modal overlays
- `<details>`/`<summary>` for accordions and disclosures
- native form validation attributes over JavaScript validation
- `loading="lazy"` on images below the fold
- `<picture>` with `srcset`/`sizes` for responsive images

## CSS Preferences
- `@layer reset, base, components, utilities` for cascade management
- `@scope` for component styles instead of BEM or class prefixes
- CSS nesting for related selectors
- `:has()` for parent and previous-sibling styling
- `@container` for component-responsive layouts
- `clamp()` for fluid typography: `clamp(1rem, 2vw + 0.5rem, 1.5rem)`
- `color-mix(in oklch, ...)` for color variations
- `light-dark()` with `color-scheme` for theme colors
- `dvh`/`svh`/`lvh` over `vh` for mobile viewport heights
- `accent-color` to theme form controls
- `:user-valid`/`:user-invalid` over `:valid`/`:invalid` for form feedback

## File Structure
```
index.html
styles/
  main.css
images/
scripts/  (only if JS approved)
```
