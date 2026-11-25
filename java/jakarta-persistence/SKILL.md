---
name: jakarta-persistence
description: 'Persist and query entities with Jakarta Persistence 3.2 (JPA) in MicroProfile / Jakarta EE servers: `@Entity` classes with field access and behavior, record embeddables and projections, JPQL named queries, the `EntityManager` in the `@Transactional` boundary with controls only where they add logic, a Flyway-owned schema the mapping is validated against. Use on demand — when the request names "JPA", "Jakarta Persistence", "EntityManager", "@Entity", "JPQL", "persistence unit", "ORM", or "Hibernate", or when the project already depends on a Jakarta Persistence provider. Not the default for an unqualified "add persistence" — use `/memory-persistence`. Not for plain SQL — use `/jdbc-persistence`. Not for CLI applications or NoSQL.'
argument-hint: "[BC or entity to persist]"
---

Add Jakarta Persistence using $ARGUMENTS. Apply all rules below strictly; `/bce`, `/java-conventions`, and `/microprofile-server` hold wherever this skill is silent.

This skill is picked on demand: when the request names no persistence technology and the project's build carries no Jakarta Persistence provider, stop and use `/memory-persistence` instead.

## Platform

- Jakarta Persistence 3.2, the release for Jakarta EE 11 — a floor above the `/microprofile-server` platform rule; before using an API marked (3.2) below, read the managed `jakarta.persistence-api` version from the effective POM (`mvn help:effective-pom`, which resolves an imported BOM), and when it is older, stay within that version and never change it without asking
- code imports `jakarta.persistence.*` only, never a provider package (`org.hibernate.*`, `org.eclipse.persistence.*`) — the provider is a runtime decision; provider settings live in configuration, never in annotations or code
- server only: a `/java-cli-app` persists with `/jdbc-persistence` — a provider and its transitive dependencies contradict the zero-dependency build

## Principle

The persistence context is the state of the unit of work: an entity loaded inside the boundary's transaction is managed, a change is a call to one of its methods, and the provider writes the difference at commit — unlike `/jdbc-persistence`, where a change is a new record followed by an explicit `UPDATE`.

- entities are classes with state and behavior; records carry what has no identity — embeddable value objects, primary key classes, projections
- an entity lives for one boundary call; no detached entity is kept or cached between calls — a full replacement arrives as a new instance and is merged
- what the provider generates stays predictable — every association is lazy, and a query states what it fetches
- the schema is not derived from the mapping; the mapping is validated against the schema

## Placement (BCE)

