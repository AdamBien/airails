---
name: web-components
description: Architecture and coding rules for single-page applications using web components, BCE layering, lit-html templating, unidirectional Redux-style state management (reduction.js, standards-based), and standards-based client-side routing (Navigation API + URLPattern). Web standards and web platform first, minimal external dependencies. Composes with `web-conventions` (semantic HTML, accessibility, design tokens, Baseline policy). Use when creating, scaffolding, generating, writing, or reviewing web component applications, custom elements, state management, client-side routing, or frontend BCE architecture. Not for server-side rendering or framework-heavy applications, and not for static content sites without client-side state — use `web-static` for those.
---

Build or maintain a web component application using $ARGUMENTS. Apply all rules below strictly.

## Guiding Principles

- web standards and web platform first
- minimal external dependencies — every dependency must justify its existence, and is continuously replaced with web standards
- no frameworks — use the platform: custom elements, ES modules, import maps
- no build system — dependencies are vendored as self-contained ES modules; nothing to compile, bundle, or transpile
- progressive enhancement over JavaScript-first design

## Composition

Compose with `/web-conventions` — it provides the baseline rules for semantic HTML, accessibility,
modern CSS, design tokens, and the Baseline browser-support policy. Rules in this skill override it
(e.g. container-query-first responsiveness). For static content sites without
client-side state, use `/web-static` instead. For experiments and PoCs, `/web-latest`
may be composed on top to lift the Baseline browser-support constraints.

## Reference Implementation

This skill is based on the [bce.design](https://github.com/AdamBien/bce.design) quickstarter.
The naming rules below are stricter than the single-module quickstarter (which registers unprefixed
tags like `b-add`) — multi-module applications need module-prefixed tag names.

## Allowed Dependencies

Only one runtime dependency is permitted. Do not add others without explicit approval:

1. **lit-html** — templating and efficient DOM rendering

State management needs no dependency — `reduction.js`, a minimal in-app store
(~70 lines), implements the used Redux Toolkit API (`configureStore`, `createAction`,
`createReducer`) on top of the `structuredClone()` web standard (see State Management below).
The original Redux Toolkit may be vendored as an optional, import-map-switchable alternative.

Client-side routing needs no dependency — it is implemented with web standards
(Navigation API + URLPattern, both Baseline Newly Available; see Routing below).

Development tooling: any static web server with an `index.html` fallback (e.g. zws,
bundled with `web-conventions`, where `--single` provides the fallback and `--live`
adds reload-on-save while authoring) and Playwright (E2E tests). There is no build system.

## BCE Architecture (Boundary Control Entity)

Structure code using the Boundary Control Entity pattern. Organize by feature module, not by technical layer:

```
app/src/
├── BElement.js              # base class for all custom elements
├── store.js                 # store configuration
├── reduction.js             # minimal standards-based store (Redux Toolkit API subset)
├── router.js                # standards-based client-side router (Navigation API + URLPattern)
├── app.js                   # entry point, route configuration, persistence
├── app.config.js            # application metadata (name, version)
├── index.html               # single HTML entry point
├── tokens.css               # design-token custom properties
├── style.css                # application styles
├── libs/                    # vendored third-party ESM modules (lit-html)
├── [feature-name]/          # one folder per feature module (business component)
│   ├── package-info.md      # the module's responsibility & design decisions
│   ├── boundary/            # UI web components
│   ├── control/             # actions and dispatchers
│   └── entity/              # reducers and state shape
└── localstorage/
    └── control/
        └── StorageControl.js  # persistence layer
```

`DESIGN.md` and the W3C Design Tokens file `tokens.json` live at the repository root
(see `/web-conventions`); `tokens.css` is their CSS custom-property projection.

### Boundary (UI Components)

- all components extend `BElement` — never extend `HTMLElement` directly
- components are the only layer that imports `html` from lit-html
- components dispatch actions through control functions — never dispatch directly to the store
- override `view()` to return a lit-html template — this is the only required method
- override `extractState(reduxState)` to select the relevant state slice
- access extracted state via `this.state` inside `view()`
- register every component with `customElements.define('prefix-name', ClassName)`
- use `b-` prefix for component tag names
- composite components import and compose child components
- each module's boundary exposes one primary entry point named after the module — `bookmarks/boundary/Bookmarks.js` registering `b-bookmarks` — the module's default routing target
- additional routed views and child components carry the module prefix in their tag name (`b-bookmarks-add`) — tag names are global, so unprefixed names collide across modules

```javascript
import BElement from "../../BElement.js";
import { html } from "lit-html";
import { deleteItem } from "../control/CRUDControl.js";

class ItemList extends BElement {
    extractState({ items: { list } }) {
        return list;
    }
    view() {
        return html`
        <ol>
            ${this.state.map(item => html`
                <li>${item.label} <button @click="${_ => deleteItem(item.id)}">delete</button></li>
            `)}
        </ol>
        `;
    }
}
customElements.define('b-items-list', ItemList);
```

### Control (Actions & Dispatchers)

- define actions with `createAction` imported from `@reduxjs/toolkit` — the import map resolves it to `reduction.js`
- export both the action creator and a dispatcher function for each action
- dispatcher functions encapsulate `store.dispatch()` — boundary components call dispatchers, never dispatch directly
- use `Date.now()` for generating entity IDs on the client

```javascript
import { createAction } from "@reduxjs/toolkit";
import store from "../../store.js";

export const itemUpdatedAction = createAction("itemUpdatedAction");
export const itemUpdated = (name, value) => {
    store.dispatch(itemUpdatedAction({ name, value }));
}

export const newItemAction = createAction("newItemAction");
export const newItem = _ => {
    store.dispatch(newItemAction(Date.now()));
}
```

### Entity (Reducers & State)

- use `createReducer` with builder callback (imported from `@reduxjs/toolkit`, resolved to `reduction.js`)
- case handlers receive a `structuredClone` of the current state — mutate it freely or return a replacement; there is no Immer
- define `initialState` as a plain object with sensible defaults
- use a temporal cache object pattern for form input before committing to a list
- keep state shape flat — avoid deep nesting

```javascript
import { createReducer } from "@reduxjs/toolkit";
import { itemUpdatedAction, newItemAction } from "../control/CRUDControl.js";

const initialState = {
    list: [],
    item: {}
}

export const items = createReducer(initialState, (builder) => {
    builder.addCase(itemUpdatedAction, (state, { payload: { name, value } }) => {
        state.item[name] = value;
    }).addCase(newItemAction, (state, { payload }) => {
        state.item["id"] = payload;
        state.list = state.list.concat(state.item);
    });
});
```

## Documentation

`package-info.md` is the web counterpart of Java's `package-info.java` — a Markdown file at the
feature-module root (`app/src/[feature-name]/package-info.md`) that documents the business
component at the folder level, co-located with its code.

- at minimum, state the module's single responsibility and the design decisions behind it — not its contents or a file listing
- keep it focused; do not restate the BCE pattern itself or explain boundary/control/entity generically

When the project is driven by **SBCE** (`/sbce`), this file *is* the capability spec — it holds
the BC's full boundary contract (responsibility, `## Boundary` operations, `## Requirements` as
EARS statements, optional `## Entities`, `## Out of scope`). One `package-info.md` per business
component; it is the single source of truth that `/sbce apply` converges the code to.

