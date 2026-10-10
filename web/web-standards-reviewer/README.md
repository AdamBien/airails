# web-standards-reviewer

An [AIrails.dev](https://airails.dev) skill reviewing an existing web project for web-standards use and essential dependencies only, against the rules of [web-conventions](../web-conventions), [javascript-conventions](../javascript-conventions), and the stack skill the project composes, with one line per finding and one line when clean.

## Scope

- Build: bundlers, transpilers, preprocessors, `package.json` in the application root, `node_modules` and CDN imports
- Dependencies: every runtime dependency against the stack skill's allowlist, each with its platform replacement
- Standards: hand-rolled code the platform covers — dialogs, form validation, URL parsing, cloning, events; Baseline status as judgment calls
- Full review of a project, or incremental review of uncommitted changes or a git range

Review only — no code changes, no report file, not part of any verification loop.

## Composition

Detects the stack from the project and takes the dependency allowlist from its skill: [web-static](../web-static) and [web-sprinkles](../web-sprinkles) allow nothing at runtime, [web-components](../web-components) allows lit-html. BCE layering belongs to [bce-reviewer](../../bce/bce-reviewer), performance to [web-performance-reviewer](../web-performance-reviewer), rendered-page verification to web-static. Listing, searching, and reading files follows the [zllm-scripts](../../java/zllm-scripts) skill.

## Scripts

Evidence is collected with `files`, `search`, `show`, and `imports` — read-only, root-confined scripts from [AdamBien/zllm-scripts](https://github.com/AdamBien/zllm-scripts), bundled in [scripts](scripts) and called by path as the [zllm-scripts](../../java/zllm-scripts) skill prescribes. Nothing is installed into the OS. Requires Java 25+.

## Usage

```
/web-standards-reviewer              # diff if the working tree is dirty, full otherwise
/web-standards-reviewer full
/web-standards-reviewer full app/src
/web-standards-reviewer diff         # uncommitted changes
/web-standards-reviewer diff main    # branch range
```
