# sbce

An [AIrails.dev](https://airails.dev) skill for **SBCE** (pronounced *"space"*) — Spec-driven
BCE — the workflow where one capability spec equals one business component (same name) and the
spec is the boundary contract. One slash-invocable skill, three modes: `new → apply → gc`
(declare → converge → reap).

## Scope

- One skill, three modes — invoke as `/sbce new|apply|gc <capability>` (or by intent):
  - **new** (declare) — author `specs/<path>/spec.md` from the bundled template and scaffold empty `boundary/control/entity` BC dirs
  - **apply** (converge) — close the gap between spec and BC, then loop the stack's test suite until green (the kubectl/terraform "make it so" step)
  - **gc** (reap) — archive a converged spec by **moving** it to `archive/specs/<path>/`; never deletes, BC source stays
- The capability is dotted and maps 1:1 to the source path: `airhacks.sbce.checkout` ↔ `src/main/java/airhacks/sbce/checkout/`
- Stack-neutral — owns only the workflow and the spec↔BC mapping; no transport, types, or framework verbs in a spec
- No binary required: the **stack's own test loop is the oracle for "done"**.
## Composition

`sbce` owns only the workflow and the spec↔BC mapping; it delegates everything else, so install
these alongside it (all ship via airails `installSkills`):

- [`bce`](../bce) — architecture invariants (BCE layering, naming).
- a **stack skill** — code idioms *and* verification: [`java-cli-app`](../java-cli-app)
  (`zunit`/`zb`), [`microprofile-server`](../microprofile-server) (integration + system tests),
  or [`web-components`](../web-components) (system tests + Playwright).

`sbce` never names a runner or a test kind; it asks the stack skill "are you green?". The spec
format and rules live in [SKILL.md](SKILL.md).

## Usage

Invoke `/sbce <mode> <capability>` (or just describe the intent — "declare a checkout
capability", "converge this BC to its spec"). Run the lifecycle in order:

1. **Declare** — `/sbce new airhacks.sbce.checkout`
   Writes `specs/airhacks/sbce/checkout/spec.md` from the template and scaffolds empty
   `boundary/control/entity` dirs. Fill in the spec: boundary operations, SHALL requirements
   with GIVEN/WHEN/THEN scenarios. Format: [references/spec-template.md](references/spec-template.md).
2. **Converge** — `/sbce apply airhacks.sbce.checkout`
   Closes the gap (boundary methods, a test per requirement, the code) and loops the stack's
   tests until green. Idempotent — re-run any time; an in-sync, green BC is a no-op.
3. **Reap** — `/sbce gc airhacks.sbce.checkout`
   Once green, moves the spec to `archive/specs/...`. Never deletes; BC source stays.

Omit the mode and the skill infers it: no spec yet → `new`; spec exists but not green →
`apply`; converged → `gc`.

## Test

```
/sbce new airhacks.sbce.checkout
```

Edit the generated spec, then converge and archive:

```
/sbce apply airhacks.sbce.checkout
/sbce gc airhacks.sbce.checkout
```
