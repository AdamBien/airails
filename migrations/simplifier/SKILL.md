---
name: simplifier
description: Replatform mode ("lift, tinker, and shift") — simplify a 1:1-lifted legacy system in place: modernize code and remove external dependencies, one pass at a time, each bracketed by characterization-tests replays, structure preserved. Owns the stack-neutral pass discipline; the transformation catalog and dependency ladder come from the composed stack ecosystem (Java: java-distiller, Java SE → MicroProfile/Jakarta EE → copied source; web: web-conventions/web-components, web platform first → vendored source). A legitimate terminal state, or preparation for concept-extractor. Invoke explicitly as /simplifier. Not a rearchitecture — structural moves belong to bc-carver and sbce.
disable-model-invocation: true
---

# Simplifier

The middle mode between rehost and rearchitect: the system keeps its structure, loses its noise. Every pass is semantics-preserving — and proven so, not assumed: the characterization suite is the green bar every pass must hold. This skill owns the pass discipline; what to transform and what replaces a dependency comes from the composed stack ecosystem.

## Preconditions

- The system runs on the target platform (1:1 lift completed).
- Characterization recordings exist and replay green. No green bar, no simplification — run `/characterization-tests` first.

## Pass Loop

1. Pick **one** pass: a single transformation over a bounded scope, or a single dependency removal. Never batch.
2. Apply it.
3. Verify with the stack's own build/check loop, then replay the characterization suite.
4. **Green** → commit the pass. **Diff** → reconcile (an accepted behavior change is a human decision, per characterization-tests) or revert the pass entirely.
5. Log the pass in `migration/SIMPLIFICATION.md` — one line: date, pass, result.
6. Repeat until the catalog finds no high-impact transformation and the dependency list is settled.

## Code Passes

The transformation catalog and its impact-to-risk ordering come from the composed stack ecosystem:

- **Java** — java-distiller: modern syntax, API upgrades, pattern adoption, dead-code removal.
- **Web frontend** — web-conventions + web-components: semantic HTML, modern CSS over framework classes, custom elements over framework widgets, standards-based routing and state.

Constraint on top, any stack: preserve the public surface and the structure — packages, components, routes stay where they are. Internals may simplify, dead code may go; moves, splits, and merges are carving, and carving is `/bc-carver` + `/sbce` territory.

## Dependency Passes

One dependency per pass. The ladder shape is stack-neutral — platform first, then copied source, then keep with justification; the rungs are the stack's:

| Stack | Ladder | Manifest |
|---|---|---|
| Java | Java SE → MicroProfile/Jakarta EE → copied source (zjson pattern) | `pom.xml` |
| Web frontend | Web platform, Baseline (`fetch`, `querySelector`, custom elements, modern CSS) → vendored source | `package.json`, script tags |

- Stop at the first rung that fits; keep only with a one-line justification in `migration/SIMPLIFICATION.md` under `## Kept Dependencies`.
- Remove the dependency from the manifest in the same pass — replaced but still declared is an unfinished pass.

## Rules

- The green bar is law: no pass survives a red replay unreconciled.
- Semantics shift in disguise — `java.time` timezone handling, stream ordering, third-party null handling; jQuery event semantics vs `addEventListener`, framework lifecycle vs custom-element callbacks, CSS cascade changes. That is why the bracket exists; trust the replay, not the reviewer's confidence.
- Stopping here is legitimate: a distilled, dependency-free system on a modern platform is a valid migration outcome.
- Continuing is better prepared: simplified code is cleaner input for `/concept-extractor` — half the naming noise the extractor fights is what these passes remove.
