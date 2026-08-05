---
name: javascript-conventions
description: Generic, composable JavaScript (ECMAScript) code conventions — modules, declarations, functions, collections, asynchrony, errors, JSDoc types, naming, and the Baseline lookup that decides which language and standard-library features may be used. Technology-neutral within the JavaScript world; meant to be composed with context-specific skills (e.g. `web-components`) alongside `web-conventions` (HTML/CSS). Use when writing, generating, or reviewing JavaScript anywhere the composed skill does not already specify style. Triggers on "JavaScript conventions", "JS style", "modern JavaScript", "ECMAScript", "ES2025", "idiomatic JavaScript", "ES modules", "JSDoc types", "async/await style", "which JS feature may I use", or any request to write or review JavaScript where context-specific skills do not already cover style. Not for HTML or CSS rules — see `web-conventions`.
---

Apply all rules below strictly to any JavaScript you write, generate, or review.

## Scope

- Language-level rules for JavaScript only — modules, syntax, functions, data, asynchrony, errors, JSDoc types, naming, comments.
- HTML, CSS, accessibility, and design tokens are **not** in this skill — see `web-conventions`.
- Architecture, state management, routing, dependency and build policy, project layout, dev server, and verification loops are **not** in this skill — see `web-components` (SPAs) and `web-sprinkles` (static sites with bounded enhancements); `web-static` forbids JavaScript entirely.
- When a composed skill specifies a rule, the composed skill wins; this skill is the fallback baseline.

## Language Level

- target the newest ECMAScript syntax and standard-library APIs the Baseline policy permits — the decision is **per feature**, never per year: there is no "ES2020 target" to configure, only a feature's Baseline status
- no build step to reach a language feature — no transpiler (Babel, SWC, esbuild), no polyfill bundle; if a feature needs compiling, its Baseline status already says not to use it
- no TypeScript, no JSX — plain ES modules with JSDoc type annotations
- modules are strict mode already — never write `'use strict'`

## Baseline Lookup for JavaScript

The `web-conventions` Baseline policy governs JavaScript exactly as it governs CSS:

- **Widely Available** — use freely
- **Newly Available** — requires feature detection with a graceful fallback
- **Limited availability** — must not be used

CSS detects with `@supports`; JavaScript's equivalent is an existence check before first use:

```javascript
const groupBy = Object.groupBy ?? ((items, key) => items.reduce(intoGroups(key), {}));
if (!document.startViewTransition) return render(url);
if (!('URLPattern' in globalThis)) return matchWithRegExp(url);
```

