# migration-advisor

An [AIrails.dev](https://airails.dev) skill that is the **front door for legacy migrations** — assesses the system, recommends an entry path and an ordered sequence of skills, and hands off. It routes; it never executes.

## Scope

- Gathers **shallow, read-only signals** from three optional sources: a repo probe (always), git history (if present), and a reachable running instance (if any).
- Runs a short interview (`AskUserQuestion`), pre-filled from the probes, for the routing-critical facts code cannot show — must it keep running, is a domain expert available, is the goal a clean runtime or full BCE, is there a deadline.
- Writes `migration/PLAN.md`: findings with evidence and caveats, a recommended path with its reasoning, and the skill sequence to invoke.

Stays under a minute. Deep vocabulary mining and behavior capture are steps in the plan, not prerequisites — `/concept-extractor` and `/characterization-tests` own those.

## Composition

- Recommends and sequences every other skill in this category — [`characterization-tests`](../characterization-tests), [`simplifier`](../simplifier), [`concept-extractor`](../concept-extractor), [`concept-clarifier`](../concept-clarifier), [`bc-carver`](../bc-carver), [`concept-annotator`](../concept-annotator) — plus [`sbce`](../../bce/sbce) and a stack skill for convergence.
- Unlike its siblings it is **model-invocable**: it is the entry point, meant to trigger on "help me migrate this legacy app," where the pipeline steps are explicitly invoked within a chosen plan.

## Usage

Invoke as `/migration-advisor`, or let it trigger from a migration request. The output is a recommendation with its evidence shown — a human edits `migration/PLAN.md` before invoking the first step.
