---
name: concerns
description: Introduce and maintain a @Concern marker annotation vocabulary in Java projects — a SOURCE-retained annotation recording why a type or package exists when the reason is a technical concern (observability, external-system communication) rather than the business component's responsibility. Ships starter kinds with settled definitions plus the admission discipline for project-specific kinds. Composes with `/java-conventions` for style and `/bce` for placement; applies in any Java context (`java-cli-app`, `microprofile-server`). Use when introducing concern markers, adding a kind, or deciding what to mark. Triggers on "concern annotation", "@Concern", "mark concerns", "technical concern marker", "concern vocabulary", "concern kinds". Not for migration notes on legacy packages — use `concept-annotator`.
---

One annotation, `@Concern(KIND)`, records why a class or package exists when that reason is a
technical concern rather than the BC's responsibility. An import search says what a class
touches; the marker says what it is for. One search over the marker returns a complete,
intent-based surface — including members no import search finds (an in-process engine binding
is an external system without a single network import).

## The Annotation

- one `Concern.java` in the project root package — an application-wide vocabulary no BC owns
- `@Documented`, `@Retention(SOURCE)`, `@Target({TYPE, PACKAGE})`, single element `Kind value()`, nested `enum Kind`
- SOURCE retention — every consumer (reader, agent, javadoc) reads source; the jar stays free of the metadata
- full template with both starter kinds: [references/concern-template.md](references/concern-template.md)

## Placement

- package-level (on `package-info.java`) when the whole BC serves the concern — state it once, not on every class
- type-level when members are scattered across BCs that serve something else
- never the same kind on both a package and a type inside it — one fact, one place
- a type may carry a kind its package does not declare

## The Sole-Purpose Rule

- mark only what would not exist without the concern; litmus: "this class would not exist without ___"
- participation is not membership — a domain class emitting an event in passing, or an orchestrator triggering provider calls, is not marked
- each kind's doc comment states what does not qualify; the exclusions are where the marker earns its keep

## Kind Admission

- keep the enum a small closed set (about five kinds); every added constant dilutes the others
- a kind that fits nearly every class carries no information — no `LOGGING`
- do not duplicate a canonical stack marker — `@Transactional`, `@Liveness`, `@Retry` already say it; a second vocabulary for the same fact drifts
- grep-findability does not disqualify a kind — naming the intent is worth it even when an import search finds the set
- heuristic: a candidate kind is usually non-functional-ish and at least partially scattered across BCs; one that is neither is a business component or noise, not a kind
- a kind whose membership must be argued case by case is a tagging system, not a classification — sharpen the definition or drop the kind

## Kind Naming

- name the judgment, not the mechanism — `EXTERNAL_SYSTEM`, not `EGRESS` or `OUTBOUND`; direction and transport words misclassify edge members (the in-process engine, the loopback client)
- the name must classify every member without commentary; a name that needs a footnote is worse than a longer one
- no dangling adjectives — the name completes "exists to/for ___"

## Starter Kinds

- `OBSERVABILITY` — exists to record what the process did, whatever the transport: JFR events, OTEL spans and metrics, dedicated diagnostic loggers; the emitting API is not the criterion; a class that emits or logs in passing is not this
- `EXTERNAL_SYSTEM` — exists to communicate with a system that has its own lifecycle and failure modes: another process, a remote service, or a foreign in-process engine; ownership is not the criterion — the project's own server counts; owning the shared HTTP client or merely triggering the call is not this
- candidates that pass admission when the project has them: `MIGRATION` (exists only to bridge to the legacy system — the deletion list), `SECURITY` (authn/authz decision points)

## Boundaries

- `/bce` and `/java-conventions` naming rules hold unchanged: packages and classes stay named after responsibilities — the annotation carries the technical-concern identity so names never have to
- the marked set is maintained by review — keep each kind small enough to audit in one reading; no generated enforcement test
- docs and markers only; introducing the vocabulary changes no behavior

## Composition

- `/java-conventions` owns language-level style; `/bce` owns where BCs, packages, and `package-info.java` live
- this skill owns only the concern vocabulary and its discipline; the composed skill always specializes, never contradicts
