# simplifier

An [AIrails.dev](https://airails.dev) skill for **replatform mode** ("lift, tinker, and shift") — simplify a 1:1-lifted system in place: modernize code and remove external dependencies, structure preserved.

## Scope

- Owns the **pass discipline**: one transformation or one dependency removal per pass, each bracketed by a characterization replay — green commits, a diff reconciles or reverts.
- Dependency removal follows a stack-neutral ladder: platform first → copied/vendored source → keep with justification.
- Preserves the public surface and the structure — packages, components, routes stay put. Moves and splits are carving.

A **valid terminal state** (a distilled, dependency-free system on a modern stack) or preparation for `concept-extractor`.

## Composition

- [`characterization-tests`](../characterization-tests) — the green bar every pass must hold; a precondition.
- the composed stack ecosystem supplies the transformation catalog and dependency rungs: [`java-distiller`](../../java/java-distiller) (Java SE → MicroProfile/Jakarta EE → copied source), [`web-conventions`](../../web/web-conventions) / [`web-components`](../../web/web-components) (web platform → vendored source).

## Usage

Invoke explicitly as `/simplifier` after a 1:1 lift, once characterization recordings replay green. Each pass is logged in `migration/SIMPLIFICATION.md`.
