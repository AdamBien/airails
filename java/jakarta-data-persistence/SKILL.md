---
name: jakarta-data-persistence
description: 'Persist and query Jakarta Persistence entities through Jakarta Data 1.0 repositories in MicroProfile / Jakarta EE servers: a `@Repository` interface in the control layer named after its responsibility, explicit `@Insert` / `@Update` / `@Delete` statements without a persistence context, `@Find` and `@Query` methods checked by the annotation processor, record projections, a Flyway-owned schema the mapping is validated against. Use on demand — when the request names "Jakarta Data", "data repository", "@Repository", "@Find", "JDQL", or "stateless session", or when the project build already carries `jakarta.data-api`. Not the default for an unqualified "add persistence" — use `/memory-persistence`. Not for the `EntityManager` and managed entities — use `/jakarta-persistence`. Not for plain SQL — use `/jdbc-persistence`. Not for Spring Data, CLI applications, or NoSQL.'
argument-hint: "[BC or entity to persist]"
---

Add Jakarta Data persistence using $ARGUMENTS. Apply all rules below strictly; `/bce`, `/java-conventions`, and `/microprofile-server` hold wherever this skill is silent.

This skill is picked on demand: when the request names no persistence technology and the project's build carries no `jakarta.data-api`, stop and use `/memory-persistence` instead.

Among the durable skills, `/jdbc-persistence` is the successor of `/memory-persistence`. This skill takes a BC when the request names Jakarta Data or the project already persists through it; in a project that already maps entities with Jakarta Persistence it fits a BC whose writes are single statements known in advance and whose reads are queries. A BC whose boundary call changes a loaded graph of entities in ways not known in advance belongs to `/jakarta-persistence` — nothing here derives a statement from a changed entity.

## Platform

- Jakarta Data 1.0, the release for Jakarta EE 11, over a Jakarta Persistence provider; read the managed `jakarta.data-api` version from the effective POM (`mvn help:effective-pom`) and never change it without asking
- the repository implementation is generated at compile time by the provider's annotation processor — a build change (see Configuration); without the processor a repository interface has no implementation
- code imports `jakarta.data.*` for repositories and `jakarta.persistence.*` for the mapping, never a provider package (`org.hibernate.*`) — which also rules out a resource accessor method returning the provider's session
- the artifacts, the processor, configuration keys, and test support of a runtime live in its reference — Quarkus: [references/quarkus.md](references/quarkus.md), read before changing build, configuration, or tests
- server only: a `/java-cli-app` persists with `/jdbc-persistence`

## Principle

A repository method is one statement, executed when it is called. There is no persistence context: an entity returned by a repository is detached, changing it changes nothing in the database, and nothing is written until an `@Update` method receives it — unlike `/jakarta-persistence`, where the provider writes the difference at commit.

- entities are the `@Entity` classes of `/jakarta-persistence`; records carry embeddable value objects, primary key classes, and projections
- every read returns a new instance — two finds of the same id are not the same object, and nothing is cached between calls
- no cascade: a repository writes the entity it was handed and nothing reachable from it
- the schema is not derived from the mapping; the mapping is validated against the schema

## Placement (BCE)

- `entity` — the `@Entity` classes, their `@Embeddable` records, and the projection records; the only layer that carries mapping annotations
- `control` — the repository: an interface annotated `@Repository`, named after its responsibility (`Customers`, `OrderReports`), never `*Repository`, `*Dao`, `*Store`; it extends no built-in supertype (`BasicRepository`, `CrudRepository`, `DataRepository`) — each brings methods the BC never asked for, among them the `save` that hides whether a row is inserted or updated; one repository has one entity its lifecycle methods take, and the queries that start from that entity
- `boundary` — owns the unit of work: `@Transactional` on the class or the project's `@Boundary` stereotype (`/microprofile-server`), one transaction per call; it injects the repository (`@Inject Customers customers;`) and calls it directly — no control class is written between the two, since it could only forward (`/java-conventions`); a control class exists where logic spans several repositories or another BC calls the operation
- the boundary's transaction is not optional: a repository also works without one, every statement committing on its own, so a missing `@Transactional` raises no error and leaves a boundary call of several writes half done
- a reference to another BC's entity is a unidirectional `@ManyToOne(fetch = FetchType.LAZY)` or a plain id field, backed by the `FOREIGN KEY` the referencing BC's migration declares; the other BC writes its own table

## Entities

