# sbce

An [AIrails.dev](https://airails.dev) skill for **SBCE** (pronounced *"space"*) — Spec-driven
BCE — the workflow where one capability spec equals one business component (same name) and the
spec is the boundary contract. One slash-invocable skill, two modes: `new → apply`
(declare → converge).

## Scope

- One skill, two modes — invoke as `/sbce new|apply <capability-or-feature>` (or by intent):
  - **new** (declare) — author the spec into the BC's package doc (`package-info.java` / `package-info.md`) and scaffold the BC's empty `boundary/control/entity` dirs. Accepts a BC name (one precise spec) **or** a natural-language feature description that decomposes into one or several BCs — coining new ones or extending existing specs, after you confirm the carving
  - **apply** (converge) — close the gap between spec and BC, then loop the stack's test suite until green (the kubectl/terraform "make it so" step)
- The identity is the **BC name** (`checkout`); the spec **is** the BC's package doc, co-located with the code — Java: `package-info.java` (`///` Markdown, [JEP 467](https://openjdk.org/jeps/467)), web: `package-info.md`. No separate `specs/` tree
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

Invoke `/sbce <mode> <bc-name>` (or just describe the intent — "declare a checkout
capability", "converge this BC to its spec"). Run the lifecycle in order:

1. **Declare** — `/sbce new checkout` *(BC name)* or
   `/sbce new "let a customer check out a cart"` *(feature description)*.
   A BC name writes the spec into the BC's package doc (`package-info.java` as `///` Markdown, or
   `package-info.md` in web) and asks the stack skill to scaffold the BC's
   `boundary/control/entity` dirs. A feature description scans existing BCs, proposes a set (new
   ones + existing to extend) for you to confirm, then authors/extends one package doc per BC.
   Fill in the spec: boundary operations, requirements as EARS statements.
   Format: [references/spec-template.md](references/spec-template.md).
2. **Converge** — `/sbce apply checkout`
   Closes the gap (boundary methods, a test per EARS statement, the code) and loops the stack's
   tests until green. Idempotent — re-run any time; an in-sync, green BC is a no-op.

Omit the mode and the skill infers it: a BC name or a feature description with no spec yet →
`new`; spec exists but not green → `apply`.

## Test

```
/sbce new checkout
```

Edit the generated spec, then converge:

```
/sbce apply checkout
```
