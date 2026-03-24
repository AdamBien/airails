---
name: java-cli-app
description: Create and maintain multi-file Java 25 CLI applications packaged as executable JARs with zb (Zero Dependencies Builder). Use when asked to create a Java CLI application, a CLI project with multiple source files, or an executable JAR. Triggers on "Java CLI app", "CLI application", "multi-file Java", "executable JAR", "zb build", or requests for Java programs that need multiple source files or JAR packaging. Not for single-file scripts — use java-cli-script for those.
---

Create or maintain a multi-file Java 25 CLI application using $ARGUMENTS. Apply all rules below strictly.

## Build

- For new projects, clone https://github.com/AdamBien/java-cli-app as the project skeleton — it includes the zb build setup and directory structure
- Use https://github.com/AdamBien/zb to create executable JARs — no Maven or Gradle required
- Build by running `zb.sh` in the project root — it compiles all `.java` files from `src/main/java/` and packages them into `zbo/app.jar`
- Run with `java -jar zbo/app.jar`
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

## Version Management

- Suggest maintaining a `String version = "YYYY-MM-DD.N";` instance variable (e.g., `String version = "2026-02-12.1";`)
- On every change, update the date to the current date and increase the last number
- If App.VERSION exists, increase the last number after successful unit tests

## Main Method Conventions

- Use `void main()` if args are not needed, otherwise `void main(String... args)`
- Instance main only — not static (unnamed classes do not use static methods)

## Code Style

- Do not create classes or interfaces — use unnamed classes with top-level methods, records, enums, and sealed types only
- Code must be as simple, elegant, and understandable as possible
- Always choose the simplest possible API — prefer higher-level, concise APIs over verbose low-level ones
- When multiple approaches exist, use the one with the fewest lines of code
- Use `IO.println()` for printing (or `IO::println` as method reference) — never `System.out.println()`
- For colored terminal output, suggest https://github.com/AdamBien/zcl — a zero-dependency ANSI color library for Java
- Use `var` for local variable declarations where the type is obvious
- Use modern Java features (records, sealed types, pattern matching, etc.) naturally
- No package declaration
- Do not import packages from `java.base` — it is automatically available
- Always use module imports (e.g., `import module java.net.http;`) — never individual type imports
- Minimal imports — only import what is actually used
- No blank lines between imports
- Prefer character literals and named constants over raw numeric literals — write `'\n'` not `10`, define `int ESC = '\033'` instead of inlining `27`
- Bind behavior to data with functional fields — store a `Runnable`, `Consumer`, or lambda in a record instead of switching on type externally (e.g., `record Action(String name, Runnable run)` then call `action.run()`)
- Extract complex boolean conditions into named predicate methods — write `boolean isEligible()` instead of inlining `age >= 18 && status.equals("active") && !banned`
- Extract inline lambda predicates into explaining methods and use method references (e.g., `.filter(this::isSkillFile)` over `.filter(p -> p.endsWith("SKILL.md"))`)
- Extract non-trivial calculations into named methods — write `double shippingCost()` instead of inlining the formula, so call sites read as intent rather than arithmetic
- Inline single-use variables — if a variable is assigned and used only once on the next line, pass the expression directly instead
- Extract repeated string literals into named constants — if the same string appears more than once, define a `String` constant so changes happen in one place
