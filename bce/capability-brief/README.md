# capability-brief

An [AIrails.dev](https://airails.dev) skill that captures one capability, or one addition to a
capability, as a **brief** — a use case or a user story complete enough that
[`/sbce new`](../sbce) authors the spec in one shot. The clarifying interview `new` runs
against a vague feature description moves here, to authoring time with the domain owner, and its
answers are recorded once.

## Scope

- Three modes, one skill — invoke as `/capability-brief uc|story|review <description-or-file>` (or by intent, mode omitted):
  - **uc** — a **use case** for a new capability with several steps, actors or kept state
  - **story** — a **user story** for one behaviour added to an existing capability, or a new capability with a single operation
  - **review** — run the checklist over an existing brief and complete only its gaps
- Mode omitted: the skill routes by the input's shape (kept state or several calls → use case, an addition to an existing capability → story) and switches when the shape changes. Mode given: the author's choice stands; when the input outgrows it, the skill warns and asks instead of switching
- An **interview** that fills the template from the author's words, scans the repository read-only (existing package docs, the system doc's decisions and language, whether a stack is declared), then asks one gap per question — only what a human can answer: result shape, input parts and validation, repeated requests, external failures, lifecycle end, rejected alternatives
- A **completeness checklist** every brief passes before it is emitted: every actor step named as an operation, every input with its parts, every operation with its failure cases, every supporting actor tagged as a capability or external, lifecycle and entities stated or `none`, decisions with rejected alternatives, open issues holding only unanswered questions
- Stack-neutral — what the system promises, never how it is built; a type, transport or framework offered by the author is kept out
- The brief is **inception input, not a source of truth** — the same status as the README seed `/sbce new` reads. Once the spec exists in the package doc, the spec is authoritative
- Templates: [references/use-case-template.md](references/use-case-template.md), [references/user-story-template.md](references/user-story-template.md); each carries a filled example and states how its sections become the spec

## Composition

- [`sbce`](../sbce) consumes the brief: its `new` mode treats a filled brief as the completed clarify loop, its `## Capability` as the confirmed carving, each `record:` as a confirmed decision, and asks only what `## Open issues` lists
- [`ears-tests`](../ears-tests) is downstream of the spec; a brief never contains test wording
- No stack skill is involved; the brief is upstream of any stack

```mermaid
graph LR
    Brief([capability-brief<br>use case / user story])
    SBCE([sbce new<br>spec in the package doc])
    EARS([ears-tests<br>tests from the spec])

    Brief -->|one-shot input| SBCE
    SBCE -->|delegates spec→test| EARS

    classDef intent fill:#ffe6cc,stroke:#d79b00,color:#000
    classDef workflow fill:#dae8fc,stroke:#6c8ebf,color:#000
    classDef transform fill:#e1d5e7,stroke:#9673a6,color:#000
    class Brief intent
    class SBCE workflow
    class EARS transform
```

## Usage

Invoke `/capability-brief <mode> <description>` or describe the capability in domain language and
let the skill pick the format. It interviews until the checklist passes and prints the filled
brief. Save it where you like (`<capability>-brief.md` beside the repository `README.md` is the
default), read it, then hand it to `/sbce new <file>`. Two commands on purpose: the pause between
them is where the domain owner signs off before the brief becomes a spec.

## Example Prompts

- `/capability-brief uc "a shopper browses the store and buys products"`

  New capability with several steps and a kept selection: use case; the interview asks for the result shape, the parts of payment and delivery details, what a second confirmation does, and when the selection expires.

- `/capability-brief story "as a shopper I want to search products by keyword"`

  Addition to the existing `shop` capability: user story; the interview asks for the match rule, ordering, limit, minimum keyword length and the response when stock control is unavailable.

- `/capability-brief story "as a shopper I want to keep products in a wishlist across visits"`

  Explicit mode outgrown: the wishlist is kept state, so the skill warns that the story hides a lifecycle and asks whether to continue as story or switch to a use case — it never switches on its own once a mode was given.

- `/capability-brief review shop-brief.md`

  Review: runs the checklist over the file, asks only for the rows that fail, and returns the completed brief.

- `write a use case for the returns process`

  Mode omitted, triggered by intent: the skill routes by shape (several steps, a returned item tracked across them → use case) and says so.

## Test

```
/capability-brief story "as a librarian I want to register a returned book"
```

Then, on the emitted brief:

```
/sbce new library-brief.md
```

`new` should author the spec without a question beyond the brief's `## Open issues`.
