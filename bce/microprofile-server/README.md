# microprofile-server

An [AIrails.dev](https://airails.dev) skill defining architecture and coding rules for long-running Java MicroProfile / Jakarta EE server applications / microservices.

## Scope

- [BCE (Boundary-Control-Entity)](https://bce.design) layering and business component (BC) structure
- JAX-RS resources and exception mapping via `WebApplicationException`
- JSON-P preferred over JSON-B; `toJSON` / `fromJSON` on record entities
- Testing conventions: unit, integration (`*IT`), and system tests in a dedicated `-st` module using microprofile-rest-client
- Maven project structure
- Minimal dependencies (Java SE → MicroProfile → Jakarta EE)
- OTEL / OpenTelemetry for metrics and observability
Not intended for serverless deployments.

## Composition

This skill composes with [`java-conventions`](../../java/java-conventions), which provides all language-level Java 25 rules (modern syntax, code style, naming, visibility, structure, methods, streams, exceptions, and documentation). `microprofile-server` adds the server-specific architecture and conventions on top.

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
