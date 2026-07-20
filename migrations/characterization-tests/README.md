# characterization-tests

An [AIrails.dev](https://airails.dev) skill that **pins the observed behavior of a running legacy system as replayable golden masters**. A regular test fails when behavior is wrong; a characterization test fails when behavior is *different*.

## Scope

- Owns the **record → replay → reconcile** discipline: record normalized stimulus/observation pairs, replay them against the migrated system, report every diff.
- The observable **surface is system-dependent** — HTTP (bundled), messaging, batch/file, CLI, database state. Assert the outermost surface that shows the behavior.
- A diff is a question, not a verdict: regression (fix the system) or accepted change (update the recording with an `accepted:` line, a human decision).

These are [system tests](https://en.wikipedia.org/wiki/System_testing) whose expectations are **recorded, not specified** — the executable oracle that brackets every code-changing migration step.

## Composition

- [`system-tests`](../../bce/system-tests) — owns the system-test contract (black box, coordinates as configuration, total reporting).
- a **stack skill** — supplies the replay-suite syntax (`microprofile-server` `-st`/JUnit, `java-cli-app`/zunit).
- [`simplifier`](../simplifier) and [`sbce`](../../bce/sbce) — the transformations this suite brackets.

## Usage

Invoke explicitly as `/characterization-tests record` before a migration step and `/characterization-tests replay` after. Recordings live in `migration/characterization/`. The surface-neutral format is in [references/recording-format.md](references/recording-format.md); HTTP specifics in [references/http.md](references/http.md).
