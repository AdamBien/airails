# microprofile-server

An [AIrails.dev](https://airails.dev) skill defining architecture and coding rules for long-running Java MicroProfile / Jakarta EE server applications / microservices.

## Scope

- [BCE (Boundary-Control-Entity)](https://bce.design) layering
- JAX-RS resources, CDI, JSON-P
- Testing conventions: unit, integration, and system tests
- Maven project structure
- Java 25 with modern syntax
- Minimal dependencies (Java SE → MicroProfile → Jakarta EE)


Not intended for serverless deployments.

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
