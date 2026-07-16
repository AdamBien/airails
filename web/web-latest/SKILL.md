---
name: web-latest
description: Composable modifier for experimentation and PoCs that lifts the `/web-conventions` Baseline browser-support policy so the newest web platform features — Newly Available, Limited availability, even experimental or flag-gated — can be used freely, without `@supports` wrappers or fallbacks. Composes on top of `web-static` or `web-components`; only the browser-support policy changes, every other rule of the composed skills stays in force. Use whenever a prototype, proof of concept, demo, or experiment should showcase the most recent browser features. Triggers on "latest browser features", "newest CSS", "most recent web features", "bleeding edge", "cutting edge", "experimental feature", "PoC", "proof of concept", "prototype", "try out a new feature", "ignore baseline", "modern browsers only", "no fallbacks needed". Not for production sites — without this modifier, the composed skills' default Baseline policy applies.
---

Use the most recent web platform features freely in $ARGUMENTS. This is a modifier skill —
it changes exactly one thing about the composed stack: the browser-support policy.

## Purpose & Precedence

Experiments and PoCs exist to explore what the newest browsers can do; a support policy
designed for production audiences would defeat their purpose. When composed with
`/web-static` or `/web-components` (and their shared `/web-conventions`), this skill overrides:

- the **Baseline Policy** section of `/web-conventions`
- the **Baseline check** of `/web-static`'s verification loop (replaced below)
- any `@supports` or existence-check *requirements* the stack skills impose on
  Newly Available features — feature detection becomes optional

Everything else in the composed skills remains binding. Explicitly **not** relaxed:

- accessibility rules, including `prefers-reduced-motion` and `prefers-color-scheme`
- semantic HTML and heading hierarchy
- `/web-static`'s no-JavaScript constraint
- `/web-components`' dependency and architecture rules
- design-token discipline

A PoC that is inaccessible or unsemantic demonstrates nothing about the feature it showcases.

## Relaxed Policy

- any Baseline status is usable: **Widely Available**, **Newly Available**, **Limited
  availability**, and experimental or flag-gated features
- no `@supports` wrappers required, no fallbacks required, no graceful-degradation argument needed
- the view-transition exemptions carved out in `/web-static` and `/web-components` are
  subsumed by this policy — no special casing needed
- when a feature exists in several maturity stages (e.g. a spec revision only Canary ships),
  prefer the newest form that runs in at least one stable or pre-release browser

## Honesty Stays

The policy changes; the lookup discipline does not. A PoC is only useful if the viewer can
actually run it, so:

- still determine each feature's status from the `/web-conventions` bundled snapshot
  (`references/baseline-snapshot.md`); a feature missing from the snapshot is likely newer
  than it — verify against webstatus.dev or the feature's spec/release notes
- declare the **support floor** in the deliverable: a short section in the README (or a
  comment at the top of the stylesheet/entry module) listing every below-Widely feature
  used, its status, and which browser/version runs it
- for flag-gated features, name the flag and how to enable it
  (e.g. `chrome://flags/#enable-experimental-web-platform-features`)
- never present the PoC as production-ready; when an experiment graduates, drop this
  modifier and converge to the composed skill's default Baseline policy

## Verification Loop Amendment (with `/web-static`)

The verification loop runs unchanged with one substitution: the **Baseline check** becomes a
**support-floor report** — an inventory, not a gate:

- list each below-Widely feature the site uses with its status (Newly / Limited /
  experimental) and the browsers that run it; confirm the declared support floor matches
- all other standard checks (console, structure, responsive, preferences, Lighthouse) and
  every `checks.md` line still apply and still define **green**
- if a feature does not run in the DevTools Chrome, that is still red — unless the
  support-floor declaration documents it as needing a flag or a different browser, in which
  case report it as "not verifiable in this browser" rather than silently passing it

## What NOT to Do

- do not use this modifier for production sites — it exists for experiments, PoCs, and demos
- do not drop accessibility, reduced-motion, or semantic-HTML rules because "it's just a PoC"
- do not skip the support-floor declaration — an unrunnable demo is a failed demo
- do not claim Baseline compliance in any report while this modifier is active
- do not invent feature support from memory — cite the snapshot or verify upstream
