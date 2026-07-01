# System doc template (base package)

The system doc is the **optional** spec one altitude above the BC specs. It lives in the base
package's doc — there is no separate `specs/` tree:

- **Java**: `src/main/java/<base>/package-info.java` — each line prefixed `///` ([JEP 467](https://openjdk.org/jeps/467) Markdown doc comments), ending with the `package <base>;` declaration.
- **web**: `app/src/package-info.md` — plain Markdown.

Each BC's own `package-info` answers *what that one boundary promises*. The system doc answers only
the questions that **span** BCs and have nowhere else to live. Add a section only when a real
cross-BC concern appears — a one-BC system needs no system doc.

Rules for filling it in:

- **Cross-BC only.** Anything that belongs to a single boundary stays in that BC's spec.
- **Never duplicate a BC's one-liner.** A hand-maintained BC index drifts and breaks the single-source-of-truth rule — *the gap is read, not stored*. If you want a BC map, mark it **generated** and regenerate it from the per-BC docs; never hand-type it.
- **Composition is concrete wiring**, not generic rules — `/bce` owns BCE layering and naming bans; the system doc owns *this system's* allowed dependencies and integration events.
- **Vision is the one exception.** Optional, one aspirational sentence — the outcome the assembly chases, distinct from the Charter's mandate ("what this assembly is"). Pure rationale: it carries **no `Sn`**, traces to **no test**, and is the single non-verifiable line in the doc. May be proposed by `/sbce new` distilling a README seed (the human accepts or edits). Omit it unless a real aspiration exists.
- **System invariants** are EARS `shall` statements (the system is the assembly, not one BC). Same six patterns as a BC spec, and the same traceability: each carries a stable id `Sn` that ≥1 test embeds, so the spec↔test binding stays bijective at system altitude too.
- **Ubiquitous language** defines shared nouns once, so each BC's `## Entities` stays terse — names plus a one-line meaning, no fields, no types.
- It is not a tasks file and not a gap registry.

The Markdown body (this is the whole system doc — every section optional except the charter):

```markdown
# <System Name>
> One sentence: what the whole assembly of BCs promises.

## Vision
<!-- optional; the aspirational outcome the assembly chases; rationale, not contract; no Sn, no test -->
- Make checkout so fast the customer never abandons a cart.

## Components
<!-- this system's concrete wiring — direction matters; each BC's own contract lives in its package-info -->
- `checkout` may call `inventory`; never the reverse.
- `payment` is reached only via `checkout`'s `place-order`.

## System invariants
<!-- cross-cutting EARS `shall` statements no single BC owns; the system is the assembly; each carries a stable id Sn that >=1 test embeds -->
- S1 — The system shall never expose an unconfirmed order outside `checkout`.

## Ubiquitous language
<!-- shared domain nouns, defined once; names + one-line meaning, no fields, no types -->
- Order — a confirmed, cancellable intent to buy. Owned by `checkout`.
- Cart — a mutable pre-order collection of items.

## Stack
<!-- the composed stack skill + package base, so `apply` reads it instead of re-inferring -->
- java-cli-app · base package `airhacks`
```

In **Java**, prefix every line with `///` and end the file with the base package declaration:

```java
/// # Airhacks Store
/// > Turn a browsing customer into a fulfilled, paid order.
///
/// ## Vision
/// - Make checkout so fast the customer never abandons a cart.
///
/// ## Components
/// - `checkout` may call `inventory`; never the reverse.
/// …
package airhacks;
```

In **web**, the same Markdown body is the entire `package-info.md` — no `///`, no `package` line.
