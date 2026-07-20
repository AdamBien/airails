---
name: concept-annotator
description: Project the mined domain concepts into the codebase — write one package-info.java (Java 25 /// Markdown doc) per legacy package with the concepts living there, candidate-BC leaning, and refactoring hints, consuming migration/CONCEPTS.md, GLOSSARY.md, and CARVING.md when present. Docs only, zero code changes; runs on the original tree or a 1:1-lifted one. Composes with concept-extractor, concept-clarifier, and bc-carver. Invoke explicitly as /concept-annotator. Not an sbce spec author — it seeds migration notes, never boundary contracts.
disable-model-invocation: true
---

# Concept Annotator

Carry the captured domain knowledge into the code itself. Every legacy package becomes self-describing — which concepts live here, where the package is heading, what the naming evidence suggests — in the exact file where the future sbce spec will live. A developer opening the package sees the migration state without leaving the IDE; an annotated system cannot quietly pretend the migration is done.

## Inputs

- `migration/GLOSSARY.md` — canonical terms and `## Decisions`: ground truth.
- `migration/CONCEPTS.md` — evidence pointers mapping concepts to packages and classes.
- `migration/CARVING.md` — when present, supplies target BC and moves per package.

Prefer running after `/concept-clarifier`; if glossary entries are still hypothesis-marked, carry the marker into the annotation verbatim — never launder a hypothesis into a fact.

## Workflow

1. List the Java packages of the source tree.
2. Map concepts to packages via the CONCEPTS.md evidence pointers; note the local aliases per package.
3. Per package, write `package-info.java` — or update only the owned section of an existing one — from [references/package-info-template.md](references/package-info-template.md).
4. Verify the build stays green: annotations are docs, never behavior.

## Annotation Content

Per package, inside the owned section:

- **Concepts** — canonical term plus the aliases as they appear locally (`Contract — here: Cntrct, VTRG`).
- **Candidate BC** — single lowercase glossary token, or `unassigned`. With `CARVING.md` present: **Target BC** plus the move rows affecting this package.
- **Refactoring hints** — evidence-backed suggestions: mixed concepts → split candidate, names to rename to canonical terms, apparent boundary/control/entity leaning per class.
- **See** — pointers to the `migration/` artifacts and open Q-ids touching this package.

## Rules

- Docs only — never touch code, imports, or resources. The smallest possible change to a runnable system is no change.
- Own only the marked section (`## Migration Notes (concept-annotator)`). Preserve existing package-info content and hand edits outside the marker; re-runs regenerate only the owned section.
- Open every generated section with the disclaimer `Migration notes — not an sbce spec.` — sbce treats package docs as authoritative boundary contracts, and these hints must never be mistaken for one. `/sbce` later replaces the notes with a real spec.
- Hints are candidates, not verdicts: phrase as evidence-backed suggestions, never contradict the glossary, carry hypothesis markers and Q-ids instead of resolving them.
