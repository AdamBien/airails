---
name: java-cli-app
description: Create and maintain multi-file Java 25 CLI applications packaged as executable JARs with zb (Zero Dependencies Builder). Use when asked to create a Java CLI application, a CLI project with multiple source files, or an executable JAR. Triggers on "Java CLI app", "CLI application", "multi-file Java", "executable JAR", "zb build", or requests for Java programs that need multiple source files or JAR packaging. Not for single-file scripts — use java-cli-script for those.
argument-hint: "[description of the CLI application to build]"
---

Create or maintain a multi-file Java 25 CLI application using $ARGUMENTS.

## Architecture

- **Default: multi-file CLI apps compose with the `/bce` skill.** The package path follows `/bce`'s `[ORGANIZATION].[PROJECT].[BC].[layer]` scheme: the organization package (e.g. `airhacks`) contains the application-level project package (e.g. `zbl`), whose direct children are the feature business components (BCs); `boundary`, `control`, `entity` only appear *inside* a BC, never directly under the application-level package.
  ```
  src/main/java/
    Main.java                                              # unnamed package, compact source
    <organization>/<project>/<bc-a>/{boundary,control,entity}/
    <organization>/<project>/<bc-b>/{boundary,control,entity}/
    <organization>/<project>/<bc-c>/control/               # BC may have only the layers it needs
  ```
  - If a design doc or input layout places `boundary/control/entity` directly under the application-level package, flag it as a BCE violation and propose feature BCs before scaffolding.
- Exception: small single-purpose tools (one screen of logic, no distinct concerns) — keep everything in one unnamed class with top-level methods, no BCs, no packages.

## Build

- For new projects, clone https://github.com/AdamBien/java-cli-app as the project skeleton — it includes the zb build setup and directory structure. When the user asks for a minimal starter, use https://github.com/adambien/z-java-cli-app instead.
- Build and run with the `/zb` skill — no Maven or Gradle required; zb compiles all `.java` files from `src/main/java/` into `zbo/app.jar`
- Offer to generate an executable launcher script with the `/java-cli-script` skill so the app runs by name instead of `java -jar zbo/app.jar`

## Dependencies

zb has no classpath or dependency resolution — all dependencies are bundled as source code directly in `src/main/java/` under their original package structure.

**How to add a dependency:**
1. **Ask the user first** whether they already have the dependency source locally (e.g., a cloned repo or a local path) — do NOT fetch from GitHub without asking
2. If the user provides a local path, copy the `.java` source files from that path into `src/main/java/<package-path>/`
3. If the user does not have it locally, clone from the dependency's source repository
4. Keep the original package declarations — zb compiles everything it finds
5. Only include the source files you actually need

Source locations and copy targets for common dependencies are owned by their skills: `/zjson` (org.json), `/zcfg` (configuration), `/zcl` (colored logging).

## Unit Testing

- **Suggest the `/zunit` skill** for testing — it is the testing approach for `java-cli-app` projects

## Version Management

- Suggest maintaining a `String version = "YYYY-MM-DD.N";` field (e.g., `String version = "2026-02-12.1";`)
- On every change, after unit tests pass, set the date to the current date and increase the last number

## Code Style

**Default: compose with the `/java-conventions` skill** for generic Java style, naming, visibility, structure, streams, exceptions, and documentation rules. The rules below override or extend it for the CLI-app context.

- Use unnamed classes with top-level methods for `Main.java` — no package declaration
- Use `IO.println()` for printing (or `IO::println` as method reference) — never `System.out.println()`
- For colored terminal output, suggest zcl via the `/zcl` skill