- the Entities rules of `/jakarta-persistence` apply: shape, field access, no `get`/`set` pairs, `toJSON()` / `fromJSON`, identity id, names, `BigDecimal` scale, types, enums, record embeddables, `fetch = FetchType.LAZY` on every to-one
- a state-changing method on the entity (`ship(Instant on)`) changes the instance only; the boundary passes the entity to the repository's `update` afterwards — the call that is implicit in `/jakarta-persistence` is written here
- `update` writes every column: a full replacement is the entity `fromJSON` created, with the id from the request path; a partial change loads the entity with `findById`, calls its method, and updates — a partially filled instance is never updated
- no `cascade` and no `orphanRemoval` on any association — the repository applies neither, so they would state a behavior that does not happen; a child is written by a lifecycle method that takes the child, inside the same boundary call
- `@Version` only when the spec asks for lost-update protection

## Repository

- every abstract method carries exactly one of `@Insert`, `@Update`, `@Delete`, `@Find`, `@Query`; a method without one is not derived from its name — the processor only warns (`repository method cannot be implemented (skipping whole repository)`), the build succeeds, no implementation is generated, and the injection fails at startup
- the method names follow the control contract of the sibling skills, so a boundary survives a migration: `save(Order order)` returns the entity with its id, `update(Order order)` returns the entity, `findById(long id)` returns `Optional`, `all()` and queries return `List`, empty, never `null`; the annotation decides what a method does, never its name
- `@Insert` assigns the generated id to the instance it received and returns that instance; a `List` parameter inserts several entities
- `@Update` and `@Delete` with an entity parameter expect the row to exist — a missing row is an `OptimisticLockingFailureException` (see Exceptions)
- `@Delete void remove(long id)` removes by the field the parameter is named after and reports nothing when no row matched — what an idempotent DELETE needs
- no `@Save`: the boundary operation already knows whether it creates or replaces, and an upsert hides a wrong id as a new row
- a `default` method is hand-written code in the control and is used for the conflict translation (see Exceptions); it never rebuilds a query in Java that a `@Find` or `@Query` method states

## Queries

- `@Find` first: every parameter is named after an entity field and compared for equality; the processor rejects an unknown name and a wrong type at compile time; a fixed order is `@OrderBy("name")`, repeatable
- `@Query` where `@Find` ends — another comparison, a projection, an aggregate, a fetched association, a bulk `update` or `delete`; the query is JPQL in a text block, its named parameters (`:roast`) are bound to the method parameters of the same name; no positional parameters, no `@Param`
- the processor checks the syntax of the query text and the binding and types of its parameters; a misspelled attribute inside the text is not reported, so every `@Query` method is executed by at least one test (see Testing)
- a single result that may be missing returns `Optional`; a non-`Optional` single result is never declared, since it turns "not found" into an `EmptyResultException`
- projections (a report row, a count with a name, a dropdown entry) are records in `entity`, filled by a constructor expression — `select new airhacks.orders.entity.OrderSummary(o.id, o.total) from Order o`; no entity is loaded to read two of its fields
- an association the caller navigates is fetched by the query: `from Order o join fetch o.lines where o.id = :id`; an unfetched collection fails with `LazyInitializationException` on access, an unfetched to-one is loaded by a further statement per entity
- the first n rows: a `Limit` parameter; `PageRequest` with a `Page` return type only when the spec asks for paging, on a method whose `@OrderBy` ends with the id, so the order is total
- a bulk statement (`update Coffee set price = price * :factor where roast = :roast`) returns `int` or `void` and does not touch entities already loaded
- a query that needs native SQL is a signal to ask whether that BC belongs to `/jdbc-persistence`

## Exceptions

- a statement runs inside the repository call, so a constraint violation surfaces there, as a `jakarta.data.exceptions.DataException` — no flush is needed to reach it
- the exception type does not name the violation: `@Insert` reports every constraint violation as `EntityExistsException`, a foreign key to a missing row included, while `@Update` and `@Delete` report the same violations as a plain `DataException`; the type is therefore never used to tell them apart
- a write the client can make conflict (a unique column, a foreign key to a row that may be gone, a row another row still references, a `@Version`) is a `default` method on the repository: it calls the annotated method inside a `try` and translates the `DataException`; the `default` method keeps the contract name, the annotated method takes the name of what the statement does — `insert` behind `save`, `replace` behind `update`, `delete` behind `remove` — and the boundary calls only the contract name; a write that cannot conflict stays the annotated method under its contract name
- the translation is the one of `/jakarta-persistence`: one `static` function per BC, in a package-private interface of the control package named after the responsibility (`Conflicts`); it walks the cause chain to the `SQLException` and switches over `getSQLState()` — `23505` (unique) and `23503` (foreign key) become `ClientErrorException(message, Response.Status.CONFLICT, cause)` with the message naming the kind and the constraint (`duplicate: uq_customer_email`, `unknown reference: fk_order_customer`); everything else becomes `InternalServerErrorException(cause)`
- `23503` names two opposite faults and does not tell which: behind an insert or update the referenced row is missing, behind a delete the row is still referenced; the `default` method of a delete therefore calls a second function of the same interface (`Conflicts.translateDelete`) that differs in this one message — `still referenced: fk_order_customer`
- `OptimisticLockingFailureException` comes from `@Update` or `@Delete` of an entity whose row is gone or whose `@Version` has moved; the function turns it into `ClientErrorException(Response.Status.CONFLICT)`; an unknown id is not left to it — the boundary answers `NotFoundException` when `findById` returns empty, before it updates, so the exception remains for a row that was removed or changed between that read and the update
- both results are `WebApplicationException`s (`/microprofile-server`): the JAX-RS runtime answers 409 or 500, `@Transactional` rolls back, the boundary catches nothing, and no `ExceptionMapper` is written; an untranslated `DataException` ends as a 500

