---
name: python-to-java
description: >
  Convert Python scripts to zero-dependency Java 25 CLI programs. Use when
  asked to migrate, convert, port, or rewrite Python scripts (.py) to Java,
  or when translating a Python codebase to Java CLI scripts or apps. Triggers
  on "convert Python to Java", "migrate Python", "port to Java", "rewrite in
  Java", or when a .py file is presented with instructions to produce a Java
  equivalent. Prefer /java-cli-script for single-file scripts; switch to
  /java-cli-app when shared code duplication across scripts exceeds ~100 lines.
---

# Python to Java Migration

Convert Python scripts to idiomatic Java 25 CLI programs with zero external dependencies.

## Decision: Single-File vs Multi-File

Evaluate **before writing any code**:

1. **Start with `/java-cli-script`** — one Java file per Python script, no build tool
2. **Switch to `/java-cli-app`** when any of these apply:
   - Multiple scripts share >100 lines of identical code (e.g. domain records)
   - A single script exceeds ~500 lines
   - Scripts import from each other in Python (`from module import func`)

When switching to `/java-cli-app`, extract shared code into common source files within the multi-file project.

## Migration Process

1. **Read all Python scripts** — understand dependencies between them (`import` graph)
2. **Identify shared code** — `utils.py`, common helpers, shared data classes
3. **Decide single-file vs multi-file** per the rules above
4. **Translate each script** using the idiom mapping below
5. **Verify** — run each script with `-version` to confirm compilation

## Python to Java 25 Idiom Mapping

See [references/idiom-mapping.md](references/idiom-mapping.md) for the complete translation table covering argument parsing, file I/O, subprocess management, concurrency, and standard library equivalents.

