---
name: zcl
description: Add colored terminal output to Java applications using zcl (Zero-dependency Colour Logger). Use when adding colored console output, terminal logging with colors, ANSI color support, or integrating zcl into a Java project. Triggers on "zcl", "colored output", "colored logging", "terminal colors", "ANSI colors", "console colors", or requests to add color-coded log output to a Java application.
---

Add colored terminal output to a Java application using $ARGUMENTS. Apply all rules below.

## What is zcl

zcl is a zero-dependency, single-enum Java logging utility using ANSI 24-bit true-colour escape sequences from the [Solarized Dark](https://github.com/altercation/solarized) accent palette. zcl is designed as a source-reuse project — the `Log.java` file is copied directly into your source tree, not consumed as a Maven/Gradle dependency or JAR. This keeps the project free of external dependencies.

- Source: https://github.com/AdamBien/zcl
- Requires: Java 21+
- API: Static methods only — all interaction is via `Log.info(...)`, `Log.error(...)`, etc.

## Integration

Copy the `Log.java` enum below into the application's source tree. Adjust the package declaration to match the target package.

```java
import java.io.PrintStream;

public enum Log {

    ERROR(Color.RED, System.err),
    USER(Color.CYAN, System.out),
    INFO(Color.GREEN, System.out),
    SYSTEM(Color.BLUE, System.out),
    WARNING(Color.YELLOW, System.out),
    DEBUG(Color.VIOLET, System.out),
    SUCCESS(Color.MAGENTA, System.out);

    private PrintStream out;

    enum Color {
        YELLOW("\033[38;2;181;137;0m"),       // Solarized Yellow #b58900
        ORANGE("\033[38;2;203;75;22m"),       // Solarized Orange #cb4b16
        RED("\033[38;2;220;50;47m"),          // Solarized Red #dc322f
        MAGENTA("\033[38;2;211;54;130m"),     // Solarized Magenta #d33682
        VIOLET("\033[38;2;108;113;196m"),     // Solarized Violet #6c71c4
        BLUE("\033[38;2;38;139;210m"),        // Solarized Blue #268bd2
        CYAN("\033[38;2;42;161;152m"),        // Solarized Cyan #2aa198
        GREEN("\033[38;2;133;153;0m");

        String code;

        Color(String code) {
            this.code = code;
        }
    }

    private final String value;
    private final static String RESET = "\u001B[0m";

    private Log(Color color, PrintStream out) {
        this.value = (color.code + "%s" + RESET);
        this.out = out;
    }

    String formatted(String raw) {
        return this.value.formatted(raw);
    }

    void out(String message) {
        var colored = formatted(message);
        this.out.println(colored);
    }

    public static void debug(String message) {
        Log.DEBUG.out(message);
    }

    public static void error(String message) {
        Log.ERROR.out(message);
    }

    public static void error(String message, Exception e) {
        Log.ERROR.out(message + ": " + e.getMessage());
        e.printStackTrace(System.err);
    }

    public static void user(String message){
        Log.USER.out(message);
    }

    public static void info(String message){
        Log.INFO.out(message);
    }

    public static void system(String message){
        Log.SYSTEM.out(message);
    }

    public static void clearScreen(){
        System.out.println("\033c");
    }

    public static void warning(String message){
        Log.WARNING.out(message);
    }

    public static void success(String message){
        Log.SUCCESS.out(message);
    }

    public static void stop(String message){
        error(message);
        System.exit(0);
    }
}
```

## Log Levels

| Level | Solarized Color | Stream | Static Method | Purpose |
|-------|----------------|--------|---------------|---------|
| `ERROR` | Red | `System.err` | `Log.error(msg)` | Critical failures |
| `USER` | Cyan | `System.out` | `Log.user(msg)` | User messages and interactive elements |
| `INFO` | Green | `System.out` | `Log.info(msg)` | General information |
| `SYSTEM` | Blue | `System.out` | `Log.system(msg)` | System-level messages |
| `WARNING` | Yellow | `System.out` | `Log.warning(msg)` | Warning conditions |
| `DEBUG` | Violet | `System.out` | `Log.debug(msg)` | Debug information for developers |
| `SUCCESS` | Magenta | `System.out` | `Log.success(msg)` | Success messages and completions |

## Usage

All interaction uses the public static convenience methods. The instance methods `out()` and `formatted()` are package-private.

```java
Log.error("Database connection failed");
Log.error("Connection failed", exception); // logs message + full stack trace to stderr
Log.user("Welcome to the application");
Log.info("Operation completed");
Log.system("Server started on port 8080");
Log.warning("Cache miss detected");
Log.debug("Processing item #42");
Log.success("Build completed successfully");
Log.clearScreen();                          // sends ESC c to reset the terminal
Log.stop("Fatal error — shutting down");    // logs error and calls System.exit(0)
```

## Integration Rules

1. Copy `Log.java` into the project source tree — do not add it as a Maven/Gradle dependency. Only include the enum constants, `Color` entries, and static methods that the target project actually uses. Remove unused log levels and their corresponding colors and convenience methods.
2. Add a `package` declaration matching the target package at the top of the copied file
3. Use the static methods (`Log.error()`, `Log.info()`, `Log.system()`, etc.) — never call the package-private instance methods `out()` or `formatted()` from outside the enum
4. Replace `System.out.println` calls with the appropriate `Log` level
5. Use `Log.error(message, exception)` for exception logging — it prints the message and full stack trace to `System.err`
6. Use `Log.stop(message)` only for unrecoverable errors that require immediate process termination
7. Add domain-specific log levels for the application. If the project has a clear domain concept that deserves its own color (e.g., `PROMPT`, `RESPONSE`, `QUERY`, `METRIC`), add a new enum constant with an appropriate Solarized `Color` and a corresponding `public static void` convenience method. Domain-specific levels make log output immediately scannable by category.
