# package-info.java Template

Java 25 `///` Markdown doc above the `package` declaration. The section heading marks the owned block — everything outside it is human territory.

```java
/// ## Migration Notes (concept-annotator)
///
/// Migration notes — not an sbce spec.
///
/// ### Concepts
/// - **Contract** — here: `Cntrct`, `VTRG` (see glossary)
/// - **Validation**
///
/// ### Target BC
/// `contracts` — per CARVING.md:
/// - `ContractValidator` → contracts / control
/// - `ContractDTO` → contracts / entity (rename: `Contract`)
///
/// ### Refactoring Hints
/// - `ContractManagerImpl` mixes Contract and Tariff — split candidate (open: Q12)
/// - rename `Cntrct*` types to the canonical term `Contract`
///
/// ### See
/// migration/CARVING.md, migration/GLOSSARY.md, open: Q12
package com.legacy.contractmgmt;
```

Rules:

- Without `CARVING.md`, the `### Target BC` section becomes `### Candidate BC` with a single token or `unassigned` — no move rows.
- Keep every hint traceable: alias claims come from CONCEPTS.md evidence, assignments from CARVING.md, open items carry their Q-id.
- Hypothesis-marked glossary terms keep their marker: `- **Agreement** *(hypothesis — see glossary)*`.
- The disclaimer line is not optional — it is what keeps `/sbce` from reading the notes as an authoritative spec.
```
