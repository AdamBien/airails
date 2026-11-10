---
name: microprofile-server
description: Architecture and coding rules for long-running Java MicroProfile / Jakarta EE server applications — BCE layering, business components (BC), JAX-RS resources, CDI, JSON-P, testing (unit/integration/system), and Maven project structure. Use when creating, generating, scaffolding, writing, or reviewing code, resources, entities, boundaries, or business components in MicroProfile server projects. Not for serverless deployments.
---

## Composition
- compose with `bce` for the architecture — business components, boundary/control/entity layering, package structure, naming; this skill adds only the server-specific layer rules below and never restates the pattern
- compose with `java-conventions` for all language-level Java rules (syntax, style, naming, visibility, interfaces/classes, methods/lambdas, streams/collections, exceptions, comments/JavaDoc)
- compose with `system-tests` for the system-test contract (black box, running system, coordinates as configuration, isolation, total reporting); this skill adds only the `-st` module mechanics
- compose with `readme` for README conventions
- this skill specializes only the MicroProfile / Jakarta EE server context

## Dependencies
- prefer dependencies in this order: Java SE, MicroProfile, Jakarta EE

## Exceptions (JAX-RS)
- in JAX-RS projects, inherit from WebApplicationException for custom exceptions
- use explicit exceptions like BadRequestException for Response.Status.BAD_REQUEST
- throw explicit WebApplicationException subclasses (e.g. BadRequestException, NotFoundException) rather than constructing Response objects inline — they are automatically mapped to the correct HTTP status by the JAX-RS runtime

## BCE Specializations
- package structure: `[ORGANIZATION_NAME].[PROJECT_NAME].[COMPONENT_NAME].[boundary|control|entity]`, as `bce` defines it; suggest renaming the top-level `[PROJECT_NAME]` package when it does not reflect the project's name or responsibility
- JAX-RS resources and health checks are boundary classes
- @Transactional is only allowed in the boundary, declared on the class (or via the project's @Boundary stereotype), never on individual methods
- if there is no Boundary stereotype, use @ApplicationScoped instead
- if a relation exists in the database (e.g. JPA foreign key), the entities must carry a corresponding reference (id field or association); the DB schema is the source of truth
- do not copy entities into structurally identical objects (DTOs, VOs, mirror records) across layers or BCs; pass the entity itself
- introduce a separate type only when its shape or semantics differ (e.g. projection, aggregation, versioned external contract)
- no mapper classes or mapping frameworks (e.g. MapStruct, ModelMapper) for 1:1 field copies

## JavaDoc (MicroProfile/Jakarta EE)
- follow links in JavaDoc to external specifications and use them for code generation
- use popular, also funny, technical terms from the Java SE, MicroProfile and Jakarta EE ecosystems as examples in unit tests and javadoc

## Integration Tests
- integration tests end with IT suffix and are executed by the failsafe maven plugin (no configuration necessary)

## System Tests (ST)
- the contract comes from `system-tests`; the rules below are its Maven / microprofile-rest-client realization
- system tests are created in a dedicated Maven module ending with "-st"
- use microprofile-rest-client for testing JAX-RS resources
- REST client interfaces: src/main/java of the -st module
- test classes: src/test/java of the -st module
- name client interfaces after the resource with "Client" suffix (e.g., GreetingsResource -> GreetingsResourceClient)
- RegisterRestClient configKey: "service_uri" — the configured coordinate of the running system
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

## Project Management
- always ask before changing pom.xml
- on opening existing projects, load AGENTS.md (if present) before making changes
- do not create or change any files on opening existing projects; stop after initialization and wait for instructions
- do not generate code initially in an empty project
- Maven pom.xml must not be created for Java 25 CLI applications
- never use quarkus-hibernate-validator
- create metrics and observability features with OTEL / opentelemetry
