# Capability spec template

The spec **is** the BC's package doc — there is no separate `specs/` tree. Author the Markdown
below into the BC's package doc; one doc describes exactly one BC, and a single feature may
decompose into several BCs — one doc each.

- **Java**: `src/main/java/<base>/<bc-name>/package-info.java` — each line prefixed `///` ([JEP 467](https://openjdk.org/jeps/467) Markdown doc comments), ending with the `package …;` declaration.
- **web**: `app/src/<bc-name>/package-info.md` — plain Markdown.

The **BC name** is the only identity — a single lowercase token, derived from the package (Java)
or folder (web) the doc lives in. The stack skill owns the location (package base / source root).

Rules for filling it in:

- Stack-neutral. No transport, no types, no framework verbs, no *how* — only what the boundary promises.
- No frontmatter — the BC name comes from the location. An optional `status: archived` line marks a frozen contract; nothing auto-sets it.
- Boundary operations are verb-noun and transport-neutral (`place-order`, not `POST /orders`).
- Every boundary op traces to a requirement group (`Rn`); every requirement is ≥1 EARS statement, and **each statement carries a stable id `Rn.m`** that ≥1 test embeds in its trace — so the spec↔test binding is per-statement and bijective. Ids are stable: never renumber on reorder; a removed statement's id is retired, not reused.
- Keep it minimal — an empty `## Out of scope` is fine, but the heading stays to keep the boundary sharp.

Requirements are written in [EARS](https://alistairmavin.com/ears/) — each statement fits one
of six patterns, and the system is always **the BC**. Reach for the unwanted-behaviour
(`If…then`) and state-driven (`While…`) patterns to capture the error and edge cases that
otherwise go unwritten and untested.

| Pattern | Template |
|---|---|
| Ubiquitous | `The BC shall <response>.` |
| State-driven | `While <precondition>, the BC shall <response>.` |
| Event-driven | `When <trigger>, the BC shall <response>.` |
| Optional-feature | `Where <feature is included>, the BC shall <response>.` |
| Unwanted-behaviour | `If <trigger>, then the BC shall <response>.` |
| Complex | `While <precondition>, when <trigger>, the BC shall <response>.` |

The Markdown body (this is the whole spec):

```markdown
# <Bc Name>
> One sentence: this BC's single responsibility.

## Boundary
<!-- the single external entry point — operations as verb-noun, transport-neutral -->
- `place-order` — submit a cart for fulfilment
- `cancel-order` — withdraw an unfulfilled order

## Requirements
<!-- EARS statements (the system is the BC); group related ones under a titled Rn; give each statement a stable id Rn.m; every boundary op traces to a group; every statement id traces to >=1 test that embeds it -->
### R1: Place an order
- R1.1 — When a cart with at least one item is submitted, the BC shall create and confirm an order.
- R1.2 — If the cart is empty, then the BC shall reject the request.

## Entities (optional)
<!-- stateful domain nouns this BC owns — names only, no fields, no types -->
- Order, Cart

## Out of scope
<!-- what this BC deliberately does not do -->
-
```

In **Java**, prefix every line with `///` and end the file with the package declaration:

```java
/// # Checkout
/// > Accept a cart and turn it into a confirmed, cancellable order.
///
/// ## Boundary
/// - `place-order` — submit a cart for fulfilment
/// …
package <base>.checkout;
```

In **web**, the same Markdown body is the entire `package-info.md` — no `///`, no `package` line.
