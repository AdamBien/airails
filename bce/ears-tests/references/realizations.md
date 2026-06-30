# Per-stack realizations

The mapping is fixed (group `Rn` → one parameterized test; statement `Rn.m` → one labeled row); only
the *syntax* changes per stack. Pick the idiom the project already uses — never add a second.

All examples realize the canonical SBCE `checkout` group, whose two statements share the
`place-order` boundary op and deliberately mix a happy and an unhappy row:

```
### R1: Place an order
- R1.1 — When a cart with at least one item is submitted, the BC shall create and confirm an order.
- R1.2 — If the cart is empty, then the BC shall reject the request.
```

In every realization the **`R1.1` / `R1.2` id is the grep-visible trace token** (a row label / case
name), keeping the SBCE spec↔test binding bijective. The `<response>` assertion and the fixture stay
**authored** — emit them as a failing stub, never a fabricated value, so an unwritten check shows red.

## JUnit 5 — `microprofile-server`

One `@ParameterizedTest` per group; one `@MethodSource` row per statement; the `Rn.m` id is the case
`name` so it surfaces in the runner and is greppable.

```java
@ParameterizedTest(name = "{0}")
@MethodSource("placeOrderCases")
void placeOrder(String requirement, Cart cart, boolean shouldConfirm) {
    var outcome = checkout.placeOrder(cart);            // shared act — the group's boundary op
    // authored per row — TODO until /sbce apply fills the assertion:
    assertEquals(shouldConfirm, outcome.isConfirmed(), requirement);
}

static Stream<Arguments> placeOrderCases() {
    return Stream.of(
        arguments("R1.1", Cart.of(item("duke-sticker")), true),   // When item present → confirm
        arguments("R1.2", Cart.empty(),                  false)    // If empty → reject
    );
}
```

A removed `R1.x` deletes its `arguments(...)` row; a row whose first arg matches no statement is the
drift SBCE surfaces. For a single-statement group, a plain `@Test` named for the id is fine.

## zunit — `java-cli-app`

zunit has no annotations: a "parameterized test" is a **loop over a case list** inside `void main()`,
each case carrying its `Rn.m` id in the `AssertionError` message (the grep token). Throw on first
mismatch — zunit treats any thrown exception as failure.

```java
void main() {
    record Case(String req, Cart cart, boolean shouldConfirm) {}
    var cases = List.of(
        new Case("R1.1", Cart.of(item("duke-sticker")), true),   // When item present → confirm
        new Case("R1.2", Cart.empty(),                  false)    // If empty → reject
    );
    for (var c : cases) {
        var outcome = Checkout.placeOrder(c.cart());             // shared act
        if (outcome.confirmed() != c.shouldConfirm())           // authored per row
            throw new AssertionError("%s — expected confirmed=%s but was %s"
                .formatted(c.req(), c.shouldConfirm(), outcome.confirmed()));
    }
}
```

## Playwright — web stacks

One `test.describe` per group; iterate a cases array to emit one `test(...)` per statement, the
`Rn.m` id leading the test title (the grep token, visible in reports).

```js
const placeOrderCases = [
  { req: 'R1.1', cart: [{ id: 'duke-sticker' }], shouldConfirm: true },  // When item present → confirm
  { req: 'R1.2', cart: [],                        shouldConfirm: false }, // If empty → reject
];

test.describe('R1: place an order', () => {
  for (const c of placeOrderCases) {
    test(`${c.req} — place order`, async ({ page }) => {
      const outcome = await placeOrder(page, c.cart);   // shared act
      // authored per row:
      expect(outcome.confirmed).toBe(c.shouldConfirm);
    });
  }
});
```
