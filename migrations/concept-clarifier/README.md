# concept-clarifier

An [AIrails.dev](https://airails.dev) skill that **resolves the open questions and glossary hypotheses from concept-extractor and bc-carver together with a human domain expert**.

## Scope

- Builds one work queue from `migration/CONCEPTS.md` and `migration/CARVING.md` open questions plus every hypothesis-marked glossary entry.
- Two modes: a **live interview** (`AskUserQuestion`, evidence in the options) and an **async questionnaire** (`migration/INTERVIEW.md`) for a remote expert, folded back on a later run.
- Folds answers into `migration/GLOSSARY.md` with provenance (`confirmed: <name>, <date>`) and consequences — merge aliases, split homonyms, fix the canonical term; non-entry decisions go to `## Decisions`.

Never answers a question itself; acceptance is always a human call.

## Composition

- [`concept-extractor`](../concept-extractor) and [`bc-carver`](../bc-carver) — produce the questions and the Q-ids this skill resolves against.
- Distinct from [`clarify`](https://airails.dev) — that scopes a task before work; this resolves domain ambiguities in mined artifacts.

## Usage

Invoke explicitly as `/concept-clarifier` in the analyzed project. A filled-in `migration/INTERVIEW.md` triggers fold-back instead of a new interview. The questionnaire format lives in [references/interview-template.md](references/interview-template.md).
