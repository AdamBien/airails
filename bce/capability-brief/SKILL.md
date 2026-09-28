---
name: capability-brief
argument-hint: uc|story|review <description-or-brief-file>
description: Capture one capability, or one addition to an existing capability, as a use case or a user story complete enough that `/sbce new` authors the spec in one shot without asking a question. Runs the clarifying interview upstream, at authoring time with the person who knows the domain, checks the result against a completeness checklist, and emits the filled template. Invoked as `/capability-brief uc|story|review <description-or-file>`, or with the mode omitted so the skill picks the format. Stack-neutral — what the system promises, never how it is built. Use whenever someone wants to write, refine, review, or complete a use case, user story, acceptance criteria, Given/When/Then scenarios, or a feature brief for a capability; wants to prepare input for `/sbce new`; or asks "what is missing from this story", "is this use case complete", "turn these notes into a use case". Triggers on "use case", "user story", "story", "acceptance criteria", "Given/When/Then", "main scenario", "extensions", "happy path", "capability brief", "prepare for sbce", "before sbce new". Not for authoring the spec or package doc itself (`/sbce`), and not for tests (`/ears-tests`).
---

Produce a **brief**: one filled template that answers every question `/sbce new` would otherwise
ask while authoring a capability spec. The interview `new` runs against a vague feature description
is moved here, to the moment the domain owner is present, and its answers are recorded once. `new`
then authors instead of asks.

The brief is stack-neutral: what the system promises, never how it is built. No types, transports,
frameworks, endpoints, or file names — the stack skill owns those, and a brief that names them
would bind the spec to an implementation before the spec exists.

## Two guarantees, one of them yours

- **Structural completeness is yours to enforce.** Every section filled, every actor step named as an operation, every input with its parts, every operation with its failure cases. The checklist below makes this mechanical; do not emit a brief that fails it.
- **Semantic completeness is the human's.** The result shape, which parts of an input are required, what a second identical request does, which alternative was rejected — these are domain facts. You cannot invent them; you can only ask until they are answered or honestly parked under `## Open issues`. A silently guessed answer becomes a spec statement, then a test, then code that verifies your guess instead of their intent.

## Invocation modes: uc · story · review

Read the mode and the input from the invocation: `/capability-brief uc "<description>"`,
`/capability-brief story "<description>"`, `/capability-brief review <file-or-pasted-brief>`.
With the mode omitted, pick the format by the routing rule below and say which one you chose.

| Mode | Brief | When | Template |
|---|---|---|---|
| `uc` | Use case | a new capability; several steps or actors; state kept across calls | [references/use-case-template.md](references/use-case-template.md) |
| `story` | User story | one behaviour added to an existing capability; a new capability with a single operation | [references/user-story-template.md](references/user-story-template.md) |
| `review` | either | an existing brief to check; the format is the one the brief already has | — |

Routing rule when no mode is given, in order: kept state between the actor's requests → use case;
the actor calls the system more than once to reach the goal → use case; an addition to a capability
whose package doc exists → story. Route by the shape the input has *now*, and switch when it
changes: a story that grows a kept selection or a second actor mid-interview has become a use case
— say so and carry the answers over. Both formats end in the same checklist and the same hand-off,
so the switch costs nothing.

An explicit mode is the author's decision, so respect it: when the input outgrows the chosen
format, warn instead of switching — "this keeps a selection across calls; the story hides that
lifecycle, the use case captures it: continue as story or switch?" — and do what the author
answers. `story` means story until they say otherwise.

`review` runs the checklist over the given brief, asks only for the rows that fail, and returns the
completed brief in the same format. It never reopens answered sections.

## Interview

