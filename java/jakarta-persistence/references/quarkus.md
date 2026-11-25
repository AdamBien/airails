# Quarkus

Runtime specifics of `jakarta-persistence` on Quarkus. The rules in [SKILL.md](../SKILL.md) apply unchanged; this file names the artifacts, the configuration keys, and the test support that realize them.

## Dependencies

All are managed by the Quarkus BOM unless noted. Ask before changing `pom.xml`, with all of them in one question.

- `quarkus-hibernate-orm` — the provider; it brings the transaction manager, so `@Transactional` needs no further dependency
- the JDBC extension of the target DBMS — `quarkus-jdbc-postgresql` and friends
- `quarkus-flyway` and the Flyway module of that DBMS (`quarkus-flyway-postgresql`) — without the module Flyway finds no plugin for the JDBC URL
- AssertJ (`org.assertj:assertj-core`, test scope) when the module lacks it — not managed by the BOM; reuse the version another module of the project pins

## EntityManager

- `@Inject EntityManager em;`

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
- Dev Services need a container runtime; without one the tests do not start — report the mapping as unverified, do not switch the DBMS or the schema strategy
