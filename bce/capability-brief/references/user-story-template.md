<!--
Capture one addition to a capability, or a capability with a single operation, as a user story with
acceptance criteria, so `/sbce new` authors its spec without asking a question. Fill every section;
a blank one becomes a clarifying question. Describe what the system promises, never how it is built:
no types, transports, frameworks or file names. The stack is not captured here; sbce reads it from
the system doc, AGENTS.md or README — if none declares it, say so under Open issues.

How the sections become the spec:
Story                -> the capability's one-line responsibility (for a new capability)
Capability           -> the spec's identity, its `## Boundary` operations, its `## Entities`, and the
                        cross-capability wiring
Inputs               -> the `If…then` statements for invalid input
Acceptance criteria  -> one EARS statement each: Given = `While…`, When = `When…`, a failing
                        scenario = `If…then`, Then = the `shall` response and its result shape
Lifecycle            -> `While…` statements
Decisions            -> the recorded `Dn` log with rejected alternatives

Cover every scenario, not only the happy path: invalid input, nothing to act on, an absent target,
an external system unavailable, a repeated request where it changes state. A scenario you do not
write is a question you will be asked.

Use: run `/sbce new` on the file, or paste the body as the argument of `/sbce new "…"`.
-->

## Story
As a <role>, I want <action with its input> so that <outcome>.

## Capability
- name: `<one lowercase word>`
- kind: new | extends `<existing-capability>`
- responsibility: <one sentence from the system's side; for kind new, else omit>
- operations: `<verb-noun>` <one per operation this story adds or changes>
- relies on: none | capability `<name>` — <for what> | external <name> — <for what>
- entities: none | <stateful nouns this capability owns, names only>

## Inputs
<!-- every input an operation carries: its parts, which are required, what makes it invalid -->
- <input>: <parts>; required: <parts>; invalid when: <rule>

## Acceptance criteria
<!-- one titled scenario per behaviour; the Then of the happy path states the result's exact shape,
order, limit with truncation wording, and empty answer; mark an optional path "(optional)" -->
### <scenario title>
- Given <state or precondition>
- When <trigger with its input>
- Then <response and result shape>

### <failing scenario title>
- When <invalid trigger or external failure>
- Then <rejection or report, and what is left unchanged>

## Lifecycle
<!-- only if the system keeps state across requests: what starts it, what is reused, when it ends;
else "none" -->
- none

## Out of scope
<!-- what you weighed and excluded; each line stops a question -->
- <excluded behaviour> — <why, optional>

## Decisions
<!-- a choice with its rejected alternatives; write "record" for each; else "none" -->
- record: <choice> _(rejected: <alternatives>)_

## Open issues
<!-- questions the author could not answer; `/sbce new` asks exactly these and nothing else -->
- none

---

<details>
<summary>Filled example: extends <code>shop</code> with a keyword search</summary>

```markdown
## Story
As a shopper, I want to search the products by a keyword so that I find a suitable product without browsing the whole store.

## Capability
- name: `shop`
- kind: extends `shop`
- operations: `search-products`
- relies on: external stock control system — availability of each listed product
- entities: none

## Inputs
- keyword: free text; required; invalid when shorter than two characters

## Acceptance criteria
### Matching products are listed
- Given the store offers products
- When the shopper searches with a keyword
- Then the system lists every product whose name or description contains the keyword, ignoring case, with name, price and availability, most relevant first, at most 50, with a note when more matched

### Nothing matches
- When the shopper searches with a keyword no product contains
- Then the system reports that no product was found

### Keyword missing or too short
- When the shopper searches with an empty keyword or fewer than two characters
- Then the system rejects the search and names the minimum length

### Stock control unavailable
- Given the stock control system is unavailable
- When the shopper searches with a keyword
- Then the system lists the matching products with availability marked as unknown

## Lifecycle
- none

## Out of scope
- filtering by price, category or brand — a later story
- search suggestions while typing

## Decisions
- record: search matches name and description _(rejected: name only; full-text over reviews)_
- record: at most 50 results, most relevant first _(rejected: paging; unlimited results)_

## Open issues
- none
```

</details>
