---
name: sbce
argument-hint: new|apply <capability-or-feature>
description: Spec-driven BCE workflow where one capability spec equals one business component (same name) and the spec is the boundary contract. Invoked as `/sbce new|apply <capability-or-feature>` (or by intent), it drives declare → converge; the stack's own test loop is the oracle for "done". `new` accepts a BC name or a natural-language feature description that may decompose into one or several BCs (new or existing). Stack-neutral — composes with `/bce` and a stack skill (`/java-cli-app`, `/microprofile-server`, `/web-components`, …) for code shape and verification. Use when authoring or converging a capability spec, declaring a feature as one or more BCs, mapping a spec to a BC, or running `/sbce new`, `/sbce apply`. Triggers on "SBCE", "capability spec", "spec-driven BCE", "declare a feature", "spec to BC", "converge to spec", "/sbce new", "/sbce apply".
---

Drive the spec-driven BCE workflow — one skill owns both the rules and the steps. Invoke as
`/sbce <mode> <capability>` (`new` or `apply`), or let it trigger from intent. Apply every rule
strictly.

## Guiding principles

- The spec **is** the boundary contract — *what the boundary promises*, never *how*.
- The spec **lives in the BC's package doc** (`package-info.java` with `///` Markdown, or `package-info.md` for web), co-located with the code it governs — no separate `specs/` tree. In Java `javadoc` renders it, so one file is source-of-truth *and* published contract.
- One capability spec ≡ one business component, named the same. No translation between "what" and "where".
- The task list is the **gap**, read off `spec` vs `BC` on demand — never a hand-maintained tasks file.
- One spec per capability, the single source of truth — never diff or merge two specs.
- Split by determinism: tooling runs tests and places the BC; this skill writes the spec, the code, and closes the gap.
- "Done" is a green run of the stack's test loop — never your own opinion.

## Spec ↔ BC mapping

The **BC name** is the only identity — a single lowercase noun (`checkout`), valid as a package
segment: never a dotted path, never a `bc`/`capability` field, never verb-noun (that is the shape of
a boundary *op*). The stack skill owns where it lands:

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
`references/system-doc-template.md`, which carries the per-section rules. Sections:

- **Mission** — the `>` one-liner under the title, the only mandatory line.
- **Vision** *(optional)* — one aspirational sentence. Rationale, not contract: no `Sn`, no test, the single non-verifiable line in the doc. May be proposed by `/sbce new` from a README seed; the human accepts or edits.
- **Components** — this system's concrete wiring: which BC may call which, which integration events cross boundaries (`/bce` owns the generic layering).
- **System invariants** — cross-cutting EARS `shall` statements, id `Sn`, each tested.
- **Ubiquitous language** — shared domain nouns defined once, so each BC's `## Entities` stays terse.
- **Decisions** *(optional)* — append-only `Dn — <choice>. _(why: …; rejected: …)_`, superseded never edited; rationale, no test. Litmus: testable behaviour → EARS statement; standing project rule → README `## Conventions`; point-in-time choice with rejected alternatives → `Dn`. A one-BC decision lives in that BC's doc under the same rules.
- **References** *(optional)* — external specs several BCs conform to; same rules as a BC's `## References`.
- **Stack** — the composed stack skill + package base, so neither mode re-infers it.

Never duplicate a BC's one-liner — a hand-typed BC index drifts; the gap is read, not stored. A BC
map is **generated** from the per-BC docs.

## Top-level README (optional projection)

An **optional** repo-root `README.md` — a human on-ramp that is a **projection of the specs, not a
source of truth**. Author from `references/readme-template.md`. Two slices, handled oppositely:

- **Generated** (never hand-edited), fenced by `<!-- sbce:generated:start -->` / `<!-- sbce:generated:end -->`: Mission + Vision, a BC map (name, `>` one-liner, link to its `package-info`), and a Mermaid `flowchart` of the **declared** `## Components` wiring — projection, never inference from code; no `## Components` → nodes only, or omit. `apply` regenerates this slice; no markers → README untouched.
- **Hand-maintained** (outside the markers, so it can't drift): `## Conventions` — project-specific, **non-behavioral** standards ("money is always cents"), declared not verified, no `Sn`, distinct from a tested system invariant and from a `Dn` — plus build/run/test delegated to the stack skill and free-form meta. This prose is also the **inception seed** `/sbce new` (no argument) reads, never rewrites.

## Determinism boundary

| Concern | Owner | Deterministic? |
|---|---|---|
| Run tests / "is it green" | the **stack's verification loop** (the composed stack skill) | yes — existing tooling, no LLM |
| Decompose a feature into BCs | this skill, over a read-only scan of the package docs, **user-confirmed** | no — semantic |
| Record a confirmed choice as a `Dn` | this skill offers, **user-confirmed** — never recorded silently | no — semantic |
| Author the spec content | this skill's judgment | no — semantic |
| Place the BC — package doc + layer dirs | the stack skill (owns the source root) | yes — stack-defined |
| Structural sync, **both directions** (spec→code: op→method, `Rn.m`→test · code→spec: method→op, trace id→statement, entity→`## Entities`) | this skill, checkable via the `Rn.m` trace form `/ears-tests` defines | grep-level |
| "Does this code satisfy the requirement" | this skill, **grounded by the statement's passing test** | no — semantic |
| Regenerate the README generated block from the package docs | this skill | yes — mechanical projection |

- Ask the stack skill "are you green?" — never name a runner or test kind, and never self-certify convergence.

## Resolving the stack

Both modes need the composed stack skill and the package base before touching a file. Resolve in
order: the system doc's `Stack` line if present, else `AGENTS.md`/`README`, else ask once (a brief
may have parked "stack not declared" under `## Open issues`). Read the system doc's `## Decisions`
if present — never propose or implement an alternative a `Dn` records as rejected. The stack skill
owns the source root: ask it where the package doc lives and to place the doc and layer dirs.

