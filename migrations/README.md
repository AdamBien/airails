# Migrations

Skills for migrating legacy, overengineered enterprise systems — Java or web — onto a modern stack. Three entry paths (see below): rehost the system unchanged, replatform it by simplifying in place, or rearchitect it toward BCE by recovering the domain vocabulary from the system's names and carving business components. Every step is bracketed by recorded behavior; the analysis steps produce documentation for human review before any code changes.

```mermaid
graph TD
    Legacy([Legacy System]) -->|rearchitect first| Extractor([concept-extractor])
    Legacy -.->|lift-and-shift first| Lift([1:1 lift — planned])
    Lift -.-> Extractor
    Extractor -->|CONCEPTS.md| Clarifier([concept-clarifier])
    Clarifier -->|GLOSSARY.md| Carver([bc-carver])
    Carver -->|CARVING.md| SBCE([sbce + stack skill])
    Carver -.-> Annotator([concept-annotator])
    Annotator -.->|package-info notes| SBCE
    Legacy -.->|record| Charact([characterization-tests])
    Lift -.->|replay| Charact
    SBCE -.->|replay| Charact
    Lift -.-> Simplifier([simplifier])
    Simplifier -.->|replay per pass| Charact
    Simplifier -.-> Extractor

    classDef bc fill:#dae8fc,stroke:#6c8ebf,color:#000
    classDef ext fill:#fff2cc,stroke:#d6b656,color:#000,stroke-dasharray:5 5
    class Extractor,Clarifier,Carver,Annotator,Charact,Simplifier bc
    class Legacy,Lift,SBCE ext
```

## Skills

- [**concept-extractor**](concept-extractor/) — Mines candidate domain concepts from every naming source (database schema, UI labels, external contracts, code); writes `migration/CONCEPTS.md`, seeds `migration/GLOSSARY.md`
- [**concept-clarifier**](concept-clarifier/) — Resolves open questions and glossary hypotheses with a domain expert, live or via `migration/INTERVIEW.md`; records answers with provenance
- [**bc-carver**](bc-carver/) — Clusters confirmed concepts into candidate business components; documents the as-is → to-be diff in `migration/CARVING.md`
- [**concept-annotator**](concept-annotator/) — Projects concepts, target BC, and refactoring hints into per-package `package-info.java` migration notes
- [**characterization-tests**](characterization-tests/) — Pins observed behavior of the running legacy system as replayable golden masters in `migration/characterization/`; system tests with recorded expectations, records before, replays after the lift and each carving step; surface-dependent (HTTP bundled), composes with [system-tests](../bce/system-tests/) and the stack skill
- [**simplifier**](simplifier/) — Replatform mode: modernizes code and removes external dependencies on the lifted system, one pass at a time under the characterization green bar; structure preserved, transformation catalog and dependency ladder from the composed stack ecosystem (java-distiller, web-conventions/web-components)

## Conventions

- All skills are invoked explicitly (`/concept-extractor`, …) and produce candidates for human review
- Pipeline artifacts live in the analyzed project's `migration/` folder; open questions carry stable Q-ids across artifacts
- `GLOSSARY.md` records confirmed terms and decisions; extractor and carver re-runs treat it as ground truth
- `migration/` is deleted after the migration; `GLOSSARY.md` is promoted into the new system

## Entry Paths

- **Lift-and-shift first** — 1:1 lift onto the target runtime, then the pipeline on the lifted tree; for systems that must keep running during a long migration or under platform EOL pressure
- **Replatform** — 1:1 lift, then [simplifier](simplifier/) passes under the characterization green bar; a valid terminal state, or preparation for the concept pipeline
- **Rearchitect directly** — the pipeline on the original tree; for small or well-understood systems

Steps are optional: on small systems the clarifier can be a single live round; the annotator can be skipped.

## Open Gaps

- **1:1 lift skill** — planned; a j2ee-migration PoC exists outside this repo
