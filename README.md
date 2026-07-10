# web-components

An [AIrails.dev](https://airails.dev) skill defining architecture and coding rules for building single-page applications with web components, BCE architecture, and zero frameworks.

## Stack

- **Custom Elements** via `BElement` base class with automatic Redux integration
- **lit-html** for templating and efficient DOM rendering
- **Redux Toolkit** for predictable state management
- **Navigation API + URLPattern** for standards-based client-side routing — no router dependency
- **BCE layering** (Boundary / Control / Entity) organized by feature module

## Principles

- Web standards and web platform first
- Minimal external dependencies — only lit-html and Redux Toolkit
- No frameworks, no build step for development, no Shadow DOM by default
- Plain ES modules with import maps

## Composition

Composes with [web-conventions](../web-conventions) — the shared baseline for semantic HTML, accessibility, design tokens, and the Baseline browser-support policy. Rules in this skill override it. For static content sites without client-side state, use [web-static](../web-static).

Serves as a stack skill for [SBCE](https://sbce.dev) — spec-driven BCE, where each capability spec lives in the BC's `package-info.md` and converges via `/sbce apply`.

## Project Structure

```
app/src/
├── BElement.js          # base class for all custom elements
├── store.js             # Redux store configuration
├── router.js            # standards-based router (Navigation API + URLPattern)
├── app.js               # entry point, route configuration, persistence
├── index.html           # single HTML entry point
├── libs/                # bundled third-party ESM modules
└── [feature]/
    ├── boundary/        # UI web components
    ├── control/         # actions and dispatchers
    └── entity/          # reducers and state shape
```

## Data Flow

```
User Event → Boundary → Control (dispatcher) → Redux Action
→ Entity (reducer) → Store → BElement → view() → lit-html → DOM
```

## Usage

`SKILL.md` contains the full architectural reference. It can be used as a coding guide for developers or as a skill file for AI coding assistants.
