---
name: system-brief
argument-hint: new|review <description, notes, or brief file>
description: Capture one system — the whole assembly of capabilities — as a brief complete enough that each capability line is a one-shot input to `/capability-brief` and the mission, language, and decisions seed the sbce system doc without a question. Runs the stakeholder interview once, at inception, checks the result against a completeness checklist, and emits the filled template — mission, vision, actors, the carving into named capabilities with one-liners, their interactions, shared language, cross-capability invariants, success measures, out of scope, decisions, open issues. Accepts any text naming a system to build — a sentence, a pitch, meeting notes, a product requirements document, a legacy system description. Stack-neutral — why the system exists and which capabilities it needs, never how any of them works or is built. Use whenever someone wants to start a new system or product, scope a greenfield project, carve a product into capabilities or business components, write a mission, vision or product brief, decide what is in and out of scope, prepare the inception seed for `/sbce new`, or asks "what capabilities does this need", "where does one BC end and the next begin", "is this scope complete", "turn this PRD into capabilities". Triggers on "system brief", "product brief", "mission", "vision", "scope the system", "carve into capabilities", "which business components", "inception", "greenfield", "new product", "PRD", "prepare for capability-brief". Not for a single capability (`/capability-brief`), not for authoring specs or the system doc (`/sbce`), and not for recovering capabilities from legacy code (`/migration-advisor`).
---

Produce a **system brief**: one filled template that answers every question `/capability-brief`
and `/sbce new` would otherwise ask about the system as a whole — what it is for, who uses it,
which capabilities it consists of and where each one ends. The stakeholder conversation that
usually happens in fragments across the first ten capability interviews is moved here, to the one
moment all stakeholders are present, and its answers are recorded once.

The brief is stack-neutral: why the system exists and which capabilities it needs, never how any
capability works or how the system is built. No operations, inputs, screens, types, transports,
frameworks, or file names. A capability's inside belongs to its own brief; the stack belongs to the
stack skill. A system brief that names either binds the carving to a solution before the
capabilities exist.

## Two guarantees, one of them yours

- **Structural completeness is yours to enforce.** Every section filled, every capability one word with one sentence and a named actor, every actor in at least one capability, every shared noun defined once, no overlapping responsibilities left unacknowledged. The checklist below makes this mechanical; do not emit a brief that fails it.
- **Semantic completeness is the stakeholders'.** Where one capability ends and the next begins, what the system will never do, which capability comes first, what was rejected and why — these are business facts. You cannot invent them; you can only ask until they are answered or honestly parked under `## Open issues`. A guessed carving becomes a business component, then a package, then a team boundary that nobody chose.

## Invocation modes: new · review

Read the mode and the input from the invocation: `/system-brief new "<description>"`,
`/system-brief review <file-or-pasted-text>`. With the mode omitted, `review` when the input is
already a system brief (recognisable by its `## System` … `## Open issues` sections), else `new`.

| Mode | When |
|---|---|
| `new` | a system that does not exist yet, or an existing system with no system brief and no system doc |
| `review` | an existing system brief to check, or existing text — a pitch, notes, a product requirements document — to complete into a brief |

`review` runs the checklist over the given brief, asks only for the rows that fail, and returns the
completed brief in the same section order. It never reopens answered sections. Given text that is
not a brief, `review` fills the template from the text and continues the same way: only the
failing rows are asked.

## Input

Any text is accepted — a sentence, a pitch, meeting notes, a product requirements document,
pasted or as a file. The minimum is **one outcome the system should deliver for someone**:
"employees handle leave, expenses and onboarding without emailing HR" is enough. Actors, the
carving, the scope boundary and the language are what the interview is for; a missing one is a
question, never a reason to refuse.

| Input | Response |
|---|---|
| names only a solution — "we need a platform", "a portal with single sign-on", "a Kubernetes-based backend" | ask what someone should be able to do once it exists; continue from the answer |
| describes one capability — one actor reaching one goal, one responsibility | say it is one capability and hand it to `/capability-brief`; a one-capability system needs no system brief, exactly as a one-BC system needs no system doc |
| describes a bug, a refactoring, a migration of a running system, or a technical task | say it is not a system to carve, and stop; for a legacy system point to `/migration-advisor`, which recovers capabilities from code rather than from stakeholders |