- `entity` — the `@Entity` classes, their `@Embeddable` records, and the projection records; the only layer that carries mapping annotations
- `boundary` — owns the unit of work: `@Transactional` on the class or the project's `@Boundary` stereotype (`/microprofile-server`), one transaction per call; for plain create, read, replace, and remove it injects the `EntityManager` and calls it directly — the `EntityManager` is already the persistence facade, and a control that only forwards to it is the empty delegate `/java-conventions` rules out; storing and finding an entity is not the business logic `/microprofile-server` keeps out of a resource; JPQL text stays on the entity as named queries, never in the boundary
- `control` — exists only where it adds something to the `EntityManager` call: a write the client can make conflict (see Exceptions), an operation another BC calls (another BC never uses a boundary — `/bce`), logic across several entities, a query assembled from runtime clauses; named after its responsibility (`Customers`, `OrderReports`), never `*Repository`, `*Dao`, `*Store`; no Panache, no Jakarta Data repository interface, no generic `Crud<T>` base class — each puts a technical type where the `EntityManager` or a responsibility-named control belongs
- a reference to another BC's entity is a unidirectional `@ManyToOne(fetch = FetchType.LAZY)` or a plain id field, either one backed by the `FOREIGN KEY` the referencing BC's migration declares (see Schema); no `cascade` and no `orphanRemoval` across BCs — the other BC writes its own table; an association mapped from both BCs (`mappedBy` in the other BC's entity) is the coupling signal `/bce` names

## Entities

- shape: a non-final top-level class with a `protected` no-arg constructor for the provider and a constructor or static factory that takes the mandatory state and checks its invariants; an enum, record, or interface cannot be an entity
- field access: `@Id` sits on the field, so the provider reads and writes fields and no accessor is required; fields are package-private and never `final` (`/java-conventions`)
- no `get`/`set` pairs: record-style accessors where a caller needs the value, and methods named after the state change (`ship(Instant on)`, `rename(String name)`) that guard the invariant they touch — a setter per field is the anemic shape `/bce` rules out
- `toJSON()` and `static fromJSON(JsonObject)` as `/microprofile-server` defines them for records; `fromJSON` creates a new entity, or the complete replacement of an existing one with the id taken from the request path; a partial change reads its values in the boundary and passes them to the managed entity's method — a partially filled instance is never merged, because `merge` overwrites every column, including those the client did not send
- id: `@Id @GeneratedValue(strategy = GenerationType.IDENTITY) long id;`, the counterpart of `BIGINT GENERATED ALWAYS AS IDENTITY` in the DDL
- names: tables and columns are `snake_case`, one singular table per entity (`order_line`), primary key `id`; the defaults are the entity name and the field name unchanged, so every multi-word entity carries `@Table(name = "order_line")` and every multi-word field `@Column(name = "feature_number")`; a project that already configures a naming strategy keeps it and omits them — never both
- a `BigDecimal` field is set to the scale of its column in the constructor and in every method that assigns it (`setScale`) — the database returns `4.20` for a stored `4.2`, and the two are not `equals`
- types: `Instant`, `OffsetDateTime`, `LocalDate`, `BigDecimal`, `UUID`; never `java.util.Date`, `Calendar`, the `java.sql` date types, or `@Temporal` — deprecated for new applications (3.2)
- enums: `@Enumerated(EnumType.STRING)`, never ordinal — the DDL `CHECK` lists the `name()` values; a stored code that differs from the constant name is a `final` field of the enum annotated `@EnumeratedValue` (3.2), not an `AttributeConverter`
- value objects: a record annotated `@Embeddable` (3.2), its compact constructor holding the invariants; a composite primary key class is a record as well
- associations: every `@ManyToOne` and `@OneToOne` declares `fetch = FetchType.LAZY` — their default is `EAGER`, which loads the target with every read; to-many associations keep their lazy default; `EAGER` is never written
- `@Version` only when the spec asks for lost-update protection

## Boundary and Control

- both obtain the `EntityManager` the same way: `@Inject EntityManager em;`; a runtime that does not offer it to `@Inject` uses `@PersistenceContext EntityManager em;`
- plain operations in the boundary: `this.em.persist(order)` — the instance is managed and carries its id afterwards; `this.em.find(Order.class, id)`, answered with `NotFoundException` when `null`; `this.em.createNamedQuery(Order.ALL, Order.class).getResultList()`; `this.em.remove(order)` on the found entity
- full replacement (a PUT carrying the complete representation): `this.em.merge(order)` — the returned instance is the managed one, the argument stays detached and is not used afterwards; an unknown id is answered with `NotFoundException` before the merge, since what `merge` does with one is not relied on
- a partial change: the boundary finds the entity and calls its method; the provider writes the difference at commit
- a control method, where one exists, has the shape of the sibling skills: `save(Order order)` returns the managed instance with its id, `update(Order order)` returns the merged one, `findById(long id)` returns `Optional`, queries return `List`, empty, never `null`; a state change the client can make conflict (see Exceptions) receives the managed entity the boundary found, calls the entity's method, and flushes
- an operation moves from the boundary into a control when it gains one of the reasons in Placement; it never exists in both
- never `merge` a new or a partially filled entity; no transaction handling besides the boundary's `@Transactional`

## Queries

- JPQL in a text block; a static query is a `@NamedQuery` on the entity it returns, its name a `static final String` constant on that entity prefixed with the entity name (`Order.OPEN = "Order.open"`) — named queries are parsed when the persistence unit starts, so a broken query fails the startup, not the first request
- typed execution: `this.em.createNamedQuery(Order.OPEN, Order.class)`; where the project already generates the static metamodel, `this.em.createQuery(reference)` with its `TypedQueryReference` (3.2); adding the metamodel processor is a `pom.xml` change — ask
- a dynamic clause selects between named queries through an enum or a `switch`; the Criteria API only when clauses combine at runtime beyond what a handful of named queries cover
- a lookup that may find nothing: `Optional.ofNullable(query.getSingleResultOrNull())` (3.2); `NoResultException` is never caught to steer logic
- projections (a report row, a count with a name, a dropdown entry) are records in `entity`, filled by a constructor expression — `select new airhacks.orders.entity.OrderSummary(o.id, o.total) from Order o`; no `Object[]` or `Tuple` leaves the method that ran the query, and no entity is loaded to read two of its fields
- a query whose result is navigated across an association fetches it with `join fetch` — the counterpart of the lazy default, never one query per parent
- JPQL covers `union`, `intersect`, `except`, `cast`, `left`, `right`, `replace`, and `||` (3.2); a query that still needs native SQL is a signal to ask whether that BC belongs to `/jdbc-persistence`

## Exceptions

- a constraint violation surfaces when the provider executes the statement: inside `persist` for an identity column, otherwise at flush, by default at commit — after the boundary method has returned and outside any `catch`; left there, a duplicate ends as a rollback and a 500
- a write the client can make conflict (a unique column, a foreign key to a row that may be gone) therefore runs `persist`, `merge`, or the entity's method **and** `this.em.flush()` inside one `try` in a control method — the reason that write lives in a control — and translates the `PersistenceException` at that point, directly into the exception the stack understands; the original exception travels as the cause
- the translation is one package-private `static` function per BC in the control package — on the control when the BC has one, otherwise in an interface named after the responsibility (`Conflicts`): it walks the cause chain to the `SQLException` and switches over `getSQLState()` — `23505` (unique) and `23503` (foreign key) become `ClientErrorException(message, Response.Status.CONFLICT, cause)` with the message naming the kind and the constraint (`duplicate: uq_customer_email`, `unknown reference: fk_order_customer`); everything else becomes `InternalServerErrorException(cause)`; the same mapping as `/jdbc-persistence`, so no cast to a provider exception type
- the constraint name is taken from the driver's message with a one-line regex in that function, never with a parsing library
- `OptimisticLockException` becomes `ClientErrorException(Response.Status.CONFLICT)`
- both are `WebApplicationException`s (`/microprofile-server`): the JAX-RS runtime answers 409 or 500, `@Transactional` rolls back, the boundary catches nothing, and no `ExceptionMapper` is written
- a missing entity is the boundary's decision — `NotFoundException` when `find` returns `null` — also before it hands the entity to a state-changing control method

## Schema

- the Schema rules of `/jdbc-persistence` apply unchanged: hand-written DDL for the target DBMS (PostgreSQL unless the request or the project names another), one Flyway migration per BC per change (`db/migration/V<yyyyMMddHHmm>__<bc>_<change>.sql`), every constraint and index named — `pk_<table>`, `uq_<table>_<column>`, `fk_<table>_<referenced_table>`, `ck_<table>_<rule>`, `ix_<table>_<column>` — because the conflict message (see Exceptions) carries exactly that name; every foreign key column is indexed
- the agent derives the **first** migration from the entities with the Type Mapping of `/jdbc-persistence`, one column per persistent field; from then on a schema change starts in SQL and the entity follows
- the provider validates and never generates: `quarkus.hibernate-orm.schema-management.strategy=validate`; never `drop-and-create`, `create`, or `update`, in any profile — the tests run against the migrations that production runs; `validate` compares tables, columns, and types only, so constraints are covered by the round-trip and conflict tests
- no schema-generation members on mapping annotations (`length`, `unique`, `columnDefinition`, `@Index`, `@CheckConstraint`, `@ForeignKey`) — nothing generates DDL from them, so they are an unchecked second copy of the migration; `optional = false` on a to-one association stays, because it changes the generated join
- constraints mirror the invariants the entity's constructor and methods check, as in `/jdbc-persistence`, and the entity checks what a column bounds (`VARCHAR(n)` length, `NUMERIC(p, s)` precision and scale)

## Configuration and Drivers

- Quarkus: `quarkus-hibernate-orm` (it brings the transaction manager), the JDBC extension of the target DBMS (`quarkus-jdbc-postgresql` and friends), `quarkus-flyway`, and the Flyway module of that DBMS (`quarkus-flyway-postgresql`) — without it Flyway finds no plugin for the JDBC URL
- ask before changing `pom.xml`, once, with every build change in the one question — including AssertJ as a test dependency when the module lacks it
- the persistence unit is configured in `application.properties`, no `persistence.xml`: `quarkus.datasource.db-kind`, `quarkus.hibernate-orm.schema-management.strategy=validate`, `quarkus.flyway.migrate-at-start=true`
- another Jakarta EE runtime: `META-INF/persistence.xml` in version 3.2 with a `jta-data-source`
- connection coordinates come from the environment (`QUARKUS_DATASOURCE_*`), never from a committed URL line — it would also switch off Dev Services; the password never lands in a source file

## Health

- a database-backed BC needs a readiness check that proves the pool hands out a working connection; this skill writes none — the runtime provides it (Quarkus registers a datasource readiness check through `quarkus-smallrye-health`) and `/microprofile-server` owns any custom `@Readiness` check

## Testing

- mapping, queries, and controls are tested against the **same DBMS** as production: `@QuarkusTest` with Dev Services, schema applied by the Flyway migrations; these are integration tests in `/microprofile-server` terms (`IT` suffix, failsafe)
- the test class carries `@TestTransaction` — a control needs the transaction a boundary would open, and the rollback removes the rows
- the mapping is tested through a round trip on an injected `EntityManager`: `persist`, `flush()`, `clear()`, `find`; the reloaded entity is another instance, so assert `reloaded.toJSON()` equal to `saved.toJSON()` — one assertion over every serialized field, and independent of the fields a provider adds to an entity class
- the conflict translation is tested once per BC: the second `save` of a duplicate throws `ClientErrorException`; it is the last database call of its test, because the DBMS aborts the transaction after the violation
- a test that was skipped has not passed: when the build skips integration tests or no container runtime is available, run them explicitly or report that the mapping is unverified
- system tests stay black box: they exercise the boundary over HTTP and never read the tables (`/system-tests`)

## Checklist Before Finishing

- the application starts with `validate` against the migrations
- no to-one association without `fetch = FetchType.LAZY`; no `EAGER` anywhere
- no provider import, no control method that only forwards to the `EntityManager`, no `merge` of a new or partially filled entity
- every write the client can make conflict runs `persist` or `merge` and `flush` inside the translating `try`
- a round-trip test exists for every entity and was executed, not skipped

## Composition

- `/sbce`: an entity and the code that persists it are converged like any other code — the spec names the boundary operation, this skill shapes the storage behind it
- `/jdbc-persistence` is the plain-SQL sibling with the same placement, schema, and conflict mapping; this skill reads its Schema section (Type Mapping, Constraints and Indexes) instead of restating it, so the two are installed together; a BC uses one of the two, never both — the persistence context does not see rows written through a `DataSource`
- `/memory-persistence` is the default start; a BC that began there migrates by turning its records into entity classes, replacing a control that only stored and found with the `EntityManager` in the boundary — a control with logic keeps its `save`, `update`, and `findById` signatures — and adding `@Transactional` (or the `@Boundary` stereotype) to the boundary
