---
name: java-cli-script
description: Create zero-dependency, single-file executable Java scripts for system-wide use via PATH. Use when asked to create a single-file Java shell script, system utility, PATH-installed Java tool, or shebang-launched Java program without the .java extension. Triggers on "Java script", "Java utility", "PATH script", "system script", or requests for single-file Java programs installed in /usr/local/bin or similar PATH directories. Not for multi-file Java applications — use java-cli-app for those.
---

Create or maintain a zero-dependency, single-file executable Java script using $ARGUMENTS. Apply all rules below strictly.

## Purpose

These are self-contained, single-file Java scripts installed in a PATH directory (e.g., `/usr/local/bin`) and invoked like any shell command — no project structure, no build tool, no `.java` extension.

## Single-File Constraint

- Everything lives in one file — all logic, records, enums, sealed types, and helper methods
- If the script grows beyond what fits comfortably in a single file, suggest switching to the `java-cli` skill instead
- No multi-file source-code programs — this skill is strictly for single-file scripts

## Build

- Use Java 25 source-file mode — no build tool, no compilation step
- Never use the `.java` extension — scripts are executable files with a lowercase filename and a shebang
- Never use `--enable-preview` in the shebang — Java 25 is a GA release, all features used here are standard
- The shebang line is always:

```
#!/usr/bin/env -S java --source 25
```

- Save with a short, descriptive, lowercase command name (e.g., `jsonformat`, `portcheck`, `sysinfo`)
- Never use dashes in filenames (e.g., `hello-world`) — Java source-file mode does not support them. Use camelCase instead (e.g., `helloWorld`)
- Mark executable: `chmod +x scriptname`
- Install by copying or symlinking to a PATH directory: `cp scriptname /usr/local/bin/` or `ln -s $(pwd)/scriptname /usr/local/bin/scriptname`

## Script Structure

A minimal script:

```
#!/usr/bin/env -S java --source 25

void main(String... args) {
    // ...
}
```

A script with helper methods and records:

```
#!/usr/bin/env -S java --source 25

record Result(String name, int value) {}

Result process(String input) {
    // ...
}

void main(String... args) {
    // ...
}
```

## Version Management

- Suggest maintaining a `String version = "YYYY-MM-DD.N";` instance variable (e.g., `String version = "2026-02-20.1";`)
- On every change, update the date to the current date and increase the last number
- Support `--version` flag to print the version

## Main Method Conventions

- Use `void main(String... args)` — PATH-installed scripts always accept arguments
- Instance main only — not static (unnamed classes do not use static methods)
- Parse arguments manually — no argument-parsing libraries

## Argument Handling

- Support `--help` and `--version` flags in every script
- Print usage information to stdout when `--help` is passed or when required arguments are missing
- Use clear, descriptive error messages for invalid input
- Exit with code 0 on success, non-zero on failure

```
void main(String... args) {
    if (args.length == 0 || args[0].equals("--help")) {
        IO.println("Usage: scriptname <input> [options]");
        IO.println("  --help     Show this help");
        IO.println("  --version  Show version");
        return;
    }
    if (args[0].equals("--version")) {
        IO.println("scriptname " + version);
        return;
    }
    // ...
}
```

## Zero Dependencies

- Only use the `java.base` module and standard JDK modules
- Never add external JARs or classpath entries to the shebang
- Use `java.net.http.HttpClient` for HTTP, `javax.crypto` for crypto, `java.nio.file` for file I/O
- If a task genuinely requires external libraries or multiple files, suggest using the `java-cli-app` skill with zb instead

## Code Style

- Do not create classes or interfaces — use unnamed classes with top-level methods, records, enums, and sealed types only
- Code must be as simple, elegant, and understandable as possible
- Always choose the simplest possible API — prefer higher-level, concise APIs over verbose low-level ones
- When multiple approaches exist, use the one with the fewest lines of code
- Use `IO.println()` for printing (or `IO::println` as method reference) — never `System.out.println()`
- Use `var` for local variable declarations where the type is obvious
- Use modern Java features (records, sealed types, pattern matching, etc.) naturally
- No package declaration
- Do not import packages from `java.base` — it is automatically available
- Always use module imports (e.g., `import module java.net.http;`) — never individual type imports
- Minimal imports — only import what is actually used
- No blank lines between imports
- Prefer character literals and named constants over raw numeric literals — write `'\n'` not `10`, define `int ESC = '\033'` instead of inlining `27`
- Bind behavior to data with functional fields — store a `Runnable`, `Consumer`, or lambda in a record instead of switching on type externally
- Extract complex boolean conditions into named predicate methods
- Extract non-trivial calculations into named methods
- Inline single-use variables — if a variable is assigned and used only once on the next line, pass the expression directly instead
- Extract repeated string literals into named constants

## stdin / stdout / stderr

- Read from stdin when no file argument is given — support piping (e.g., `cat file | scriptname`)
- Write normal output to stdout
- Write errors and diagnostics to stderr using `System.err.println()`
- Support both piped input and file arguments where applicable

## Installation Guidance

When presenting the finished script, always include installation instructions:

```
# Make executable
chmod +x scriptname

# Install system-wide (pick one)
sudo cp scriptname /usr/local/bin/
# or symlink for development
sudo ln -s $(pwd)/scriptname /usr/local/bin/scriptname
```