Long text is the risk, not short text. A sentence is visibly incomplete and gets asked; a product
document looks complete and tempts you to read a carving into it — a chapter per capability, a
heading per actor. Chapters follow the author's outline, not the system's responsibilities. Fill
only what the text states, read it back, and let the stakeholders draw the lines.

## Interview

1. **Fill from the stakeholders' words first.** Place everything they said into the template before asking anything. Fill only what the text states: every entry is quoted or closely paraphrased from the input, or later confirmed in the interview. A value the text does not state is a proposed default to ask, never an entry. Vague wording — "all the usual HR stuff", "the standard integrations", "etc." — fills nothing; it is a gap. Two passages that contradict each other become one question quoting both.
2. **Scan the repository, read-only**, before the first question:
   - a system doc (base package `package-info.java` / `app/src/package-info.md`) → the system already has a mission, language and decisions; use `review` semantics against it and never propose what a `Dn` records as rejected;
   - existing package docs → the capabilities that already exist, with their one-line responsibilities; list them under `## Capabilities` tagged `existing`, so the brief shows the whole assembly and no new capability duplicates one; a capability the stakeholders decide to grow is retagged `extends` and gets a line of its own for the addition;
   - the README's hand-written prose → an inception seed may already state intent; take it as input text;
   - whether a stack is declared → not yours to decide; if none is declared, note "stack not declared" under `## Open issues` so `/sbce new` knows it must ask.
3. **Read back what was taken from the input** before the first question, grouped by template section, so a misreading is corrected before it counts as an answer.
4. **Report what the scan found**: existing capabilities and what they own, nouns already defined in the ubiquitous language, decisions already recorded. Each finding becomes a question with options — extend the existing capability / add a new one beside it; adopt the existing term / rename. With nothing found, say "greenfield, nothing existing" and continue.
5. **Propose the carving, do not announce it.** From the actors and outcomes, propose a capability set with one-liners and ask for confirmation, naming the alternative you rejected — "I'd carve `leave` and `expenses` apart because different people decide their rules; the alternative is one `requests` capability. Confirm or correct." Carving has no test oracle; the stakeholders' confirmation is the only one it gets.
6. **Carve by responsibility, not by outline.** A capability is one responsibility an actor would name, with one party who decides its rules. Signs of a cut too coarse: an "and" in the one-liner joining two outcomes, or two actors reaching different goals. Signs of a cut too fine: a capability whose one-liner is a single step of another, or that no actor would ask for by itself — that is a behaviour of an existing capability and belongs in a user story, not here. Signs of a wrong cut: two capabilities that cannot be described without each other.
7. **Ask one gap per question**, specific over generic, with enumerable options and an "other" escape. Skip what context already answers. No meta-questions about the process.
8. **Ask what only the stakeholders know**: who the actors are and what each achieves; where one capability ends and the next begins; which external systems the assembly relies on and which capability relies on each; which capability is built first and why; what the system will not do, even though someone asked; how the stakeholders will know the system delivers; which alternative a decision rejected.
9. **Answer yourself what is derivable**: capability names (one lowercase noun from the responsibility), the section a fact belongs to, the ordering of sections, which nouns appear in more than one line and so need a definition.
10. **Loop.** Each answer can expose the next gap; re-run the checklist after every round. Stop when it passes.
11. **Park, never guess.** A question the stakeholders cannot or will not answer goes under `## Open issues` in their words. That section is the only thing downstream will ask about the system, so it must hold every unanswered question and no answered one.
12. **Refuse the inside and the how.** If a stakeholder offers an operation, an input, a screen, a type, a transport, a framework or a product, keep it out of the brief. An operation or input is a note for that capability's brief — say so and carry it forward when `/capability-brief` runs. A technology choice that is really a business decision made against alternatives is recorded in domain terms under `## Decisions`; otherwise drop it and say so.

## Completeness checklist

Run this before emitting; every row maps to a question that would otherwise be asked downstream,
once per capability.

