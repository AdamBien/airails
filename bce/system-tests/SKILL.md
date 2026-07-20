---
name: system-tests
description: Generic, composable conventions for system tests — black-box tests exercising the running system from the outside through its public surface. Defines the stack-neutral contract (no internals, environment coordinates as configuration, test isolation, total verdict reporting) and the taxonomy of expectation origins (specified, spec-derived via ears-tests, recorded via characterization-tests). Technology-neutral; composes with stack skills (microprofile-server, java-cli-app, web-components) that own launch mechanics, module layout, and test syntax. Use when writing, generating, or reviewing system tests in any stack. Triggers on "system test", "system tests", "ST", "-st module", "e2e test", "end-to-end test", "black-box test", "test against the running server".
---

# System Tests

A system test exercises the **running system** from the **outside** through its **public surface** — HTTP, CLI, messaging, files. It observes what a consumer observes; nothing else.

## Contract

- **Black box** — assert only through the public surface: no mocks, no internal classes, no database access unless the database is the public surface. Behavior not observable from outside is a unit or integration concern.
- **Running system** — tests target a started instance and never instantiate production classes. Starting and deploying is the stack skill's concern.
- **Coordinates as configuration** — base URL, broker address, working directory come from configuration (e.g. microprofile-server's `service_uri` configKey), never hardcoded. The same suite runs unchanged against dev, CI, and a migration target.
- **Isolation** — each test runs alone and in any order; seed-data assumptions are declared, not implied.
- **Total reporting** — every test appears in the result with a verdict; skipped is a verdict, not an omission.

## Expectation Origins

Same test level, different oracle:

| Origin | Expected values come from | Skill |
|---|---|---|
| Specified | hand-written from known requirements | (default) |
| Spec-derived | EARS statements of a capability spec | ears-tests |
| Recorded | golden masters of the running legacy system | characterization-tests |

## Composition

- This skill owns the contract above.
- The composed stack skill owns launch mechanics, module layout, and test syntax — microprofile-server: `-st` Maven module, JUnit 5, rest client; java-cli-app: zunit; web-components: browser-driven checks.
- ears-tests and characterization-tests own their expectation origins and compose with both this skill and the stack skill.
