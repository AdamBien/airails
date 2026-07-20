---
name: characterization-tests
description: Pin the observed behavior of a running legacy system as replayable golden masters — record normalized stimulus/observation pairs into migration/characterization/, replay them against the lifted or re-architected system, report behavior diffs. Owns the record/replay/reconcile discipline; the observable surface (HTTP, messaging, batch files, CLI, database state) is system-dependent, and test syntax follows the composed stack skill. Characterization tests are system tests whose expectations are recorded, not specified. Brackets every code-changing migration step; retires as sbce specs and ears-tests take over. Invoke explicitly as /characterization-tests record|replay. Not for spec-derived tests — use ears-tests.
disable-model-invocation: true
---

# Characterization Tests

A regular test fails when behavior is wrong; a characterization test fails when behavior is *different*. Record what the system does — bugs included — and pin it as the expected value. Correctness is the information the legacy system lost; observed behavior is the only surviving specification, and downstream consumers may depend on the bugs.

Characterization tests are **system tests**: black-box, against the running system, from the outside. The only difference from ordinary system tests is the origin of the expectations — recorded, not specified.

## Surfaces

The observable surface is a property of the system, not of this skill. Inventory all of them:

| Surface | Stimulus → Observation | Mechanics |
|---|---|---|
| HTTP (REST, server-rendered pages) | request → response | [references/http.md](references/http.md) |
| Messaging (JMS, queues) | message in → message(s) out | same discipline; document per system |
| Batch / file exchange | input file → output file, report | same discipline; document per system |
| CLI | invocation → exit code, output | same discipline; document per system |
| Database side effects | any stimulus → state delta | record only when no outer surface shows the effect |

HTTP mechanics are bundled; for other surfaces apply the identical record/normalize/replay discipline and document the surface mechanics next to the recordings. Prefer the outermost surface that shows the behavior — assert database state only when nothing above it does.

## Record

1. Inventory the system's surfaces and their entry points (JAX-RS resources, `web.xml` mappings, queue names, batch jobs, and the external contracts in `migration/CONCEPTS.md` when present).
2. Ask the user: target coordinates (base URL, broker, job trigger — never guess), environment, credentials, and which business flows matter most. Confirm the target is not an unprotected production system.
3. Observe-only by default: record stimuli without side effects. Record mutating stimuli (writes, consumed messages, batch runs) only after explicit confirmation, only against a disposable environment with resettable seed data.
4. Execute each stimulus, normalize the observation per [references/recording-format.md](references/recording-format.md) and the surface reference, store one file per interaction under `migration/characterization/recordings/`.
5. Each recording must replay in isolation — note seed-data assumptions in its header.

## Replay

1. Ask for the target coordinates of the migrated system.
2. Re-execute every recording; normalize the observation with the same rules; compare against the stored expectation.
3. Write `migration/characterization/REPORT.md`: one verdict per recording (`same` | `different` with diff excerpt | `unreachable`), plus summary counts. Every recording appears in the report — skipped is a verdict, not an omission.
4. For repeated replays (CI, per-carving checks), generate a replay suite following the composed stack skill's system-test conventions — an `-st` module with JUnit 5 for microprofile-server targets, zunit for java-cli-app, a zero-dependency single-file Java 25 runner when no stack skill is composed.

## Reconcile

A diff is a question, not a verdict: behavior changed — intended?

- **Regression** → the migrated system is fixed; the recording stays.
- **Accepted change** (e.g. a bug deliberately fixed) → update the recording and append a decision line to its header: `accepted: <name>, <YYYY-MM-DD> — <reason>`.

Never update a recording silently, and never decide yourself — acceptance is a human call, same rule as the clarifier.

## Rules

- Record actual observations, including apparently wrong ones — correctness judgments happen at reconcile time, by a human.
- Normalize before storing and before comparing; never diff raw observations.
- The suite encodes what *is*, not what *should be* — it is scaffolding with an expiry: retire recordings BC by BC as sbce specs and ears-tests encode intent, delete the rest with `migration/`.
- Recording exercises a real system: respect rate limits, stop on unexpected error storms, and never authenticate with credentials the user has not explicitly provided for this purpose.
