# Quarkus

Runtime specifics of `jakarta-data-persistence` on Quarkus. The rules in [SKILL.md](../SKILL.md) apply unchanged; this file names the artifacts, the annotation processor, the configuration keys, and the test support that realize them, and the behavior of the provider the rules refer to. Provider, JDBC driver, Flyway, and the configuration keys are those of the `jakarta-persistence` Quarkus reference.

## Dependencies

All are managed by the Quarkus BOM unless noted. Ask before changing `pom.xml`, with all of them in one question.

- `quarkus-hibernate-orm` — the provider behind the repositories; it brings the transaction manager, so `@Transactional` needs no further dependency
- `jakarta.data:jakarta.data-api` — not brought by `quarkus-hibernate-orm`
- the JDBC extension of the target DBMS — `quarkus-jdbc-postgresql` and friends
- `quarkus-flyway` and the Flyway module of that DBMS (`quarkus-flyway-postgresql`) — without the module Flyway finds no plugin for the JDBC URL
- AssertJ (`org.assertj:assertj-core`, test scope) when the module lacks it — not managed by the BOM; reuse the version another module of the project pins

## Annotation Processor

The processor generates the repository implementations (`Coffees_`) and the static metamodel. It is declared on the `maven-compiler-plugin`, with its version taken from the BOM:

```xml
<configuration>
    <annotationProcessorPathsUseDepMgmt>true</annotationProcessorPathsUseDepMgmt>
    <annotationProcessorPaths>
        <path>
            <groupId>org.hibernate.orm</groupId>
            <artifactId>hibernate-processor</artifactId>
        </path>
    </annotationProcessorPaths>
</configuration>
```

- `org.hibernate.orm:hibernate-processor` is managed by the Quarkus BOM (checked with 3.35.1); where the BOM of the project's Quarkus version manages `io.quarkus:quarkus-data-processor`, the Quarkus guide names that artifact instead — read the effective POM and use the one that is managed, never a hand-picked version
- the two elements are added to the existing `<configuration>` of the plugin; nothing else in it changes
- the generated sources land in `target/generated-sources/annotations`; a repository without a generated `<Repository>_` there has no implementation
- an abstract repository method without an annotation is only a warning — `repository method cannot be implemented (skipping whole repository)`; the build succeeds and no `<Repository>_` is generated

## Repository

- `@Inject Coffees coffees;` — the generated implementation is a `@Dependent` bean
- `@Insert` reports every constraint violation as `EntityExistsException`, a foreign key to a missing row included; `@Update` and `@Delete` report the same violations as a plain `DataException`
- an unfetched collection fails with `org.hibernate.LazyInitializationException` on access; an unfetched to-one is loaded by a further statement per entity

## Configuration

`application.properties`; no `persistence.xml`:

```
quarkus.datasource.db-kind=postgresql
quarkus.hibernate-orm.schema-management.strategy=validate
quarkus.flyway.migrate-at-start=true
```

- the strategy is set explicitly: unset, Quarkus switches it to `drop-and-create` when Dev Services run and no extension manages the schema
- never `drop-and-create`, `create`, or `update`, in any profile
- no `quarkus.datasource.jdbc.url` line in committed configuration — it switches off Dev Services; production coordinates arrive as `QUARKUS_DATASOURCE_*` environment variables

## Health

- `quarkus-smallrye-health` registers a readiness check for the datasource; no custom check is written

## Testing

- `@QuarkusTest`; Dev Services start the configured DBMS in a container and Flyway migrates it
- `@TestTransaction` on the test class runs every test method in a transaction that is rolled back
- Quarkus enhances entity classes with additional fields, which is why the round trip compares `toJSON()` and not the fields
- a build that sets `skipITs` skips the `IT`-suffixed tests in `mvn verify`; run `mvn verify -DskipITs=false`
- Dev Services need a container runtime; without one the tests do not start — report the mapping and the queries as unverified, do not switch the DBMS or the schema strategy
