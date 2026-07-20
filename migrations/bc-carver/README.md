# bc-carver

An [AIrails.dev](https://airails.dev) skill that **carves candidate business components from the confirmed domain concepts and documents the as-is → to-be diff** — without changing any code.

## Scope

- Clusters concepts by `CONCEPTS.md` co-occurrence into candidate BCs; settled homonym splits are the strongest boundary signals.
- Names each BC with one canonical glossary term (a single lowercase token — the future sbce BC name) and a one-line responsibility.
- Writes `migration/CARVING.md`: the components, the `current location → target BC / layer` diff, an explicit `## Unmapped` bucket, and its own `## Open Questions`.

The diff is the backlog for **incremental (strangler) refactoring**. Candidates, not verdicts — a human confirms the cut before anything downstream runs.

## Composition

- [`concept-extractor`](../concept-extractor) / [`concept-clarifier`](../concept-clarifier) — supply the confirmed concepts and co-occurrence graph.
- [`sbce`](../../bce/sbce) + a stack skill — execute the carving, one BC at a time; `bc-carver` never restructures code.

## Usage

Invoke explicitly as `/bc-carver` in the analyzed project. Open questions continue the shared Q-id space and resolve through `/concept-clarifier`. Re-runs respect glossary `## Decisions` and a human-edited `CARVING.md` (regenerate to `CARVING.new.md`).
