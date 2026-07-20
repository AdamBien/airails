---
name: migration-advisor
description: Front door for legacy migrations — assess a legacy system from shallow read-only signals (repo probe, git history, a reachable running instance) plus a short interview, then recommend an entry path (rehost, replatform, rearchitect) and an ordered sequence of migration skills to invoke. Produces migration/PLAN.md with its evidence shown; routes, never executes. Use when someone wants to migrate, modernize, or port a legacy enterprise Java or web application and needs to know where to start. Triggers on "migrate this legacy app", "how do I modernize this", "port this old system", "where do I start with this legacy codebase", "plan a migration".
---

# Migration Advisor

The triage front door. Legacy migration is judgment-heavy and the entry-path choice is a human decision — this skill gathers the cheap signals that inform that choice, recommends a path and a sequence, and hands off. It **routes, never executes**: every step it recommends is invoked explicitly by a human, and each produces a reviewable artifact.

Stay shallow. The value is finishing in under a minute on a repo you have never seen. Deep vocabulary mining is `/concept-extractor`; behavior capture is `/characterization-tests`. Those are steps *in* the plan, not prerequisites to producing it.

## Data Sources

Use whatever is present; state in the plan which were available — a plan built without git or a runnable instance is more heuristic, and must say so.

### Repo probe (always)

Read-only counts and pattern hits, never comprehension:

- **Platform / age** — build files: `javax.*` vs `jakarta.*`, servlet/EJB/JSF/Spring versions, target JDK; or `package.json` with jQuery/AngularJS/Backbone.
- **Size / shape** — module, package, and file counts; LOC order of magnitude; density of `*Manager`/`*Impl`/`*Util`/`*Facade` names (overengineering signal → concept pipeline pays off; small and clean → rearchitect directly).
- **External dependencies** — count and identity of libraries with Java SE or platform equivalents (the replatform/simplifier signal).
- **Test coverage** — presence and size of a test source tree (near-zero → characterization must come first; "1:1 lift" is risky).
- **Surfaces** — JAX-RS resources, `web.xml` mappings, queue config, batch jobs; whether a runnable artifact exists (is characterization feasible, on which surface).

### Git history (if `.git` present)

Aggregate `git log` / `shortlog` / `--numstat`, not per-file blame:

- **Liveness** — last commit date, commit frequency: frozen (experts likely gone → lean on recorded behavior and async questionnaires) vs. active (strangle alongside ongoing work).
- **Bus factor** — `git shortlog -sne`: contributor count and whether the top authors are recent. Pre-fills the interview's expert-availability question with names.
- **Hotspots** — churn per file/package: where behavior is volatile and characterization should concentrate.
- **Temporal coupling** — files that change together: empirical evidence that latent structure exists and the carve path will pay off. Note it; do not carve — that is `/bc-carver`.
- **Code age** — old untouched code is safe to lift; recently churned code is where lift risk concentrates.

Distrust the signal when history is shallow-cloned, squashed/rebased, dominated by reformatting or dependency-bump commits, or broken by large file moves. Say so rather than presenting distorted churn as fact.

### Running instance (if reachable)

Confirm a surface responds; note base URL and health. Do not exercise it — that is characterization's job.

## Interview

The routing-critical facts live in people's heads. Ask 3–4 enumerable questions with `AskUserQuestion`, **pre-filled from the probes** so the human confirms or corrects rather than starts blank:

- Must the system keep running during migration? → rehost/replatform first vs. rearchitect directly.
- Is a domain expert available? (offer the git bus-factor names) → concept path viability; live vs. async clarification.
- Goal: clean modern runtime, or full BCE? → replatform-terminal vs. carve to sbce.
- Platform EOL / compliance deadline? → urgency; favors lift-first.

## Output

Write `migration/PLAN.md`:

- **Findings** — the gathered signals, each with its source and the caveats that apply. Spot-checkable, never trust-based.
- **Recommended path** — rehost, replatform, or rearchitect — with the *reasoning* (which signals drove it), so a human who knows the system can override a bad inference.
- **Sequence** — the ordered skills to invoke (`/characterization-tests record` → … ), with steps marked skippable and why, and the decision points that need a human call flagged.
- **Reference, don't restate** — link each step to its own skill README for detail; keep the plan thin so it does not drift from the skills.

The plan is a recommendation with its evidence shown, not a verdict — candidates-not-verdicts applied to the routing decision itself. The human edits it before step one.

## Entry Paths

- **Rehost** — 1:1 lift onto the target runtime, nothing else. Keeps running through a long migration or under EOL pressure.
- **Replatform** — lift, then `/simplifier` passes under the characterization green bar. A valid terminal state, or preparation for the concept pipeline.
- **Rearchitect** — recover vocabulary (`/concept-extractor` → `/concept-clarifier`), carve (`/bc-carver`), converge to specs (`/sbce` + stack skill). For overengineered systems where the structure hides the domain.
