---
name: zcl
description: Add colored terminal output to Java applications using zcl (Zero-dependency Colour Logger). Use when adding colored console output, terminal logging with colors, ANSI color support, or integrating zcl into a Java project. Triggers on "zcl", "colored output", "colored logging", "terminal colors", "ANSI colors", "console colors", or requests to add color-coded log output to a Java application.
---

Add colored terminal output to a Java application using $ARGUMENTS. Apply all rules below.

## What is zcl

zcl is a zero-dependency, single-enum Java logging utility with ANSI color support for enhanced console output readability. zcl is designed as a source-reuse project — the `Log.java` file is copied directly into your source tree, not consumed as a Maven/Gradle dependency or JAR. This keeps the project free of external dependencies.

- Source: https://github.com/AdamBien/zcl
- Requires: Java 21+

## Integration

Copy the `Log.java` enum below into the application's source tree. Adjust the package declaration to match the target package.

```java
import java.io.PrintStream;

public enum Log {

    ERROR(Color.BRIGHT_RED, System.err),
    USER(Color.BRIGHT_CYAN, System.out),
    INFO(Color.BRIGHT_GREEN, System.out),
    SYSTEM(Color.SKY_BLUE, System.out),
    WARNING(Color.WARM_YELLOW, System.out),
    DEBUG(Color.SOFT_PURPLE, System.out);

    PrintStream out;

    enum Color {
        SOFT_GRAY("\033[38;5;246m"),
        WARM_YELLOW("\033[38;5;220m"),
        BRIGHT_BLUE("\033[38;5;33m"),
        BRIGHT_WHITE("\033[38;5;255m"),
        BRIGHT_RED("\033[38;5;196m"),
        BRIGHT_GREEN("\033[38;5;46m"),
        BRIGHT_YELLOW("\033[38;5;226m"),
        SKY_BLUE("\033[38;5;39m"),
        SOFT_PURPLE("\033[38;5;129m"),
        BRIGHT_CYAN("\033[38;5;51m"),
        BLACK_ON_WHITE("\033[38;5;232;48;5;255m");

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

    public String formatted(String raw) {
        return this.value.formatted(raw);
    }

    public void out(String message) {
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
        Log.INFO.out(message);
    }

    public static void warning(String message){
        Log.WARNING.out(message);
    }

    public static void stop(String message){
        error(message);
        System.exit(0);
    }
}
```

## Log Levels

| Level | Color | Stream | Purpose |
|-------|-------|--------|---------|
| `ERROR` | Bright Red | `System.err` | Critical failures |
| `USER` | Bright Cyan | `System.out` | User messages and interactive elements |
| `INFO` | Bright Green | `System.out` | Success messages and general information |
| `SYSTEM` | Sky Blue | `System.out` | System-level messages |
| `WARNING` | Warm Yellow | `System.out` | Warning conditions |
| `DEBUG` | Soft Purple | `System.out` | Debug information for developers |

## Usage

Two equivalent patterns — enum constants and static convenience methods:

```java
// Enum constants
Log.ERROR.out("Critical error occurred");
Log.INFO.out("Operation completed");
Log.WARNING.out("Resource usage high");
Log.DEBUG.out("Entering method X");
Log.USER.out("Welcome");
Log.SYSTEM.out("Server started on port 8080");

// Static convenience methods
Log.error("Database connection failed");
Log.error("Connection failed", exception); // logs message + full stack trace
Log.user("Welcome to the application");
Log.warning("Cache miss detected");
Log.debug("Processing item #42");
Log.stop("Fatal error — shutting down");   // logs error and calls System.exit(0)
```

## Integration Rules

1. Copy `Log.java` as-is into the project source tree — do not add it as a Maven/Gradle dependency
2. Add a `package` declaration matching the target package at the top of the copied file
3. Use `Log.ERROR` (to `System.err`) for errors and `Log.INFO`/`Log.SYSTEM`/`Log.USER`/`Log.WARNING`/`Log.DEBUG` (to `System.out`) for everything else
4. Replace `System.out.println` calls with the appropriate `Log` level
5. Use `Log.error(message, exception)` for exception logging — it prints the message and full stack trace to `System.err`
6. Use `Log.stop(message)` only for unrecoverable errors that require immediate process termination
