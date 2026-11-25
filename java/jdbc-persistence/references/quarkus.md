# Quarkus

Runtime specifics of `jdbc-persistence` on Quarkus, for the `/microprofile-server` stack. The rules in [SKILL.md](../SKILL.md) apply unchanged; this file names the artifacts, the configuration keys, and the test support that realize them.

## Dependencies

All are managed by the Quarkus BOM unless noted. Ask before changing `pom.xml`, with all of them in one question.

- the JDBC extension of the target DBMS — `quarkus-jdbc-postgresql` and friends; it brings the driver and the Agroal connection pool
- `quarkus-flyway` and the Flyway module of that DBMS (`quarkus-flyway-postgresql`) — without the module Flyway finds no plugin for the JDBC URL
- AssertJ (`org.assertj:assertj-core`, test scope) when the module lacks it — not managed by the BOM; reuse the version another module of the project pins

## Configuration

`application.properties`:

```
quarkus.datasource.db-kind=postgresql
quarkus.flyway.migrate-at-start=true
```

- no `quarkus.datasource.jdbc.url` line in committed configuration — it switches off Dev Services; production coordinates arrive as `QUARKUS_DATASOURCE_*` environment variables

## Health

- `quarkus-smallrye-health` registers a readiness check for the datasource when the Agroal extension is present; no custom check is written

## Testing

- `@QuarkusTest`; Dev Services start the configured DBMS in a container and Flyway migrates it
- a build that sets `skipITs` skips `IT`-suffixed tests in `mvn verify`; run `mvn verify -DskipITs=false`
- Dev Services need a container runtime; without one the tests do not start — report the persistence as unverified, do not switch the DBMS
