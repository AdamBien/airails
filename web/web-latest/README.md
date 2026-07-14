# web-latest

Composable modifier for experiments, prototypes, and PoCs: lifts the Baseline browser-support policy of the composed stack so the newest web platform features — Newly Available, Limited availability, even experimental or flag-gated — can be used without `@supports` wrappers or fallbacks. Not for production sites.

## Composition

Composes on top of [web-static](../web-static) or [web-components](../web-components) and their shared [web-conventions](../web-conventions). It overrides exactly one concern — the browser-support policy:

- the `web-conventions` Baseline Policy
- the Baseline check in the `web-static` verification loop, replaced by a support-floor report (an inventory, not a gate)
- any `@supports`/feature-detection requirements on Newly Available features

Everything else stays binding: accessibility (including `prefers-reduced-motion` and `prefers-color-scheme`), semantic HTML, the `web-static` no-JavaScript constraint, dependency rules, and design-token discipline.

## Support Floor Instead of Fallbacks

Feature statuses are still looked up in the `web-conventions` [baseline snapshot](../web-conventions/references/baseline-snapshot.md), never guessed from model memory. Every deliverable declares its support floor: each below-Widely feature, its status, and the browsers/versions (or flags) that run it. When an experiment graduates to production, drop this modifier and converge to the default Baseline policy.

## Usage

`SKILL.md` contains the full modifier rules. Apply it together with a stack skill — it does nothing on its own.
