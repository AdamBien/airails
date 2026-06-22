---
name: sbce
description: Spec-driven BCE workflow where one capability spec equals one business component (same name) and the spec is the boundary contract. Invoked as `/sbce new|apply <capability-or-feature>` (or by intent), it drives declare → converge; the stack's own test loop is the oracle for "done". `new` accepts a BC name or a natural-language feature description that may decompose into one or several BCs (new or existing). Stack-neutral — composes with `/bce` and a stack skill (`/java-cli-app`, `/microprofile-server`, `/web-components`, …) for code shape and verification. Use when authoring or converging a capability spec, declaring a feature as one or more BCs, mapping a spec to a BC, or running `/sbce new`, `/sbce apply`. Triggers on "SBCE", "capability spec", "spec-driven BCE", "declare a feature", "spec to BC", "converge to spec", "/sbce new", "/sbce apply".
---

Drive the spec-driven BCE workflow. This one skill owns the whole loop — both the rules and
the steps. Invoke it as `/sbce <mode> <capability>` where mode is `new` or `apply`
(see "Invocation modes"), or let it trigger from natural-language intent. Apply every rule
below strictly.

## Guiding principles

- The spec **is** the boundary contract — it answers *what the boundary promises*, never *how*.
- One capability spec ≡ one business component, named the same way. No translation step between "what" and "where".
- Split work by determinism: deterministic parts (run tests, scaffold, move files) belong to **tooling**; judgment (write the spec, write the code, close the gap) belongs to **this skill**.
- The task list is the **gap** read off `spec` vs `BC` on demand — never a hand-maintained tasks file.
- The spec is the single source of truth. Never diff or merge two specs; there is exactly one per capability.
- Never declare "done" on your own opinion — a green run of the stack's test loop is the only signal.

## Spec ↔ BC mapping

The identity is the **BC name** — a single lowercase token (`checkout`), never a dotted path. The
spec location is mechanical and stack-neutral; the **source** location is the stack skill's call
(it owns the package base / source root), so SBCE never hand-types a package coordinate.

```
checkout                                    # the BC name == the only identity
  → spec:   specs/checkout/spec.md          # mechanical, stack-neutral
  → source: <stack skill resolves it>       # Java: src/main/java/<base>/checkout/{boundary,control,entity}/
                                            # web:  app/src/checkout/{boundary,control,entity}/
```

- `## Boundary` ops ↔ the BC's `boundary` layer (one operation → one entry-point method).
- `## Requirements` + EARS statements ↔ behavior **and** the tests that cover it.
- `## Entities` ↔ the BC's `entity` layer.
- `capability` frontmatter == the BC name. This is the only identity field; there is no dotted
  path and no `bc` field. The dotted form is a Java package detail the stack skill derives — never the user's input.

## Determinism boundary

| Concern | Owner | Deterministic? |
|---|---|---|
| Run tests / "is it green" | the **stack's verification loop** (the composed stack skill) | yes — existing tooling, no LLM |
| Decompose a feature into BCs (new vs existing) | this skill's judgment, over a read-only scan of `specs/` + source, **user-confirmed** | no — semantic |
| Scaffold the spec dir `specs/<bc-name>/` | this skill + filesystem | trivially yes |
| Resolve BC name → source location + scaffold layer dirs | the composed stack skill (owns the package base / source root) | yes — stack-defined |
| Structural sync (op→method, requirement→test present) | this skill, made checkable by the stack's traceability convention | grep-level |
| "Does this code satisfy the requirement" | this skill's judgment, **grounded by the requirement's passing test** | no — semantic |

Rules:

- Ask the stack skill "are you green?" — never name a runner or a test kind yourself.
- The green test run is the independent signal; do not self-certify convergence.

## Invocation modes: new · apply

Read the mode and the capability from the invocation (`/sbce apply checkout`). If the mode is
missing, infer it — a BC name **or** a feature description with no spec yet → `new`; spec exists
but not converged → `apply` — or ask. The spec lives at `specs/<bc-name>/spec.md`; the source
location is the stack skill's call. One skill, two modes; never split it.

### new — declare

Declare a new feature. Accepts either a **BC name** (one precise BC) or a **natural-language
feature description** (which may decompose into one *or several* BCs, some new, some existing).
The novelty is the *intent brought*, not the artifact — coining a fresh BC and extending an
existing one are both "new" here.

**BC name** (`/sbce new checkout`):

1. Validate the name: a single lowercase token, no dots, spaces, or uppercase (the stack maps it to a package segment in Java, a folder in web). Reject otherwise.
2. If `specs/<bc-name>/spec.md` exists, do **not** overwrite — report and stop unless the user confirms a rewrite.
3. Author `specs/<bc-name>/spec.md` from the bundled `references/spec-template.md`: one-line responsibility, boundary ops, requirements as EARS statements, optional entities, out-of-scope. Stack-neutral — no types, transports, frameworks, or *how*.
4. Ask the stack skill to scaffold the BC's empty layer dirs (`boundary`/`control`/`entity`) at the source location it owns, and to write the BC's one-line responsibility into its package-level doc — `package-info.java` (Java) / `package-info.md` (web). Write **no** BC source.
5. Report the open gap (counts of ops / requirements) and point the user to `/sbce apply <bc-name>`.

**Feature description** (`/sbce new "let a customer check out a cart"`):

