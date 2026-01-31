---
name: static-web-development
description: Build modern static websites using semantic HTML and CSS without external dependencies or build systems
keywords: html, css, semantic markup, web standards, static website, baseline 2025, accessibility, responsive design, no javascript
version: 0.0.1
author: adam-bien.com
---

# Static Web Development Power

Build modern, performant static websites using web standards, semantic markup, and CSS. This power guides development without external dependencies, build systems, or JavaScript, focusing on pure HTML/CSS capabilities and Google Baseline 2025 features.

## Core Principles

**Web Standards First**: Rely exclusively on standardized HTML and CSS features available in modern browsers.

**Semantic Markup**: Use HTML elements according to their semantic meaning to improve accessibility, SEO, and maintainability.

**No Build Systems**: Write code that runs directly in browsers without transpilation, bundling, or preprocessing.

**No External Dependencies**: Pure HTML and CSS only - no frameworks, libraries, or JavaScript.

**Progressive Enhancement**: Start with functional, accessible HTML, enhance presentation with CSS.

**Baseline 2025 Compatibility**: Use CSS features widely available across modern browsers as defined by Google's Baseline initiative.

## Project Structure

```
project/
├── index.html          # Entry point
├── about.html          # Additional pages
├── contact.html
├── css/                # or styles/
│   ├── style.css
│   ├── components.css
├── images/             # or img/
│   ├── logo.svg
│   └── hero.jpg
├── fonts/
│   └── custom-font.woff2
└── icons/
    └── favicon.ico
```

Alternative flat structure for smaller sites:
```
project/
├── index.html
├── about.html
├── style.css           # Single stylesheet
└── images/
    └── photo.jpg
```

Note: Directory naming conventions vary. Common alternatives include `styles/` vs `css/`, `images/` vs `img/`, or grouping all static files under `assets/` or `static/`. Choose what works best for your project and stay consistent.

## HTML Best Practices

### Semantic Structure

Use semantic HTML5 elements to convey document structure and meaning:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta name="description" content="Page description for SEO">
  <title>Page Title</title>
  <link rel="stylesheet" href="styles/style.css">
</head>
<body>
  <header>
    <nav aria-label="Main navigation">
      <ul>
        <li><a href="/">Home</a></li>
        <li><a href="/about.html">About</a></li>
      </ul>
    </nav>
  </header>

  <main>
    <article>
      <h1>Article Title</h1>
      <p>Content goes here</p>
    </article>
  </main>

  <aside>
    <section>
      <h2>Related Content</h2>
    </section>
  </aside>

  <footer>
    <p>&copy; 2025 Company Name</p>
  </footer>
</body>
</html>
```

### Semantic Element Guidelines

- `<header>`: Site or section header with branding, navigation
- `<nav>`: Navigation links (use `aria-label` for multiple navs)
- `<main>`: Primary page content (one per page)
- `<article>`: Self-contained content (blog posts, news articles)
- `<section>`: Thematic grouping of content with heading
- `<aside>`: Tangentially related content (sidebars, callouts)
- `<footer>`: Site or section footer with metadata
- `<figure>` + `<figcaption>`: Images, diagrams with captions
- `<time>`: Dates and times with `datetime` attribute
- `<address>`: Contact information

### Forms and Input

```html
<form action="/submit" method="post">
  <fieldset>
    <legend>Contact Information</legend>
    
    <label for="name">Name:</label>
    <input type="text" id="name" name="name" required>
    
    <label for="email">Email:</label>
    <input type="email" id="email" name="email" required>
    
    <label for="message">Message:</label>
    <textarea id="message" name="message" rows="5" required></textarea>
  </fieldset>
  
  <button type="submit">Send</button>
</form>
```

### Form Control Styling

Native form controls can be customized with CSS. Use `appearance: none` to remove default browser styling:

```css
/* Custom text input */
input[type="text"],
input[type="email"],
textarea {
  appearance: none;
  border: 1px solid #ccc;
  border-radius: 4px;
  padding: 0.75rem;
  font: inherit;
  inline-size: 100%;
}

input:focus,
textarea:focus {
  outline: 2px solid var(--color-primary);
  outline-offset: 2px;
  border-color: var(--color-primary);
}

/* Custom checkbox */
input[type="checkbox"] {
  appearance: none;
  inline-size: 1.25rem;
  block-size: 1.25rem;
  border: 2px solid #ccc;
  border-radius: 4px;
  cursor: pointer;
}

input[type="checkbox"]:checked {
  background-color: var(--color-primary);
  border-color: var(--color-primary);
  background-image: url("data:image/svg+xml,..."); /* checkmark icon */
}

