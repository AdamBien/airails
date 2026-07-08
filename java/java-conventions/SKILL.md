---
name: java-conventions
description: Generic, composable Java 25 code conventions — modern syntax, code style, naming, visibility, structure, methods, streams, exceptions, and documentation rules that apply across all Java contexts (single-file scripts, CLI apps, MicroProfile/Jakarta EE servers, libraries). Technology-neutral within the Java world; meant to be composed with context-specific skills (e.g. `java-cli-script`, `java-cli-app`, `microprofile-server`, `bce`). Use when writing, generating, or reviewing Java code anywhere the composed skill does not already specify style. Triggers on "Java conventions", "Java style", "Java code style", "modern Java", "Java 25", "idiomatic Java", or any request to write or review Java code where context-specific skills do not already cover style.
---

## Scope

- Language-level rules for Java 25 only — syntax, style, naming, visibility, structure, methods, streams, exceptions, comments.
- Architecture (BCE, layering, packaging) is **not** in this skill — see `bce`.
- Build, packaging, framework, and protocol rules are **not** in this skill — see `java-cli-script`, `java-cli-app`, `microprofile-server`, `zb`, etc.
- When a composed skill specifies a rule, the composed skill wins; this skill is the fallback baseline.

## Java Version & Syntax

- target Java 25; assume all features are GA — never use `--enable-preview`
- use modern syntax naturally: `var`, records, sealed types, pattern matching, text blocks, switch expressions
- use `var` for local variable declarations where the type is obvious from the right-hand side
- use module imports (e.g. `import module java.net.http;`) over individual type imports
- do not import packages from `java.base` — it is automatically available
- use switch expressions with arrow syntax (`case X -> ...`) over old `case X:` statements with `break`
- use pattern matching for `instanceof` — `if (o instanceof String s)` over cast-and-assign
- use pattern matching in `switch` for type-based dispatch
- use the diamond operator `<>` for generic type inference
- never use raw generic types — always parameterize
- prefer `void main()` / `void main(String... args)` over `public static void main(String[] args)`; instance main, not static

## Java SE APIs

- use Java SE APIs over writing custom code — the standard library has it, usually in `java.util`, `java.nio.file`, `java.net.http`, or `java.time`
- prefer the most specific Java SE type for the domain — `Path` over `String`, `URI` over `String`, `Duration` over `long millis`, `Instant`/`LocalDate` over `Date`/`long`

## Visibility & Modifiers

- avoid `private` methods (including `private static` helpers) — prefer package-private (default) so same-package unit tests can exercise them directly, including edge cases, without round-tripping through the public API
- avoid `private` fields — prefer package-private so same-package tests can read or seed state without reflection or extra accessors
- reserve `private` for genuinely sensitive state (credentials, security tokens, invariants that must never be observed externally); the burden of justification is on `private`, not on package-private
- do not use `final` on fields — exception: `static final` for constants like `LOGGER`
- do not use `final` on local variables or parameters
- do not use constructor injection — prefer field injection in CDI contexts
- avoid mutable static fields

## Interfaces & Classes

- only use interfaces with multiple implementations or for the strategy pattern; never create an interface whose only purpose is to be implemented by one class
- for stateless or procedural logic, prefer interfaces with `static` methods over classes with private constructors
- in utility interfaces, prefer `static` over `default` methods
- avoid anonymous inner classes — extract them into named, testable top-level classes (e.g. a CDI bean produced via `@Produces`) instead of instantiating an interface inline
- use records by default for value types and data carriers
- use sealed interfaces or sealed classes for closed type hierarchies (pairs well with pattern matching)
- prefer factory methods (static `of`, `from`, etc.) in records over passing `null` to constructors
- prefer composition over inheritance
- create multiple classes only if it decreases complexity and increases readability

## Naming

- name classes, modules, and files after their responsibilities, not technical concerns
- avoid meaningless suffixes: `*Impl`, `*Service`, `*Manager`, `*Creator`
- class names must not end with `Control`
- reserve protocol- or pattern-specific suffixes for elements that actually fulfill that role: `Resource` for JAX-RS classes, `Factory` for actual GoF factories, `Builder` for classes with method chaining
- avoid the `get` prefix; use the record-style convention — `configuration()` not `getConfiguration()`

## Methods & Lambdas

- keep methods short, cohesive, and testable
- create well-named methods for coarse-grained, self-contained logic
- never use multi-statement lambdas — extract them into well-named helper methods
- prefer method references over equivalent lambdas (`String::strip` over `s -> s.strip()`, `this::isSkillFile` over `p -> p.endsWith("SKILL.md")`)
- extract inline lambda predicates into explaining methods and use method references
- split complex `.filter()` calls with multiple `&&`/`||` conditions into chained `.filter()` calls
- extract complex boolean conditions into named predicate methods — write `boolean isEligible()` instead of inlining `age >= 18 && status.equals("active") && !banned`
- extract non-trivial calculations into named methods so call sites read as intent, not arithmetic
- do not create empty delegate methods that only forward without added value

