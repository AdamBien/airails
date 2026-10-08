---
name: bce-reviewer
argument-hint: full|diff [path-or-git-ref]
description: Review business component (BC) carving and Boundary-Control-Entity layering against the `/bce` rules and answer with the shortest possible feedback — one line per finding, one line when clean. Two modes, `full` for the whole codebase and `diff` for an incremental review of uncommitted changes or a git range. Stack-neutral; composes with `/bce` as rule source and the stack skill (microprofile-server, java-cli-app, web-components, aws-cdk) for stack layer rules. Use whenever BCE structure must be checked — triggers on "BCE review", "review the architecture", "review BC carving", "check the BC layout", "check layering", "is this BCE compliant", "review my changes for BCE", "/bce-reviewer". Not for code style or conventions (the stack and conventions skills own those), not for tests or build files, and never for fixing.
---

Verify the BCE structure of $ARGUMENTS and report as briefly as possible. Review only; never modify code.

## Composition

- rules come from `/bce`; this skill owns only the procedure and the output format
- the composed stack skill supplies the stack layer rules (e.g. "JAX-RS resources are boundary classes") and what counts as an external entry point; what it explicitly allows is not a finding
- dependencies are read from the source as the stack expresses them: Java `import`, ES module `import … from`, fully qualified references
- fixing belongs to the stack skill or `/sbce apply`

## Modes

| Invocation | Scope |
|---|---|
| `full [path]` | whole source tree below `path` (default: project root) |
| `diff` | uncommitted changes: `git diff HEAD` plus untracked files (`git ls-files --others --exclude-standard`) |
| `diff <ref>` | `git diff <ref>...` plus the working tree — a branch or PR range |
| mode omitted | `diff` when the working tree is dirty, otherwise `full` |

Both modes build the same BC map first: direct children of the top-level package are BCs; record each BC's layers, the cross-BC dependency edges, and per BC two counts — internal references (file pairs inside the BC that reference each other) and outgoing cross-BC references (file pairs reaching another BC). A reference is one `import` or fully qualified use per referencing file, counted once per target file; a reference to another BC's entity counts twice, because it bypasses the control contract.

- `full` reports every finding in the map
- `diff` reports only findings the change creates or worsens: new, moved, or renamed packages and files, added imports, added public control methods, added classes. Carving checks (cycles, fan-in/fan-out) use the full map as context but are reported only when the diff introduced the edge. Pre-existing findings in touched files collapse to one trailing line.

## Checks

Carving:
- layer-first layout: `boundary`, `control`, `entity` outside a BC (e.g. `app.boundary`)
- BC named after a technical concern (`util`, `common`, `model`, `dto`, `services`, `helpers`, `api`, `impl`)
- BC whose one-line responsibility needs "and" → split candidate
- BC with a single small class consumed by exactly one other BC → merge candidate
- dependency cycle between BCs
- excessive cross-BC references or shared configuration → split, merge, or rebalance
- cohesion ratio below 1 — more outgoing cross-BC references than internal ones → merge into the referenced BC, or move the reaching code there
- BC with no internal references and more than one class → a folder, not a component; split or merge
- non-trivial code in the root package → dedicated BC

Layering:
- external entry point (protocol handler, transport adapter, UI event handler, CLI entry) outside boundary
- upward dependency: entity → control or boundary, control → boundary
- cross-cutting wrapper (transaction, authorization check, request/response mapping) in control or entity
- anemic entity: state only, its behavior lives in a control
- public control method used only inside its own BC → package-private

Naming:
- meaningless suffix: `*Impl`, `*Service`, `*Manager`, `*Creator`; any name ending with `Control`
- pattern suffix (`Resource`, `Factory`, `Builder`) on an element not fulfilling that role

## Output

Terminal only; no report file.

```
bce-reviewer diff (HEAD, 7 files): 2 findings
✗ carving  orders/util/          technical BC name → move into owning BC
? layer    orders/entity/Order   anemic, logic in orders/control/Pricer → move into Order
pre-existing: 3 (run full)
```

- header: mode, baseline or path, file count (diff) or BC count (full), finding count
- one line per finding: severity, category (`carving|layer|naming`), location, defect → fix; ~100 characters max
- severity: `✗` rule violation, `?` judgment call (split/merge candidates, anemic entities, excessive coupling, cohesion ratio)
- order: carving, layer, naming — carving errors make layer findings moot
- `full` adds one BC map line, `+` joining several targets: `BCs: billing→orders+customers, orders→customers, customers`
- `full` adds one cohesion line, per BC `internal/cross-BC` references (cross-BC weighted as counted above), `?` marking a ratio below 1: `cohesion: orders 14/3 · customers 9/1 · billing 2/7 ?`
- clean is one line: `bce-reviewer full: clean (5 BCs)`

## Rules

- every finding names a location and a fix; no finding without evidence in the code
- no prose, no BCE explanations, no praise, no summaries, no restating of rules
- never present a judgment call as a violation
- do not review language style, tests, build or dependency files
- not a gate: never part of `/sbce`, `/continuous-testing`, or a stack verification loop