1. **Fill from the user's words first.** Place everything they said into the template before asking anything; a question about something already answered wastes the domain owner's attention.
2. **Scan the repository, read-only**, before the first question:
   - existing package docs (`package-info.java` / `package-info.md`) → the capabilities that exist, their operations and entities; decides `kind: extends` versus `new`, and stops you from coining an operation that exists;
   - the system doc's `## Decisions` → never propose an alternative a `Dn` records as rejected; the `## Ubiquitous language` → reuse its nouns;
   - whether a stack is declared (system doc `Stack` line, `AGENTS.md`, `README.md`) → if not, `/sbce new` will have to ask for it; put "stack not declared" under `## Open issues` so the author knows it is not theirs to answer.
3. **Ask one gap per question**, specific over generic, with enumerable options and an "other" escape. When you propose a default, name it — "I'd assume at most 50 results, most relevant first; confirm or correct". Skip what context already answers. No meta-questions about the process.
4. **Ask what only the human knows**: the result's shape, order, limit and empty answer; each input's parts and which are required; what makes an input invalid; the response when an external system is unavailable; what a repeated request does; who ends kept state and when; the alternatives a decision rejected.
5. **Answer yourself what is derivable**: operation names (verb-noun from the actor's step), the format, the `capability`/`external` tag of a supporting actor when a package doc names it, the section a fact belongs to.
6. **Loop.** Each answer can expose the next gap; re-run the checklist after every round. Stop when it passes.
7. **Park, never guess.** A question the author cannot or will not answer goes under `## Open issues` in their words. That section is the only thing `/sbce new` will ask, so it must hold every unanswered question and no answered one.
8. **Refuse the how.** If the author offers a type, transport, framework, or file name, keep it out of the brief. If it is really a choice made against alternatives, record the choice in domain terms under `## Decisions`; otherwise drop it and say so.

## Completeness checklist

Run this before emitting; every row maps to a part of the spec that would otherwise be guessed.

| Check | Feeds |
|---|---|
| Capability name is one lowercase word; `kind` is `new` or `extends` a capability whose package doc exists | the spec's identity |
| `responsibility` is one sentence from the system's side | the spec's `>` line |
| Every step where the primary actor addresses the system names a verb-noun operation; system-only steps name none | `## Boundary`, one op each |
| Every input an operation carries lists its parts, the required ones, and what makes it invalid | the `If…then` rows per operation |
| The last step, or the happy `Then`, states the result's shape, order, limit with truncation wording, and the empty answer | the assert of the happy `When` |
| Per operation the extensions cover: invalid input, nothing to act on, absent target, external system unavailable, repeated request where it changes state, actor stops responding; optional paths are marked | `If…then`, `Where…` |
| Every supporting actor is tagged `capability` (naming it) or `external` | the system doc's `## Components` wiring |
| Lifecycle is `none` or states what starts kept state, what is reused, and when and how it ends | `While…` |
| Entities is `none` or lists the stateful nouns this capability owns, names only | `## Entities` |
| Out of scope lists what was weighed and excluded | closes scope questions |
| Every decision reads `record: <choice> _(rejected: …)_`; else `none` | the `Dn` log |
| Open issues holds only unanswered questions | the only questions `new` asks |
| No type, transport, framework, endpoint, or file name anywhere | stack neutrality |

## Output

- Emit the filled body exactly in the template's section order, header comment removed, ending with the exact hand-off to run next: `/sbce new <file>` when the brief was written to a file, else `/sbce new` on the pasted body. Two commands, not one: the pause between them is where the domain owner reads the brief before it becomes a spec.
- Print it in the reply. Write it to a file only when asked, where the author says; a sensible default is `<capability>-brief.md` beside the repository `README.md`.
- The brief is **inception input, not a source of truth** — the same status as the README seed `/sbce new` reads. Once the spec exists in the package doc, the spec is authoritative and the brief is not kept in sync.

## Composition

- `/sbce` owns the spec format and the `new` step; its SKILL.md states that a filled brief counts as the completed clarify loop and that only `## Open issues` is asked. Never author the spec here.
- `/ears-tests` owns tests; a brief never contains test wording.
- The stack skill is not consulted — the brief is upstream of any stack; the only stack-related act is the declaration check in the interview.
