---
name: java-cli
description: Create and maintain Java 25 CLI applications in source-file mode. Use when asked to create Java CLI tools, scripts, single-file Java applications, unnamed-class programs, or shebang-launched Java scripts. Triggers on "Java CLI", "Java script", "source-file mode", "single-file Java", or requests for executable Java programs without a build tool.
---

Create or maintain a Java 25 CLI application using $ARGUMENTS. Apply all rules below strictly.

## Build

- Use Java 25 source-file mode — no build tool required
- If necessary, use https://github.com/AdamBien/zb to create executable JARs
- Never use the `.java` extension — always create executable files with a lowercase filename and a shebang:

```
#!/usr/bin/env -S java --source 25

void main(String... args) {
    // ...
}
```

  Save as e.g. `app`, mark executable with `chmod +x`, then run with: `./app`

- When asked, create a shell script wrapper for execution (replace `SCRIPT_NAME`):

```
#!/bin/sh
BASEDIR=$(dirname $0)
java ${BASEDIR}/SCRIPT_NAME "$@"
```

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
- Extract non-trivial calculations into named methods — write `double shippingCost()` instead of inlining the formula, so call sites read as intent rather than arithmetic
- Inline single-use variables — if a variable is assigned and used only once on the next line, pass the expression directly instead
- Extract repeated string literals into named constants — if the same string appears more than once, define a `String` constant so changes happen in one place