## Invocation modes: new · apply

Read mode + capability from the invocation (`/sbce apply checkout`). If mode is missing, infer: no
spec yet → `new`; spec exists but not converged → `apply`; else ask.

### new — declare

Declare a new feature from a **BC name** (one precise BC), a **natural-language feature
description** (which may decompose into one *or several* BCs, new or existing), the **repo README
seed** (`/sbce new` with no argument), or a **brief**. The novelty is the *intent*, not the
artifact — coining a BC and extending one are both "new".

**Clarify first (every path).** Resolve every ambiguity the contract needs before authoring — a
guessed spec makes the oracle verify assumptions, not intent.

- **Loop, don't stop at one.** Each answer exposes new gaps; re-derive and re-ask until resolved. Never proceed on partial answers.
- **Never assume silently.** Lean on a default only by naming it — "I'd assume create-only; confirm or correct".
- **One ambiguity per question, specific over generic.** Offer enumerable options with an "other" escape; skip what context already answers; no meta-questions.
- **Interrogate per boundary op** — trigger/response (event-driven), invalid/edge triggers (`If…then`), state constraints (`While…`) — and across the BC: scope (create-only vs full lifecycle), entities and fields (required/optional, identity, validation), and what "done" means.
- **Stop only when** every boundary op and EARS statement, happy *and* unhappy, is answerable from the user's words such that another engineer would author the same spec. If in doubt, ask one more.

**Brief input — the loop already ran.** A use case or user story authored with `/capability-brief`
(recognisable by its `## Capability` … `## Open issues` sections) is the clarify loop run upstream
with the domain owner, so its answers are the user's confirmed word: `## Capability` is the
confirmed carving (name, `new`/`extends`, responsibility) — it replaces the carving confirmation
below; each `record:` line under `## Decisions` is a confirmed `Dn` to log without offering it
again; a supporting actor tagged `capability` is declared cross-BC wiring for the system doc;
`## Out of scope` closes the scope questions. Ask exactly what `## Open issues` lists plus any
section left blank — nothing else — then author from the brief's own section→spec mapping: named
operations → `## Boundary`; `Inputs`, `Extensions` and failing `Acceptance criteria` scenarios →
`If…then` (an invalid input is rejected naming every invalid part unless an extension or scenario
states another response); optional paths → `Where…`; `Lifecycle` → `While…`; `Entities` →
`## Entities`. The brief is inception input, not a source of truth: once the spec exists, the spec
is authoritative.

**System brief input — the carving already ran.** A system brief authored with `/system-brief`
(recognisable by its `## System` … `## Open issues` sections) is the stakeholder conversation
about the whole assembly, signed off upstream: `## Capabilities` is the confirmed carving — it
replaces the carving confirmation below, each line tagged `new`, `extends` or `existing`;
`## System` is the proposed mission and vision for the system doc (the human accepts or edits the
vision, `## Success measures` is its rationale); `## Interactions`, `## Language` and
`## Invariants` seed `## Components`, `## Ubiquitous language` and `## System invariants` —
invariants arrive in domain terms and are authored here as EARS `Sn` statements; each `record:`
under `## Decisions` is a confirmed `Dn`. The system brief carries no capability's inside, so each
`new` or `extends` capability still needs its clarify loop: prefer the capability briefs the system
brief hands off to, and run the feature-description interview only for one without. Like every
brief it is inception input, not a source of truth.

**BC name** (`/sbce new checkout`):

1. Validate the name — a single lowercase noun, no dots/hyphens/spaces/uppercase. Reject otherwise.
2. Ask the stack skill where the package doc lives. If it exists, do **not** overwrite — report and stop unless the user confirms a rewrite.
3. Author the spec into the package doc from `references/spec-template.md`: one-line responsibility, boundary ops, EARS requirements, optional entities, out-of-scope.
4. Ask the stack skill to place the doc and scaffold empty `boundary`/`control`/`entity` dirs. Write **no** BC source.
5. Report the open gap (counts of ops / requirements) and point to `/sbce apply <bc-name>`.

**Feature description** (`/sbce new "let a customer check out a cart"`):