### JSDoc

Plain ES modules carry no type annotations — JSDoc provides them. Every method and function
ships with a JSDoc comment declaring its types:

- declare parameter and return types with `@param {Type} name` and `@returns {Type}`
- describe entity and state shapes once with `@typedef` in the entity layer and reference them from the other layers
- keep the comments type-focused — add prose only for intent the types cannot express

## BElement Base Class

The base class provides automatic store integration for all components:

- `connectedCallback()` — subscribes to the store, triggers initial render
- `disconnectedCallback()` — unsubscribes to prevent memory leaks
- `triggerViewUpdate()` — extracts state, generates template, renders via lit-html
- `view()` — abstract, must be overridden to return a lit-html template
- `extractState(reduxState)` — override to select a state slice (default: entire state)
- `getRenderTarget()` — override to change render target (default: `this`)

Do not modify the BElement base class unless adding cross-cutting concerns that affect all components.

## State Management

### reduction.js — a minimal redux

State management is unidirectional Redux-style data flow without the Redux dependency.
`reduction.js` implements the used Redux Toolkit API surface — `configureStore`,
`createAction`, `createReducer` — in ~70 lines of standards-based code. Immutability
comes from `structuredClone()` instead of Immer: each case handler receives a clone of
the current state and may mutate it freely, or return a replacement state.

The implementation is selected in the import map (`index.html`): `@reduxjs/toolkit`
resolves to `/reduction.js` by default. The original Redux Toolkit, vendored in `libs/`,
remains a one-line import-map switch away — application code is identical for both.

### Store Configuration

```javascript
import { configureStore } from "@reduxjs/toolkit";
import { load } from "./localstorage/control/StorageControl.js";
import { items } from "./items/entity/ItemsReducer.js";

const reducer = { items }
const preloadedState = load();
const config = preloadedState ? { reducer, preloadedState } : { reducer };
const store = configureStore(config);
export default store;
```

