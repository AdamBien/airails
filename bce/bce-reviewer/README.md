# bce-reviewer

An [AIrails.dev](https://airails.dev) skill reviewing business component carving and [Boundary-Control-Entity (BCE/ECB)](https://bce.design) layering against the [bce](../bce) rules, with one line per finding and one line when clean.

## Scope

- BC carving: layer-first layouts, technical BC names, split and merge candidates, cycles, coupling, and the per-BC cohesion ratio (internal vs. cross-BC references)
- Layering: entry points outside boundary, upward dependencies, misplaced cross-cutting concerns, anemic entities, control visibility
- Naming: BC names against the project's own vocabulary and responsibility statements, generic and activity names, suffix bans from `bce`
- Full review of a codebase, or incremental review of uncommitted changes or a git range

Review only — no code changes, no report file, not part of any verification loop.

## Composition

Composes `bce` as rule source with a stack skill (`microprofile-server`, `java-cli-app`, `web-components`, `aws-cdk`) supplying stack layer rules. Stack conventions, tests, and build files are out of scope.

## Usage

```
/bce-reviewer              # diff if the working tree is dirty, full otherwise
/bce-reviewer full
/bce-reviewer full src/main/java/airhacks/orders
/bce-reviewer diff         # uncommitted changes
/bce-reviewer diff main    # branch range
```
