---
name: microprofile-server
description: Architecture and coding rules for long-running Java MicroProfile / Jakarta EE server applications — BCE layering, business components (BC), JAX-RS resources, CDI, JSON-P, testing (unit/integration/system), and Maven project structure. Use when creating, generating, scaffolding, writing, or reviewing code, resources, entities, boundaries, or business components in MicroProfile server projects. Not for serverless deployments.
---

## Composition
- compose with `java-conventions` for all language-level Java rules (syntax, style, naming, visibility, interfaces/classes, methods/lambdas, streams/collections, exceptions, comments/JavaDoc)
- this skill specializes only the MicroProfile / Jakarta EE server context — it does not restate language-level rules

## Dependencies
- prefer dependencies in this order: Java SE, MicroProfile, Jakarta EE

## Exceptions (JAX-RS)
- in JAX-RS projects, inherit from WebApplicationException for custom exceptions
- use explicit exceptions like BadRequestException for Response.Status.BAD_REQUEST
- throw explicit WebApplicationException subclasses (e.g. BadRequestException, NotFoundException) rather than constructing Response objects inline — they are automatically mapped to the correct HTTP status by the JAX-RS runtime

## Logging
- use java.lang.System.Logger instead of System.out statements
- never use java.util.logging.Logger
- Logger fields must be named LOGGER (uppercase) and marked as static final

## BCE/ECB Architecture
- structure code using the Boundary Control Entity (BCE/ECB) pattern
- package structure: [ORGANIZATION_NAME].[PROJECT_NAME].[COMPONENT_NAME].[boundary|control|entity]
- top level package reflects the application responsibility or name
- business components are children of top level package, named after their responsibilities
- a BC may represent a domain concept or a shared concern; both are valid when the responsibility has a name and is reused across the application
- boundary, control, entity packages are only allowed in business components
- a BC does not need every layer; a BC may consist of only a control layer when its responsibility is procedural and consumed by other BCs
- not every BCE component needs a dedicated boundary package; control package contents can be accessed directly
- prefer a dedicated BC over the root application package when a shared concern carries domain or protocol semantics, exposes more than one operation, or is expected to grow
- reserve the root application package for trivial single-class plumbing with no business semantics and no protocol coupling
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

## Entity Layer
- maintain domain objects, data classes, and entities in the entity package
- entities maintain state and corresponding behavior
- model value objects as enums
- direct references between entities from independent BCs are allowed, but always aim for Maximal Cohesion and Minimal Coupling between BCs
- if a relation exists in the database (e.g. JPA foreign key), the entities must carry a corresponding reference (id field or association); the DB schema is the source of truth
- excessive cross-BC references or shared configuration is a refactoring signal — split, merge, or rebalance the BCs to restore cohesion

## Components
- create new components with minimal business logic and essential fields only

## JavaDoc (MicroProfile/Jakarta EE)
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
- execute system tests after every major change to the service module
- in PoC mode (user-activated), system test execution is skipped

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
- on opening existing projects, load AGENTS.md (if present) before making changes
- do not create or change any files on opening existing projects; stop after initialization and wait for instructions
- do not generate code initially in an empty project
- Maven pom.xml must not be created for Java 25 CLI applications
- never use quarkus-hibernate-validator
- create metrics and observability features with OTEL / opentelemetry