/* Custom radio button */
input[type="radio"] {
  appearance: none;
  inline-size: 1.25rem;
  block-size: 1.25rem;
  border: 2px solid #ccc;
  border-radius: 50%;
  cursor: pointer;
}

input[type="radio"]:checked {
  border-color: var(--color-primary);
  background: radial-gradient(var(--color-primary) 40%, transparent 45%);
}

/* Custom select box */
select {
  appearance: none;
  border: 1px solid #ccc;
  border-radius: 4px;
  padding: 0.75rem 2.5rem 0.75rem 0.75rem;
  font: inherit;
  background-color: white;
  background-image: url("data:image/svg+xml,..."); /* dropdown arrow */
  background-repeat: no-repeat;
  background-position: right 0.75rem center;
  cursor: pointer;
}

/* Custom button */
button {
  appearance: none;
  border: none;
  padding: 0.75rem 1.5rem;
  font: inherit;
  background-color: var(--color-primary);
  color: white;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.2s;
}

button:hover {
  background-color: var(--color-primary-dark);
}

button:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: 2px;
}
```

Note: Always maintain visible focus indicators for accessibility. The `accent-color` property provides simpler theming for checkboxes, radios, and range inputs without removing native styling:

```css
:root {
  accent-color: var(--color-primary);
}
```

### Accessibility Requirements

- Always include `lang` attribute on `<html>`
- Provide `alt` text for images (empty `alt=""` for decorative images)
- Use ARIA labels when semantic HTML is insufficient
- Maintain proper heading hierarchy (h1 → h2 → h3)
- Use `<a>` for navigation with descriptive link text
- Include skip links for keyboard users
- Ensure sufficient color contrast (WCAG AA minimum 4.5:1)
- Use `<label>` elements properly associated with form inputs

## CSS Best Practices

### File Organization

Maintain styles in dedicated CSS files, never inline:

```css
/* style.css - Global styles and CSS custom properties */
:root {
  --color-primary: #0066cc;
  --color-secondary: #6c757d;
  --color-text: #212529;
  --color-background: #ffffff;
  --spacing-unit: 8px;
  --font-family-base: system-ui, -apple-system, sans-serif;
  --font-size-base: 1rem;
  --line-height-base: 1.5;
}

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: var(--font-family-base);
  font-size: var(--font-size-base);
  line-height: var(--line-height-base);
  color: var(--color-text);
  background-color: var(--color-background);
}
```

### Modern CSS Features (Baseline 2025)

**Container Queries**:
```css
.card-container {
  container-type: inline-size;
  container-name: card;
}

@container card (min-width: 400px) {
  .card {
    display: grid;
    grid-template-columns: 1fr 2fr;
  }
}
```

**CSS Nesting**:
```css
.navigation {
  display: flex;
  gap: var(--spacing-unit);
  
  & ul {
    list-style: none;
    display: flex;
    gap: calc(var(--spacing-unit) * 2);
  }
  
  & a {
    text-decoration: none;
    color: var(--color-text);
    
    &:hover {
      color: var(--color-primary);
    }
  }
}
```

**Cascade Layers**:
```css
@layer reset, base, components, utilities;

@layer reset {
  * { margin: 0; padding: 0; }
}

@layer base {
  body { font-family: var(--font-family-base); }
}

@layer components {
  .button { padding: 0.5rem 1rem; }
}
```

**Logical Properties**:
```css
.content {
  margin-block: 2rem;
  margin-inline: auto;
  padding-inline: 1rem;
  max-inline-size: 70ch;
}
```

**Modern Layout**:
```css
/* Grid */
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: var(--spacing-unit);
}

/* Flexbox */
.flex {
  display: flex;
  flex-wrap: wrap;
  gap: var(--spacing-unit);
}

/* Subgrid */
.card-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1rem;
}

.card {
  display: grid;
  grid-template-rows: subgrid;
  grid-row: span 3;
}
```

**Color Functions**:
```css
:root {
  --color-primary: oklch(60% 0.15 250);
  --color-primary-light: oklch(from var(--color-primary) calc(l + 0.2) c h);
  --color-primary-dark: oklch(from var(--color-primary) calc(l - 0.2) c h);
}
```

### Transitions and Animations

CSS transitions provide smooth visual feedback for state changes without JavaScript.

**Basic Transitions**:
```css
.button {
  background-color: var(--color-primary);
  transition: background-color 0.2s ease-in-out;
}

.button:hover {
  background-color: var(--color-primary-dark);
}

