---
name: sbce
description: Spec-driven BCE workflow where one capability spec equals one business component (same name) and the spec is the boundary contract. Invoked as `/sbce new|apply <capability-or-feature>` (or by intent), it drives declare → converge; the stack's own test loop is the oracle for "done". `new` accepts a BC name or a natural-language feature description that may decompose into one or several BCs (new or existing). Stack-neutral — composes with `/bce` and a stack skill (`/java-cli-app`, `/microprofile-server`, `/web-components`, …) for code shape and verification. Use when authoring or converging a capability spec, declaring a feature as one or more BCs, mapping a spec to a BC, or running `/sbce new`, `/sbce apply`. Triggers on "SBCE", "capability spec", "spec-driven BCE", "declare a feature", "spec to BC", "converge to spec", "/sbce new", "/sbce apply".
---

Drive the spec-driven BCE workflow — one skill owns both the rules and the steps. Invoke as
`/sbce <mode> <capability>` (`new` or `apply`), or let it trigger from intent. Apply every rule
strictly.

## Guiding principles

- The spec **is** the boundary contract — *what the boundary promises*, never *how*.
- The spec **lives in the BC's package doc** — `package-info.java` (Java, `///` Markdown) or `package-info.md` (web) — co-located with the code it governs. There is no separate `specs/` tree and no hand-typed package coordinate. In Java the `///` doc renders via `javadoc`, so the same file is source-of-truth *and* published contract.
- One capability spec ≡ one business component, named the same. No translation between "what" and "where".
- The task list is the **gap**, read off `spec` vs `BC` on demand — never a hand-maintained tasks file.
- One spec per capability, the single source of truth — never diff or merge two specs.
- Split by determinism: tooling runs tests and places the BC; this skill writes the spec, the code, and closes the gap.
- "Done" is a green run of the stack's test loop — never your own opinion.

## Spec ↔ BC mapping

The **BC name** is the only identity — a single lowercase token (`checkout`), never a dotted path
or a `bc`/`capability` field. The stack skill owns where it lands:

```
checkout                                      # BC name == the only identity
  spec = package doc, co-located with code:
    Java: src/main/java/<base>/checkout/package-info.java   # `///` Markdown (JEP 467)
    web:  app/src/checkout/package-info.md                  # Markdown
  code = same folder, {boundary,control,entity}/            # stack skill places it
```

- `## Boundary` op ↔ one `boundary` entry-point method.
- `## Requirements` (EARS) ↔ behaviour **and** the tests covering it.
- `## Entities` ↔ the `entity` layer.

## System doc (base package)

An **optional** doc one altitude up — the base package's `package-info.java` /
`app/src/package-info.md` — for concerns that **span** BCs and have no other home. Add a section
only when a real cross-BC concern appears; a one-BC system needs none. Author from
`references/system-doc-template.md`.

- **Charter** — one sentence for the whole assembly.
- **Components** — this system's concrete wiring: which BC may call which, which integration events cross boundaries (`/bce` owns the generic layering; this owns the concrete dependencies).
- **System invariants** — cross-cutting EARS `shall` statements (id `Sn`) no single BC owns.
- **Ubiquitous language** — shared domain nouns defined once, so each BC's `## Entities` stays terse.
- **Stack** — the composed stack skill + package base, so `apply` reads it instead of re-inferring.

Never duplicate a BC's one-liner — a hand-typed BC index drifts; the gap is read, not stored. For
a BC map, mark it **generated** and regenerate it from the per-BC docs.

## Determinism boundary

| Concern | Owner | Deterministic? |
|---|---|---|
| Run tests / "is it green" | the **stack's verification loop** (the composed stack skill) | yes — existing tooling, no LLM |
| Decompose a feature into BCs (new vs existing) | this skill's judgment, over a read-only scan of the source tree's package docs, **user-confirmed** | no — semantic |
| Author the spec content (boundary ops, EARS requirements) | this skill's judgment | no — semantic |
| Place the BC — package doc + layer dirs at the source location | the composed stack skill (owns the package base / source root) | yes — stack-defined |
| Structural sync, **both directions** (spec→code: op→method, `Rn.m`→test · code→spec: method→op, test-id→statement, entity→`## Entities`) | this skill, made checkable by the stack's traceability convention | grep-level |
| "Does this code satisfy the requirement" | this skill's judgment, **grounded by the requirement's passing test** | no — semantic |

