---
name: web-components
description: Architecture and coding rules for single-page applications using web components, BCE layering, lit-html templating, Redux Toolkit state management, and standards-based client-side routing (Navigation API + URLPattern). Web standards and web platform first, minimal external dependencies. Composes with `web-conventions` (semantic HTML, accessibility, design tokens, Baseline policy). Use when creating, scaffolding, generating, writing, or reviewing web component applications, custom elements, state management, client-side routing, or frontend BCE architecture. Not for server-side rendering or framework-heavy applications, and not for static content sites without client-side state — use `web-static` for those.
---

Build or maintain a web component application using $ARGUMENTS. Apply all rules below strictly.

## Guiding Principles

- web standards and web platform first
- minimal external dependencies — every dependency must justify its existence
- no frameworks — use the platform: custom elements, ES modules, import maps
- no build step required for development — bundling is only for packaging third-party dependencies
- progressive enhancement over JavaScript-first design

## Composition

Compose with `/web-conventions` — it provides the baseline rules for semantic HTML, accessibility,
modern CSS, design tokens, and the Baseline browser-support policy. Rules in this skill override it
(e.g. Bulma utility classes, container-query-first responsiveness). For static content sites without
client-side state, use `/web-static` instead.

## Reference Implementation

This skill is based on the [bce.design](https://github.com/AdamBien/bce.design) quickstarter.

## Allowed Dependencies

Only these two runtime dependencies are permitted. Do not add others without explicit approval:

1. **lit-html** — templating and efficient DOM rendering
2. **Redux Toolkit** — predictable state management with unidirectional data flow

Client-side routing needs no dependency — it is implemented with web standards
(Navigation API + URLPattern, both Baseline Newly Available; see Routing below).

Development tooling: Vite (dev server), Rollup (dependency bundling), Playwright (E2E tests).

## BCE Architecture (Boundary Control Entity)

Structure code using the Boundary Control Entity pattern. Organize by feature module, not by technical layer:

```
app/src/
├── BElement.js              # base class for all custom elements
├── store.js                 # Redux store configuration
├── router.js                # standards-based client-side router (Navigation API + URLPattern)
├── app.js                   # entry point, route configuration, persistence
├── app.config.js            # application metadata (name, version)
├── index.html               # single HTML entry point
├── style.css                # application styles
├── libs/                    # bundled third-party ESM modules
├── [feature-name]/          # one folder per feature module (business component)
│   ├── package-info.md      # the module's responsibility & design decisions
│   ├── boundary/            # UI web components
│   ├── control/             # actions and dispatchers
│   └── entity/              # reducers and state shape
└── localstorage/
    └── control/
        └── StorageControl.js  # persistence layer
```

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
- when a module exposes a composite entry point, it is a facade component named after the module: `bookmarks/boundary/Bookmarks.js` registering `b-bookmarks` — this is the module's default routing target

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
customElements.define('b-item-list', ItemList);
```

### Control (Actions & Dispatchers)

- define Redux actions with `createAction` from Redux Toolkit
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

- use `createReducer` from Redux Toolkit with builder callback
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

The base class provides automatic Redux integration for all components:

- `connectedCallback()` — subscribes to Redux store, triggers initial render
- `disconnectedCallback()` — unsubscribes to prevent memory leaks
- `triggerViewUpdate()` — extracts state, generates template, renders via lit-html
- `view()` — abstract, must be overridden to return a lit-html template
- `extractState(reduxState)` — override to select a state slice (default: entire state)
- `getRenderTarget()` — override to change render target (default: `this`)

Do not modify the BElement base class unless adding cross-cutting concerns that affect all components.

## State Management

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

- persist entire Redux state to localStorage on every store update via `store.subscribe()`
- use `JSON.stringify` / `JSON.parse` for serialization
- derive storage key from `appName` in `app.config.js`
- wrap save/load in try-catch — never let persistence errors crash the application
- preload persisted state on store initialization

### Data Flow

```
User Event → Boundary Component → Control (dispatcher) → Redux Action
→ Entity (reducer) → Store → BElement.triggerViewUpdate()
→ extractState() → view() → lit-html render() → DOM
```

## Routing

Routing is implemented with web standards — no router library. URLPattern matches routes,
the Navigation API intercepts same-origin navigations (link clicks, back/forward) with
built-in focus and scroll handling.

- keep the router mechanics in `router.js`; `app.js` only declares the route table
- define routes mapping paths to custom element tag names
- import all routed components before initializing the router
- use standard `<a href="...">` for navigation — no programmatic routing unless necessary
- non-matching and cross-origin URLs must fall through to regular browser navigation
- serve the app with an `index.html` fallback for unknown paths (deep links)

```javascript
// router.js
export const initRouter = (outlet, routeConfig) => {
    const routes = routeConfig.map(({ path, component }) => ({
        pattern: new URLPattern({ pathname: path }),
        component
    }));

    const render = url => {
        const route = routes.find(({ pattern }) => pattern.test(url));
        if (!route) return false;
        outlet.replaceChildren(document.createElement(route.component));
        return true;
    };

    const reducedMotion = matchMedia('(prefers-reduced-motion: reduce)');

    const transitionTo = url => {
        if (!document.startViewTransition || reducedMotion.matches) return render(url);
        return document.startViewTransition(() => render(url)).finished;
    };

    navigation.addEventListener('navigate', event => {
        if (!event.canIntercept || event.hashChange || event.downloadRequest) return;
        const url = new URL(event.destination.url);
        if (!routes.some(({ pattern }) => pattern.test(url))) return;
        event.intercept({ handler: () => transitionTo(url) });
    });

    render(new URL(location.href));
}
```

```javascript
// app.js
import { initRouter } from "./router.js";

initRouter(document.querySelector('.view'), [
  { path: '/',     component: 'b-item-list' },
  { path: '/add',  component: 'b-items' }
]);
```

For parameterized routes, extend `router.js` only: match with `pattern.exec(url)` and pass
the named groups (`/items/:id`) to the created element.

### View Transitions (Same-Document)

Route changes animate with same-document view transitions, as in the router above — the SPA
counterpart of `web-static`'s cross-document `@view-transition` for multipage sites.

- same-document view transitions are Baseline Newly Available — the `document.startViewTransition`
  existence check in `transitionTo` is the required feature detection; browsers without it render instantly
- skip the transition under `prefers-reduced-motion: reduce` — reduced motion means instant route changes
- give elements that persist across routes (header, logo) a `view-transition-name` for continuity;
  each name must be unique within the rendered page
- customize the animation in CSS via `::view-transition-old(root)` / `::view-transition-new(root)` —
  the default cross-fade needs no CSS at all

## HTML Structure

- single `index.html` entry point with semantic HTML (`<main>`, `<header>`, `<nav>`, `<section>`, `<footer>`)
- declare import maps in `<head>` to map bare module specifiers to local bundled files
- load `init.js` (non-module) before `app.js` (module) to set up globals required by dependencies
- the router outlet is a `<section>` element — router replaces its content with route components
- use native `<dialog>` with `showModal()` for modal dialogs (Widely Available) — focus trapping, Esc dismissal, and `::backdrop` styling come for free; close with `dialog.close()` or `<form method="dialog">`

```html
<script type="importmap">
{
  "imports": {
    "lit-html": "./libs/lit-html.js",
    "@reduxjs/toolkit": "./libs/redux-toolkit.modern.js"
  }
}
</script>
```

## CSS Rules

Design tokens, logical properties, modern CSS features, accessibility, and the Baseline policy
follow `/web-conventions`. This stack adds:

- align design-token custom properties with Bulma's CSS variables (`--bulma-primary`, …) where they are overridden
- use CSS Grid for page-level layout with named grid areas
- use container queries (`@container`) over media queries for responsive design
- set `container-type: inline-size` on `body`
- use `dvh` units for viewport height (`min-height: 100dvh`)
- style custom elements directly by tag name (`b-list { ... }`)
- use a lightweight CSS framework (Bulma) for utility classes — no Tailwind
- keep custom styles in a separate `style.css`
- use flexbox for component-level layout

## Dependency Management

- third-party dependencies are installed in a separate `libs/` directory (not the app root)
- Rollup bundles dependencies from `node_modules` as ESM into `app/src/libs/`
- import maps in `index.html` map bare specifiers to the bundled local files
- the application source never imports from `node_modules` — always through import maps

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

```javascript
class TagPicker extends BElement {
    static formAssociated = true;
    constructor() {
        super();
        this.internals = this.attachInternals();
    }
    onTagsChanged(tags) {
        this.internals.setFormValue(tags.join(','));
        tags.length
            ? this.internals.setValidity({})
            : this.internals.setValidity({ valueMissing: true }, 'Add at least one tag');
    }
}
customElements.define('b-tag-picker', TagPicker);
```

## What NOT to Do

- do not use Shadow DOM unless encapsulation is explicitly required
- do not use TypeScript — plain ES modules with JSDoc type declarations only
- do not install dependencies in the application root
- do not import from `node_modules` paths — use import maps
- do not add CSS preprocessors (Sass, Less, PostCSS)
- do not add bundlers for application code — only for third-party dependency packaging
- do not use React, Angular, Vue, Svelte, or any component framework
- do not add a routing library (Vaadin Router, page.js, …) — route with the Navigation API and URLPattern
- do not use npm scripts for development workflow beyond `npx vite`
- do not create package.json in the application root
- do not dispatch Redux actions directly from boundary components — always go through control functions
- do not put business logic in boundary components — delegate to control layer
- do not hand-roll modal overlays from `<div>`s — use `<dialog>`
