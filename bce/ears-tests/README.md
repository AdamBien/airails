# ears-tests

An [AIrails.dev](https://airails.dev) skill that generates **parameterized (table-driven) tests
from [EARS](https://alistairmavin.com/ears/) requirement statements** — the deterministic transform
behind a [SBCE](https://sbce.space) spec's `## Requirements`.

## Scope

- The EARS→test mapping: requirement **group `Rn`** → one parameterized test method; **statement
  `Rn.m`** → one labeled row.
- The EARS-pattern → row-shape table (which column is arrange/act/assert, and the row's happy/unhappy role).
- Preserving the **spec↔test trace** — the `Rn.m` id stays grep-visible as the row's display name.
- The line between what is generated mechanically (structure, ids, roles, the bijection check) and
  what stays authored (fixtures, `<response>` assertions).

Stack-neutral. No runner, framework verb, or test syntax — those come from the composed stack skill.

## Composition

- [`sbce`](../sbce) — provides the spec and the stable `Rn.m` ids this skill consumes.
- [`bce`](../bce) — where the BC and its tests live.
- a **stack skill** — the test syntax and the green/red oracle:
  [`microprofile-server`](../microprofile-server) (JUnit 5 `@ParameterizedTest`),
  [`java-cli-app`](../java-cli-app) (zunit), or `web-components` (Playwright).

Per-stack realization examples live in [references/realizations.md](references/realizations.md).

## Usage

Activated when turning EARS requirements — or a `/sbce` capability spec's `## Requirements` — into
tests. A requirement group with ≥2 statements becomes one parameterized test; each statement `Rn.m`
becomes a row whose display name embeds its id. The rules in [SKILL.md](SKILL.md) drive the mapping;
the composed stack skill supplies the syntax.