## Schema, Configuration, Health

- the Schema, Configuration, and Health rules of `/jakarta-persistence` apply unchanged: Flyway migrations per BC with named constraints, the first one derived from the entities, the provider validating and never generating, a JTA data source, coordinates from the environment, the runtime's readiness check
- the build additions of this skill — the Jakarta Data API and the provider's annotation processor — join the one `pom.xml` question: ask before changing `pom.xml`, once, with every build change in it

## Testing

- repositories are tested against the **same DBMS** as production, started and migrated by the runtime's test support (see its reference); these are integration tests in `/microprofile-server` terms (`IT` suffix, failsafe)
- every test runs in a transaction that is rolled back at its end
- the mapping is tested through a round trip on the injected repository: `save`, then `findById`; the found entity is another instance without any clearing, so assert `reloaded.toJSON()` equal to `saved.toJSON()`
- every `@Query` method is executed once with a row that matches — the only check of the attribute names in its text
- the conflict translation is tested once per BC: the second `save` of a duplicate throws `ClientErrorException`; a translated delete is tested the same way, by removing a row another row references; each is the last database call of its test, because the DBMS aborts the transaction after the violation
- a test that was skipped has not passed: when the build skips integration tests or no container runtime is available, run them explicitly or report that the mapping and the queries are unverified
- system tests stay black box: they exercise the boundary over HTTP and never read the tables (`/system-tests`)

## Checklist Before Finishing

- the build generated an implementation for every repository (`<Repository>_` under the generated sources) and reported no "cannot be implemented" warning
- no abstract repository method without an annotation, no built-in supertype, no `@Save`, no provider import
- no `cascade`, no `orphanRemoval`; every changed entity is passed to `update`
- the boundary is `@Transactional`; no control class that only forwards to a repository
- every write the client can make conflict is called through its translating `default` method
- a round-trip test exists for every entity, every `@Query` method is executed by a test, and none of them was skipped

## Shape

```java
@Repository
public interface Coffees {

    @Insert
    Coffee insert(Coffee coffee);

    @Update
    Coffee replace(Coffee coffee);

    @Delete
    void remove(long id);

    @Find
    Optional<Coffee> findById(long id);

    @Find
    @OrderBy("name")
    List<Coffee> all();

    @Find
    @OrderBy("name")
    List<Coffee> byRoast(Roast roast);

    @Query("""
            select new airhacks.coffee.entity.CoffeeSummary(c.id, c.name)
            from Coffee c
            where c.price < :limit
            order by c.name
            """)
    List<CoffeeSummary> cheaperThan(BigDecimal limit);

    default Coffee save(Coffee coffee) {
        try {
            return insert(coffee);
        } catch (DataException e) {
            throw Conflicts.translate(e);
        }
    }

    default Coffee update(Coffee coffee) {
        try {
            return replace(coffee);
        } catch (DataException e) {
            throw Conflicts.translate(e);
        }
    }
}
```

## Composition

- `/sbce`: an entity and its repository are converged like any other code — the spec names the boundary operation, this skill shapes the storage behind it
- `/jakarta-persistence` is the sibling with a persistence context; this skill reads its Entities, Schema, Configuration, and Health sections instead of restating them, so the two are installed together; a BC uses one of the two, never both — a repository statement bypasses the persistence context, which neither sees the write nor refreshes what it has loaded; two BCs of one project may differ
- `/jdbc-persistence` is the plain-SQL sibling and owns the Schema rules (Type Mapping, Constraints and Indexes) both entity-based skills read
- `/memory-persistence` is the default start; a BC that began there migrates by turning its records into entity classes, replacing the control class with a repository interface of the same method names, and adding `@Transactional` (or the `@Boundary` stereotype) to the boundary
