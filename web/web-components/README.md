# web-components

Architecture and coding rules for building single-page applications with web components, BCE architecture, and zero frameworks.

## Stack

- **Custom Elements** via `BElement` base class with automatic Redux integration
- **lit-html** for templating and efficient DOM rendering
- **Redux Toolkit** for predictable state management
- **Vaadin Router** for client-side routing
- **BCE layering** (Boundary / Control / Entity) organized by feature module

## Principles

- Web standards and web platform first
- Minimal external dependencies — only lit-html, Redux Toolkit, and Vaadin Router
- No frameworks, no build step for development, no Shadow DOM by default
- Plain ES modules with import maps

## Project Structure

```
app/src/
├── BElement.js          # base class for all custom elements
├── store.js             # Redux store configuration
├── app.js               # entry point, routing, persistence
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
