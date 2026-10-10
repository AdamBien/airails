---
name: web-standards-reviewer
argument-hint: full|diff [path-or-git-ref]
description: Review an existing web project for web-standards use and essential dependencies only, against the rules of `/web-conventions`, `/javascript-conventions`, and the stack skill the project composes (`web-static`, `web-sprinkles`, `web-components`), and answer with the shortest possible feedback — one line per finding, one line when clean. Evidence is collected with the bundled read-only zllm-scripts (`scripts/files`, `search`, `show`, `imports`) per `/zllm-scripts`, requiring Java 25+. Three categories — build step, dependencies beyond the stack allowlist, hand-rolled code the platform covers — and two modes, `full` for the whole project and `diff` for uncommitted changes or a git range. Use whenever the frontend stack must be checked — triggers on "web standards review", "review dependencies", "dependency review", "is this standards-based", "what can the platform replace", "do we need this library", "review the frontend stack", "too many dependencies", "/web-standards-reviewer". Not for BCE layering (`bce-reviewer`), performance (`web-performance-reviewer`), rendered-page verification (`web-static`), and never for fixing or migrating.
---

Review the build, dependency, and web-standards posture of $ARGUMENTS and report as briefly as possible. Review only; never modify code.

## Composition

- rules come from `/javascript-conventions` (platform-first list, ES module rules), `/web-conventions` (Baseline policy, snapshot lookup), and the detected stack skill (dependency allowlist, build rules, layout); this skill owns only the procedure and the output format
- what the stack skill explicitly allows is not a finding: `/web-components` allows lit-html, vendored Redux Toolkit as an import-map switch, zws plus Playwright as development tooling, and the Navigation API plus URLPattern without existence check in `router.js`; `reduction.js` is application code, not a dependency; `/web-static` and `/web-sprinkles` allow nothing at runtime
- files are listed, searched, and read per `/zllm-scripts` with the scripts bundled in `scripts/`; that skill owns the output format, exit codes, denials, and fallbacks
- `/web-latest` in effect (a declared support floor in the project) suppresses Newly Available findings
- BCE layering belongs to `/bce-reviewer`, performance to `/web-performance-reviewer`, rendered-page verification to `/web-static`, fixing to the stack skill

## Stack Detection

| Evidence | Stack |
|---|---|
| import map mapping `lit-html`, or `BElement.js`, `reduction.js`, `router.js` present | `web-components` |
| ES modules without lit-html; the HTML works with the scripts removed | `web-sprinkles` |
| no JavaScript | `web-static` |
| none of the above (framework, bundler) | `unknown` — reviewed against `/web-conventions` and `/javascript-conventions` with an empty allowlist: every runtime dependency is a finding; replacements name the web-components shape for applications, the web-sprinkles shape for sites |

## Modes

| Invocation | Scope |
|---|---|
| `full [path]` | whole project below `path` (default: project root) |
| `diff` | uncommitted changes: `git diff HEAD` plus untracked files (`git ls-files --others --exclude-standard`) |
| `diff <ref>` | `git diff <ref>...` plus the working tree — a branch or PR range |
| mode omitted | `diff` when the working tree is dirty, otherwise `full` |

The application root is the directory holding `index.html` (and the import map, if any). Both modes detect the stack first, then build the same dependency map over the whole application: every runtime import (bare specifiers, `node_modules` paths, CDN URLs, `<script src>`, external stylesheet links), every build and config file, the import map.

- `full` reports every finding in the map
- `diff` reports only findings the change introduces: a new import, a new config file, a new manifest entry, new hand-rolled code. A new import that depends on an unchanged file (an import map without an entry for it) is still introduced by the change. Pre-existing findings in touched files collapse to one trailing line.

Granularity: one finding per runtime dependency (location: the manifest or import map entry, else the first import), dependencies with the same replacement share a line; one finding per build concern (bundler, transpiler, CSS pipeline, manifest, CDN script), not per config file; development-only dependencies (bundler, transpiler, PostCSS, test runner) surface as build findings and stay out of the `deps:` line; a CSS framework is a `deps` finding, its PostCSS pipeline a `build` finding.

Evidence: `package.json`, lockfiles, and `node_modules` from the application root up to the repository root; `vite.config.*`, `webpack.*`, `rollup.*`, `esbuild.*`, `tsconfig.json`, `babel.config.*`, `.babelrc`, `postcss.config.*`, `tailwind.config.*`; `<script>` and `<link>` tags and the import map in `index.html`; `import … from` and `require(` across `*.js`, `*.mjs`, `*.ts`, `*.jsx`, `*.tsx`; file extensions `*.ts`, `*.tsx`, `*.jsx`, `*.scss`, `*.less`. Baseline status is looked up in the `/web-conventions` snapshot, never recalled from memory; only features the snapshot or `/javascript-conventions` list as Newly Available or Limited are checked, Widely Available features are not inventoried.

## Scripts

`scripts/` holds unmodified copies of `files`, `search`, `show`, and `imports` from https://github.com/AdamBien/zllm-scripts. Call them by absolute path, `<this skill's directory>/scripts/<name>`, from the application root; a script of the same name on the PATH is not used. Requires Java 25+.

| Evidence | Command |
|---|---|
| build and config files, manifests, lockfiles | `files package.json 'vite.config.*' 'webpack.*' 'rollup.*' 'esbuild.*' tsconfig.json 'babel.config.*' .babelrc 'postcss.config.*' 'tailwind.config.*'` |
| `node_modules` | `files -type d -all -max 1 node_modules` — ignored directories are skipped without `-all` |
| transpiled and preprocessed sources | `files -count '*.{ts,tsx,jsx,scss,less}'` |
| file count for the header | `files -count '*.{html,css,js,mjs}'` |
| runtime dependencies for the `deps:` line | `imports -external -count -package` |
| import locations, imported names, import-map targets, CDN URLs, `<script src>`, stylesheet links | `imports -names` |
| hand-rolled code | `search '<regex>' -glob '*.{js,mjs,html}'` |
| reading a finding's location | `show <file> -lines <from>-<to>` |

