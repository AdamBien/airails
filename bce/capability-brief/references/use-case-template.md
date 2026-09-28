<!--
Capture one capability as a use case, so `/sbce new` authors its spec without asking a question.
Fill every section; a blank one becomes a clarifying question. Describe what the system promises,
never how it is built: no types, transports, frameworks or file names. The stack is not captured
here; sbce reads it from the system doc, AGENTS.md or README — if none declares it, say so under
Open issues.

How the sections become the spec:
Capability     -> the spec's identity and its one-line responsibility
Actors         -> who calls the system; a supporting capability is cross-capability wiring
Main scenario  -> the named operations become `## Boundary`; each step a `When…shall` statement;
                  the last step the result shape
Inputs         -> the `If…then` statements for invalid input
Extensions     -> `If…then` statements, one per extension; optional paths -> `Where…`
Lifecycle      -> `While…` statements
Entities       -> `## Entities`
Decisions      -> the recorded `Dn` log with rejected alternatives

Use: run `/sbce new` on the file, or paste the body as the argument of `/sbce new "…"`.
-->

## Use case
- name: <verb-noun, e.g. "browse and shop">
- goal: <one sentence: what the primary actor achieves>

## Capability
- name: `<one lowercase word>`
- kind: new | extends `<existing-capability>`
- responsibility: <one sentence: what the capability promises, from the system's side>

## Actors
<!-- primary: who triggers it; supporting: only what this use case relies on, tagged as another
capability or as external to the system -->
- primary: <e.g. "a shopper">
- supporting: <name> — capability `<name>` | external; <what this use case relies on it for>

## Trigger
<the event that starts the use case, with the input it carries>

## Main scenario
<!-- numbered happy path; a step where the primary actor addresses the system names its operation
in backticks, verb-noun; system-only steps name none; do not repeat the Extensions; the last step
states the result: shape, order, limit and truncation wording, empty answer -->
1. `<operation>` — <actor> <does what, with which input>
2. The system <does what>
n. The system returns <shape>; <order>; <limit and truncation>; when nothing matches, <empty answer>.

## Inputs
<!-- every input an operation carries: its parts, which are required, what makes it invalid -->
- <input>: <parts>; required: <parts>; invalid when: <rule>

## Extensions
<!-- one per thing that can go wrong or differ, keyed to its step; cover per operation: invalid
input, nothing to act on, absent target, external system unavailable, repeated request where it
changes state, actor stops responding; mark an optional path "(optional)"; "*" = any step -->
- 1a. <condition> -> <system response>
- *a. <condition> -> <system response>

## Lifecycle
<!-- state or an external resource kept across calls: what starts it, what is reused, when and how
it ends; else "none" -->
- none

## Entities
<!-- stateful nouns this capability owns, names only; else "none" -->
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
<summary>Filled example: <code>shop</code>, use case "browse and shop" — adapted from Jacobson & Cockburn, <a href="https://alistaircockburn.com/Use%20Case%20Foundation.pdf">Use-Case Foundation</a> v1.1</summary>

```markdown
## Use case
- name: browse and shop
- goal: a shopper finds the most suitable product for their needs and purchases it

## Capability
- name: `shop`
- kind: new
- responsibility: list the products of the online music store and turn a shopper's selection into a paid, confirmed order

## Actors
- primary: a shopper of the online music store
- supporting: the stock control system — external; availability of a product
- supporting: the payment system — external; validating payment details and taking the payment
- supporting: a sales advisor — external; specialist or high-value products; the system hands the shopper over, the advisor never places a purchase

## Trigger
the shopper asks for products, optionally narrowed by a keyword

## Main scenario
1. `list-products` — the shopper asks for products, optionally with a keyword.
2. The system lists the available products with name, price and availability; by name; at most 50, with a note when more matched; when none match it reports that no product was found.
3. `select-product` — the shopper selects a product and a quantity for purchase; repeated for each product.
4. `provide-payment` — the shopper provides payment details.
5. `provide-delivery` — the shopper provides delivery details.
6. `confirm-purchase` — the shopper confirms the purchase.
7. The system takes the payment, reserves the products and returns the purchase confirmation: an order number, the purchased products with quantities, the total charged and the delivery details.

## Inputs
- keyword: free text; optional; invalid when shorter than two characters
- selection: product and quantity; required: both; invalid when the product is unknown or the quantity is below one
- payment details: card holder, card number, expiry, security code; required: all; invalid when a part is missing, the number fails its check digit, or the expiry is past
- delivery details: recipient, street, postal code, city, country; required: all; invalid when a part is missing

## Extensions
- 1a. the keyword is invalid -> reject the request and name the minimum length
- 1b. the shopper asks for expert advice (optional) -> hand the shopper over to a sales advisor; the selection is kept
- 2a. the stock control system is unavailable -> list the products without availability and mark it as unknown
- 3a. the selection is invalid -> reject it and name the invalid part
- 3b. the selected product is out of stock -> reject the selection and report the product as unavailable
- 3c. the same product is selected again -> add the quantity to the existing line, never a second line
- 4a. payment details invalid -> reject them and name the invalid part
- 4b. the shopper has stored payment details (optional) -> offer them for reuse before asking for new ones
- 5a. delivery details invalid -> reject them and name every missing part
- 5b. the shopper has stored delivery details (optional) -> offer them for reuse before asking for new ones
- 6a. nothing selected at confirmation -> reject the confirmation and ask for a selection
- 6b. payment or delivery details not yet provided -> reject the confirmation and name what is missing
- 6c. the purchase is confirmed a second time -> take no second payment; return the same confirmation
- 7a. a product went out of stock since selection -> take no payment, report the product and let the shopper amend the selection
- 7b. the payment system is unavailable -> take no payment, report the purchase as temporarily unavailable, keep the selection
- 7c. the payment is declined -> report the decline, keep the selection, let the shopper provide other payment details
- *a. the shopper quits at any step -> nothing is charged; the selection is kept for the next visit
- *b. the shopper stops responding beyond the inactivity period -> the selection expires; nothing is charged

## Lifecycle
- one selection per shopper, started at the first selected product, kept across visits until purchase, quit or expiry after the inactivity period

## Entities
- Selection, Order

## Out of scope
- the sales advisor's conversation with the shopper — a separate capability
- shipment, delivery tracking and returns — separate capabilities
- shopper accounts and the storage of payment and delivery details — reused here, owned elsewhere

## Decisions
- record: one capability `shop` from browsing to confirmation _(rejected: separate `catalog`, `cart` and `checkout` capabilities)_
- record: payment is taken only at confirmation _(rejected: authorising at the moment payment details are given)_
- record: availability comes from stock control at listing and again at confirmation _(rejected: trusting the listing at confirmation)_

## Open issues
- none
```

</details>
