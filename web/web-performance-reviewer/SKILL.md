---
name: web-performance-reviewer
description: Review web frontends for performance issues by driving the rendered site through Chrome DevTools MCP — throttled performance traces, Core Web Vitals judged against thresholds, network waterfall analysis, and heap-snapshot leak checks for SPAs. Composes on top of `web-static` or `web-components`. Strictly opt-in — never part of the standard verification loop; "verify the site" and "is it green" stay with `web-static`. Use whenever a site or SPA must be measured for speed: triggers on "performance review", "review performance", "core web vitals", "LCP", "CLS", "INP", "lighthouse performance", "performance trace", "why is this slow", "is it fast", "page weight", "audit performance", "memory leak", "before publishing, check performance". Not for fixing — only for measuring, identifying, and prioritizing issues.
argument-hint: "[site root or URL to review]"
---

Review the runtime performance of $ARGUMENTS by measuring the rendered site in a real browser
through the Chrome DevTools MCP tools (`chrome-devtools`). Produce a prioritized, evidence-backed
report. If the `chrome-devtools` MCP tools are unavailable, report the review as not runnable —
do not judge performance from source code alone.

## Composition

Composes on top of `/web-static` or `/web-components` (and their shared `/web-conventions`).
This skill is a reviewer, not a builder: it measures and prioritizes; fixing belongs to the
composed stack skill, and re-verification after a fix belongs to that stack's own loop.

**Never part of green.** The `/web-static` verification loop stays fast and deterministic;
performance numbers are slow to gather and jitter run-to-run, which makes them a bad gate.
This review runs only on explicit request — before publishing, after adding images/fonts/JS,
or when something feels slow. A red finding here never turns the composed stack's green red.

## Philosophy

- **Lab, not field** — everything measured here is lab data from one Chrome on one machine;
  useful for finding problems and comparing before/after, never a claim about real-user metrics
- **Unthrottled localhost lies** — a site served by zws on localhost loads in milliseconds
  regardless of how bloated it is; only throttled runs reveal what users on mid-range phones see
- **Numbers jitter** — a single trace is an anecdote; conclusions come from the median of
  repeated runs, and high variance is itself a finding to report
- **The trace is the oracle** — never diagnose from reading source; the trace, the waterfall,
  and the heap snapshot are the evidence, source reading only explains what they found
- **If it is fast, say so** — do not invent issues; a clean report is a valid outcome

## Serve

Serve the site root with zws exactly as the composed stack prescribes: `java zws <site-root>`
(or the copy bundled with `web-conventions`: `java <web-conventions skill dir>/scripts/zws <site-root>`).
Never use `--live` during measurement — the injected reload script and its SSE stream would
appear in every trace and waterfall and contaminate the evidence. Measure against a plain instance.

## Measurement Loop

### 1. Establish lab conditions

Before any trace, throttle via `emulate`: **4× CPU slowdown** and **Slow 4G network**, with the
page resized to 375×667 (`resize_page`) — the mobile mid-range profile is the honest default.
Repeat the headline trace at 1280×800 unthrottled only as a secondary data point, never as the verdict.

### 2. Trace

Run `performance_start_trace` with a page reload, then `performance_stop_trace`. Extract from
the trace result: **LCP**, **CLS**, and **TBT** (the lab proxy for INP when no interaction is
traced). Run **at least 3 traces** and judge the median; when the spread across runs exceeds
~20% on any metric, report the variance instead of pretending precision.

### 3. Drill into insights

For each metric that misses its threshold, run `performance_analyze_insight` on the insights the
trace flags (LCP breakdown, render blocking, document latency, CLS culprits, third parties) —
the insight names the culprit element, resource, or task; put that in the finding, not a guess.

### 4. Read the waterfall

`list_network_requests` (and `get_network_request` for suspects): total transfer size, request
count, render-blocking CSS and fonts, images without modern formats (`WebP`/`AVIF`) or
`loading="lazy"`, oversized images relative to display size, missing compression, and cache
headers. This is the check that stays meaningful even when the trace looks fine on a fast machine.

