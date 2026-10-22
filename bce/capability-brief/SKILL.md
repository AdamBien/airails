---
name: capability-brief
argument-hint: uc|story|review <description, notes, or brief file>
description: Capture one capability, or one addition to an existing capability, as a use case or a user story complete enough that `/sbce new` authors the spec in one shot without asking a question. Runs the clarifying interview upstream, checks the result against a completeness checklist, and emits the filled template. Accepts any text naming one outcome the system should deliver — a sentence, notes, an email, a requirements document — and always emits a use case or user story. Modes uc, story, and review; with the mode omitted the skill picks the format. Stack-neutral — what the system promises, never how it is built. Use whenever someone wants to write, refine, review, or complete a use case, user story, acceptance criteria, Given/When/Then scenarios, or a feature brief; wants to prepare input for `/sbce new`; or asks "what is missing from this story", "is this use case complete", "turn these notes into a use case", "turn this requirements document into a brief". Triggers on "use case", "user story", "acceptance criteria", "Given/When/Then", "main scenario", "happy path", "capability brief", "prepare for sbce". Not for authoring the spec or package doc itself (`/sbce`), and not for tests (`/ears-tests`).
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
`/capability-brief story "<description>"`, `/capability-brief review <file-or-pasted-text>`.
With the mode omitted, pick the format by the routing rule below and say which one you chose.

| Mode | Brief | When | Template |
|---|---|---|---|
| `uc` | Use case | a new capability; several steps or actors; state kept across calls | [references/use-case-template.md](references/use-case-template.md) |
| `story` | User story | one behaviour added to an existing capability; a new capability with a single operation | [references/user-story-template.md](references/user-story-template.md) |
| `review` | either | an existing brief to check, in the format it already has; or existing text — notes, an email, a requirements document — to complete into a brief | — |

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
completed brief in the same format. It never reopens answered sections. Given text that is not a
brief, `review` routes by the rule above, fills the template from the text, and continues the same
way: only the failing rows are asked.

## Input

Any text is accepted — a sentence, notes, an email, a requirements document, pasted or as a file.
The minimum is **one outcome the system should deliver**: "employees request leave and their
manager decides" is enough. Actors, inputs, result shape and failures are what the interview is
for; a missing one is a question, never a reason to refuse. The input's form never lowers the
checklist; it only changes how many questions the interview asks. The brief is emitted as a use
case or a user story, whatever the input's form — `/sbce new` reads only those.

| Input | Response |
|---|---|
| names only a solution — "add a message queue for orders", "we need an endpoint for X" | ask what someone should be able to do once it exists; continue from the answer |
| spans several capabilities — "the whole HR portal" | list the capabilities found and ask which one to brief; the others go under `## Out of scope` or into later briefs |
| describes no system behaviour, or is a bug report, a refactoring, or a technical task | say it is not a capability, and stop |

Long text is the risk, not short text. A one-liner is visibly incomplete and gets asked; a document
looks complete and tempts you to read answers into it — "show the matching products" into an order
and a limit, "the manager approves" into a timeout. Each such reading fills a section, passes the
structural checklist, and becomes a guessed spec statement. The interview rules below and the
provenance row of the checklist close that gap.

## Interview

1. **Fill from the user's words first.** Place everything they said into the template before asking anything; a question about something already answered wastes the domain owner's attention. Fill only what the text states: every entry is quoted or closely paraphrased from the input, or later confirmed in the interview. A value the text does not state is a proposed default to ask, never an entry. Vague wording — "a few", "fast", "the usual checks", "as before", "etc." — fills nothing; it is a gap. Two passages that contradict each other become one question quoting both.
2. **Scan the repository, read-only**, before the first question:
   - existing package docs (`package-info.java` / `package-info.md`) → the capabilities that exist, their operations and entities; decides `kind: extends` versus `new`, and stops you from coining an operation that exists;
   - the system doc's `## Decisions` → never propose an alternative a `Dn` records as rejected; the `## Ubiquitous language` → reuse its nouns;
   - whether a stack is declared (system doc `Stack` line, `AGENTS.md`, `README.md`) → if not, `/sbce new` will have to ask for it; put "stack not declared" under `## Open issues` so the author knows it is not theirs to answer.
