# java-cli-app

An [AIrails.dev](https://airails.dev) skill for creating and maintaining multi-file Java 25 CLI applications packaged as executable JARs.

## Scope

- Multi-file CLI app structure composing with the [`/bce`](https://bce.design) skill — feature business components (BCs) as direct children of the project package, with `boundary`, `control`, `entity` only inside a BC
- Build and packaging via [`zb`](https://github.com/AdamBien/zb) (Zero Dependencies Builder) — no Maven or Gradle
- Project skeleton from https://github.com/AdamBien/java-cli-app
- Source-bundled dependencies (no classpath resolution) — common libraries like `org.json`, `zcfg`, `zcl`
- Unit testing via the `zunit` skill
- Version management using `YYYY-MM-DD.N` convention
- Unnamed classes with top-level methods for `Main.java`; `IO.println()` instead of `System.out.println()`

Not intended for single-file scripts — use `java-cli-script` for those.

## Composition

This skill composes with [`java-conventions`](../../java/java-conventions), which provides all language-level Java 25 rules (modern syntax, code style, naming, visibility, structure, methods, streams, exceptions, and documentation). Builds and packaging are handled by the [`zb`](https://github.com/AdamBien/zb) skill. `java-cli-app` adds the CLI application architecture and conventions on top.

## Usage

This skill is activated automatically when creating, writing, or reviewing code in Java CLI application projects. The rules in [SKILL.md](SKILL.md) guide code generation and review decisions.

## Test

Clone the project skeleton:

```
git clone https://github.com/AdamBien/java-cli-app
```

Then use the following prompt to test BC creation:

```
create coffee BC
```
