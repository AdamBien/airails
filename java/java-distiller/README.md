# java-distiller

A Claude Code skill that transforms Java code into its cleanest, most modern, idiomatic Java 25 form. Removes verbosity, legacy patterns, and accidental complexity — keeps only the essence.

## Transformation Priority

Applied in order — high-impact, low-risk first:

```mermaid
graph LR
    A[Syntax Modernization] --> B[API Upgrades]
    B --> C[Pattern Adoption]
    C --> D[Functional Style]
    D --> E[Concurrency Modernization]
    E --> F[Structural Simplification]
```

| Category | Examples |
|---|---|
| **Syntax Modernization** | `var`, text blocks, switch expressions, module imports, `void main()` |
| **API Upgrades** | `java.time`, `java.nio.file`, `HttpClient`, `List.of`/`Map.of`, `String.formatted` |
| **Pattern Adoption** | Records, sealed interfaces, pattern matching (`instanceof`, `switch`, deconstruction) |
| **Functional Style** | Streams over loops, method references, lambdas, `Optional` over null checks |
| **Concurrency** | Virtual threads, structured concurrency, scoped values |
| **Structural** | try-with-resources, remove redundant modifiers, sequenced collections |

## Usage

Invoke via Claude Code:

```
/java-distiller
```

Trigger phrases: "distill", "simplify", "modernize", "upgrade", "refactor", "clean up", "beautify", "make idiomatic".

## Rules

- Preserves behavior exactly — distilling is refactoring, not rewriting
- Does not add comments, javadoc, logging, or error handling beyond what exists
- Does not introduce new dependencies
- When the code is already clean and modern, says so
