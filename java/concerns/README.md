# concerns

A `@Concern(KIND)` marker annotation vocabulary for Java projects: one SOURCE-retained
annotation in the root package recording why a type or package exists when the reason is a
technical concern rather than the business component's responsibility. One search over the
marker returns a complete, intent-based surface — including members no import search finds.

Starter kinds:

- `OBSERVABILITY` — exists to record what the process did
- `EXTERNAL_SYSTEM` — exists to communicate with a system that has its own lifecycle and failure modes

Project-specific kinds pass the admission discipline in [SKILL.md](SKILL.md); the annotation
template lives in [references/concern-template.md](references/concern-template.md).

Composes with [java-conventions](../java-conventions) for style and
[bce](../../bce/bce) for package placement. Names stay responsibility-based —
the annotation carries the technical-concern identity so names never have to.
