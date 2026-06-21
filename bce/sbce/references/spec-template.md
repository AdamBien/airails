# Capability spec template

Copy the fenced block below into `specs/<capability-as-path>/spec.md` and fill it in.

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
- Every boundary op traces to a requirement; every requirement has ≥1 scenario and ≥1 test.
- Keep it minimal — an empty `## Out of scope` is fine, but the heading stays to keep the boundary sharp.

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
<!-- SHALL statements; each has >=1 scenario; every boundary op traces to a requirement -->
### R1: Place an order
The BC SHALL accept a cart and produce a confirmed order.
#### Scenario: valid cart
- GIVEN a cart with at least one item
- WHEN place-order is invoked
- THEN an order is created and confirmed

## Entities (optional)
<!-- stateful domain nouns this BC owns — names only, no fields, no types -->
- Order, Cart

## Out of scope
<!-- what this BC deliberately does not do -->
-
```