3. **Read back what was taken from the input** before the first question, grouped by template section, so the author corrects a misreading before it counts as an answer. For a one-sentence input this is one line.
4. **Report what the scan found** before the first question, in a short list the author can correct — the author may not know what the system already does:
   - **existing functionality** — a capability or operation whose responsibility overlaps the request, named with its package: "`shop` already offers `findProducts`; this reads as an extension of `shop`, not a new capability";
   - **reuse candidates** — an existing operation or entity the new capability could rely on instead of re-declaring, offered as a supporting actor tagged `capability`;
   - **overloaded terms** — a word of the author's that already means something else in a package doc or the ubiquitous language ("order" as a purchase versus a sort order), two words for one thing (the author's "basket", the system's "cart"), or a proposed capability, operation or entity name already taken.

   Each finding becomes a question with options — reuse or extend the existing capability / keep it separate; adopt the existing term / rename / keep both meanings. A rejected reuse or a deliberately kept second meaning is a choice against an alternative: record it under `## Decisions`. With no package doc and no ubiquitous language, say "greenfield, nothing to reuse" and continue.
5. **Ask one gap per question**, specific over generic, with enumerable options and an "other" escape. When you propose a default, name it — "I'd assume at most 50 results, most relevant first; confirm or correct". Skip what context already answers. No meta-questions about the process.
6. **Ask what only the human knows**: the result's shape, order, limit and empty answer; each input's parts and which are required; what makes an input invalid; the response when an external system is unavailable; what a repeated request does; who ends kept state and when; the alternatives a decision rejected.
7. **Answer yourself what is derivable**: operation names (verb-noun from the actor's step), the format, the `capability`/`external` tag of a supporting actor when a package doc names it, the section a fact belongs to.
8. **Loop.** Each answer can expose the next gap; re-run the checklist after every round. Stop when it passes.
9. **Park, never guess.** A question the author cannot or will not answer goes under `## Open issues` in their words. That section is the only thing `/sbce new` will ask, so it must hold every unanswered question and no answered one.
10. **Refuse the how.** If the author offers a type, transport, framework, or file name, keep it out of the brief. If it is really a choice made against alternatives, record the choice in domain terms under `## Decisions`; otherwise drop it and say so.

## Completeness checklist

Run this before emitting; every row maps to a part of the spec that would otherwise be guessed.

| Check | Feeds |
|---|---|
| Capability name is one lowercase word; `kind` is `new` or `extends` a capability whose package doc exists | the spec's identity |
| `responsibility` is one sentence from the system's side | the spec's `>` line |
| No responsibility overlaps an existing capability unless `kind: extends` it, it is a supporting actor, or the overlap is rejected under Decisions; no name clashes with an existing capability, operation or entity; every noun matches the ubiquitous language or is new, and a second meaning is recorded under Decisions | no duplicate BC, no overloaded term in the spec |
| Every step where the primary actor addresses the system names a verb-noun operation; system-only steps name none | `## Boundary`, one op each |
| Every input an operation carries lists its parts, the required ones, and what makes it invalid; its rejection names every invalid part unless an extension or scenario states another response | the `If…then` rows per operation |
| The last step, or the happy `Then`, states the result's shape, order, limit with truncation wording, and the empty answer | the assert of the happy `When` |
| Per operation the extensions cover: nothing to act on, absent target, external system unavailable, repeated request where it changes state, actor stops responding; optional paths are marked | `If…then`, `Where…` |
| Every supporting actor is tagged `capability` (naming it) or `external` | the system doc's `## Components` wiring |
| Lifecycle is `none` or states what starts kept state, what is reused, and when and how it ends | `While…` |
| Entities is `none` or lists the stateful nouns this capability owns, names only | `## Entities` |
| Out of scope lists what was weighed and excluded | closes scope questions |
| Every decision reads `record: <choice> _(rejected: …)_`; else `none` | the `Dn` log |
| Every entry is stated in the input or confirmed in the interview; no value is inferred, and no vague wording ("a few", "fast", "usual") stands in for a value | one-shot authoring without guessed spec statements |
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
