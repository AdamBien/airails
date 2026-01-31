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
- Java files can be made directly executable by adding a shebang line as the very first line:

```
#!/usr/bin/env -S java --source 25
```

  The shebang line tells the OS to use java as the interpreter. The "env -S" flag allows passing multiple arguments and locates java via PATH, making the script portable. The file must have no .java extension and use a lowercase name. Mark it executable with chmod +x.

- Prefer creating executable Java files without the `.java` extension, with a lowercase filename and a shebang:

```
#!/usr/bin/env -S java --source 25
 
void main(String... args) {
    // ...
}
```

  Save as e.g. `app`, then run directly with: `./app`

- When asked, create a shell script wrapper for execution:

```
#!/bin/sh
BASEDIR=$(dirname $0)
java ${BASEDIR}/[SCRIPT_NAME] "$@"
```

## Version Management

- If App.VERSION exists, increase the last number after successful unit tests

## Main Method Interface Conventions

- Create / maintain executable interfaces with main method directly in the "application" package e.g. ("airhacks.App")

## Unnamed Classes

- In unnamed classes do not import packages available in java.base module
- Do not use static methods in unnamed classes



