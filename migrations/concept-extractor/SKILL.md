---
name: concept-extractor
description: Mine candidate domain concepts and their vocabulary from every naming source of a legacy system — database schema, UI labels, REST/URL paths, configuration, code identifiers, documentation. Produces a regenerable CONCEPTS.md (aliases, evidence, frequency, co-occurrence) and seeds a human-owned GLOSSARY.md with canonical terms, both in the analyzed project's migration/ folder. First step of a legacy-to-BCE migration; composes with bce and sbce. Invoke explicitly as /concept-extractor on the system to analyze. Not for mechanical 1:1 ports (use j2ee-migration) and not for carving business components — it only produces the vocabulary later steps consume.
disable-model-invocation: true
---

# Concept Extractor

Recover the domain language buried in a legacy system's names. The technical structure of overengineered systems lies about the domain — packages reflect patterns, not business capabilities. Names are where the original domain knowledge survived. Produce candidates for human review, never verdicts.

## Workflow

1. Inventory which naming sources exist in the system (see ranking below).
2. Mine every source. Tokenize compound names: `CustomerContractValidationServiceFactoryImpl` → Customer, Contract, Validation.
3. Strip pattern noise using [references/noise-words.md](references/noise-words.md). Keep the discards for the noise report.
4. Cluster aliases into one concept per meaning (Customer / Client / Kunde / `CUST`). Expand abbreviations from surrounding context — column names, labels, Javadoc.
5. Record co-occurrence: two concepts co-occur when they share a class, table, URL path, or UI view.
6. Write `migration/CONCEPTS.md` in the analyzed project — all pipeline artifacts live in this folder, never at the legacy project's root.
7. Seed `migration/GLOSSARY.md` if absent. If present, append proposals under `## Proposed` — never modify existing entries.

## Naming Sources — Ranked by Trustworthiness

Mine all that exist. When sources conflict, trust the higher-ranked one:

1. **Database schema** — tables, columns, constraints. Survived every refactoring fashion; usually the most honest source.
2. **UI labels** — JSP/JSF/HTML text, i18n bundles, report headers. Closest to the language the business actually speaks.
3. **External contracts** — REST/URL paths, queue and topic names, WSDL, exchanged file names.
4. **Configuration keys, enum values, exception names.**
5. **Code identifiers** — packages, classes, methods, fields. Most numerous, least reliable.
6. **Comments, Javadoc, test names, documentation.**

## CONCEPTS.md Format

Regenerable, evidence-oriented, diffable. Use this structure:

```markdown
# Concepts: <system name>

## Concepts
### <Concept>
- **Aliases:** <all forms found, incl. abbreviations>
- **Sources:** DB / UI / contracts / code — with concrete pointers (table, file, path)
- **Frequency:** <n> | **Confidence:** high | medium | low
- **Co-occurs with:** <Concept> (<count>), ...

## Co-occurrence
<table or Mermaid graph of concept pairs with counts>

## Noise Report
<stripped words with counts — makes the extraction auditable>

## Open Questions
### Q<n> — <one-line question>
<evidence pointers; candidate answers when enumerable — suspected synonyms
across sources, homonyms in different contexts (often the first hint of a
BC boundary), unexpandable abbreviations>
```

Q-ids are stable across re-runs: never renumber, new questions get new ids. They are the contract `concept-clarifier` resolves against.

Confidence: **high** = present in several source kinds, **medium** = one kind with many occurrences, **low** = single source or unresolved abbreviation. Order concepts by frequency — the reader reviews the top entries carefully and skims the tail.

## GLOSSARY.md Format

Human-owned, durable — the ubiquitous language of the future system. Seed one entry per high- and medium-confidence concept:

```markdown
### <Concept>
**Definition (inferred):** <one sentence>. *(hypothesis — derived from <evidence>; confirm with domain expert)*
**Canonical term:** <term> — supersedes: <aliases>
**Related:** [[<Concept>]], [[<Concept>]]
```

Every definition is inferred from usage, not knowledge — mark it as a hypothesis until a human removes the marker. The canonical term is a naming decision for the generated code (entities, boundaries, REST paths, BC names); for mixed-language systems, pick one language and state it at the top of the glossary.

The glossary outlives `migration/`: its canonical terms feed the new system's sbce system doc and BC naming. It is authored here but meant to be promoted into the new system — the folder is disposable, the glossary is cargo. State this in the glossary header.

## Rules

- Every claim carries an evidence pointer — review must be spot-checkable, not trust-based.
- All output goes to `migration/` — a well-known path is the contract downstream skills read from, and one folder keeps the legacy repo clean (one `.gitignore` line, one `rm -r` after migration).
- Never overwrite a human-edited file: regenerate `CONCEPTS.md` to `migration/CONCEPTS.new.md` when the original was modified; only ever append to `GLOSSARY.md`.
- Confirmed glossary entries and its `## Decisions` section are ground truth: on re-runs keep their merges, splits, and canonical terms, and do not re-open answered questions — resolved Q-ids stay resolved in `CONCEPTS.new.md`.
- Noise is contextual — a pattern word can be a domain term (Booking in payments, Position in trading). When a noise-list word appears in DB columns or UI labels, keep it as a concept.
- Do not carve business components or propose architecture — downstream skills (`bce`, `sbce`) consume the output.
