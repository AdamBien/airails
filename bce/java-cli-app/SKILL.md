---
name: java-cli-app
description: Create and maintain multi-file Java 25 CLI applications packaged as executable JARs with zb (Zero Dependencies Builder). Use when asked to create a Java CLI application, a CLI project with multiple source files, or an executable JAR. Triggers on "Java CLI app", "CLI application", "multi-file Java", "executable JAR", "zb build", or requests for Java programs that need multiple source files or JAR packaging. Not for single-file scripts — use java-cli-script for those.
---

Create or maintain a multi-file Java 25 CLI application using $ARGUMENTS. Apply all rules below strictly.

## Architecture

- **Default: multi-file CLI apps compose with the `/bce` skill.** Feature business components (BCs) are direct children of the project package; `boundary`, `control`, `entity` only appear *inside* a BC, never directly under the project root.
  ```
  src/main/java/
    Main.java                                  # unnamed package, compact source
    <project>/<bc-a>/{boundary,control,entity}/
    <project>/<bc-b>/{boundary,control,entity}/
    <project>/<bc-c>/control/                  # BC may have only the layers it needs
  ```
  - Name BCs after their domain responsibilities (e.g., `auth`, `users`, `billing`), not technical concerns.
  - If a design doc or input layout places `boundary/control/entity` directly under the project root, flag it as a BCE violation and propose feature BCs before scaffolding.
- Exception: small single-purpose tools (one screen of logic, no distinct concerns) — keep everything in one unnamed class with top-level methods, no BCs, no packages.

## Build

- For new projects, clone https://github.com/AdamBien/java-cli-app as the project skeleton — it includes the zb build setup and directory structure
- Alternatively, bootstrap from the minimalistic template https://github.com/adambien/z-java-cli-app — a stripped-down starter for fresh `/java-cli-app` projects
- Use https://github.com/AdamBien/zb to create executable JARs — no Maven or Gradle required
- Build by running `zb.sh` in the project root — it compiles all `.java` files from `src/main/java/` and packages them into `zbo/app.jar`
- Run with `java -jar zbo/app.jar`
- Offer to generate an executable launcher script with the `/java-cli-script` skill so the app runs by name instead of `java -jar zbo/app.jar`
- Never use `--enable-preview` — Java 25 is a GA release, all features used here are standard
- Source files use the `.java` extension and live in the project directory

## Dependencies

zb has no classpath or dependency resolution — all dependencies are bundled as source code directly in `src/main/java/` under their original package structure.

**How to add a dependency:**
1. **Ask the user first** whether they already have the dependency source locally (e.g., a cloned repo or a local path) — do NOT fetch from GitHub without asking
2. If the user provides a local path, copy the `.java` source files from that path into `src/main/java/<package-path>/`
3. If the user does not have it locally, clone from the source URL listed below
4. Keep the original package declarations — zb compiles everything it finds
5. Only include the source files you actually need

**Common dependencies and their source locations:**

| Dependency | Ask user for local path to | Fallback source | Copy to |
|-----------|---------------------------|----------------|---------|
| org.json (JSON processing) | org.json / z-JSON-java repo | `https://github.com/AdamBien/z-JSON-java` | `src/main/java/org/json/` |
| zcfg (configuration) | zcfg repo | `https://github.com/AdamBien/zcfg` | `src/main/java/airhacks/zcfg/` |
| zcl (colored logging) | zcl repo | `https://github.com/AdamBien/zcl` | `src/main/java/` |

## Unit Testing

- **Suggest the `/zunit` skill** for testing — use it to generate and run unit tests for the project
- `/zunit` is the testing approach for `java-cli-app` projects; do not use JUnit or other frameworks here

## Version Management

- Suggest maintaining a `String version = "YYYY-MM-DD.N";` instance variable (e.g., `String version = "2026-02-12.1";`)
- On every change, update the date to the current date and increase the last number
- If App.VERSION exists, increase the last number after successful unit tests

## Code Style

**Default: compose with the `/java-conventions` skill** for generic Java style, naming, visibility, structure, streams, exceptions, and documentation rules. The rules below override or extend it for the CLI-app context.

- Use unnamed classes with top-level methods for `Main.java` — no package declaration
- Use `IO.println()` for printing (or `IO::println` as method reference) — never `System.out.println()`
- For colored terminal output, suggest https://github.com/AdamBien/zcl — a zero-dependency ANSI color library for Java
