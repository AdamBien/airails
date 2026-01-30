---
description: Create and maintain Java 25 CLI applications in source-file mode. Use when asked to create Java CLI tools, scripts, or single-file Java applications.
argument-hint: "[description of the CLI application to create]"
---

Create or maintain a Java 25 CLI application using $ARGUMENTS. Apply all rules below strictly.

## Language

- Use Java 25

## Build

- Prefer Java source files without any build tool
- If necessary, use https://github.com/AdamBien/zb to create executable JARs
- Java 25 CLI applications will be executed in source-file mode
- When asked, create a script for execution:

```
#!/bin/sh
BASEDIR=$(dirname $0)
java ${BASEDIR}/[SCRIPT_NAME].java "$@"
```

## Version Management

- If App.VERSION exists, increase the last number after successful unit tests

## Main Method Interface Conventions

- Create / maintain executable interfaces with main method directly in the "application" package e.g. ("airhacks.App")

## Unnamed Classes

- In unnamed classes do not import packages available in java.base module
- Do not use static methods in unnamed classes



