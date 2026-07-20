# system-tests

An [AIrails.dev](https://airails.dev) skill with the **stack-neutral contract for [system tests](https://en.wikipedia.org/wiki/System_testing)** ([ISTQB definition](https://glossary.istqb.org/en_US/term/system-testing)) — black-box tests exercising the running system from the outside through its public surface.

## Scope

- The contract: black box (public surface only), running system (never instantiate production classes), environment coordinates as configuration, test isolation, total verdict reporting.
- The expectation-origin taxonomy — same test level, different oracle: specified (hand-written), spec-derived ([ears-tests](../ears-tests)), recorded ([characterization-tests](../../migrations/characterization-tests)).
- No launch mechanics, module layout, or test syntax — those come from the composed stack skill.

## Composition

- a **stack skill** — launch mechanics, module layout, and test syntax:
  [`microprofile-server`](../microprofile-server) (`-st` Maven module, JUnit 5, rest client),
  [`java-cli-app`](../java-cli-app) (zunit), or [`web-components`](../../web/web-components) (browser-driven checks).
- [`ears-tests`](../ears-tests) — system tests with spec-derived expectations.
- [`characterization-tests`](../../migrations/characterization-tests) — system tests with recorded expectations, for legacy migrations.

## Usage

Activated when writing, generating, or reviewing system tests in any stack. The contract in [SKILL.md](SKILL.md) constrains what a system test may observe and how it reports; the composed stack skill supplies everything concrete.
