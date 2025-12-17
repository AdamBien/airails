## Java Version & Syntax
- use Java 25 with modern syntax (var, pattern matching, records, text blocks)
- prefer dependencies in this order: Java SE, MicroProfile, Jakarta EE
- use Java SE APIs over writing custom code
- prefer unchecked over checked exceptions; never throw generic exceptions like java.lang.Exception
- throw RuntimeException subclasses, not directly; inherit from WebApplicationException in JAX-RS projects
- consider using Java records instead of classes with final fields
- prefer factory methods in records over passing null in constructors

## Logging
- use java.lang.System.Logger instead of System.out statements
- never use java.util.logging.Logger
- Logger fields must be named LOGGER (uppercase) and marked as static final

## BCE/ECB Architecture
- structure code using the Boundary Control Entity (BCE/ECB) pattern
- package structure: [ORGANIZATION_NAME].[PROJECT_NAME].[COMPONENT_NAME].[boundary|control|entity]
- top level package reflects the application responsibility or name
- business components are children of top level package, named after their responsibilities
- boundary, control, entity packages are only allowed in business components
- classes with cross-cutting functionality are located in the root application package
- not every BCE component needs a dedicated boundary package; control package contents can be accessed directly
- do not explain the BCE pattern in documentation

## Package Naming
- create application level package with name derived from maven project or context
- name packages after their domain responsibilities
- create package-info.java for top level packages with JavaDoc documenting design decisions and responsibilities (not contents)
- document only domain-specific packages with package-info.java where the purpose is not self-evident

## Boundary Layer
- keep coarse-grained classes in the boundary package
- place facades in the boundary package
- health checks must be placed in the boundary package
- @Transactional is only allowed in boundary layer
- if there is no Boundary stereotype, use ApplicationScope instead

## Control Layer
- implement procedural business logic in the control package
- prefer interfaces with static methods over classes for stateless/procedural logic
- in utility interfaces prefer static over default methods

## Entity Layer
- maintain domain objects, data classes, and entities in the entity package
- entities maintain state and corresponding behavior
- model value objects as enums

## Class Naming Conventions
- name classes after their responsibilities
- avoid meaningless suffixes: *Impl, *Service, *Manager, *Creator
- class names must not end with "Control"
- only use "Resource" suffix for JAX-RS classes
- only use "Factory" suffix for actual GoF Factory pattern
- only use "Builder" suffix for classes with typical builder structure (method chaining)
- stacks should end with "Stack" suffix

## Visibility & Modifiers
- avoid private visibility; prefer package-private (default) visibility
- avoid "private static" methods; prefer default visibility
- do not use final for fields (exception: static final for LOGGER)
- do not use constructor injection

## Interfaces & Classes
- only use interfaces with multiple implementations or for strategy pattern
- do not create interfaces with abstract methods implemented by a single class; use classes directly
- create multiple classes only if it decreases complexity and increases readability

## Method Naming & Design
- avoid "getter" methods starting with "get"; prefer record convention (e.g., configuration() not getConfiguration())
- keep methods short and testable
- create well-named methods for coarse-grained, cohesive, self-contained logic
- if a lambda requires multiple statements or braces {}, extract it into a well-named helper method
- do not create multiline lambda expressions; use method references instead
- extract repeated calculations or string concatenations into helper methods (DRY principle)
- do not create empty delegates which just call methods without added value

## Stream & Collections
- prefer java.util.stream.Stream API over for loops
- avoid forEach; prefer Stream methods
- prefer Stream.of to Arrays.stream
- prefer toList() to .collect(Collectors.toList())
- prefer List.of over String[] or new ArrayList<>()
- avoid creating unnecessary intermediate collections when streaming arrays
- prefer variable declaration over lengthy method chaining

