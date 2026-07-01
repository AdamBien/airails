# Top-level README template (projection)

The repo-root `README.md` is an **optional** human on-ramp to the system. It is a **projection of
the specs, not a source of truth** — the per-BC `package-info` docs (and the system doc) stay
authoritative; the README only renders a human-readable view *over* them, the repo-altitude analog
of how `javadoc` publishes each BC's `package-info`. Add it only when a human on-ramp helps; a
one-BC scratch project needs none.

It has two slices, handled oppositely:

- **Generated** — everything derivable from the specs (charter, vision, the BC map). Regenerated on
  `apply`; **never hand-edited**.
- **Hand-maintained** — only what *no spec covers*, so it cannot drift from one: project-local
  standards, build/run/test, and free-form meta.

Rules for filling it in:

- **Projection, not source.** Never hand-write capability, charter, vision, or BC content into the README — that lives in the specs and is *generated* here. Hand-typing it recreates the drift SBCE bans (*"the gap is read, not stored"*).
- **The generated slice is fenced** by `<!-- sbce:generated:start -->` / `<!-- sbce:generated:end -->`. `apply` replaces only what is between the markers and preserves everything outside. **No markers → `apply` leaves the README untouched** — a README is opt-in, never forced.
- **Generated content** = the system doc's **Charter** + **Vision** (if present) + a **BC map**: each BC name, its `>` responsibility one-liner, and a link to its `package-info`. Read from the per-BC docs — the *"mark it generated, regenerate it"* rule made concrete at repo altitude.
- **Hand-maintained content** = only what no spec covers:
  - **`## Conventions`** — project-local, non-behavioral standards (coverage target, "money is always cents", review policy). **Declared, not verified**: not EARS, no `Sn`, not checked by the test oracle — binding team *policy*, distinct from Vision (aspiration) and from a `System invariant` (which must be behavioral *and* tested).
  - **Build / run / test** — delegate to the composed stack skill; it owns the commands.
  - Free-form meta — license, links, motivation.

The Markdown body (plain Markdown at the repo root — no `///`, no `package` line):

```markdown
# <System Name>

<!-- sbce:generated:start — projection of the specs; do not edit; `apply` regenerates from the system doc + per-BC package docs -->
> Turn a browsing customer into a fulfilled, paid order.

**Vision:** Make checkout so fast the customer never abandons a cart.

## Capabilities
- **checkout** — accept a cart and turn it into a confirmed, cancellable order · [`spec`](src/main/java/airhacks/checkout/package-info.java)
- **inventory** — track and reserve stock · [`spec`](src/main/java/airhacks/inventory/package-info.java)
<!-- sbce:generated:end -->

## Conventions
<!-- hand-maintained; project-local, non-behavioral standards no spec covers; declared, not verified — no Sn, no test -->
- Money is always integer cents.
- Every PR needs two reviews and ≥90% line coverage.

## Build & run
<!-- delegate to the stack skill; it owns the commands -->
See the composed stack skill for build, run, and test commands.

## License
MIT
```