### localStorage Persistence

- persist the entire application state to localStorage on every store update via `store.subscribe()`
- use `JSON.stringify` / `JSON.parse` for serialization
- derive storage key from `appName` in `app.config.js`
- wrap save/load in try-catch — never let persistence errors crash the application
- preload persisted state on store initialization

### Data Flow

```
User Event → Boundary Component → Control (dispatcher) → Action
→ Entity (reducer) → Store → BElement.triggerViewUpdate()
→ extractState() → view() → lit-html render() → DOM
```

## Routing

Routing is implemented with web standards — no router library. URLPattern matches routes,
the Navigation API intercepts same-origin navigations (link clicks, back/forward) with
built-in focus and scroll handling.

- keep the router mechanics in `router.js`; `app.js` only declares the route table
- define routes mapping paths to custom element tag names
- named path parameters (`/items/:id`) are passed to the routed component as attributes
- import all routed components before initializing the router
- use standard `<a href="...">` for navigation — no programmatic routing unless necessary
- non-matching and cross-origin URLs must fall through to regular browser navigation
- never intercept reloads (`event.navigationType === 'reload'`) — intercepting would turn them
  into same-document re-renders with stale modules and in-memory state; reload means reload
- serve the app with an `index.html` fallback for unknown paths (deep links)

```javascript
// router.js
export const initRouter = (outlet, routeConfig) => {
    const routes = routeConfig.map(({ path, component }) => ({
        pattern: new URLPattern({ pathname: path }),
        component
    }));

    const render = url => {
        const match = routes
            .map(({ pattern, component }) => ({ result: pattern.exec(url), component }))
            .find(({ result }) => result);
        if (!match) return false;
        const routeComponent = document.createElement(match.component);
        Object.entries(match.result.pathname.groups)
            .filter(([, value]) => value !== undefined)
            .forEach(([name, value]) => routeComponent.setAttribute(name, value));
        outlet.replaceChildren(routeComponent);
        return true;
    };

    navigation.addEventListener('navigate', event => {
        if (!event.canIntercept || event.hashChange || event.downloadRequest) return;
        if (event.navigationType === 'reload') return;
        const url = new URL(event.destination.url);
        if (!routes.some(({ pattern }) => pattern.test(url))) return;
        event.intercept({ handler: () => render(url) });
    });

    render(new URL(location.href));
}
```

```javascript
// app.js
import { initRouter } from "./router.js";

initRouter(document.querySelector('.view'), [
  { path: '/',          component: 'b-items' },
  { path: '/add',       component: 'b-items-add' },
  { path: '/edit/:id',  component: 'b-items-add' }
]);
```

URLPattern uses the same `:param` syntax as router libraries (both inherit it from
`path-to-regexp`). A routed component reads its parameters with `this.getAttribute('id')`
and loads the matching entity through the control layer.

### View Transitions (Same-Document) — Optional

Route changes may animate with same-document view transitions — the SPA counterpart of
`web-static`'s cross-document `@view-transition`. The reference implementation renders
instantly; add transitions only when the application calls for them, by wrapping `render`:

```javascript
const reducedMotion = matchMedia('(prefers-reduced-motion: reduce)');
const transitionTo = url => {
    if (!document.startViewTransition || reducedMotion.matches) return render(url);
    return document.startViewTransition(() => render(url)).finished;
};
// in the navigate listener: event.intercept({ handler: () => transitionTo(url) });
```

- same-document view transitions are Baseline Newly Available — the `document.startViewTransition`
  existence check is the required feature detection; browsers without it render instantly
- skip the transition under `prefers-reduced-motion: reduce` — reduced motion means instant route changes
- give elements that persist across routes (header, logo) a `view-transition-name` for continuity;
  each name must be unique within the rendered page
- customize the animation in CSS via `::view-transition-old(root)` / `::view-transition-new(root)` —
  the default cross-fade needs no CSS at all

## HTML Structure

- single `index.html` entry point with semantic HTML (`<main>`, `<header>`, `<nav>`, `<section>`, `<footer>`)
- declare import maps in `<head>` to map bare module specifiers to local vendored files
- use root-absolute URLs (`/style.css`, `/app.js`, `/libs/...`) for all resources — with the
  `index.html` fallback the document is also served for nested routes like `/edit/17`, where
  relative URLs would resolve against `/edit/` and 404
- load `init.js` (non-module) before `app.js` (module) only when a dependency needs globals —
  Redux Toolkit requires `window.process.env.NODE_ENV`; with `reduction.js` no globals are needed