## Code Style
- prefer multiple simpler lines to one more complex line
- prefer multiline Strings (text blocks) over String concatenations
- prefer imports over fully qualified class names
- use "this" to reference instance fields
- remove unused imports
- extract variables to eliminate duplication
- prefer enums over plain Strings for finite, well-defined values
- reuse enum constants as values if possible; enum constants do not have to follow naming conventions
- prefer try-with-resources over explicitly closing resources

## Simplicity Principles
- keep the design KISS and YAGNI
- always implement the simplest possible solution
- write simple code first; ask before implementing enhancements or optional features
- never over-engineer; ask about adding optional features or extension points
- create new components with minimal business logic and essential fields only

## Exceptions
- create custom exceptions only if it significantly improves robustness or maintainability
- use explicit exceptions like BadRequestException for Response.Status.BAD_REQUEST
- always use WebApplicationException in compact constructors in JAX-RS projects
- do not re-throw exceptions with "throw e" without adding value

## JavaDoc
- do not write obvious JavaDoc comments that rephrase code
- document the intentions and the "why", not implementation details
- either describe the "why" or do not comment at all
- follow links in JavaDoc to external specifications and use them for code generation
- use popular, also funny, technical terms from the Java SE, MicroProfile and Jakarta EE ecosystems as examples in unit tests and javadoc

## README Guidelines
- write brief, 'to the point' README.md files for advanced developers
- use precise and concise language; avoid generic adjectives like "simple", "lightweight"
- do not include detailed project structure (file/folder listings); high-level module descriptions are acceptable
- never list REST resources in READMEs
- if modules are listed, provide links
- do not use "Orchestrates" term; use more specific alternatives

## Testing
- unit test methods must not start with "test" or "should"
- avoid writing repetitive or trivial unit tests; keep only essential tests verifying core functionality
- do not write tests for implementations that cannot fail (enums, records, getters/setters)
- create minimalistic tests first
- generate at most three tests per class under test (applies separately to UTs, ITs, and STs)
- use AssertJ library instead of JUnit assertions
- the presence of isEqualTo assertion makes less specific checks (startsWith, isNotNull) obsolete
- do not use private visibility in tests

## Integration Tests
- integration tests end with IT suffix and are executed by the failsafe maven plugin (no configuration necessary)

## System Tests (ST)
- system tests are created in a dedicated Maven module ending with "-st"
- use microprofile-rest-client for testing JAX-RS resources
- REST client interfaces: src/main/java of the -st module
- test classes: src/test/java of the -st module
- name client interfaces after the resource with "Client" suffix (e.g., GreetingsResource -> GreetingsResourceClient)
- RegisterRestClient configKey: "service_uri"
- STs end with "IT" suffix
- do not use RestAssured. Write e2e test in the -st module

## JAX-RS
- resources should be named in plural (e.g., SpeakersResource not SpeakerResource)
- @Consumes and @Produces should be declared on class-level
- do not implement business logic in JAX-RS resources; delegate instead
- prefer returning JAX-RS Response over JsonObject in resources
- do not create new "@RegisterRestClient(configKey," - reuse existing

## JSON Serialization (JSON-P)
- prefer JSON-P over JSON-B
- record entities should ship with toJSON method returning a JSON-P object
- always map JSON-P in the boundary to entities
- create record entities from JSON-P JsonObject in static method: fromJSON(JsonObject json)

## HTTP Client
- prefer synchronous HTTPClient APIs
- use asynchronous Http APIs (HttpClient.sendAsync) only if explicitly requested

## Project Management
- always ask before changing pom.xml
- do not create or change any files on opening existing projects; stop after initialization and wait for instructions
- do not generate code initially in an empty project
- Maven pom.xml must not be created for Java 25 CLI applications
- Java 25 CLI applications will be executed in source-file mode
- never use quarkus-hibernate-validator
- create metrics and observability features with OTEL / opentelemetry

## Unnamed Classes
- in unnamed classes do not import packages available in java.base module
- do not use static methods in unnamed classes