1. Scan existing BCs — read their package docs for responsibilities (and the system doc's `## Decisions`, see above).
2. **Propose** a BC set — each tagged **new** (coin a single lowercase noun) or **extend-existing**, each with the one-line responsibility it owns.
3. **Confirm the carving before any write** — decomposition has no test oracle, so the human approves the BC set (a brief's `## Capability` / `## Capabilities` is that approval).
4. Realise each entry via the BC-name steps: **new** → fresh doc + dirs; **extend-existing** → add ops / requirements to its **single** existing doc, never a second spec.
5. If the carving introduces cross-BC wiring (a call, a shared noun, a system invariant), record it in the system doc — user-confirmed.
6. A carving or stack choice the user confirmed **against** a proposed alternative is a decision — offer to record it as a `Dn`; never record one silently (a brief's `record:` lines are already confirmed).

**README seed** (`/sbce new`, no argument):

1. Read only the hand-written prose **outside** the `sbce:generated` markers and treat it as the feature description: scan → propose carving → confirm → author, with the clarify loop filling every gap the prose leaves.
2. Additionally **propose a `## Vision` line** distilled from the seed; the human accepts or edits it — never author it silently.
3. The seed is inception input, not a source of truth: once specs exist they are authoritative; the seed stays human-owned and is not kept in sync. Re-running `/sbce new` re-reads it and re-proposes through the same confirm-first path.

Guard: one capability ≡ one BC — output is 1..N package-doc specs, never a persisted feature artifact.

### apply — converge

Make reality match the declared spec — the "make it so" step. Idempotent.

1. Locate the package doc (the spec); if missing, tell the user to run `/sbce new <bc-name>` first and stop.
2. Resolve the stack (see above).
3. Run the stack's test loop. Green **and** no structural gap **in either direction** → report "already converged" and go to 6.
4. Else read the gap — both directions — and close it:
   - **spec → code** (this skill closes it): each undeclared boundary op → a `boundary` method; each untested statement id `Rn.m` → a traceable test via `/ears-tests` (one parameterized test per `### Rn`, one labeled row per `Rn.m`); then write code to pass them. Invoke `/bce` (invariants) + the stack skill (idioms).
   - **code → spec** (surface, never auto-author): a `boundary` method with no declared op, a test tracing an id no statement carries, an `entity` type absent from `## Entities` — report each as drift and stop on it. The user decides: declare it (`/sbce new` / extend the doc) or delete the orphan. Never edit the spec to match code.
5. Re-run. Repeat 3–5, bounded to **≤3 passes**, then surface remaining failures/drift to the user.
6. If the README carries `sbce:generated` markers, regenerate that slice from the system doc + package docs; touch nothing outside the markers.

Green build + no structural gap or drift is the only definition of done.

## Composition

- Own the **workflow** and the **spec↔BC mapping**; delegate everything else.
- BCE layering + naming bans (`*Impl`/`*Service`) → `/bce`. EARS→test transform and the `Rn.m` trace form → `/ears-tests`. Code idioms, placement + verification → the stack skill.
- Never duplicate or contradict `/bce`, `/ears-tests` or the stack skill; if any conflicts with a spec, surface it, don't guess.

## Spec format rules

- Sections in order: `# Title` + one-line responsibility, `## Boundary`, `## Requirements`, optional `## Entities`, optional `## Decisions` (BC-local, same rules as the system doc's), optional `## References`, `## Out of scope`.
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

- Every statement uses `shall` and is mandatory **and** tested — no `SHOULD`/`MAY` gradation (the oracle is binary); express optionality with the `Where` pattern.
- Ids are **stable**: never renumber on reorder; a removed id is retired, not reused.
- Every boundary op traces to a group `Rn`; every statement `Rn.m` traces to ≥1 test that **embeds its id**, checked in both directions: a new `Rn.m` with no test is a gap; a method, trace id, or `entity` type with no spec counterpart is drift (surfaced, never absorbed into the spec). `/ears-tests` defines the trace form per stack; SBCE only requires the id be grep-visible.
- **Optional `why`.** Any `Rn.m` statement (or boundary op) may carry a trailing `_(why: …)_` — terse *origin/intent*. Rationale, not contract: non-verified, not a trace target, bound to its id (retires with it; the `Rn.m` prefix stays first). Capture *why the rule exists*, never *how it currently works*.
- **Optional `## References`.** External specifications the BC implements (RFC, Jakarta/MicroProfile chapter, regulation) — format and rules in the spec template. Rationale, not contract: a reference **informs** requirements, never replaces them — behaviour is still a tested `Rn.m`, never an untested "shall conform to X". The stack skill may read the linked document while closing the gap.
- The `control` layer is implementation — pure *how* — so it has **no spec section**; only `## Boundary`, `## Requirements`, and `## Entities` map to code.
- Stack-neutral throughout: no types, transports, framework verbs, or *how*.

## Reference spec

The worked example — a `checkout` BC — lives in `references/spec-template.md`: no frontmatter, the
name comes from the folder; `place-order` → `placeOrder`; `R1.1` → a test whose trace embeds it.
