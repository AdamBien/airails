---
name: zargs
description: Add zero-dependency argument parsing to Java CLI applications using the zargs pattern — an enum-based argument parser. Use when adding CLI argument parsing, command-line option handling, or adding options to a Java project. Triggers on "zargs", "argument parsing", "CLI arguments", "command-line options", "parse arguments", "add options", or requests to add argument/option parsing to a Java CLI application without external dependencies.
---

Add zero-dependency argument parsing to a Java CLI application using $ARGUMENTS. Apply all rules below.

## What is zargs

zargs is a zero-dependency, enum-based argument parsing pattern for Java CLI applications. It is not a library or dependency — it is a set of conventions and code patterns to apply directly when generating argument parsing code.

- Reference: https://github.com/AdamBien/zeeds/blob/main/zargs
- Requires: Java 21+

## Argument Enum Pattern

Define an enum where each constant represents a CLI option. The enum provides matching, lookup, and usage generation:

```java
import java.util.stream.Collectors;
import java.util.stream.Stream;

public enum Arg {
    HELP("-help", "Show this help"),
    VERBOSE("-verbose", "Enable verbose output"),
    OUTPUT("-output", "Specify output file");

    final String option;
    final String description;

    Arg(String option, String description) {
        this.option = option;
        this.description = description;
    }

    boolean matches(String value) {
        return this.option.equals(value)
            || value.startsWith(this.option + ":");
    }

    static Arg from(String value) {
        return Stream.of(values())
            .filter(a -> a.matches(value))
            .findFirst()
            .orElse(null);
    }

    static String usage() {
        return Stream.of(values())
            .map(a -> "  %-12s %s".formatted(a.option, a.description))
            .collect(Collectors.joining("\n"));
    }
}
```

## Parsed Arguments Record

Define a record matching the application's parsed arguments:

```java
import java.util.List;

public record Args(boolean verbose, String output, List<String> files) {}
```

## Argument Syntax

zargs supports two value styles for options that take a value:

| Style | Example | Description |
|-------|---------|-------------|
| Colon-separated | `-output:result.txt` | Value follows `:` immediately |
| Space-separated | `-output result.txt` | Value is the next argument |

Boolean flags (like `-verbose`) require no value.

## Parsing Pattern

Use pattern matching with `switch` and `case null` to parse arguments:

```java
Args parseArgs(String[] args) {
    var verbose = false;
    String output = null;
    var files = new ArrayList<String>();

    for (int i = 0; i < args.length; i++) {
        var arg = args[i];
        switch (Arg.from(arg)) {
            case HELP -> printUsage();
            case VERBOSE -> verbose = true;
            case OUTPUT -> output = extractValue(arg, args, i++);
            case null -> {
                if (arg.startsWith("-")) Log.stop("Unknown option: " + arg);
                files.add(arg);
            }
        }
    }
    return new Args(verbose, output, List.copyOf(files));
}

String extractValue(String arg, String[] args, int index) {
    if (arg.contains(":")) {
        return arg.substring(arg.indexOf(":") + 1);
    }
    if (index + 1 >= args.length) {
        Log.stop("Missing value for " + arg);
    }
    return args[index + 1];
}

void printUsage() {
    Log.user("""
        Usage: appname [options] <files...>

        Options:
        %s
        """.formatted(Arg.usage()));
    System.exit(0);
}
```

## Key Design Principles

1. **Enum constants define all options** — each option is a constant with its flag string and description
2. **`Arg.from()` returns `null` for unknown args** — enables `case null` in switch for positional arguments and unknown option detection
3. **`matches()` handles both syntaxes** — exact match (`-output`) or colon-prefixed (`-output:value`)
4. **`extractValue()` extracts values** — from colon syntax or next argument, with missing-value error handling
5. **`usage()` auto-generates help text** — no manual formatting needed; derived from enum constants
6. **`Args` record is the parse result** — immutable, typed container for all parsed values

## Applying the Pattern

When generating argument parsing code, adapt these parts for the specific application:

1. **Enum constants** — define constants matching the application's specific options
2. **`Args` record fields** — match the fields to the application's parsed result shape
3. **`parseArgs` switch cases** — add a case for each enum constant
4. **Usage text** — update the program name and description in `printUsage()`

## Rules

1. Generate the enum and record directly in the application's source — there are no files to copy
2. Add `package` declarations matching the target package
3. Customize enum constants for the application's specific options
4. Use `Log.stop()` from zcl for fatal errors (missing values, unknown options)
5. Use `Log.user()` from zcl for usage/help output
6. Use `List.copyOf()` for immutable positional argument lists in the `Args` record
7. Prefer single-dash options (`-verbose`, not `--verbose`) for consistency
