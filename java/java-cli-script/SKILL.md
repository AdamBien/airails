---
name: java-cli-script
description: Create zero-dependency, single-file executable Java scripts for system-wide use via PATH. Use when asked to create a single-file Java shell script, system utility, PATH-installed Java tool, or shebang-launched Java program without the .java extension. Triggers on "Java script", "Java utility", "PATH script", "system script", or requests for single-file Java programs installed in /usr/local/bin or similar PATH directories. Not for multi-file Java applications — use java-cli-app for those.
---

Create or maintain a zero-dependency, single-file executable Java script using $ARGUMENTS. Apply all rules below strictly.

## Purpose

These are self-contained, single-file Java scripts installed in a PATH directory (e.g., `/usr/local/bin`) and invoked like any shell command — no project structure, no build tool, no `.java` extension.

## Composition

- Follow `/java-conventions` for all generic Java 25 style, syntax, naming, methods, streams, exceptions, and comments
- The rules below specialize `/java-conventions` for single-file source-mode scripts; when this skill is silent on a topic, `/java-conventions` applies

## Single-File Constraint

- Everything lives in one file — all logic, records, enums, sealed types, and helper methods
- When a script grows longer or more complex, apply [BCE Layer Grouping](#bce-layer-grouping-for-longer-or-more-complex-scripts) before considering a switch — grouping extends single-file viability considerably
- Past roughly 1000 lines even a grouped single file fights the medium — suggest switching to the `java-cli-app` skill, unless single-file deployment (copy one file to PATH, edit in place, no build) is the binding constraint
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

void main(String... args) throws Exception {
    // ...
}
```

## BCE Layer Grouping (for Longer or More Complex Scripts)

Composes with `/bce`. Short scripts stay flat: top-level methods, records, and enums ordered Boundary → Control → Entity, `main` last. When a script grows beyond roughly two screens (~150–250 lines) or accumulates several records and many methods, group its members into three interfaces named after the BCE layers:

- `interface Boundary` — the coarse-grained facade named after the script's responsibility (e.g. `listBusinessComponents`), output adapters (e.g. a `Log` enum), and the `NAME`/`VERSION` constants as bare interface fields (`String NAME = ...` — `public static final` is implicit, never write it)
- `interface Control` — stateless static functions owning all I/O and traversal, coarsest function first
- `interface Entity` — records and enums maintaining state and behavior on that state, no I/O; records keep their own explicit `static final` fields (records are classes, not interfaces)

Rules:

- The interfaces are namespaces for developer experience (IDE outline, per-layer folding, layer-labeled call sites) — never add enforcement ceremony: no constructors, no `final`, no explicit visibility modifiers
- Interfaces beat grouping classes (a class generates a default constructor that pollutes completion) and grouping enums (`values()`/`valueOf()` pollute completion; the zero-constant `;` is cryptic)
- Interface methods with bodies require the explicit `static` modifier
- Inside a nested type, derive the script name with `MethodHandles.lookup().lookupClass().getEnclosingClass().getName()` — plain `lookupClass()` reports the nested type, not the script
- Cross-layer references are qualified (`Entity.Layer` in `Control` signatures) — accepted cost: the qualifier labels the layer at every call site
- `main` cannot move into an interface — it stays top-level at the very bottom and contains exactly one statement, the invocation of the boundary facade:

```
void main(String... args) throws Exception {
    Boundary.listBusinessComponents(List.of(args));
}
```

- The layer names `Boundary`, `Control`, `Entity` are the sanctioned exception to the `/bce` rule that no type name may end with `Control` — they are layer namespaces, not domain types
- Reference implementation: `zlsbc` in the zeeds repository

## Naming Convention

- The script's filename is its application name — use it consistently in all output: version strings, help text, usage lines, and error messages
- Derive the application name dynamically at startup using `MethodHandles.lookup().lookupClass().getName()` — in source-file mode, the class name matches the filename, so there is nothing to hardcode or keep in sync

## Version Management

- Suggest maintaining a `String version = "YYYY-MM-DD.N";` instance variable (e.g., `String version = "2026-02-20.1";`)
- On every change, update the date to the current date and increase the last number
- Print the script name and version on startup as the first line of output (e.g., `IO.println(name + " " + version);`)
- Support `-version` flag to print the version — only for scripts that accept additional arguments

## Main Method Conventions

- Use `void main(String... args) throws Exception` — PATH-installed scripts always accept arguments; declaring `throws Exception` keeps the code clean by avoiding try-catch boilerplate for checked exceptions like `IOException`
- Parse arguments manually for simple scripts with few flags
- For scripts with multiple options or complex argument handling, use the `/zargs` skill to generate an enum-based argument parser — it remains zero-dependency and single-file

## Argument Handling

- Scripts with no additional arguments: do not introduce `-help` or `-version` flags — the script should just do its work
- Scripts with additional arguments: support both `-help` and `-version` flags
- Use clear, descriptive error messages for invalid input
- Exit with code 0 on success, non-zero on failure

Script with additional arguments — include `-help` and `-version`:

```
String name = MethodHandles.lookup().lookupClass().getName();
String version = "2026-02-20.1";

void main(String... args) throws Exception {
    IO.println(name + " " + version);
    if (args.length == 0 || args[0].equals("-help")) {
        IO.println("""
                Usage: %s <input> [options]
                  -help     Show this help
                  -version  Show version
                """.formatted(name));
        return;
    }
    if (args[0].equals("-version")) {
        return;
    }
    // ...
}
```

## Convenience Scripts for an Existing java-cli-app or JAR

When asked for an executable convenience script for a `/java-cli-app` project or an existing JAR, do not replicate the application's logic in the script. The script stays a single-file source script whose `main` calls into the JAR's classes. Instead:

- Keep source-file mode and add the JAR to the classpath in the same shebang — both are required so the source file compiles and can resolve the app's classes:

```
#!/usr/bin/env -S java --source 25 -cp /path/to/app.jar
```

- This is the one sanctioned exception to the "never add classpath entries to the shebang" rule under [Zero Dependencies](#zero-dependencies) — it applies only when wrapping an existing app, never for standalone scripts
- Delegate to the existing functionality by calling the app's public API. Only fall back to invoking its `main(String[])` when no other entry point exists — calling `main` directly re-runs the app's startup banner and argument parsing
- If the existing code is not callable from a single-file launcher (e.g., no accessible entry point, tightly coupled logic), recommend refactoring the `java-cli-app` to expose a public API or executable entry point rather than duplicating its behavior in the script
- Never copy or paraphrase the application's logic into the convenience script

## Zero Dependencies

- Only use the `java.base` module and standard JDK modules
- Never add external JARs or classpath entries to the shebang
- Use `java.net.http.HttpClient` for HTTP, `javax.crypto` for crypto, `java.nio.file` for file I/O
- If a task genuinely requires external libraries or multiple files, suggest using the `java-cli-app` skill with zb instead

## Code Style

Generic Java style comes from `/java-conventions`. The rules below are script-specific specializations.

- Do not create classes — use unnamed classes with top-level methods, records, enums, and sealed types; the only sanctioned named interfaces are the `Boundary`/`Control`/`Entity` layer namespaces from [BCE Layer Grouping](#bce-layer-grouping-for-longer-or-more-complex-scripts) in longer or more complex scripts
- No package declaration
- Use `IO.println()` for printing (or `IO::println` as method reference) — never `System.out.println()` and never `System.Logger` (scripts use stdout directly)
- For multi-line output, use a single `IO.println` with a text block — never multiple `IO.println` calls for consecutive lines

## stdin / stdout / stderr

- Read from stdin when no file argument is given — support piping (e.g., `cat file | scriptname`)
- Write normal output to stdout
- Write errors and diagnostics to stderr using `System.err.println()`
- For colored terminal output (status messages, errors, warnings), use the `/zcl` skill — it adds ANSI color support with zero dependencies
- Support both piped input and file arguments where applicable

## Configuration (Optional)

- For scripts that need external configuration (properties files, environment overrides), use the `/zcfg` skill to add zero-dependency configuration loading
- Most scripts should prefer command-line arguments over configuration files — only use `/zcfg` when persistent settings are genuinely needed

## Companion Libraries

The following zero-dependency libraries can be embedded directly into single-file scripts. They may need to be cloned first:

- **zcl** — colored terminal output: `git clone https://github.com/AdamBien/zcl`
- **zargs** — enum-based argument parsing pattern: https://github.com/AdamBien/zeeds/blob/main/zargs (no repo to clone — the `/zargs` skill generates the code directly)
- **zcfg** — configuration loading: `git clone https://github.com/AdamBien/zcfg`

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
