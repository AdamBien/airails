---
name: sbce
description: Spec-driven BCE workflow where one capability spec equals one business component (same name) and the spec is the boundary contract. Invoked as `/sbce new|apply|gc <capability>` (or by intent), it drives declare → converge → reap; the stack's own test loop is the oracle for "done". Stack-neutral — composes with `/bce` and a stack skill (`/java-cli-app`, `/microprofile-server`, `/web-components`, …) for code shape and verification. Use when authoring or converging a capability spec, mapping a spec to a BC, or running `/sbce new`, `/sbce apply`, `/sbce gc`. Triggers on "SBCE", "capability spec", "spec-driven BCE", "converge to spec", "spec to BC", "/sbce new", "/sbce apply", "/sbce gc".
---

Drive the spec-driven BCE workflow. This one skill owns the whole loop — both the rules and
the steps. Invoke it as `/sbce <mode> <capability>` where mode is `new`, `apply`, or `gc`
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

Mechanical, never reinterpreted. Capability is dotted; the path is dots → slashes.

```
airhacks.sbce.checkout
  → spec:   specs/airhacks/sbce/checkout/spec.md
  → source: src/main/java/airhacks/sbce/checkout/{boundary,control,entity}/
```

- `## Boundary` ops ↔ the BC's `boundary` layer (one operation → one entry-point method).
- `## Requirements` + EARS statements ↔ behavior **and** the tests that cover it.
- `## Entities` ↔ the BC's `entity` layer.
- `capability` frontmatter == the directory path. This is the only identity field; there is no `bc` field.

## Determinism boundary

| Concern | Owner | Deterministic? |
|---|---|---|
| Run tests / "is it green" | the **stack's verification loop** (the composed stack skill) | yes — existing tooling, no LLM |
| Scaffold spec + BC layer dirs | this skill + filesystem | trivially yes |
| Move spec → archive | this skill + filesystem (`mv`) | trivially yes |
| Structural sync (op→method, requirement→test present) | this skill, made checkable by the stack's traceability convention | grep-level |
| "Does this code satisfy the requirement" | this skill's judgment, **grounded by the requirement's passing test** | no — semantic |

Rules:

- Ask the stack skill "are you green?" — never name a runner or a test kind yourself.
- The green test run is the independent signal; do not self-certify convergence.

## Invocation modes: new · apply · gc

Read the mode and the dotted capability from the invocation (`/sbce apply checkout`). If the
mode is missing, infer it — no spec yet → `new`; spec exists but not converged → `apply`;
converged → `gc` — or ask. Compute the path by replacing dots with slashes
(`airhacks.sbce.checkout` → `airhacks/sbce/checkout`). One skill, three modes; never split it.

### new — declare

1. Validate the name: lowercase, dot-separated, no spaces or uppercase. Reject otherwise.
2. If `specs/<path>/spec.md` exists, do **not** overwrite — report and stop unless the user confirms a rewrite.
3. Author `specs/<path>/spec.md` from the bundled `references/spec-template.md`: one-line responsibility, boundary ops, requirements as EARS statements, optional entities, out-of-scope. Stack-neutral — no types, transports, frameworks, or *how*.
4. Scaffold empty BC layer dirs `src/main/java/<path>/{boundary,control,entity}/` (add `.gitkeep` if the stack needs empty dirs tracked). Write **no** BC source.
5. Report the open gap (counts of ops / requirements) and point the user to `/sbce apply <capability>`.

Guard: never hand-write a tasks file; never write BC source here; never overwrite without confirmation.

### apply — converge

Make reality match the declared spec — the kubectl/terraform "make it so" step. Idempotent.

1. Locate `specs/<path>/spec.md`; if missing, tell the user to run `/sbce new <capability>` first and stop.
2. Detect the composed stack skill from `AGENTS.md`/`README` (here `java-cli-app`); ask once if it cannot be inferred.
3. Run the stack's test loop. If it is green **and** the structural sync step finds no gap, stop — idempotent no-op; report "already converged".
4. Else read the gap and close it: invoke `/bce` (invariants) + the stack skill (code idioms) to implement each undeclared boundary op as a method, add a traceable test per untested requirement, and write code to make the tests pass.
5. Re-run the test loop. Repeat steps 3–5, bounded to **≤3 passes**, then surface remaining failures/drift to the user.

Guard: green build + no structural drift is the only definition of done; never self-certify; never bake in a stack or runner — ask the stack skill "are you green?".

### gc — reap

Archive a converged capability by **moving** its spec. Never delete; BC source stays.

1. If no capability is given, scan `specs/` and list candidates, then ask which to reap.
2. Run the converge gate (test loop green + structural sync clean). If it fails, stop and tell the user to run `/sbce apply <capability>` first.
3. Move the spec: `mv specs/<path>/ archive/specs/<path>/` (create `archive/specs/` if needed). Do **not** touch `src/main/java/<path>/` — the BC source remains.
4. Confirm the result as `from → to`.

Guard: archive only when the gate is green; **move, never delete**; BC source is never removed.

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
capability: airhacks.sbce.checkout
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

This maps to `src/main/java/airhacks/sbce/checkout/` with `placeOrder`/`cancelOrder` in the
boundary, and a test tracing each EARS statement under R1, R2.

## What NOT to do

- Don't hand-author a tasks file — the gap is read, not stored.
- Don't diff or merge specs — there is one spec per capability.
- Don't bake in a stack or a test runner — delegate code idioms and verification.
- Don't self-certify convergence — let the stack's tests speak.
- Don't delete on archive — `gc` moves, never deletes; BC source stays.
- Don't put *how* (types, transports, frameworks) into a spec.
- Don't split this into three skills or three commands — one skill, three modes.
- Don't build the `sbce` binary until a headless need is real.