/* Multiple properties */
.card {
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.card:hover {
  transform: translateY(-4px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

/* Shorthand for all properties */
.link {
  transition: all 0.15s ease-in-out;
}
```

**CSS Animations**:
```css
@keyframes fade-in {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.fade-in {
  animation: fade-in 0.3s ease-out;
}

/* Continuous animation */
@keyframes pulse {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.05); }
}

.pulse {
  animation: pulse 2s ease-in-out infinite;
}
```

**Respect User Preferences**:
```css
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

### Responsive Design

Use mobile-first approach with logical breakpoints:

```css
/* Mobile first (default) */
.container {
  padding-inline: 1rem;
}

/* Tablet */
@media (min-width: 768px) {
  .container {
    padding-inline: 2rem;
  }
}

/* Desktop */
@media (min-width: 1024px) {
  .container {
    max-inline-size: 1200px;
    margin-inline: auto;
  }
}

/* Prefer reduced motion */
@media (prefers-reduced-motion: reduce) {
  * {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}

/* Dark mode */
@media (prefers-color-scheme: dark) {
  :root {
    --color-text: #f8f9fa;
    --color-background: #212529;
  }
}
```

## Interactive Elements with Pure CSS

### CSS-Only Interactions

Many interactive patterns can be achieved with pure CSS using pseudo-classes and the `:target`, `:checked`, and `:focus-within` selectors.

**Accordion with Details/Summary**:
```html
<details>
  <summary>Click to expand</summary>
  <p>Hidden content that appears when expanded.</p>
</details>
```

```css
details {
  border: 1px solid #ccc;
  border-radius: 4px;
  padding: 0.5rem;
}

summary {
  cursor: pointer;
  font-weight: bold;
  padding: 0.5rem;
}

summary:hover {
  background-color: #f0f0f0;
}

details[open] summary {
  margin-block-end: 0.5rem;
}
```

**CSS-Only Modal with :target**:

Note: HTML provides a native `<dialog>` element with built-in accessibility and focus management, but it requires JavaScript (`.showModal()`) to open. The `:target` approach below works without JavaScript:
```html
<a href="#modal">Open Modal</a>

<div id="modal" class="modal">
  <div class="modal-content">
    <a href="#" class="modal-close">&times;</a>
    <h2>Modal Title</h2>
    <p>Modal content</p>
  </div>
</div>
```

```css
.modal {
  display: none;
  position: fixed;
  inset: 0;
  background-color: rgba(0, 0, 0, 0.5);
  z-index: 1000;
}

.modal:target {
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal-content {
  background-color: white;
  padding: 2rem;
  border-radius: 8px;
  max-inline-size: 500px;
  position: relative;
}

.modal-close {
  position: absolute;
  inset-block-start: 1rem;
  inset-inline-end: 1rem;
  font-size: 2rem;
  text-decoration: none;
  color: #333;
}
```

**CSS-Only Tabs with Radio Buttons**:
```html
<div class="tabs">
  <input type="radio" name="tabs" id="tab1" checked>
  <input type="radio" name="tabs" id="tab2">
  <input type="radio" name="tabs" id="tab3">
  
  <div class="tab-labels">
    <label for="tab1">Tab 1</label>
    <label for="tab2">Tab 2</label>
    <label for="tab3">Tab 3</label>
  </div>
  
  <div class="tab-content">
    <div class="tab-panel">Content 1</div>
    <div class="tab-panel">Content 2</div>
    <div class="tab-panel">Content 3</div>
  </div>
</div>
```

```css
.tabs input[type="radio"] {
  display: none;
}

.tab-labels {
  display: flex;
  gap: 0.5rem;
  border-block-end: 2px solid #ccc;
}

.tab-labels label {
  padding: 0.75rem 1.5rem;
  cursor: pointer;
  border-block-end: 2px solid transparent;
  margin-block-end: -2px;
}

.tab-labels label:hover {
  background-color: #f0f0f0;
}

#tab1:checked ~ .tab-labels label[for="tab1"],
#tab2:checked ~ .tab-labels label[for="tab2"],
#tab3:checked ~ .tab-labels label[for="tab3"] {
  border-block-end-color: var(--color-primary);
  font-weight: bold;
}

.tab-panel {
  display: none;
  padding: 1rem;
}

#tab1:checked ~ .tab-content .tab-panel:nth-child(1),
#tab2:checked ~ .tab-content .tab-panel:nth-child(2),
#tab3:checked ~ .tab-content .tab-panel:nth-child(3) {
  display: block;
}
```

**Dropdown Menu with :focus-within**:
```html
<nav class="dropdown">
  <a href="#" class="dropdown-trigger">Menu</a>
  <ul class="dropdown-menu">
    <li><a href="/page1.html">Page 1</a></li>
    <li><a href="/page2.html">Page 2</a></li>
    <li><a href="/page3.html">Page 3</a></li>
  </ul>
</nav>
```

```css
.dropdown {
  position: relative;
}

.dropdown-menu {
  display: none;
  position: absolute;
  inset-block-start: 100%;
  inset-inline-start: 0;
  background-color: white;
  border: 1px solid #ccc;
  list-style: none;
  padding: 0;
  min-inline-size: 200px;
}

.dropdown:hover .dropdown-menu,
.dropdown:focus-within .dropdown-menu {
  display: block;
}

.dropdown-menu a {
  display: block;
  padding: 0.75rem 1rem;
  text-decoration: none;
  color: inherit;
}

.dropdown-menu a:hover {
  background-color: #f0f0f0;
}
```

## Performance Optimization

### Loading Strategies

```html
<!-- Preload critical resources -->
<link rel="preload" href="styles/style.css" as="style">
<link rel="preload" href="fonts/main.woff2" as="font" type="font/woff2" crossorigin>

<!-- Lazy load images -->
<img src="image.jpg" alt="Description" loading="lazy">


```

### Image Optimization

```html
<!-- Responsive images -->
<picture>
  <source srcset="image-large.webp" media="(min-width: 1024px)" type="image/webp">
  <source srcset="image-medium.webp" media="(min-width: 768px)" type="image/webp">
  <source srcset="image-small.webp" type="image/webp">
  <img src="image-small.jpg" alt="Description" loading="lazy">
</picture>

<!-- srcset for resolution switching -->
<img 
  srcset="image-320w.jpg 320w,
          image-640w.jpg 640w,
          image-1280w.jpg 1280w"
  sizes="(max-width: 768px) 100vw, 50vw"
  src="image-640w.jpg"
  alt="Description">
```

### CSS Performance

```css
/* Use contain for performance */
.card {
  contain: layout style paint;
}

/* content-visibility for off-screen content */
.section {
  content-visibility: auto;
  contain-intrinsic-size: 0 500px;
}

/* will-change for animations */
.animated {
  will-change: transform;
}

.animated:hover {
  transform: scale(1.1);
}
```

## Common Patterns

### Responsive Navigation Menu

```html
<nav aria-label="Main navigation">
  <input type="checkbox" id="menu-toggle" class="menu-toggle">
  <label for="menu-toggle" class="menu-icon">
    <span></span>
    <span></span>
    <span></span>
  </label>
  <ul class="menu">
    <li><a href="/">Home</a></li>
    <li><a href="/about.html">About</a></li>
    <li><a href="/contact.html">Contact</a></li>
  </ul>
</nav>
```

```css
.menu-toggle {
  display: none;
}

.menu-icon {
  display: none;
  flex-direction: column;
  gap: 4px;
  cursor: pointer;
  padding: 0.5rem;
}

.menu-icon span {
  display: block;
  inline-size: 25px;
  block-size: 3px;
  background-color: #333;
  transition: transform 0.3s;
}

.menu {
  display: flex;
  gap: 1rem;
  list-style: none;
}

@media (max-width: 767px) {
  .menu-icon {
    display: flex;
  }
  
  .menu {
    display: none;
    flex-direction: column;
    inline-size: 100%;
  }
  
  .menu-toggle:checked ~ .menu {
    display: flex;
  }
  
  .menu-toggle:checked ~ .menu-icon span:nth-child(1) {
    transform: rotate(45deg) translate(5px, 5px);
  }
  
  .menu-toggle:checked ~ .menu-icon span:nth-child(2) {
    opacity: 0;
  }
  
  .menu-toggle:checked ~ .menu-icon span:nth-child(3) {
    transform: rotate(-45deg) translate(7px, -6px);
  }
}
```

### Card Grid Layout

```html
<section class="card-grid">
  <article class="card">
    <img src="image1.jpg" alt="Description">
    <h3>Card Title</h3>
    <p>Card description text.</p>
    <a href="/details.html" class="card-link">Read more</a>
  </article>
  <!-- More cards -->
</section>
```

```css
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 2rem;
  padding: 2rem;
}

.card {
  border: 1px solid #ddd;
  border-radius: 8px;
  overflow: hidden;
  transition: transform 0.2s, box-shadow 0.2s;
}

.card:hover {
  transform: translateY(-4px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.card img {
  inline-size: 100%;
  block-size: 200px;
  object-fit: cover;
}

.card h3,
.card p,
.card-link {
  padding-inline: 1rem;
}

.card h3 {
  margin-block-start: 1rem;
}

.card-link {
  display: inline-block;
  margin-block: 1rem;
  color: var(--color-primary);
  text-decoration: none;
}

.card-link:hover {
  text-decoration: underline;
}
```

### Hero Section

```html
<section class="hero">
  <div class="hero-content">
    <h1>Welcome to Our Site</h1>
    <p>Build amazing static websites with pure HTML and CSS</p>
    <a href="/get-started.html" class="button">Get Started</a>
  </div>
</section>
```

```css
.hero {
  min-block-size: 60vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  text-align: center;
  padding: 2rem;
}

.hero-content {
  max-inline-size: 800px;
}

.hero h1 {
  font-size: clamp(2rem, 5vw, 4rem);
  margin-block-end: 1rem;
}

.hero p {
  font-size: clamp(1rem, 2vw, 1.5rem);
  margin-block-end: 2rem;
}

.button {
  display: inline-block;
  padding: 1rem 2rem;
  background-color: white;
  color: #667eea;
  text-decoration: none;
  border-radius: 4px;
  font-weight: bold;
  transition: transform 0.2s;
}

.button:hover {
  transform: scale(1.05);
}
```

## Testing and Validation

### HTML Validation
- Use W3C Markup Validation Service
- Check semantic structure and accessibility
- Validate ARIA attributes

### CSS Validation
- Use W3C CSS Validation Service
- Check browser compatibility with caniuse.com
- Test responsive breakpoints
- Verify CSS-only interactions work without JavaScript

### Accessibility Testing
- Use browser DevTools accessibility inspector
- Test keyboard navigation (Tab, Enter for links)
- Verify screen reader compatibility
- Check color contrast ratios (WCAG AA minimum 4.5:1)
- Test with browser extensions (axe, WAVE)
- Ensure interactive elements are keyboard accessible
- Verify focus indicators are visible

### Performance Testing
- Use Lighthouse in Chrome DevTools
- Check Core Web Vitals (LCP, FID, CLS)
- Test on slow network connections
- Verify image optimization

## Deployment

Static sites can be deployed to any web server or static hosting service:

- GitHub Pages
- Netlify
- Vercel
- Cloudflare Pages
- AWS S3 + CloudFront
- Traditional web hosting (Apache, Nginx)

No build step required - upload files directly to hosting service.

## Resources

- [MDN Web Docs](https://developer.mozilla.org/) - Comprehensive web platform documentation
  - [HTML Reference](https://developer.mozilla.org/en-US/docs/Web/HTML)
  - [CSS Reference](https://developer.mozilla.org/en-US/docs/Web/CSS)
  - [Web APIs](https://developer.mozilla.org/en-US/docs/Web/API) - Browser APIs (requires JavaScript)
- [W3C Technical Reports](https://www.w3.org/TR/) - Official web standards and specifications
- [WHATWG Standards](https://spec.whatwg.org/) - Living standards for HTML, DOM, and web APIs
- [Browser Specs](https://github.com/w3c/browser-specs) - Curated, machine-readable list of current web specifications
- [web.dev](https://web.dev/) - Modern web development best practices
- [Can I Use](https://caniuse.com/) - Browser compatibility tables
- [Baseline](https://web.dev/baseline) - Widely available web platform features
- [WCAG Guidelines](https://www.w3.org/WAI/WCAG21/quickref/) - Accessibility standards
- [HTML Living Standard](https://html.spec.whatwg.org/) - HTML specification
- [CSS Specifications](https://www.w3.org/Style/CSS/) - CSS standards

## Anti-Patterns to Avoid

- Do not use inline styles - maintain CSS in separate files
- Do not use non-semantic divs when semantic elements exist
- Do not use `<br>` for spacing - use CSS margins/padding
- Do not use tables for layout - use CSS Grid or Flexbox
- Do not include JavaScript files or inline scripts
- Do not include external dependencies or frameworks
- Do not use build tools, transpilers, or preprocessors
- Do not use CSS frameworks like Bootstrap or Tailwind
- Do not use CSS preprocessors like Sass or Less
- Do not rely on JavaScript for core functionality
- Do not use `onclick` or other inline event handlers
- Do not create interactions that require JavaScript when CSS alternatives exist

## Limitations of Static HTML/CSS Sites

Be aware of what cannot be achieved without JavaScript:

- Dynamic content loading (AJAX requests)
- Form validation beyond HTML5 built-in validation
- Complex state management
- Real-time updates
- Local storage persistence
- API integrations
- Single-page application routing
- Complex animations triggered by scroll position

For these features, consider whether they are truly necessary or if simpler alternatives exist. Many websites can function effectively with pure HTML and CSS.