- look up the feature id in `web-conventions/references/baseline-snapshot.md` — **never from model memory**; widely-known features can still be Limited (top-level await is, in the 2026-07-08 snapshot)
- a feature missing from the snapshot or newer than its header date is uncertain — verify against [webstatus.dev](https://api.webstatus.dev/v1/features) before recommending it, or treat it as Limited
- core syntax may have no feature id at all — optional chaining (`?.`) returns 404 from the API (verified 2026-08-04), meaning untracked, not Limited; judge such syntax on its own rather than by lookup
- when recommending a newer API, cite its status so the reader can judge support

### Widely Available — use freely

Statuses from the 2026-07-08 snapshot, with the date each became Widely Available.

| Feature id | Feature | Since |
|---|---|---|
| `let-const` | `const` / `let` | 2019-03-20 |
| `js-modules` | ES modules | 2020-11-09 |
| `destructuring`, `spread`, `template-literals` | destructuring, spread, template literals | 2022-07-15 |
| `nullish-coalescing`, `logical-assignments` | `??`, `??=`, `\|\|=`, `&&=` | 2023-03-16 |
| `optional-catch-binding` | `catch { }` without a binding | 2022-07-15 |
| `numeric-separators` | `86_400_000` | 2023-01-28 |
| `async-await`, `async-iterators` | `async`/`await`, `for await…of` | 2019-10-05 / 2022-07-15 |
| `promise-allsettled`, `promise-any` | `Promise.allSettled()`, `Promise.any()` | 2023-01-28 / 2023-03-16 |
| `aborting`, `abortable-fetch` | `AbortController`, abortable `fetch` | 2021-09-25 |
| `array-flat`, `array-at`, `array-findlast` | `flat()`/`flatMap()`, `at()`, `findLast()` | 2022-07-15 / 2024-09-14 / 2025-02-23 |
| `array-by-copy` | `toSorted()`, `toReversed()`, `toSpliced()`, `with()` | 2026-01-04 |
| `object-hasown` | `Object.hasOwn()` | 2024-09-14 |
| `structured-clone` | `structuredClone()` | 2024-09-14 |
| `error-cause` | `new Error(msg, { cause })` | 2024-03-20 |
| `string-replaceall`, `string-at`, `string-wellformed` | `replaceAll()`, `at()`, `isWellFormed()` | 2023-02-27 / 2024-09-14 / 2026-04-24 |
| `url`, `url-canparse` | `URL`, `URL.canParse()` | 2015-07-29 / 2026-06-07 |
| `intl` | `Intl` formatting | 2020-03-28 |
| `import-maps` | import maps | 2025-09-27 |
| `queuemicrotask`, `broadcast-channel` | `queueMicrotask()`, `BroadcastChannel` | 2023-01-28 / 2024-09-14 |

### Newly Available — existence check plus fallback

| Feature id | Feature | Newly since |
|---|---|---|
| `array-fromasync` | `Array.fromAsync()` | 2024-01-25 |
| `abortsignal-any`, `abortsignal-timeout` | `AbortSignal.any()`, `AbortSignal.timeout()` | 2024-03-19 / 2024-04-18 |
| `array-group` | `Object.groupBy()`, `Map.groupBy()` | 2024-03-05 |
| `promise-withresolvers`, `promise-try` | `Promise.withResolvers()`, `Promise.try()` | 2024-03-05 / 2025-01-07 |
| `set-methods` | `union()`, `intersection()`, `difference()`, `isSubsetOf()` | 2024-06-11 |
| `intl-segmenter`, `intl-duration-format` | `Intl.Segmenter`, `Intl.DurationFormat` | 2024-04-16 / 2025-03-04 |
| `iterator-methods`, `iterator-concat` | iterator helpers, `Iterator.concat()` | 2025-03-31 / 2026-03-24 |
| `json-modules` | `import data from "./d.json" with { type: "json" }` | 2025-04-29 |
| `regexp-escape` | `RegExp.escape()` | 2025-05-01 |
| `urlpattern`, `navigation` | `URLPattern`, Navigation API | 2025-09-15 / 2026-01-13 |
| `uint8array-base64-hex` | `Uint8Array.fromBase64()` / `toBase64()` | 2025-09-05 |
| `getorinsert` | `Map.getOrInsert()` | 2026-02-14 |

### Limited — do not use

| Feature id | Feature | Use instead |
|---|---|---|
| `top-level-await` | `await` at module top level | export an `async` init function the entry point calls |
| `explicit-resource-management` | `using` / `await using` | explicit `try` / `finally` |
| `temporal` | `Temporal` | `Date` plus `Intl` formatting |
| `import-assertions` | `assert { type: "json" }` | `json-modules` `with { type: "json" }` (Newly) or `fetch().then(r => r.json())` |
| `observable` | `Observable` | `addEventListener` |
| `requestidlecallback`, `scheduler` | `requestIdleCallback()`, `scheduler.postTask()` | `queueMicrotask()` or `setTimeout()` |
| `proto` | `__proto__` | `Object.getPrototypeOf()` / `Object.create()` |
| `regexp-static-properties` | `RegExp.$1` | named capture groups on the match result |

## Modules

- one ES module per file — `import` / `export` only; never CommonJS (`require`, `module.exports`), UMD, or globals
- named exports by default; a default export only when the module *is* the single thing it exports (a class, an element)
- resolve bare specifiers through an import map — never a relative path into `node_modules`
- keep imports at the top of the file; use dynamic `import()` only for code that is genuinely conditional
- importing a module must have no side effects beyond registration (`customElements.define`) — never fetch, write storage, or touch the DOM at import time
- no top-level `await` (Limited) — export an `async init()` and call it from the entry point

## Declarations & Syntax

- `const` by default, `let` only where reassignment is real — never `var`
- `===` / `!==` only — never `==`; test absence explicitly (`=== undefined`, `=== null`)
- template literals over string concatenation
- destructure in the parameter list to name exactly what a function needs — `({ target: { name, value } })`
- optional chaining `?.` and `??` over `&&` guard chains and `||` defaults — `||` also swallows `0` and `""`
- `??=` / `||=` / `&&=` for conditional assignment
- spread for copying and merging over `Object.assign`
- numeric separators for long literals — `86_400_000`, not `86400000`
- `for…of` when a loop is genuinely unavoidable — never `for (let i = 0; …)` over an iterable
- never `eval`, `with`, or implicit globals

## Functions

- arrow functions by default; `function` only when `this`, `arguments`, or hoisting is genuinely required
- keep inline callbacks — array operations, template event bindings — to a single expression; extract a multi-statement body into a named function
- pass named functions directly (`items.map(toLabel)`) — never wrap one in a redundant arrow
- never pass a multi-parameter builtin as a callback — `["1", "10"].map(parseInt)` does not do what it reads like; `map(Number)` does
- default parameter values over `||` fallbacks in the body; rest parameters over `arguments`
- name a deliberately unused parameter `_`
- extract complex boolean conditions into named predicates — `isEligible(user)`, not an inlined `&&` chain
- guard clauses (early returns) over nested `if` / `else`
- keep functions short, cohesive, and independently testable

## Classes & Objects

- prefer plain functions and object literals — use a class only where the platform requires one (custom elements) or identity and behavior genuinely belong together
- avoid `#private` fields: module scope already encapsulates, and `#` blocks same-module tests from reading state without a workaround; reserve it for genuinely sensitive values
- no getter/setter pairs that only pass a value through
- prefer composition over inheritance; never build a deep class hierarchy
- avoid meaningless suffixes: `*Impl`, `*Service`, `*Manager`, `*Helper`, `*Utils`
- avoid the `get` prefix — `configuration()`, not `getConfiguration()`

## Data & Collections

- treat data as immutable: `toSorted()`, `toReversed()`, `toSpliced()`, `with()` (Widely since 2026-01-04) over the mutating `sort()`, `reverse()`, `splice()`
- `structuredClone()` (Widely since 2024-09-14) over `JSON.parse(JSON.stringify(…))`
- array methods over loops — `map`, `filter`, `reduce`, `find`, `findLast`, `flatMap`, `some`, `every`
- avoid `forEach` — prefer operations that return a value; a `forEach` body is usually a `map`, `filter`, or `reduce` in disguise
- `at(-1)` over `array[array.length - 1]`
- `Map` over object literals for dynamic keyed lookups; `Set` over arrays for membership tests
- `Object.entries()` / `Object.fromEntries()` for transforming records
- `Object.hasOwn(target, key)` over `target.hasOwnProperty(key)`
- return empty arrays and objects, never `null` or `undefined`, from functions producing collections
- do not put `null` into collections
- mutate only values you own — never a parameter, a caller's object, or shared state; copy first with `structuredClone()`, spread, or `with()`

## Asynchrony

- `async` / `await` over `.then()` chains
- run independent work concurrently — `Promise.all` for all-or-nothing, `Promise.allSettled` when partial failure is acceptable
- never `await` inside a loop over independent items — collect the promises, then `await Promise.all(…)`
- pass an `AbortSignal` to every `fetch` a user action can cancel or supersede
- never leave a promise floating — `await` it or attach a rejection handler; an unhandled rejection is a silent failure
- no `new Promise(…)` wrapped around something that already returns a promise

## Errors

- `throw new Error("what failed and with which input")` — never throw a string or a plain object
- subclass `Error` only when callers branch on the type
- chain with `cause` (Widely since 2024-03-20): `new Error("loading failed", { cause })` — never string-concatenate the original message
- never swallow an error — log with context or rethrow wrapped
- `catch { }` (optional catch binding) when the binding is genuinely unused — never `catch (e) { }` with an ignored `e`
- check `response.ok` after every `fetch` — `fetch` rejects on network errors only, never on 4xx or 5xx

## Types & Documentation

- JSDoc carries the types that plain ES modules cannot declare
- every exported function declares `@param {Type} name` and `@returns {Type}`
- describe entity and state shapes once with `@typedef` and reference them from everywhere else
- keep JSDoc type-focused — prose only for intent the types cannot express
- default to no comments; comment only where the *why* is non-obvious: a hidden constraint, a workaround, behavior that would surprise a reader
- never explain *what* the code does — well-named identifiers do that

## Naming

- `camelCase` for functions and variables, `PascalCase` for classes and the files exporting them, `UPPER_SNAKE_CASE` for module-level constants that are genuinely fixed
- name modules and functions after their responsibility, not a technical concern
- boolean names read as predicates — `isEmpty`, `hasItems`, `canSubmit`
- name handlers after what happens, not after the event — `deleteItem`, not `onClick`

## Platform APIs First

- reach for the platform before any dependency — every dependency must justify its existence
- `URL` and `URLSearchParams` over string surgery on paths and query strings; `URL.canParse()` (Widely since 2026-06-07) over `try` / `catch` around `new URL(…)`
- `Intl` for all number, currency, date, list, and relative-time formatting — never hand-rolled formatting
- `Date` plus `Intl` for dates — `Temporal` is Limited
- `fetch` with `AbortSignal` for HTTP; `JSON.parse` / `JSON.stringify` for JSON
- `structuredClone()` for deep copies, `queueMicrotask()` for deferring
- no lodash, underscore, moment, date-fns, or axios — array methods, `Intl`, and `fetch` already cover them

## Code Style

- KISS and YAGNI — implement the simplest solution that works; never add extension points on speculation
- prefer several simple lines over one dense chain; name an intermediate value when a chain stops reading as intent
- inline single-use variables assigned and consumed on the next line
- extract repeated string literals into named constants — define once, change once
- separate side effects from conditions — do the work, then branch on the result
- terminate statements with semicolons; indent with 4 spaces
- no dead code, no commented-out code, no leftover `console.log` — `console.error` only for genuine failures

## Testing

- test user-visible behavior through the public surface, never internals
- create minimal tests first; no tests for code that cannot fail
- at most three tests per unit under test
- the framework, layout, and run command come from the composed stack skill

## What NOT to Do

- do not use TypeScript, JSX, or any syntax requiring a transpiler
- do not add a bundler, transpiler, or polyfill to reach a newer language feature
- do not use CommonJS, `var`, `==`, `eval`, `with`, or `__proto__`
- do not use top-level `await`, `using` declarations, or `Temporal` — all Limited
- do not guess a feature's Baseline status from memory — look it up in the snapshot
- do not use a Newly Available API without an existence check and a fallback
- do not swallow errors or leave promises floating
- do not mutate parameters, shared state, or arrays in place — copy, then change the copy
- do not reimplement platform APIs — URL parsing, formatting, deep cloning are already there

## Composition with Other Skills

- this skill defines only language-level JavaScript conventions; HTML, CSS, accessibility, and design tokens come from `web-conventions`, and architecture, state management, routing, dependency policy, project layout, dev server, and verification come from the stack skill (`web-components`, `web-sprinkles`) and `bce`
- the Baseline snapshot is bundled with `web-conventions` (`references/baseline-snapshot.md`); this skill applies that policy to JavaScript syntax and APIs
- `web-latest` overrides the Baseline policy for experiments and PoCs — any status becomes usable, with a declared support floor instead of fallbacks
- when a composed skill adds or refines a rule, apply it on top of these; the composed skill always specializes, never contradicts
- if a composed skill is silent on a topic covered here, these rules apply by default
