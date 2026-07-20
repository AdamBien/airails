---
name: bc-carver
description: Carve candidate business components from the confirmed domain concepts — cluster by CONCEPTS.md co-occurrence, name each BC with a canonical glossary term — and document the as-is → to-be diff in migration/CARVING.md without changing any code. Step between concept-clarifier and sbce in a legacy-to-BCE migration; the diff is the backlog for incremental (strangler) refactoring. Invoke explicitly as /bc-carver in the analyzed project. Never restructures code — /sbce and the composed stack skill execute the carving.
disable-model-invocation: true
---

# BC Carver

Decide the decomposition, don't perform it. Carving and executing are different acts: the carver produces a written, reviewable cut of the system into business components — the diff a human argues about *before* the first class moves. Candidates, not verdicts.

## Inputs

- `migration/GLOSSARY.md` — canonical terms and `## Decisions`: ground truth, never contradicted.
- `migration/CONCEPTS.md` — co-occurrence graph and evidence pointers.

No mining, no new concepts. Vocabulary missing here is an extractor defect — stop and re-run `/concept-extractor`, don't improvise.

## Carving Heuristics

- BCs are the dense clusters in the co-occurrence graph; cut along sparse edges.
- Homonym splits settled by the clarifier are the strongest boundary signals — a word that means two things marks two components.
- Name each BC with one canonical glossary term as a single lowercase token (`contracts`, `pricing`) — this is the future sbce BC name.
- Give each BC a one-line responsibility phrased in glossary language. If the line needs "and", reconsider the cut.
- Inter-BC references are directed (`uses`); cycles between candidates are a carving error, not a documentation fact.

## CARVING.md Format

Write `migration/CARVING.md` — regenerable, diffable:

```markdown
# Carving: <system name>

## Components
### <bc-name>
- **Responsibility:** <one line, glossary language>
- **Concepts:** <canonical terms>
- **Uses:** <bc-name>, ...

## Diff
| Current | Target BC | Layer |
|---|---|---|
| com.legacy.contractmgmt | contracts | — |
| com.legacy.common.ContractValidator | contracts | control |

## Unmapped
<technical noise, dead-code candidates, shared utilities — with reasons>

## Open Questions
### Q<n> — <one-line question>
<evidence; candidate assignments>
```

- **Diff granularity:** package-level rows by default; class-level rows only where a package splits across BCs — that is where the information matters, and full class listings on large systems are enormous and instantly stale.
- **Layer** (boundary | control | entity) only where the evidence supports it; leave `—` otherwise.
- **Unmapped is mandatory** — silent omission reads as "covered".
- **Open Questions** continue the shared Q-id space from `CONCEPTS.md`: never reuse or renumber ids, continue counting. Ambiguous assignments ("Tariff → contracts or pricing?") are questions, not guesses — resolve via `/concept-clarifier`.

## Rules

- Documentation only — zero code changes; `/sbce` + the stack skill execute the carving, one BC at a time.
- A human confirms the cut before anything downstream consumes it.
- Respect `GLOSSARY.md` `## Decisions` on re-runs: resolved assignments stay resolved.
- Never overwrite a human-edited `CARVING.md` — regenerate to `migration/CARVING.new.md` (same convention as `CONCEPTS.md`).
