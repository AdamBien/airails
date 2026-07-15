# web-performance-reviewer

Reviews web frontends for performance issues by measuring the rendered site in a real browser through Chrome DevTools MCP: performance traces under 4× CPU / Slow 4G throttling, Core Web Vitals judged as the median of repeated runs against the good/poor thresholds, network waterfall analysis, and heap-snapshot leak checks for SPAs. Produces a prioritized, evidence-backed report — it measures and prioritizes, it never fixes.

## Composition

Composes on top of [web-static](../web-static) or [web-components](../web-components) and their shared [web-conventions](../web-conventions). It is strictly opt-in: the `web-static` verification loop stays fast and deterministic and deliberately excludes performance — noisy, slow measurements make a bad gate. This review runs on explicit request (before publishing, after adding images/fonts/JS, when something feels slow), and a red finding here never turns the composed stack's green red.

## Lab, Not Field

All results are lab data from one Chrome on one machine — useful for finding problems and comparing before/after, never a claim about real-user metrics. Unthrottled localhost numbers are meaningless by design; only throttled runs reveal what users on mid-range devices experience.

## Usage

`SKILL.md` contains the full measurement loop. Apply it together with a stack skill — it reviews what the stack built.