- `diff` mode: `git diff` and `git ls-files` determine the changed files — no script covers git; the changed files are then passed as paths to `imports`, `search`, and `show`
- vendored files in `libs/` are part of the dependency map and stay in scope; `imports libs` shows bare specifiers and sibling imports inside them
- a truncated result (`output stopped at -max N` on stderr) is narrowed or repeated with a higher `-max` before a finding count is reported
- a denied path (exit 3) is not reviewed and not retried with another tool

## Checks

A check is `✗` unless marked `?`.

Build:
- bundler or import-resolving dev server (Vite, webpack, Rollup, esbuild, Parcel) → static server with `index.html` fallback plus import map
- transpiler or type-stripping step (TypeScript, Babel, SWC, JSX) → ES modules with JSDoc types
- CSS preprocessor (Sass, Less, PostCSS) → plain CSS on design tokens
- `package.json` or `node_modules` in the application root → vendored ESM in `libs/` (web-components), nothing (web-static, web-sprinkles)
- import from a `node_modules` path or a CDN URL → import map to a local vendored file
- bare specifier without an import-map entry → import-map entry to a vendored file, or drop the import
- CommonJS, UMD, or global-script module format → ES modules
- polyfill bundle → drop the feature, or use it as its Baseline status allows

Dependencies (against the stack allowlist):
- component framework (React, Angular, Vue, Svelte) → custom elements, lit-html under web-components
- state library (Redux Toolkit installed, MobX, Zustand) → `reduction.js` or plain module state
- routing library (Vaadin Router, page.js, React Router) → Navigation API plus URLPattern
- CSS framework (Tailwind, Bulma, Bootstrap) → design tokens plus plain CSS
- utility library with a platform equivalent: lodash → array methods, `Object.groupBy()`; moment, dayjs, date-fns → `Intl`, `Date`; axios → `fetch`; uuid → `crypto.randomUUID()`; immer → `structuredClone()`; classnames → template literals
- any other runtime dependency → `?` when it ships as a single dependency-free ESM file and has no platform equivalent, otherwise `✗`
- vendored file that imports a bare specifier or a sibling file, or ships as several files → `?` signal against adoption

Standards (hand-rolled code the platform covers):
- `<div>` modal or overlay with focus trap → `<dialog>` with `showModal()`
- custom validation with a parallel error display → `reportValidity()`, `setCustomValidity()`, `:user-invalid`
- reading inputs one by one → `new FormData(form)`
- `XMLHttpRequest` or a fetch wrapper → `fetch` with `AbortSignal`; `fetch` without a `response.ok` check → add it
- hand-rolled URL or query parsing → `URL`, `URLSearchParams`, `URLPattern`
- hand-rolled deep clone → `structuredClone()`; hand-rolled pub/sub → `EventTarget` with `CustomEvent`
- JavaScript scroll hijacking → scroll snap; JavaScript breakpoint switching → container or media queries
- JavaScript show/hide of static content → `<details>`, `popover`, `hidden`
- Shadow DOM without a stated encapsulation need → `?` light DOM (web-components)
- Limited-availability feature → `✗` remove; Newly Available feature without existence check or `@supports` → `?` (suppressed under `/web-latest`)

## Output

Terminal only; no report file.

```
web-standards-reviewer full (web-components, app/src, 23 files): 4 findings
deps: lit-html ✓ · @reduxjs/toolkit (import map → reduction.js) ✓ · lodash-es ✗ · dayjs ✗
✗ build      vite.config.js            bundler resolves bare imports → static server + import map
✗ deps       orders/control/Grouping   lodash-es groupBy → Object.groupBy()
✗ deps       libs/dayjs.js             date formatting → Intl.DateTimeFormat
✗ standards  ui/Modal.js               div overlay with focus trap → <dialog>.showModal()
```

- header: mode, detected stack, path relative to the current directory (`full`) or git ref (`diff`: `HEAD` or the given ref), file count of HTML, CSS, and JavaScript files (`full`: in scope, `diff`: changed), finding count
- one `deps:` line listing every runtime dependency of the whole application in both modes, `✓` allowed by the stack skill, `✗` or `?` finding; the only line exempt from the length cap; omitted when there are none
- one line per finding: severity, category (`build|deps|standards`), location, defect → platform replacement; ~100 characters max
- severity: `✗` rule violation, `?` judgment call (justified single-file ESM dependency, Shadow DOM, Newly Available without detection)
- order: build, deps, standards — a build step makes dependency findings moot until it is gone; under a component-framework finding, `standards` findings collapse to one count line (`standards: 4 in framework code, moot until the framework is gone`) because the rewrite removes them
- `diff` adds one trailing line when touched files carry older findings: `pre-existing: 3 (run full)`
- clean is one line: `web-standards-reviewer full: clean (web-components, 1 dependency: lit-html)`, `clean (web-static, 0 dependencies)`

## Rules

- every finding names a location and the platform replacement; no finding without evidence in the code
- no prose, no explanation of why standards matter, no praise, no migration plan
- a dependency's version, license, or popularity is never a finding; only its necessity
- never present a judgment call as a violation; never contradict what the stack skill allows
- do not review BCE layering, naming, CSS style, accessibility, tests, or performance
- not a gate: never part of `/sbce`, `/continuous-testing`, `/web-system-tests`, `checks.md`, or a stack verification loop
