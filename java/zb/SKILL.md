---
name: zb
description: Build Java 21+ projects with zb (Zero Dependencies Builder). Use when compiling, building, packaging, or running Java projects that have no external dependencies. Triggers on "build with zb", "zb build", "compile and package", "create executable JAR", or when a .zb configuration file is present in the project. Not for projects requiring Maven/Gradle dependency management.
---

# zb - Zero Dependencies Builder

Build single-module Java 21+ projects into executable JARs without Maven or Gradle.

## Installation

Download `zb.jar` and `zb.sh` from [github.com/AdamBien/zb](https://github.com/AdamBien/zb) into your project root. Requires Java 21+.

## Build Command

```bash
zb.sh
```

Or directly:

```bash
java -jar zb.jar
```

## Source Directory Convention

zb auto-detects sources in order:

1. `src/main/java` (Maven convention, preferred)
2. `src/`
3. Current directory `.`

Resources auto-detected from `src/main/resources` or current directory.

## Entry Point Requirement

The project must have exactly one class with a `void main(` method (Java 21+ unnamed main).

## Configuration (.zb)

Optional `.zb` properties file in project root (auto-generated on first run):

```properties
sources.dir=<discovered by zb>
resources.dir=<discovered by zb>
classes.dir=<temp.dir>
jar.dir=zbo/
jar.file.name=app.jar
```

- `<discovered by zb>` - auto-detect directory
- `<temp.dir>` - use temporary directory, cleaned after build

## CLI Arguments (Positional)

```bash
java -jar zb.jar [sources] [classes] [jar_dir] [jar_file]
```

Precedence: CLI args > `.zb` file > defaults.

## Output

Default: `zbo/app.jar` - executable JAR with Main-Class manifest entry.

Run with:

```bash
java -jar zbo/app.jar
```

## Project Structure for zb-Compatible Projects

```
project/
├── src/main/java/       # Java sources
│   └── com/example/
│       └── App.java     # Must contain void main(
├── src/main/resources/  # Optional resources (included in JAR)
│   └── META-INF/services/  # Optional SPI configuration
├── .zb                  # Optional configuration
└── zbo/
    └── app.jar          # Build output
```

## Constraints

- Java 21+ required
- No external dependency support (no Maven/Gradle dependency resolution)
- Single-module projects only
- Exactly one main class per project
