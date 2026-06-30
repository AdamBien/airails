---
name: ears-tests
description: Generate parameterized (table-driven) tests from EARS requirement statements — the deterministic transform that turns an SBCE capability spec's `## Requirements` into one parameterized test per requirement group and one labeled row per statement id. Stack-neutral; owns only the EARS→table mapping and the spec↔test trace, and delegates the test syntax to the composed stack skill (JUnit 5, zunit, Playwright). Use whenever turning EARS requirements, acceptance criteria, or an SBCE/`/sbce` spec into tests; whenever you see "When/While/If…then, the … shall …" statements that need covering; or when asked for table-driven, data-driven, or parameterized tests from a requirement group. Triggers on "EARS", "parameterized tests", "table-driven tests", "data-driven tests", "tests from requirements", "tests from the spec", "cover requirement Rn", "generate tests for this capability", "acceptance criteria to tests".
---

Turn EARS requirement statements into parameterized tests. The leverage is structural: every
EARS pattern is a `(condition → response)` tuple, and an SBCE requirement **group** already
collects statements that share **one boundary operation** — same arrange/act skeleton, only the
data differs. That is the textbook precondition for parameterization, so the mapping is mechanical.

Own only the **transform and the trace**. The concrete test syntax is the composed stack skill's
call — never name a runner or framework verb here.

## The mapping

- One requirement **group `Rn` → one parameterized test method** (the shared boundary op is the body).
- One **statement `Rn.m` → one row** in the parameter source.
- The **statement id `Rn.m` is the row's display name** — so the spec↔test binding stays per-statement
  and grep-visible, exactly as SBCE requires (a removed id retires its row; a row with no statement is drift).
- The EARS pattern fixes the row's **role and shape** — you do not invent the columns, you read them off the pattern.

## EARS pattern → row shape

The system is always **the BC**. Each pattern decomposes into the arrange/act/assert columns of a row:

| Pattern | Statement shape | Arrange (state) | Act (trigger) | Assert (response) | Role |
|---|---|---|---|---|---|
| Event-driven | `When <trigger>, the BC shall <response>` | — | trigger | response | happy |
| Unwanted-behaviour | `If <trigger>, then the BC shall <response>` | — | bad/invalid trigger | rejection / error | unhappy |
| State-driven | `While <precondition>, the BC shall <response>` | precondition | (implicit) | response | stateful |
| Complex | `While <precondition>, when <trigger>, the BC shall <response>` | precondition | trigger | response | full tuple |
| Optional-feature | `Where <feature>, the BC shall <response>` | feature enabled | trigger | response | gated |
| Ubiquitous | `The BC shall <response>` | — | — | invariant | single case (least parameterizable — a plain test is fine) |

Mixing a happy `When` row and an unhappy `If…then` row in the same group's table is the norm — it is
how one parameterized method covers both the success and the rejection path of one boundary op.

## Deterministic vs. authored

EARS hands you the *enumeration*, not the *data*. Keep the two honestly separated so the generated
test never silently asserts an assumption:

- **Deterministic (generate without judgment):** the method-per-group, the row-per-statement, the
  `Rn.m` row labels, which boundary op the body calls, the row role from the pattern, and the
  bijection check (every `Rn.m` ↔ exactly one row).
- **Authored (leave a clearly marked stub for human/LLM):** the concrete input fixture and the
  assertion that expresses `<response>`. EARS abstracts the response in prose ("create and confirm
  an order") and carries no values — do not fabricate them. Emit a failing/`TODO` stub a human or a
  later `/sbce apply` pass fills, so an unwritten assertion shows up red, not green.

## Realization is the stack skill's call

This skill is stack-neutral. The parameter-source mechanism and display-name form belong to the
composed stack skill — read `references/realizations.md` for the per-stack shape (JUnit 5
`@ParameterizedTest` + `@MethodSource`, zunit's table form, Playwright `test.describe` per group).
Pick the one the project already uses; never introduce a second test idiom.

## Composition

- **`/sbce`** owns the spec and the stable `Rn.m` ids — this skill consumes them; it never edits the
  spec or coins ids. During `/sbce apply`, this is how the *spec→code* "a traceable test per `Rn.m`"
  step is realized when a group has ≥2 statements.
- **`/bce`** owns where tests and the BC live; the **stack skill** owns the test syntax and the
  "are you green?" oracle.
- Prefer a parameterized method only when a group has **≥2 statements**; a lone statement is a plain
  test — do not force a one-row table.
