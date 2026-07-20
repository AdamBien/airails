# concept-annotator

An [AIrails.dev](https://airails.dev) skill that **projects the mined domain concepts into the codebase** — one `package-info.java` migration note per package. Docs only, zero code changes.

## Scope

- Writes a Java 25 `///` Markdown doc per package: concepts present (canonical term + local aliases), candidate or target BC, evidence-backed refactoring hints, and pointers to the `migration/` artifacts.
- Owns only a marked section (`## Migration Notes`); preserves existing package-info content and hand edits.
- Every section opens with `Migration notes — not an sbce spec.` — the hints must never be mistaken for a boundary contract.

Runs on the original tree or a 1:1-lifted one. Hints are candidates: never contradicts the glossary, carries hypothesis markers and Q-ids instead of resolving them.

## Composition

- [`concept-extractor`](../concept-extractor) / [`concept-clarifier`](../concept-clarifier) / [`bc-carver`](../bc-carver) — supply concepts, canonical terms, and target-BC assignments.
- [`sbce`](../../bce/sbce) — later replaces the notes with an authoritative spec in the same file.

## Usage

Invoke explicitly as `/concept-annotator`. The build must stay green — annotations are documentation. The output shape lives in [references/package-info-template.md](references/package-info-template.md).
