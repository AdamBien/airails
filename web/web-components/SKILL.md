---
name: web-components
description: Architecture and coding rules for single-page applications using web components, BCE layering, lit-html templating, Redux Toolkit state management, and client-side routing. Web standards and web platform first, minimal external dependencies. Use when creating, scaffolding, generating, writing, or reviewing web component applications, custom elements, state management, client-side routing, or frontend BCE architecture. Not for server-side rendering or framework-heavy applications.
---

Build or maintain a web component application using $ARGUMENTS. Apply all rules below strictly.

## Guiding Principles

- web standards and web platform first
- minimal external dependencies — every dependency must justify its existence
- no frameworks — use the platform: custom elements, ES modules, import maps
- no build step required for development — bundling is only for packaging third-party dependencies
- progressive enhancement over JavaScript-first design

## Reference Implementation

This skill is based on the [bce.design](https://github.com/AdamBien/bce.design) quickstarter.

## Allowed Dependencies

Only these three runtime dependencies are permitted. Do not add others without explicit approval:

1. **lit-html** — templating and efficient DOM rendering
2. **Redux Toolkit** — predictable state management with unidirectional data flow
3. **Vaadin Router** — client-side routing via History API

Development tooling: Vite (dev server), Rollup (dependency bundling), Playwright (E2E tests).

## BCE Architecture (Boundary Control Entity)

Structure code using the Boundary Control Entity pattern. Organize by feature module, not by technical layer:

```
app/src/
├── BElement.js              # base class for all custom elements
├── store.js                 # Redux store configuration
├── app.js                   # entry point, routing, persistence
├── app.config.js            # application metadata (name, version)
├── index.html               # single HTML entry point
├── style.css                # application styles
├── libs/                    # bundled third-party ESM modules
├── [feature-name]/          # one folder per feature module
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

export const deleteItemAction = createAction("deleteItemAction");
export const deleteItem = (id) => {
    store.dispatch(deleteItemAction(id));
}
```

### Entity (Reducers & State)

- use `createReducer` from Redux Toolkit with builder callback
- define `initialState` as a plain object with sensible defaults
- use a temporal cache object pattern for form input before committing to a list
- keep state shape flat — avoid deep nesting

```javascript
import { createReducer } from "@reduxjs/toolkit";
import { itemUpdatedAction, deleteItemAction, newItemAction } from "../control/CRUDControl.js";

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
    }).addCase(deleteItemAction, (state, { payload }) => {
        state.list = state.list.filter(item => item.id !== payload);
    });
});
```

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

- use Vaadin Router with an HTML element as outlet
- define routes mapping paths to custom element tag names
- import all routed components before initializing the router
- use standard `<a href="...">` for navigation — no programmatic routing unless necessary

```javascript
import { Router } from "@vaadin/router";
const outlet = document.querySelector('.view');
const router = new Router(outlet, {});
router.setRoutes([
  { path: '/',     component: 'b-item-list' },
  { path: '/add',  component: 'b-items' }
]);
```

## HTML Structure

- single `index.html` entry point with semantic HTML (`<main>`, `<header>`, `<nav>`, `<section>`, `<footer>`)
- declare import maps in `<head>` to map bare module specifiers to local bundled files
- load `init.js` (non-module) before `app.js` (module) to set up globals required by dependencies
- the router outlet is a `<section>` element — router replaces its content with route components

```html
<script type="importmap">
{
  "imports": {
    "lit-html": "./libs/lit-html.js",
    "@reduxjs/toolkit": "./libs/redux-toolkit.modern.js",
    "@vaadin/router": "./libs/router.js"
  }
}
</script>
```

## CSS Rules

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

```javascript
// rollup.config.js
export default [{
  input: [
    './node_modules/lit-html/lit-html.js',
    './node_modules/@vaadin/router/dist/router.js',
    './node_modules/@reduxjs/toolkit/dist/redux-toolkit.modern.mjs'
  ],
  output: { dir: "../app/src/libs", format: "esm" },
  plugins: [nodeResolve({ browser: true })]
}]
```

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
- validate forms using native HTML validation: `form.reportValidity()` and `form.checkValidity()`

## What NOT to Do

- do not use Shadow DOM unless encapsulation is explicitly required
- do not use TypeScript — plain ES modules only
- do not install dependencies in the application root
- do not import from `node_modules` paths — use import maps
- do not add CSS preprocessors (Sass, Less, PostCSS)
- do not add bundlers for application code — only for third-party dependency packaging
- do not use React, Angular, Vue, Svelte, or any component framework
- do not use npm scripts for development workflow beyond `npx vite`
- do not create package.json in the application root
- do not dispatch Redux actions directly from boundary components — always go through control functions
- do not put business logic in boundary components — delegate to control layer
