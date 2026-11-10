# microprofile-server

An [AIrails.dev](https://airails.dev) skill defining architecture and coding rules for long-running Java MicroProfile / Jakarta EE server applications / microservices.

## Scope

- Server-specific [BCE](https://bce.design) rules on top of the `bce` skill: resources and health checks as boundary, `@Transactional` placement, entity references and no DTO mirrors
- JAX-RS resources and exception mapping via `WebApplicationException`
- JSON-P preferred over JSON-B; `toJSON` / `fromJSON` on record entities
- Testing conventions: unit, integration (`*IT`), and system tests in a dedicated `-st` module using microprofile-rest-client
- Maven project structure
- Minimal dependencies (Java SE → MicroProfile → Jakarta EE)
- OTEL / OpenTelemetry for metrics and observability
Not intended for serverless deployments.

## Composition

This skill composes with [`bce`](../bce) for the architecture pattern, [`java-conventions`](../../java/java-conventions) for all language-level Java 25 rules, [`system-tests`](../system-tests) for the system-test contract, and [`readme`](../../documentation/readme) for README conventions. `microprofile-server` adds only the server-specific specializations on top: JAX-RS, CDI, JSON-P, the `-st` module mechanics, and Maven project rules.

## Usage

This skill is activated automatically when creating, writing, or reviewing code in MicroProfile server projects. The rules in [SKILL.md](SKILL.md) guide code generation and review decisions.

## Test

Clone the sample project:

```
git clone https://github.com/adambien/quarkus-microprofile
```

Then use the following prompt to test BC creation:

```
create coffee BC
```
