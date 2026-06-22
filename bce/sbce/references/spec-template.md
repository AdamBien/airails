# Capability spec template

Copy the fenced block below into `specs/<capability-as-path>/spec.md` and fill it in. One spec
describes exactly one BC; a single feature may decompose into several specs — one per BC.

The directory segment **is** the BC name and the only identity. The capability is dotted;
the path is the same value with dots replaced by slashes:

```
airhacks.sbce.checkout
  → spec:   specs/airhacks/sbce/checkout/spec.md
  → source: src/main/java/airhacks/sbce/checkout/{boundary,control,entity}/
```

Rules for filling it in:

- Stack-neutral. No transport, no types, no framework verbs, no *how* — only what the boundary promises.
- `capability` is the **only** identity field and must equal the directory path. There is no `bc` field.
- Boundary operations are verb-noun and transport-neutral (`place-order`, not `POST /orders`).
- Every boundary op traces to a requirement group (`Rn`); every requirement is ≥1 EARS statement, and every EARS statement traces to ≥1 test.
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

```markdown
---
capability: <org>.<project>.<bc>   # == BC name == directory path under specs/ and src/main/java/
status: active                     # active | archived
---
# <Bc Name>
> One sentence: this BC's single responsibility.

## Boundary
<!-- the single external entry point — operations as verb-noun, transport-neutral -->
- `place-order` — submit a cart for fulfilment
- `cancel-order` — withdraw an unfulfilled order

## Requirements
<!-- EARS statements (the system is the BC); group related ones under a titled Rn; every boundary op traces to a requirement; every statement traces to >=1 test -->
### R1: Place an order
- When a cart with at least one item is submitted, the BC shall create and confirm an order.
- If the cart is empty, then the BC shall reject the request.

## Entities (optional)
<!-- stateful domain nouns this BC owns — names only, no fields, no types -->
- Order, Cart

## Out of scope
<!-- what this BC deliberately does not do -->
-
```