### 5. Corroborate with Lighthouse

`lighthouse_audit` with the performance category, as corroboration for the trace findings —
its opportunity list often names savings in bytes and milliseconds worth quoting as evidence.
The trace remains the primary source; when the two disagree, trust the trace and say so.

### 6. SPA extras (with `/web-components` only)

- **Interaction responsiveness** — trace with recorded interactions (click through the primary
  flows via `click`/`fill` between `performance_start_trace` and `performance_stop_trace`) and
  judge INP; long tasks blocking input are the usual culprit
- **Memory leaks** — `take_heapsnapshot`, navigate the main routes several times (5+ round
  trips), snapshot again; steadily growing retained size or accumulating detached DOM nodes
  across cycles is a leak finding, one navigation's growth is not

## Thresholds

Judge the **median of ≥3 throttled runs** against the Core Web Vitals thresholds:

| Metric | Good | Poor |
|--------|------|------|
| LCP | ≤ 2.5 s | > 4.0 s |
| CLS | ≤ 0.1 | > 0.25 |
| INP (or TBT as lab proxy) | ≤ 200 ms (TBT ≤ 200 ms) | > 500 ms (TBT > 600 ms) |
| TTFB | ≤ 0.8 s | > 1.8 s |

Between good and poor is "needs improvement" — report it as such. TTFB on localhost is nearly
meaningless; report it only when it is anomalous despite the local server.

## Analysis Categories

1. **Loading (LCP)** — render-blocking stylesheets and fonts, missing preload on the LCP
   resource, image priority, document latency
2. **Layout stability (CLS)** — images and embeds without dimensions, late-loading fonts
   without `font-display`, content injected above existing content
3. **Responsiveness (INP/TBT)** — long tasks, JavaScript weight and execution time (SPA),
   work on the critical input path
4. **Network weight** — total bytes vs. what the page shows, image formats and sizing,
   compression, caching, request count
5. **Memory (SPA)** — heap growth across route cycles, detached nodes, listeners that survive
   component disconnect

## Output Format

Store the complete report as `{current-folder-name}-performance-review.md` in the project root.

````markdown
## Performance Review: {site}

### Lab Conditions
Browser, throttling, viewport, number of runs, variance observed.

### Core Web Vitals (median of N throttled runs)

| Page | LCP | CLS | TBT | Verdict |
|------|-----|-----|-----|---------|
| /    | ... | ... | ... | good / needs improvement / poor |

### Findings

| Priority | Issue | Evidence | Location |
|----------|-------|----------|----------|
| High | ... | metric + trace/waterfall fact | file or URL |

### Detailed Analysis

#### 1. [Finding Title] (Priority: High/Medium/Low)
**Evidence:** the trace insight, waterfall numbers, or heap deltas — concrete milliseconds and bytes.
**Problem:** why this hurts, tied to the metric it moves.
**Recommendation:** specific fix, referencing the composed stack's own rules where they already
prescribe it (e.g. `web-static` Performance section: `loading="lazy"`, `<picture>` + WebP, preload).

### Summary
Priority-grouped bullets, one line each — the section read first.
````

## Rules

- Do not modify the site — only report findings
- Every finding carries numeric evidence from a trace, waterfall, or snapshot — never "seems slow"
- Judge only throttled medians; label any unthrottled number as such
- Prioritize by user impact on the measured metric, not by ease of fixing
- State the lab caveat in every report: one Chrome, one machine, no field data
- When the composed stack already has the rule the fix needs, cite it instead of inventing advice

## What NOT to Do

- Do not run as part of "verify the site", "is it green", `checks.md`, or any `/sbce` /
  `/continuous-testing` convergence loop — those belong to the composed stack and must stay fast
- Do not gate or fail the composed stack's green on a performance number
- Do not draw conclusions from a single run or from unthrottled localhost loads
- Do not use `--live` mode while measuring
- Do not present lab results as Core Web Vitals field data — CrUX and RUM are different instruments
- Do not pad the report — a fast site gets a short, clean report