- the router outlet is a `<section>` element — router replaces its content with route components
- use native `<dialog>` with `showModal()` for modal dialogs (Widely Available) — focus trapping, Esc dismissal, and `::backdrop` styling come for free; close with `dialog.close()` or `<form method="dialog">`

```html
<script type="importmap">
{
  "imports": {
    "lit-html": "/libs/lit-html.js",
    "@reduxjs/toolkit": "/reduction.js"
  }
}
</script>
```

## CSS Rules

Design tokens, logical properties, modern CSS features, accessibility, and the Baseline policy
follow `/web-conventions`. This stack adds:

- no CSS framework — plain CSS on design tokens; Bulma was removed from the reference in favor of custom properties
- keep design-token custom properties (`--color-primary`, `--spacing-md`, …) in a dedicated `tokens.css`, loaded before `style.css`; `tokens.css` projects the root-level `tokens.json` (W3C Design Tokens)
- use CSS Grid for page-level layout with named grid areas
- use container queries (`@container`) over media queries for responsive design
- set `container-type: inline-size` on `body`
- use `dvh` units for viewport height (`min-height: 100dvh`)
- style custom elements directly by tag name (`b-items { ... }`)
- keep custom styles in a separate `style.css`
- use flexbox for component-level layout

## Dependency Management

- there is no build system — no Vite, no Rollup, no npm install in the application
- runtime dependencies are vendored as self-contained ES modules in `app/src/libs/`
- update by fetching the released module straight from the npm registry with a small
  script (e.g. `update-lit-html.sh <version>`: `curl` the tarball, extract the single
  `lit-html.js` file into `libs/`) — an update is a plain file copy
- only dependencies that ship as a single dependency-free ESM file qualify — if a library
  needs bundling to become self-contained, that is a signal against adopting it
- import maps in `index.html` map bare specifiers to the vendored local files
- the application source never references `node_modules` — always import through the import map

## Testing

- E2E tests with Playwright in a separate `tests/` directory
- test across Chromium, Firefox, and WebKit
- use role selectors (`getByRole`), label selectors (`getByLabel`), and visibility assertions
- use `pressSequentially` with delay for simulating realistic text input
- code coverage in a dedicated `codecoverage/` directory using Monocart Coverage Reports
- tests verify user-visible behavior, not implementation details

## Event Handling in Templates

- use lit-html event bindings: `@click`, `@keyup`, `@input`
- destructure event objects to extract relevant data: `({ target: { name, value } })`
- use `event.preventDefault()` for form submissions
- validate forms using native HTML validation: `form.reportValidity()` and `form.checkValidity()` — both only pay off when inputs declare standard types and constraint attributes (`required`, `pattern`, `min`/`max`) per `web-conventions`
- for rules markup cannot express, use `setCustomValidity()` on the input so the error surfaces through the same native reporting — do not build a parallel error display
- read submitted values with `new FormData(form)` instead of querying individual inputs

### Form-Associated Custom Elements

Components rendering native inputs into their light DOM need nothing extra — the inputs participate
in the surrounding `<form>` natively. Make a component form-associated (Baseline Widely Available)
only when the element itself is the control — a composite input with no single native equivalent
(rating widget, tag picker) — and must contribute to `FormData` and native validation:

- declare `static formAssociated = true` and call `this.attachInternals()` in the constructor
- report the current value with `internals.setFormValue()` on every change
- express validity with `internals.setValidity()` so errors surface through the same native
  reporting (`reportValidity()`) — the same rule as `setCustomValidity()` above: no parallel error display
- prefer composing standard inputs first — reach for form association only when no standard input fits

## What NOT to Do

- do not use Shadow DOM unless encapsulation is explicitly required
- do not use TypeScript — plain ES modules with JSDoc type declarations only
- do not install dependencies in the application root
- do not import from `node_modules` paths — use import maps
- do not add CSS preprocessors (Sass, Less, PostCSS)
- do not add a CSS framework (Bulma, Tailwind, …) — plain CSS on design tokens
- do not add bundlers or a build system — dependencies are vendored, application code runs as-is
- do not use Vite as dev server — it resolves bare imports itself instead of honoring the import map; use a plain static server with `index.html` fallback
- do not use React, Angular, Vue, Svelte, or any component framework
- do not add Redux Toolkit as a default dependency — `reduction.js` covers the used API; vendored Redux Toolkit is an optional import-map switch
- do not add a routing library (Vaadin Router, page.js, …) — route with the Navigation API and URLPattern
- do not create package.json in the application root
- do not dispatch actions directly from boundary components — always go through control functions
- do not put business logic in boundary components — delegate to control layer
- do not hand-roll modal overlays from `<div>`s — use `<dialog>`
