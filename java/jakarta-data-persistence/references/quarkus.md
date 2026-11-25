# Quarkus

Runtime specifics of `jakarta-data-persistence` on Quarkus. The rules in [SKILL.md](../SKILL.md) apply unchanged. Provider, JDBC driver, Flyway, configuration keys, health, and test support are those of the [Quarkus reference of `jakarta-persistence`](../../jakarta-persistence/references/quarkus.md) — read it as well, except its `EntityManager` section; this file names only what Jakarta Data adds: one dependency, the annotation processor, and the behavior of the provider the rules refer to.

## Dependencies

Ask before changing `pom.xml`, with this and the dependencies of the `jakarta-persistence` reference in one question.

- `jakarta.data:jakarta.data-api` — managed by the Quarkus BOM, not brought by `quarkus-hibernate-orm`

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

- `org.hibernate.orm:hibernate-processor` is managed by the Quarkus BOM (checked with 3.35.1); from 3.39 the BOM also manages `io.quarkus:quarkus-data-processor`, a wrapper around it and the artifact the Quarkus guide names — read the effective POM, use `quarkus-data-processor` where it is managed and `hibernate-processor` otherwise, never a hand-picked version
- the two elements are added to the existing `<configuration>` of the plugin; nothing else in it changes
- the generated sources land in `target/generated-sources/annotations`; a repository without a generated `<Repository>_` there has no implementation
- an abstract repository method without an annotation is only a warning — `repository method cannot be implemented (skipping whole repository)`; the build succeeds and no `<Repository>_` is generated

## Repository

- `@Inject Coffees coffees;` — the generated implementation is a `@Dependent` bean
- `@Insert` reports every constraint violation as `EntityExistsException`, a foreign key to a missing row included; `@Update` and `@Delete` report the same violations as a plain `DataException`
- an unfetched collection fails with `org.hibernate.LazyInitializationException` on access; an unfetched to-one is loaded by a further statement per entity

## Testing

- the test support of the `jakarta-persistence` reference applies unchanged; where it reports the mapping as unverified, the queries are unverified as well