1. Scan existing BCs — `specs/*/spec.md` responsibilities + the stack's source tree.
2. **Propose** a BC set — each entry tagged **new** (coin a BC name from its verb-noun core) or **extend-existing**, each with the one-line responsibility it owns (its slice of the feature).
3. **Confirm the carving with the user before any write** — decomposition has no test oracle, so the human approves the BC set.
4. Realise each entry through the BC-name steps above: a **new** BC gets a fresh spec + dirs + package-level doc; an **extend-existing** BC gets the new boundary ops / EARS requirements added to its **single** existing spec (never a second spec).

Guard: one capability ≡ one BC — output is always 1..N specs, each 1:1 with a BC; a feature is ephemeral input, never a persisted artifact. Confirm the carving (and any coined name) before writing; extend the single existing spec, never fork a parallel one; never hand-write a tasks file; never write BC source here; never overwrite without confirmation.

### apply — converge

Make reality match the declared spec — the kubectl/terraform "make it so" step. Idempotent.

1. Locate `specs/<bc-name>/spec.md`; if missing, tell the user to run `/sbce new <bc-name>` first and stop.
2. Detect the composed stack skill from `AGENTS.md`/`README` (here `java-cli-app`); ask once if it cannot be inferred.
3. Run the stack's test loop. If it is green **and** the structural sync step finds no gap, stop — idempotent no-op; report "already converged".
4. Else read the gap and close it: invoke `/bce` (invariants) + the stack skill (code idioms) to implement each undeclared boundary op as a method, add a traceable test per untested requirement, and write code to make the tests pass. (`new` already wrote the BC's responsibility into its package-level doc; if absent, add it here so the intention stays co-located with the code.)
5. Re-run the test loop. Repeat steps 3–5, bounded to **≤3 passes**, then surface remaining failures/drift to the user.

Guard: green build + no structural drift is the only definition of done; never self-certify; never bake in a stack or runner — ask the stack skill "are you green?".

## Convergence loop

```
run stack test loop + structural sync
  → green & no gap?  → done (stop)
  → else: read the gap → close it (ops, requirement tests, code) → re-run
```

- Bounded: at most 3 passes, then surface remaining failures/drift to the user.
- The sole termination condition is **green build with no structural drift**.
- Each pass closes concrete gaps: an undeclared boundary op → add its method; a requirement with no test → add a traceable test; a failing test → fix the code.

## Composition

- Own only the **workflow** and the **spec↔BC mapping**. Delegate everything else.
- Architecture invariants (BCE layering, naming bans like `*Impl`/`*Service`) → `/bce`.
- Code idioms **and verification** → the project's stack skill and its test loop.
- Discover the stack skill from `AGENTS.md` / `README` (here `java-cli-app` appears in the README "Develop" section). Ask the user once if it cannot be inferred.
- Never duplicate or contradict `/bce` or the stack skill; if they conflict with a spec, surface it rather than guessing.

## Spec format rules

- Sections in order: `# Title` + one-line responsibility, `## Boundary`, `## Requirements`, optional `## Entities`, `## Out of scope`.
- Boundary operations are verb-noun and transport-neutral (`place-order`, not `POST /orders`).
- Requirements are [EARS](https://alistairmavin.com/ears/) statements — one of six patterns, the system is always **the BC**. Group related statements under a titled `### Rn`. Use the unwanted-behaviour (`If…then`) and state-driven (`While…`) patterns to capture error and edge cases.

  | Pattern | Template |
  |---|---|
  | Ubiquitous | `The BC shall <response>.` |
  | State-driven | `While <precondition>, the BC shall <response>.` |
  | Event-driven | `When <trigger>, the BC shall <response>.` |
  | Optional-feature | `Where <feature is included>, the BC shall <response>.` |
  | Unwanted-behaviour | `If <trigger>, then the BC shall <response>.` |
  | Complex | `While <precondition>, when <trigger>, the BC shall <response>.` |

- Every EARS statement uses `shall` and is mandatory **and** tested. No `SHOULD`/`MAY` gradation (RFC 2119) — the test oracle is binary, so express optionality with the `Where` (optional-feature) pattern, never a weaker keyword.
- Every boundary op traces to a requirement group; every EARS statement traces to ≥1 test.
- The **form** of the statement→test trace is the stack skill's call — an `r1…` method name for `zunit`, a system-test name, an `@requirement` tag, a Playwright spec. SBCE only requires the trace exist and be grep-visible.
- Stack-neutral throughout: no types, no transports, no framework verbs, no *how*.

## Reference spec

```markdown
---
capability: checkout
status: active
---
# Checkout
> Accept a cart and turn it into a confirmed, cancellable order.

## Boundary
- `place-order` — submit a cart for fulfilment
- `cancel-order` — withdraw an unfulfilled order

## Requirements
### R1: Place an order
- When a cart with at least one item is submitted, the BC shall create and confirm an order.
- If the cart is empty, then the BC shall reject the request.

### R2: Cancel an order
- While an order is unfulfilled, the BC shall allow it to be cancelled.
- If the order is already fulfilled, then the BC shall reject the cancellation.

## Entities
- Order, Cart

## Out of scope
- payment capture
- shipping
```

The spec lives at `specs/checkout/spec.md`. The stack skill places the BC source (e.g.
`src/main/java/<base>/checkout/` in Java, `app/src/checkout/` in web) with `placeOrder`/`cancelOrder`
in the boundary, and a test tracing each EARS statement under R1, R2.

## What NOT to do

- Don't hand-author a tasks file — the gap is read, not stored.
- Don't diff or merge specs — there is one spec per capability.
- Don't bake in a stack or a test runner — delegate code idioms and verification.
- Don't self-certify convergence — let the stack's tests speak.
- Don't put *how* (types, transports, frameworks) into a spec.
- Don't split this into separate skills or commands — one skill, two modes.
- Don't build the `sbce` binary until a headless need is real.
