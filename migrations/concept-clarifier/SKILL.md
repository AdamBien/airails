---
name: concept-clarifier
description: Resolve the open questions and hypothesis-marked glossary definitions produced by concept-extractor and bc-carver together with a human domain expert — via live interview or an async questionnaire (migration/INTERVIEW.md) — and fold confirmed answers into migration/GLOSSARY.md with provenance. Second step of a legacy-to-BCE migration; composes with concept-extractor. Invoke explicitly as /concept-clarifier in the analyzed project. Not for scoping a task or gathering requirements for a request — use clarify for that.
disable-model-invocation: true
---

# Concept Clarifier

Turn the extractor's hypotheses into confirmed domain knowledge. Machines mined the candidates; only a human who knows the business can settle them. Never invent an answer — an open question folded back wrong poisons every downstream step.

## Inputs

Build one work queue from:

1. `migration/CONCEPTS.md` → `## Open Questions`, by Q-id.
2. Every `migration/GLOSSARY.md` entry still carrying a `(hypothesis …)` marker.
3. `migration/CARVING.md` → `## Open Questions`, when present — BC-assignment questions share the same Q-id space and resolve through the same flow; their answers land in `GLOSSARY.md` `## Decisions` keyed by Q-id.

Group the queue by theme: synonym merges, homonym splits, abbreviation expansions, canonical-term and language choices, BC assignments. Do not mine the codebase — evidence shown to the expert comes verbatim from the CONCEPTS.md pointers; if evidence is missing, that is an extractor defect, not a reason to search.

## Mode Selection

First ask who answers:

- **The person in this session** → live interview.
- **A remote domain expert** → async questionnaire. This is the common case in real migrations.

If `migration/INTERVIEW.md` exists and contains filled-in answers, skip the question phase and fold back (below).

## Live Interview

Use `AskUserQuestion`, one theme per round, max 4 questions per call. Make options enumerable where possible — "Same concept / Different concepts / Depends" — with the evidence pointers in the option descriptions; free-form answers arrive via "Other". After each round, fold back immediately, then continue. Stop when the queue is empty or the user defers the remainder — deferred questions stay open.

## Async Questionnaire

Generate `migration/INTERVIEW.md` from [references/interview-template.md](references/interview-template.md): one block per question with Q-id, question, evidence, checkbox options, free-text line, and an "answered by" line. Tell the user to send it to the expert and re-run `/concept-clarifier` when it comes back filled in.

On fold-back, parse each answered block; leave unanswered blocks untouched. Partial answers are normal — fold what exists.

## Fold-Back Rules

Every confirmed answer lands in `migration/GLOSSARY.md`:

- Remove the `(hypothesis …)` marker; stamp provenance: `confirmed: <name>, <YYYY-MM-DD>`.
- Apply consequences, not just the words: merge alias clusters into one entry, split a homonym into two entries, correct the canonical term, update `Related:` links.
- Decisions with no glossary entry of their own (e.g. "keep as two concepts", "German is the canonical language") go into a `## Decisions` section, one line per Q-id. The glossary is the single durable decision store that extractor re-runs consult.
- In `CONCEPTS.md`, mark the resolved question: `~~Q3: …~~ → answered, see GLOSSARY.md`. Annotate, never delete — the audit trail must survive.
- Edit only the affected entries. Never regenerate the glossary; it is human-owned.

## Rules

- Never answer a question yourself, however obvious — plausible self-answers are how hypotheses masquerade as facts.
- Unanswered and deferred questions remain open; re-runs pick them up again.
- Record who answered, always — provenance is what separates a confirmed term from a guess with confidence.
