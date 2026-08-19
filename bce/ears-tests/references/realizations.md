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

In every realization the **`R1.1` / `R1.2` id is the runner-visible trace token** (a case label),
keeping the SBCE spec↔test binding bijective. The `<response>` assertion and the fixture stay
**authored** — emit them as a failing stub, never a fabricated value, so an unwritten check shows red.

## Java trace symbols — the generated `Requirement` annotation

Java stacks (`microprofile-server`, `java-cli-app`) materialize the trace tokens as **symbols**: one
generated file per BC, `Requirement.java` in the BC root package (beside `boundary/`, `control/`,
`entity/`) — a `@Documented` annotation with a nested `Rn` enum, one constant per statement, the
EARS sentence as its doc, `toString()` returning the literal spec id:

```java
package orders;

import java.lang.annotation.*;

/// Generated from the capability spec in [orders] — do not edit.
/// Marks the boundary method or test that realizes the given requirement statements.
@Documented
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Requirement {

    /// One constant per statement id in the spec's `## Requirements`.
    enum Rn {
        /// When a cart with at least one item is submitted, the BC shall create and confirm an order.
        R1_1("R1.1", "When a cart with at least one item is submitted, the BC shall create and confirm an order."),
        /// If the cart is empty, then the BC shall reject the request.
        R1_2("R1.2", "If the cart is empty, then the BC shall reject the request.");

        private final String id;
        private final String statement;
        Rn(String id, String statement) { this.id = id; this.statement = statement; }
        public String statement() { return statement; }
        @Override public String toString() { return id; }
    }

    Rn[] value();
}
```

- **The spec stays the single authored source.** The `## Requirements` section of the BC's
  `package-info.java` owns the statements; `/sbce apply` regenerates `Requirement.java` wholesale on
  every pass. Never hand-edit it — regeneration is the drift oracle.
- **Public, own file, per BC.** The BC spans layer subpackages, so the type must be `public`; a
  package-private type (e.g. one declared inside `package-info.java`) is invisible to
  `orders.boundary`. An annotation element must be typed to one concrete enum, so the annotation
  cannot be shared across BCs — each BC generates its own, and deleting the BC deletes its trace
  apparatus with it.
- **Usage.** `Rn` constants in parameterized rows (the per-statement trace); `@Requirement(R1_2)` on
  a plain single-statement test; `@Requirement({R1_1, R1_2})` on the boundary method that realizes
  the group. Never a statement list on a parameterized test method — it duplicates the rows and drifts.
- **Retirement is fail-closed.** A statement removed from the spec drops its constant on
  regeneration, and every row still referencing it becomes a compile error. The reverse direction —
  a new statement with no covering row — is an unused constant and compiles; that check stays with
  `/sbce`'s bijection pass.
- **The EARS sentence is emitted twice, on purpose.** As the `///` doc (IDE hover, javadoc constant
  summary) and as the `statement()` field (runtime access — failure messages print the violated
  requirement verbatim, `Rn.values()` is the requirements table via reflection). Both come from the
  one authored source on every regeneration, so the in-file duplication cannot drift. `toString()`
  stays the bare id — the runner label and grep depend on it; the sentence rides on `statement()` only.
- **Javadoc.** `@Documented` publishes `@Requirement({R1_1, R1_2})` on each boundary method's
  signature, hyperlinked to the enum page, whose constant summary doubles as the requirements table;
  the package spec links it with `[Requirement.Rn]`. Spec → vocabulary → realizing methods, all
  navigable in the generated site.
- **Scope ends at the unit/integration line.** The `-st` module is black box (no code dependency on
  the server) and keeps literal string labels, like the web stacks.

## JUnit 5 — `microprofile-server`

One `@ParameterizedTest` per group; one `@MethodSource` row per statement; the row's `Rn` constant is
the case `name` — `toString()` prints the spec id, so the runner shows `R1.1` and find-usages on the
constant lists every covering row.

```java
import static orders.Requirement.Rn.*;

@ParameterizedTest(name = "{0}")
@MethodSource("placeOrderCases")
void placeOrder(Requirement.Rn requirement, Cart cart, boolean shouldConfirm) {
    var outcome = checkout.placeOrder(cart);            // shared act — the group's boundary op
    // authored per row — TODO until /sbce apply fills the assertion:
    assertEquals(shouldConfirm, outcome.isConfirmed(),
        requirement + " — " + requirement.statement());
}

static Stream<Arguments> placeOrderCases() {
    return Stream.of(
        arguments(R1_1, Cart.of(item("duke-sticker")), true),   // When item present → confirm
        arguments(R1_2, Cart.empty(),                  false)    // If empty → reject
    );
}
```

A statement removed from the spec removes its constant, and the stale `arguments(...)` row fails to
compile — retirement is enforced by the compiler, not by grep. For a single-statement group, a plain
`@Test` annotated `@Requirement(R1_2)` is fine.

## zunit — `java-cli-app`

zunit tests use no annotations: a "parameterized test" is a **loop over a case list** inside
`void main()`, each case carrying its `Rn` constant — its `toString()` is the grep token in the
failure message. The `@Requirement` annotation still marks the realizing method in `src`. zunit runs
tests with `-ea` — a failing `assert` throws, and any thrown exception fails the file on first mismatch.

```java
import static orders.Requirement.Rn.*;

void main() {
    record Case(Requirement.Rn req, Cart cart, boolean shouldConfirm) {}
    var cases = List.of(
        new Case(R1_1, Cart.of(item("duke-sticker")), true),   // When item present → confirm
        new Case(R1_2, Cart.empty(),                  false)    // If empty → reject
    );
    for (var c : cases) {
        var outcome = Checkout.placeOrder(c.cart());             // shared act
        assert outcome.confirmed() == c.shouldConfirm()          // authored per row
            : "%s — %s — expected confirmed=%s but was %s"
                .formatted(c.req(), c.req().statement(), c.shouldConfirm(), outcome.confirmed());
    }
}
```

## Playwright — web stacks

No symbols in plain JS — the literal `Rn.m` string stays the trace token, leading the test title
(visible in reports, grep-visible against the spec). One `test.describe` per group; iterate a cases
array to emit one `test(...)` per statement. The suite around these tests — layout, configuration,
locators, and the green oracle — is `web-system-tests`.

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