| Check | Feeds |
|---|---|
| System name is one lowercase word; mission is one sentence from the system's side | the system doc's title and `>` line |
| Vision is one aspirational sentence or `none`; it states an outcome, not a feature | the system doc's `## Vision` |
| Every primary actor is named with what they achieve, and appears in at least one capability line | the `primary` actor of each capability brief |
| Every external system is named with what it is relied on for, and at least one capability relies on it | `supporting … — external` in the capability briefs |
| Every capability is one lowercase word, unique, tagged `new`, `extends` or `existing`, with one sentence naming its primary actor and the outcome the system promises; `extends` and `existing` name a capability whose package doc exists | one `/capability-brief` input per `new` or `extends` line; `kind` and `name` of the resulting brief; `existing` lines only complete the assembly |
| No capability one-liner contains an operation, an input, a step, or a screen | the capability brief owns the inside |
| No two capabilities share a responsibility; every overlap is resolved by a merge or split recorded under Decisions, or parked under Open issues | one capability ≡ one BC, no duplicate BC |
| No capability name clashes with an existing package doc unless tagged `extends` or `existing`; every noun matches the existing ubiquitous language or is new, and a second meaning is recorded under Decisions | no overloaded term across specs |
| Capabilities are in build order; the first one has a stated reason for being first | which brief to write first |
| Interactions name only listed capabilities and external systems, each with a direction and what crosses; or `none known yet` | the system doc's `## Components` wiring, proposed |
| Every shared noun — one used in two or more capability lines or actor lines — is defined once in one line, no fields, no types | the system doc's `## Ubiquitous language` |
| Every invariant is a rule in domain terms that spans two or more capabilities; a rule owned by one capability is moved to that capability's notes | candidates for the system doc's `## System invariants`, authored as EARS by `/sbce new` |
| Success measures are observable outcomes the stakeholders can check, or `none`; none is a feature | rationale for the carving and the build order, feeds Vision |
| Out of scope lists what was weighed and excluded, each with its reason | closes scope questions in every capability brief |
| Every decision reads `record: <choice> _(rejected: …)_`; else `none` | the system doc's `Dn` log |
| Every entry is stated in the input or confirmed in the interview; no value is inferred, and no vague wording ("the usual", "standard", "etc.") stands in for a value | one-shot briefing downstream without guessed capabilities |
| Open issues holds only unanswered questions | the only questions asked downstream about the system |
| No operation, type, transport, framework, product, or file name anywhere | stack neutrality; the capability briefs own the inside |

## Output

- Emit the filled body exactly in the template's section order ([references/system-brief-template.md](references/system-brief-template.md)), header comment removed, ending with the hand-off: one `/capability-brief "<capability line>"` per `new` or `extends` capability, in build order, then `/sbce new`; `existing` capabilities get no line, they are already specified. Say that the brief is read and signed off between the two steps — the pause is where the stakeholders confirm the carving before it becomes packages.
- Print it in the reply. Write it to a file only when asked, where the author says; a sensible default is `system-brief.md` beside the repository `README.md`. The brief's prose may also be pasted into the README outside the `sbce:generated` markers, where it is the inception seed `/sbce new` with no argument reads.
- Carry notes forward. Operations, inputs and rules a stakeholder mentioned for one capability are not lost: list them after the brief as "notes for `/capability-brief <name>`", one block per capability, so the capability interview starts from them. They are not part of the brief and are not checked by its checklist.
- The brief is **inception input, not a source of truth** — the same status as a capability brief and the README seed. Once the system doc and the specs exist, they are authoritative and the brief is not kept in sync; a hand-maintained capability list drifts, which is why the sbce README projection regenerates its BC map from the package docs instead.

## Composition

- `/capability-brief` consumes one capability line at a time: the line's name becomes `name`, its tag `kind` — `new` stays `new`, `extends` becomes `kind: extends <name>`, `existing` is never handed off — its actor the `primary` actor, the system's external systems the `external` supporting actors, the carried notes the start of its interview. The system brief never contains a capability's main scenario, inputs, extensions or acceptance criteria.
- `/sbce new` consumes the whole: `## Capabilities` is the proposed carving, confirmed by the stakeholders' sign-off; `## System` seeds the system doc's mission and vision; `## Interactions`, `## Language` and `## Invariants` seed its `## Components`, `## Ubiquitous language` and `## System invariants`; each `record:` is a confirmed `Dn`. The system doc is authored there, never here.
- `/bce` is the judge of a cut, not its source: its cohesion and coupling rules decide whether a carving is legal; the stakeholders decide which carving is wanted. When a proposed cut would violate `/bce` — a capability that is only a layer, or two capabilities that would share every entity — say so and propose another.
- The stack skill is not consulted — the system brief is upstream of any stack; the only stack-related act is the declaration check in the interview.
- `/migration-advisor` and `/bc-carver` do the equivalent for a legacy system from its code; this skill does it for a greenfield system from its stakeholders. The two meet in the same system doc.