- Ask the stack skill "are you green?" — never name a runner or test kind, and never self-certify convergence.

## Invocation modes: new · apply

Read mode + capability from the invocation (`/sbce apply checkout`). If mode is missing, infer: no
spec yet → `new`; spec exists but not converged → `apply`; else ask. One skill, two modes — never
split it.

### new — declare

Declare a new feature from either a **BC name** (one precise BC) or a **natural-language feature
description** (which may decompose into one *or several* BCs, new or existing). The novelty is the
*intent*, not the artifact — coining a BC and extending one are both "new".

**Clarify first (both paths).** Resolve every ambiguity the contract needs before authoring. Vague
input (`"store a session with title, description, conference, date"`) hides scope and edge cases —
a guessed spec makes the oracle verify assumptions, not intent.

- **Loop, don't stop at one.** Each answer exposes new gaps; re-derive and re-ask until resolved. Never proceed on partial answers.
- **Never assume silently.** Lean on a default only by naming it — "I'd assume create-only; confirm or correct".
- **One ambiguity per question, specific over generic.** Offer enumerable options with an "other" escape; skip what context already answers; no meta-questions.
- **Interrogate per boundary op** — its trigger/response (event-driven), invalid/edge triggers (`If…then`), state constraints (`While…`) — and across the BC: scope (create-only vs full lifecycle, in/out), entities and fields (required/optional, identity, validation), and what "done" means.
- **Stop only when** every boundary op and EARS statement, happy *and* unhappy, is answerable from the user's words such that another engineer would author the same spec. If in doubt, ask one more.

**BC name** (`/sbce new checkout`):

1. Validate the name — a single lowercase token, no dots/spaces/uppercase. Reject otherwise.
2. Ask the stack skill where the package doc lives. If it exists, do **not** overwrite — report and stop unless the user confirms a rewrite.
3. Author the spec into the package doc from `references/spec-template.md`: one-line responsibility, boundary ops, EARS requirements, optional entities, out-of-scope.
4. Ask the stack skill to place the doc and scaffold empty `boundary`/`control`/`entity` dirs. Write **no** BC source.
5. Report the open gap (counts of ops / requirements) and point to `/sbce apply <bc-name>`.

**Feature description** (`/sbce new "let a customer check out a cart"`):

1. Scan existing BCs — read their package docs for responsibilities.
2. **Propose** a BC set — each tagged **new** (coin a verb-noun name) or **extend-existing**, each with the one-line responsibility it owns.
3. **Confirm the carving before any write** — decomposition has no test oracle, so the human approves the BC set.
4. Realise each entry via the BC-name steps: **new** → fresh doc + dirs; **extend-existing** → add ops / requirements to its **single** existing doc, never a second spec.
5. If the carving introduces cross-BC wiring (a call, a shared noun, a system invariant), record it in the system doc — user-confirmed.

Guard: output is always 1..N package-doc specs, each 1:1 with a BC; a feature is ephemeral input,
never persisted. Confirm the carving and any coined name before writing; never write BC source
here; never overwrite without confirmation.

### apply — converge

Make reality match the declared spec — the "make it so" step. Idempotent.

1. Locate the package doc (the spec); if missing, tell the user to run `/sbce new <bc-name>` first and stop.
2. Resolve the composed stack skill in order: the system doc's `Stack` line if present, else `AGENTS.md`/`README`, else ask once.
3. Run the stack's test loop. Green **and** no structural gap **in either direction** → stop and report "already converged".
4. Else read the gap — both directions — and close it:
   - **spec → code** (this skill closes it): each undeclared boundary op → a `boundary` method; each untested statement id `Rn.m` → a traceable test; then write code to pass them. Invoke `/bce` (invariants) + the stack skill (idioms).
   - **code → spec** (surface, never auto-author): a `boundary` method with no declared op, a test tracing an id no statement carries, an `entity` type absent from `## Entities` — report each as drift and stop on it. The spec is the source of truth, so the user decides: declare it (`/sbce new` / extend the doc) or delete the orphan. Never edit the spec to match code.
