---
name: zunit
description: Generate and run zunit tests for java-cli-app projects. Use when asked to create tests, write tests, add tests, or generate test files for a java-cli-app project. Triggers on "zunit", "write tests", "create tests", "add tests", "test this", "generate tests", or requests to test a java-cli-app application. Also trigger when the user asks to verify or validate behavior of a java-cli-app project. Not for JUnit tests or microprofile-server projects — use continuous-testing for those.
---

Generate and run zunit tests for a java-cli-app project using $ARGUMENTS. Apply all rules below strictly.

## What is zunit

zunit is a zero-dependency test runner. It discovers `*Test.java` source files and runs each directly via `java --source 25`. No compilation step, no JUnit, no framework.

## Test Convention

- **Test files**: name ends with `Test.java`, placed in `test/` directory
- Each test file is a **self-contained Java source script** with `void main()`
- Tests run via `java --source 25 --class-path <classpath>`
- Failure: any thrown exception or non-zero exit = failed
- Success: clean exit with code 0
- All test files run **concurrently** in separate JVMs

## Test File Structure

Every test file follows this pattern:

```java
void main() {
    // arrange
    var input = ...;

    // act
    var result = SomeClass.someMethod(input);

    // assert — throw on failure
    if (!expected.equals(result))
        throw new AssertionError("expected %s but got %s".formatted(expected, result));
}
```

## Assertion Style

- No assertion libraries — use plain `if` + `throw`
- Throw `AssertionError` with a descriptive message including expected and actual values
- One test file can contain multiple assertions — the first failure stops the file
- For testing exceptions, use try/catch:

```java
void main() {
    try {
        SomeClass.methodThatShouldFail(badInput);
        throw new AssertionError("expected exception was not thrown");
    } catch (IllegalArgumentException expected) {
        // success
    }
}
```

## Test Discovery for java-cli-app Projects

### Directory Layout

```
project-root/
├── src/main/java/          # main source (compiled by zb)
├── test/                   # zunit test sources
│   ├── SomethingTest.java
│   └── AnotherTest.java
├── zbo/app.jar             # zb output (auto-detected as classpath)
└── .zb                     # zb config
```

### Classpath

zunit auto-detects `zbo/app.jar` as the classpath. Tests import main classes directly — zb packages everything into `app.jar`.

## How to Generate Tests

1. **Read the main source code** in `src/main/java/` to understand what to test
2. Create test files in `test/` directory — one test file per logical concern
3. Name test files descriptively: `ParserTest.java`, `ValidationTest.java`, `OutputTest.java`
4. Test public behavior, not internal implementation
5. Include both happy path and error/edge cases
6. Keep each test file focused — prefer multiple small test files over one large file (they run concurrently)

## Test File Rules

- No package declaration
- No imports from `java.base` — it is automatically available
- Use module imports for non-base modules (e.g., `import module java.net.http;`)
- Use `var` for local variables
- Use unnamed class style — `void main()` at top level, helper methods as needed
- Do not use `IO.println()` for assertions — throw exceptions
- Printing to stdout is fine for debugging but not required

## HTTP Client Timeouts

When tests make network calls using `java.net.http.HttpClient`, always set explicit timeouts to prevent tests from hanging:

- Set `.connectTimeout(Duration.ofSeconds(2))` on the `HttpClient`
- Set `.timeout(Duration.ofSeconds(2))` on each `HttpRequest`

## Running Tests

After generating test files:

1. **Build first**: `zb` (compiles main sources into `zbo/app.jar`)
2. **Run tests**: `zunit` (discovers `test/*Test.java`, runs against `zbo/app.jar`)
3. Alternatively: `zb && zunit`

Use `zunit -verbose` to debug classpath or discovery issues.

## Example

Given a main class in `src/main/java/Converter.java`:

```java
class Converter {
    static int toFahrenheit(int celsius) {
        return celsius * 9 / 5 + 32;
    }
}
```

Generate `test/ConverterTest.java`:

```java
void main() {
    // freezing point
    var freezing = Converter.toFahrenheit(0);
    if (freezing != 32)
        throw new AssertionError("expected 32 but got " + freezing);

    // boiling point
    var boiling = Converter.toFahrenheit(100);
    if (boiling != 212)
        throw new AssertionError("expected 212 but got " + boiling);

    // negative
    var negative = Converter.toFahrenheit(-40);
    if (negative != -40)
        throw new AssertionError("expected -40 but got " + negative);
}
```