## Stream & Collections

- prefer `java.util.stream.Stream` API over `for` loops
- avoid `forEach`; prefer terminal operations that return values
- prefer `Stream.of` over `Arrays.stream` for known elements
- prefer `.toList()` over `.collect(Collectors.toList())`
- prefer `List.of` / `Set.of` / `Map.of` over `new ArrayList<>()` and array literals for small immutable collections
- avoid creating unnecessary intermediate collections when streaming arrays
- prefer `Stream.gather(...)` with a `Gatherer` (Java 24+) over custom `Spliterator` or stateful `forEach` for stream transformations that need to keep state across elements or flush a remainder at end-of-stream
- prefer a named intermediate variable over deeply nested method chaining when readability suffers
- return empty collections (`List.of()`, `Set.of()`, `Map.of()`), never `null`
- do not put `null` values in collections

## Code Style

- KISS and YAGNI — always implement the simplest possible solution that works
- never over-engineer; ask before adding optional features, extension points, or abstractions
- code must be as simple, elegant, and understandable as possible
- always choose the simplest API — prefer higher-level, concise APIs over verbose low-level ones
- prefer multiple simpler lines to one complex line
- use text blocks (`"""`) for all multiline string content (JSON, SQL, HTML, help/usage text, templates) — never `+`-concatenation or embedded `\n` escapes
- prefer `String.formatted()` (instance) over `String.format(...)` (static) for readability at the call site
- prefer imports over fully qualified class names; remove unused imports
- prefer `Files.readString` / `Files.writeString` / `Files.lines` over `BufferedReader`/`BufferedWriter` ceremony
- no blank lines between imports
- use `this` to reference instance fields when it improves clarity
- prefer enums over plain strings for finite, well-defined values; reuse existing enum constants as values where possible (enum constants do not have to follow naming conventions when reused as values)
- prefer try-with-resources over manual `.close()` on any `AutoCloseable`
- extract repeated string literals into named constants — define once, change once
- prefer character literals and named constants over raw numeric literals — write `'\n'` not `10`, define `int ESC = '\033'` instead of inlining `27`
- inline single-use variables — if assigned and used only once on the next line, pass the expression directly
- bind behavior to data with functional fields — store a `Runnable`, `Consumer`, or lambda in a record instead of switching on type externally
- separate side effects from conditions — do the work first, then branch on the result; keep the `if` a pure decision
- use guard clauses (early returns) over deeply nested `if`/`else`
- avoid `Optional` as a parameter type; use `Optional` as a return type only when absence is a meaningful part of the contract

## Exceptions

- prefer unchecked over checked exceptions
- never throw raw `java.lang.Exception` or `RuntimeException` directly — throw a specific subclass
- do not re-throw with `throw e` adding no value
- do not catch and silently ignore exceptions — at minimum, log with context or rethrow wrapped
- use the underscore `_` for unused catch parameters (`catch (IOException _)`) instead of named variables like `e` or `ignored`
- create custom exceptions only when they significantly improve robustness or maintainability

## Logging

- use `java.lang.System.Logger` instead of `System.out` statements
- never use `java.util.logging.Logger`
- `Logger` fields must be named `LOGGER` (uppercase) and marked as `static final`

## HTTP Client

- prefer the synchronous `java.net.http.HttpClient` APIs
- use asynchronous APIs (`HttpClient.sendAsync`) only when explicitly requested

## Testing

- use AssertJ assertions instead of JUnit assertions
- unit test methods must not start with `test` or `should`
- create minimalistic tests first; avoid repetitive or trivial unit tests and keep only essential tests verifying core functionality
- do not write tests for implementations that cannot fail (enums, records, getters/setters)
- generate at most three tests per class under test (applies separately to unit, integration, and system tests)
- the presence of an `isEqualTo` assertion makes less specific checks (`startsWith`, `isNotNull`) obsolete

## Comments & JavaDoc

- default to no comments
- only comment when the *why* is non-obvious: hidden constraint, subtle invariant, workaround for a specific bug, behavior that would surprise a reader
- do not explain *what* the code does — well-named identifiers do that
- do not write JavaDoc that restates the method signature or rephrases the code
- either describe the *why* or omit the comment entirely
- when JavaDoc is written, always use Markdown JavaDoc (`///`, JEP 467); never use HTML-tagged `/** */` blocks

## Composition with Other Skills

- this skill defines only language-level Java conventions; build, packaging, file layout, frameworks, protocols, and architecture come from the composed skill (e.g. `java-cli-script`, `java-cli-app`, `microprofile-server`, `bce`, `zb`)
- when a composed skill adds or refines a rule (e.g. "use `IO.println` not `System.out.println`", "no package declaration in single-file scripts", "JAX-RS resources are boundary classes"), apply it on top of these rules; the composed skill always specializes, never contradicts
- if a composed skill is silent on a topic covered here, these rules apply by default