5. Re-run. Repeat 3–5, bounded to **≤3 passes**, then surface remaining failures/drift to the user.

Green build + no structural gap or drift is the only definition of done.

## Composition

- Own the **workflow** and the **spec↔BC mapping**; delegate everything else.
- BCE layering + naming bans (`*Impl`/`*Service`) → `/bce`. Code idioms + verification → the stack skill.
- Never duplicate or contradict `/bce` or the stack skill; if either conflicts with a spec, surface it, don't guess.
- Don't build the `sbce` binary until a headless need is real.

## Spec format rules

- Sections in order: `# Title` + one-line responsibility, `## Boundary`, `## Requirements`, optional `## Entities`, `## Out of scope`.
- Boundary operations are verb-noun and transport-neutral (`place-order`, not `POST /orders`).
- Requirements are [EARS](https://alistairmavin.com/ears/) statements — one of six patterns, the system always **the BC** — grouped under a titled `### Rn`, each statement carrying a stable id `Rn.m` (group `n`, statement `m`). Reach for `If…then` and `While…` to capture error and edge cases.

  | Pattern | Template |
  |---|---|
  | Ubiquitous | `The BC shall <response>.` |
  | State-driven | `While <precondition>, the BC shall <response>.` |
  | Event-driven | `When <trigger>, the BC shall <response>.` |
  | Optional-feature | `Where <feature is included>, the BC shall <response>.` |
  | Unwanted-behaviour | `If <trigger>, then the BC shall <response>.` |
  | Complex | `While <precondition>, when <trigger>, the BC shall <response>.` |

- Every statement uses `shall` and is mandatory **and** tested — no `SHOULD`/`MAY` gradation (the oracle is binary); express optionality with the `Where` (optional-feature) pattern.
- Ids are **stable**: never renumber on reorder; a removed id is retired, not reused.
- Every boundary op traces to a group `Rn`; every statement `Rn.m` traces to ≥1 test that **embeds its id** — per-statement and **bijective both ways**: a new `Rn.m` with no test is a gap, and a method, trace id, or `entity` type with no spec counterpart is inverse drift (surfaced, never absorbed into the spec). The trace form is the stack skill's call (an `r1_2…` method, an `@requirement("R1.2")` tag, a system-test or Playwright name); SBCE only requires the id be grep-visible.
- The `control` layer is implementation — pure *how* — so it has **no spec section**; only `## Boundary`, `## Requirements`, and `## Entities` map to code.
- Stack-neutral throughout: no types, transports, framework verbs, or *how*.

## Reference spec

In Java the spec is `src/main/java/airhacks/checkout/package-info.java`, written as `///` Markdown
(JEP 467):

```java
/// # Checkout
/// > Accept a cart and turn it into a confirmed, cancellable order.
///
/// ## Boundary
/// - `place-order` — submit a cart for fulfilment
/// - `cancel-order` — withdraw an unfulfilled order
///
/// ## Requirements
/// ### R1: Place an order
/// - R1.1 — When a cart with at least one item is submitted, the BC shall create and confirm an order.
/// - R1.2 — If the cart is empty, then the BC shall reject the request.
///
/// ### R2: Cancel an order
/// - R2.1 — While an order is unfulfilled, the BC shall allow it to be cancelled.
/// - R2.2 — If the order is already fulfilled, then the BC shall reject the cancellation.
///
/// ## Entities
/// - Order, Cart
///
/// ## Out of scope
/// - payment capture
/// - shipping
package airhacks.checkout;
```

The BC name `checkout` comes from the package — no frontmatter. The web equivalent is
`app/src/checkout/package-info.md` with the same Markdown, minus the `///` prefixes. The stack
skill places `boundary`/`control`/`entity` beside the doc with `placeOrder`/`cancelOrder` in the
boundary, and a test per statement id (`R1.1`, `R1.2`, `R2.1`, `R2.2`) whose trace embeds that id.
