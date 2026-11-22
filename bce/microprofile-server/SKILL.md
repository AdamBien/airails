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

## Platform
- Java 25, as `java-conventions` defines it
- Jakarta EE 10 or later: EE APIs live in the `jakarta.*` namespace, never `javax.*` — older examples and tutorials still use the `javax.*` names and do not compile against the current APIs; `javax.*` packages of the JDK itself (`javax.sql`, `javax.crypto`) are unaffected
- MicroProfile 6 or later: MicroProfile OpenTracing is gone, MicroProfile Telemetry (OTEL) replaces it
- the exact specification and runtime versions come from the project's BOM in `pom.xml` — read it before using a version-specific API; never assume a version and never change one (see Project Management)

## Dependencies
- prefer dependencies in this order: Java SE, MicroProfile, Jakarta EE — an API the platform already ships adds nothing to the build, the image, or the upgrade path; a third-party library is the last resort

## Concurrency
- write blocking, synchronous code and run it on virtual threads (e.g. `@RunOnVirtualThread` on Quarkus); never introduce reactive APIs (Mutiny `Uni`/`Multi`, reactive streams, `CompletionStage` chains) unless the user explicitly asks — a blocked virtual thread is cheap, so reactive code buys no scalability and costs straight-line control flow, readable stack traces, and plain debugging
- blocking calls (JDBC, REST client, file I/O) in boundaries and controls are the expected shape, not a smell

## CDI
- do not use constructor injection — prefer field injection into package-private fields: one line per dependency, and a same-package unit test assigns the field directly (`java-conventions`); constructor injection adds a constructor, the assignments, and a second no-arg constructor for the client proxy of a normal-scoped bean

## Exceptions (JAX-RS)
- in JAX-RS projects, inherit from WebApplicationException for custom exceptions — the exception carries its own status, so no `ExceptionMapper` is needed
- use explicit exceptions like BadRequestException for Response.Status.BAD_REQUEST
- throw explicit WebApplicationException subclasses (e.g. BadRequestException, NotFoundException) rather than constructing Response objects inline — they are automatically mapped to the correct HTTP status by the JAX-RS runtime

## BCE Specializations
- package structure: `[ORGANIZATION_NAME].[PROJECT_NAME].[COMPONENT_NAME].[boundary|control|entity]`, as `bce` defines it; suggest renaming the top-level `[PROJECT_NAME]` package when it does not reflect the project's name or responsibility
- JAX-RS resources and health checks are boundary classes
- the BC package doc that `bce` requires is `package-info.java` in the BC's package, written as Markdown JavaDoc (`///`); it states the responsibility and design decisions
- outbound integrations are controls, as `bce` defines it: a MicroProfile REST Client interface (`@RegisterRestClient`) or a messaging producer lives in the control package of the BC that uses it and is injected into a control, never into a JAX-RS resource; the control converts the remote representation into the BC's entities
- @Transactional is only allowed in the boundary, declared on the class (or via the project's @Boundary stereotype), never on individual methods — a boundary call is the unit of work: one call from outside, one transaction; the class-level declaration leaves no method outside it
- if there is no Boundary stereotype, use @ApplicationScoped instead
- boundaries and controls are CDI beans and therefore classes, even when stateless — this is the deliberate exception to the `java-conventions` rule that stateless types are interfaces with `static` methods; it does not apply to CLI projects without a container
- if a relation exists in the database (e.g. JPA foreign key), the entities must carry a corresponding reference (id field or association); the DB schema is the source of truth
- do not copy entities into structurally identical objects (DTOs, VOs, mirror records) across layers or BCs; pass the entity itself — a mirror type duplicates every field change and adds a mapping without adding information
- introduce a separate type only when its shape or semantics differ (e.g. projection, aggregation, versioned external contract)
- no mapper classes or mapping frameworks (e.g. MapStruct, ModelMapper) for 1:1 field copies

## JavaDoc (MicroProfile/Jakarta EE)
- follow links in JavaDoc to external specifications and use them for code generation
- use popular, also funny, technical terms from the Java SE, MicroProfile and Jakarta EE ecosystems as examples in unit tests and javadoc

## Integration Tests
- integration tests end with IT suffix and are executed by the failsafe maven plugin (no configuration necessary)

## System Tests (ST)
- the contract comes from `system-tests`; the rules below are its Maven / microprofile-rest-client realization
- system tests are created in a dedicated Maven module ending with "-st" — a separate module cannot see the service's classes, which enforces the black box
- use microprofile-rest-client for testing JAX-RS resources — the typed client interface exercises the API the way a consumer would
- REST client interfaces: src/main/java of the -st module
- test classes: src/test/java of the -st module
- name client interfaces after the resource with "Client" suffix (e.g., GreetingsResource -> GreetingsResourceClient)
- RegisterRestClient configKey: "service_uri" — the configured coordinate of the running system
- the coordinate is one property line in the -st module's `application.properties`: `service_uri/mp-rest/url=http://localhost:8080`; a different environment overrides it through MicroProfile Config (e.g. the env var `SERVICE_URI_MP_REST_URL`), never by editing the file
- STs end with "IT" suffix — failsafe executes them, as it does integration tests
- do not use RestAssured — it is a second HTTP client and DSL next to the rest client the platform already ships (see Dependencies). Write e2e test in the -st module
- execute system tests after every major change to the service module
- in PoC mode (user-activated), system test execution is skipped

## JAX-RS
- resources should be named in plural (e.g., SpeakersResource not SpeakerResource) — a resource addresses a collection (`/speakers`, `/speakers/{id}`)
- @Consumes and @Produces should be declared on class-level — one declaration covers every method; a method overrides it only when its media type differs
- do not implement business logic in JAX-RS resources; delegate instead — a resource is a boundary: it adapts HTTP, the control decides
- prefer returning JAX-RS Response over JsonObject in resources — status code and headers stay explicit
- do not create new "@RegisterRestClient(configKey," - reuse existing — one configKey is one configured coordinate; a second key for the same system is a second place to misconfigure

## JSON Serialization (JSON-P)
- prefer JSON-P over JSON-B — the explicit mapping keeps the wire format visible, testable, and independent of field names; no reflection and no binding annotations on entities
- record entities should ship with toJSON method returning a JSON-P object
- always map JSON-P in the boundary to entities — control and entity never see the wire format
- create record entities from JSON-P JsonObject in static method: fromJSON(JsonObject json)

## Project Management
- always ask before changing pom.xml — a dependency is an architecture decision (see Dependencies)
- on opening existing projects, load AGENTS.md (if present) before making changes
- do not create or change any files on opening existing projects
- do not generate code initially in an empty project
- prefer explicit validation in code over quarkus-hibernate-validator (Bean Validation annotations) — as with JSON-P over JSON-B, the explicit check gives control over the response and its message; a constraint violation produces the framework's generic report
- create metrics and observability features with OTEL / opentelemetry — one vendor-neutral API for traces, metrics, and logs, and the basis of MicroProfile Telemetry
