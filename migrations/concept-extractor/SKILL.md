---
name: concept-extractor
description: Mine candidate domain concepts and their vocabulary from every naming source of a legacy system — database schema, UI labels, REST/URL paths, configuration, code identifiers, documentation. Produces a regenerable CONCEPTS.md (aliases, evidence, frequency, co-occurrence) and seeds a human-owned GLOSSARY.md with canonical terms. First step of a legacy-to-BCE migration; composes with bce and sbce. Use when analyzing a legacy or unfamiliar system for its domain vocabulary or recovering the ubiquitous language. Triggers on "extract concepts", "concept extraction", "mine the domain", "domain concepts", "domain vocabulary", "ubiquitous language", "concept inventory", "glossary from code". Not for mechanical 1:1 ports (use j2ee-migration) and not for carving business components — it only produces the vocabulary later steps consume.
---

# Concept Extractor

Recover the domain language buried in a legacy system's names. The technical structure of overengineered systems lies about the domain — packages reflect patterns, not business capabilities. Names are where the original domain knowledge survived. Produce candidates for human review, never verdicts.

## Workflow

1. Inventory which naming sources exist in the system (see ranking below).
2. Mine every source. Tokenize compound names: `CustomerContractValidationServiceFactoryImpl` → Customer, Contract, Validation.
3. Strip pattern noise using [references/noise-words.md](references/noise-words.md). Keep the discards for the noise report.
4. Cluster aliases into one concept per meaning (Customer / Client / Kunde / `CUST`). Expand abbreviations from surrounding context — column names, labels, Javadoc.
5. Record co-occurrence: two concepts co-occur when they share a class, table, URL path, or UI view.
6. Write `CONCEPTS.md` to the analyzed project's root.
7. Seed `GLOSSARY.md` if absent. If present, append proposals under `## Proposed` — never modify existing entries.

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
<unresolved ambiguities: suspected synonyms across sources, homonyms
appearing in different contexts — often the first hint of a BC boundary>
```

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

## Rules

- Every claim carries an evidence pointer — review must be spot-checkable, not trust-based.
- Never overwrite a human-edited file: regenerate `CONCEPTS.md` to `CONCEPTS.new.md` when the original was modified; only ever append to `GLOSSARY.md`.
- Noise is contextual — a pattern word can be a domain term (Booking in payments, Position in trading). When a noise-list word appears in DB columns or UI labels, keep it as a concept.
- Do not carve business components or propose architecture — downstream skills (`bce`, `sbce`) consume the output.
