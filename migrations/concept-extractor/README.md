# concept-extractor

An [AIrails.dev](https://airails.dev) skill that **mines candidate domain concepts and their vocabulary from every naming source of a legacy system** — the first step of a legacy-to-BCE migration.

## Scope

- Mines every naming source, ranked by trustworthiness: database schema, UI labels, external contracts (REST/URL, queues), configuration, code identifiers, documentation.
- Strips pattern noise (`Manager`, `Impl`, `Facade`, …) to expose the domain nucleus; clusters aliases (`Customer` / `Client` / `Kunde` / `CUST`) into one concept.
- Records evidence, frequency, confidence, and co-occurrence for every concept.
- Writes `migration/CONCEPTS.md` (regenerable) and seeds `migration/GLOSSARY.md` (human-owned).

Produces **candidates for human review, never verdicts**. Does not carve business components — that is `bc-carver`.

## Composition

- [`concept-clarifier`](../concept-clarifier) — resolves the `## Open Questions` (by Q-id) this skill emits.
- [`bc-carver`](../bc-carver) and [`sbce`](../../bce/sbce) — downstream consumers of the confirmed vocabulary.

## Usage

Invoke explicitly as `/concept-extractor` on the system to analyze. All output lands in the analyzed project's `migration/` folder. Re-runs respect a human-edited `CONCEPTS.md` (regenerate to `CONCEPTS.new.md`) and treat confirmed glossary entries as ground truth. The noise-word list lives in [references/noise-words.md](references/noise-words.md).
