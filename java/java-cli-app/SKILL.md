---
name: java-cli-app
description: Create and maintain multi-file Java 25 CLI applications packaged as executable JARs with zb (Zero Dependencies Builder). Use when asked to create a Java CLI application, a CLI project with multiple source files, or an executable JAR. Triggers on "Java CLI app", "CLI application", "multi-file Java", "executable JAR", "zb build", or requests for Java programs that need multiple source files or JAR packaging. Not for single-file scripts — use java-cli-script for those.
---

Create or maintain a multi-file Java 25 CLI application using $ARGUMENTS. Apply all rules below strictly.

## Build

- Use https://github.com/AdamBien/zb to create executable JARs — no Maven or Gradle required
- Never use `--enable-preview` — Java 25 is a GA release, all features used here are standard
- Source files use the `.java` extension and live in the project directory

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
