---
name: java-distiller
description: Simplify, modernize, refactor, and beautify Java code to produce clean, idiomatic Java 25. Use when asked to distill, simplify, modernize, upgrade, refactor, clean up, beautify, or improve Java code. Triggers on "distill this Java", "modernize this code", "simplify this Java", "upgrade to modern Java", "refactor to Java 25", "clean up this code", "make this idiomatic", "beautify this Java", or when Java code needs transformation to current standards. Not for writing new code from scratch — use java-development for that.
---

# Java Distiller

Transform Java code into its cleanest, most modern, idiomatic form. Distill away verbosity, legacy patterns, and accidental complexity — keep only the essence.

## Philosophy

- **Distill, don't decorate** — remove complexity, don't add abstractions
- **Modern Java first** — use Java 25 features wherever they simplify the code
- **Functional over imperative** — prefer streams, lambdas, and declarative style when clearer
- **Less code, more meaning** — every remaining line should earn its place

## Workflow

1. Read the input code carefully
2. Identify transformation opportunities (see [references/transformations.md](references/transformations.md))
3. Apply transformations — prioritize those with the highest impact-to-risk ratio
4. Present the distilled code with a brief explanation of what changed and why

## Transformation Priority

Apply in this order — high-impact, low-risk first:

1. **Syntax modernization** — var, text blocks, switch expressions, module imports
2. **API upgrades** — java.time, java.nio.file, HttpClient, List.of/Map.of
3. **Pattern adoption** — records, sealed interfaces, pattern matching
4. **Functional style** — streams over loops, lambdas over anonymous classes
5. **Concurrency modernization** — virtual threads, structured concurrency, scoped values
6. **Structural simplification** — flatten hierarchies, inline trivial methods, remove dead code

## Output Format

````markdown
## Distilled

```java
// transformed code
```

### Changes
- <what changed> — <why it's better>
- ...
````

## Rules

- Preserve behavior exactly — distilling is refactoring, not rewriting
- Do not add comments, javadoc, or logging that wasn't there
- Do not add error handling beyond what exists unless the original code has a bug
- Do not introduce new dependencies
- Remove unnecessary blank lines, redundant modifiers, and dead imports
- Use `var` where the type is obvious from the right-hand side
- Use module imports (`import module java.net.http;`) over individual type imports
- Prefer `void main()` over `public static void main(String[] args)` when appropriate
- When multiple transformations compete, choose the one that produces fewer lines
- If the code is already clean and modern, say so — don't force changes

## Detailed Transformation Catalog

See [references/transformations.md](references/transformations.md) for the complete catalog of before/after transformation patterns organized by category.
