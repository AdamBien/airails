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
- **Vision** *(optional)* — one aspirational sentence: the outcome the assembly chases. Rationale, not contract — no `Sn`, no test; the single non-verifiable line in the doc, and the deliberate exception to the traceability invariant. May be proposed by `/sbce new` distilling a README seed (human accepts/edits).
- **Components** — this system's concrete wiring: which BC may call which, which integration events cross boundaries (`/bce` owns the generic layering; this owns the concrete dependencies).
- **System invariants** — cross-cutting EARS `shall` statements (id `Sn`) no single BC owns.
- **Ubiquitous language** — shared domain nouns defined once, so each BC's `## Entities` stays terse.
- **Stack** — the composed stack skill + package base, so `apply` reads it instead of re-inferring.

Never duplicate a BC's one-liner — a hand-typed BC index drifts; the gap is read, not stored. For
a BC map, mark it **generated** and regenerate it from the per-BC docs.

## Top-level README (optional projection)

An **optional** repo-root `README.md` — a human on-ramp that is a **projection of the specs, not a
source of truth**. Author from `references/readme-template.md`. Two slices, handled oppositely:

- **Generated** (never hand-edited) — the system doc's Charter + Vision + a BC map (each BC name, its `>` one-liner, a link to its `package-info`), fenced by `<!-- sbce:generated:start -->` / `<!-- sbce:generated:end -->`. This is the "mark it generated, regenerate from the per-BC docs" rule made concrete at repo altitude.
- **Hand-maintained** (outside the markers, since no spec covers it — so it can't drift): `## Conventions`, build/run/test delegated to the stack skill, plus free-form meta (license, links, motivation).

- Never hand-write capability / charter / vision / BC content **into the generated block** — it is projected from the specs; hand-typing it there recreates the drift SBCE bans. (Free-form vision/intent framing in the seed prose *outside* the markers is fine — it seeds, it doesn't duplicate the canonical line.)
- **Doubles as the inception seed.** The hand-written prose outside the markers is what `/sbce new` (no argument) reads to bootstrap vision + specs (see `new`); SBCE reads it, never rewrites it.
- `## Conventions` is the home for **project-specific, non-behavioral standards** (coverage target, "money is always cents", review policy): **declared, not verified** — no `Sn`, no test — and distinct from a `System invariant`, which must be behavioral *and* tested.
- Optional: a one-BC project needs none. No markers → `apply` leaves the README untouched.

## Determinism boundary

| Concern | Owner | Deterministic? |
|---|---|---|
| Run tests / "is it green" | the **stack's verification loop** (the composed stack skill) | yes — existing tooling, no LLM |
| Decompose a feature into BCs (new vs existing) | this skill's judgment, over a read-only scan of the source tree's package docs, **user-confirmed** | no — semantic |
| Author the spec content (boundary ops, EARS requirements) | this skill's judgment | no — semantic |
| Place the BC — package doc + layer dirs at the source location | the composed stack skill (owns the package base / source root) | yes — stack-defined |
| Structural sync, **both directions** (spec→code: op→method, `Rn.m`→test · code→spec: method→op, test-id→statement, entity→`## Entities`) | this skill, made checkable by the stack's traceability convention | grep-level |
| "Does this code satisfy the requirement" | this skill's judgment, **grounded by the requirement's passing test** | no — semantic |
| Regenerate the README generated block (Charter/Vision/BC map) from the package docs | this skill | yes — mechanical projection |

- Ask the stack skill "are you green?" — never name a runner or test kind, and never self-certify convergence.

## Invocation modes: new · apply

Read mode + capability from the invocation (`/sbce apply checkout`). If mode is missing, infer: no
spec yet → `new`; spec exists but not converged → `apply`; else ask. One skill, two modes — never
split it.

### new — declare

Declare a new feature from a **BC name** (one precise BC), a **natural-language feature
description** (which may decompose into one *or several* BCs, new or existing), or the **repo
README seed** (`/sbce new` with no argument). The novelty is the *intent*, not the artifact —
coining a BC and extending one are both "new".

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

**README seed** (`/sbce new`, no argument):

The repo-root `README.md` doubles as an optional **inception seed** — a human (often a product
owner or analyst) writes free-form intent there and SBCE bootstraps from it. Read only the
hand-written prose **outside** the `sbce:generated` markers.

1. Treat the seed prose as the **feature description** and run the feature-description steps above (scan → propose carving → confirm → author), with the clarify loop filling every gap the prose leaves.
2. Additionally **propose a `## Vision` line** distilled from the seed; the human accepts or edits it — never author it silently.
3. The seed is **inception input, not a source of truth**: once specs exist they are authoritative; the seed prose stays human-owned and is not kept in sync. Re-running `/sbce new` simply re-reads it and re-proposes through the same confirm-first path.

Guard: one capability ≡ one BC — output is 1..N package-doc specs, never a persisted feature artifact.

### apply — converge

Make reality match the declared spec — the "make it so" step. Idempotent.

1. Locate the package doc (the spec); if missing, tell the user to run `/sbce new <bc-name>` first and stop.
2. Resolve the composed stack skill in order: the system doc's `Stack` line if present, else `AGENTS.md`/`README`, else ask once.
3. Run the stack's test loop. Green **and** no structural gap **in either direction** → stop and report "already converged".
4. Else read the gap — both directions — and close it:
   - **spec → code** (this skill closes it): each undeclared boundary op → a `boundary` method; each untested statement id `Rn.m` → a traceable test (delegate the EARS→table transform to `/ears-tests` — one parameterized test per `### Rn`, one labeled row per `Rn.m`); then write code to pass them. Invoke `/bce` (invariants) + the stack skill (idioms).
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
- Every boundary op traces to a group `Rn`; every statement `Rn.m` traces to ≥1 test that **embeds its id** — per-statement and **bijective both ways**: a new `Rn.m` with no test is a gap, and a method, trace id, or `entity` type with no spec counterpart is inverse drift (surfaced, never absorbed into the spec). The trace form is the stack skill's call (an `r1_2…` method, a `@requirement R1.2` JavaDoc tag on the test, a system-test or Playwright name); SBCE only requires the id be grep-visible.
- **Optional `why`.** Any `Rn.m` statement (or boundary op) may carry a trailing `_(why: …)_` — terse *origin/intent* for the rule. **Rationale, not contract**: non-verified (the oracle ignores it), **not a trace target** (the statement still needs its test; a `why` is never drift), and **bound to its id** (retires with the statement, never orphans; the `Rn.m` prefix stays first so id-grep is unaffected). Capture *why the rule exists*, never *how it currently works* — immutable origin, not a description to re-sync.
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
/// - R1.2 — If the cart is empty, then the BC shall reject the request. _(why: empty carts were the top source of phantom orders)_
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
