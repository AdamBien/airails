---
name: jdbc-persistence
description: Persist and query Java records with plain JDBC and Data-Oriented Programming: records as row shapes, sealed interfaces for variants, hand-written SQL, no ORM. Use when adding persistence, a table, a query, a repository, or a database-backed BC; when mapping `ResultSet` rows to records; or on "JDBC", "plain SQL", "DataSource", "PreparedStatement", "DDL", "CREATE TABLE", "migration". Not for JPA/Hibernate or NoSQL.
argument-hint: "[BC, record, or query to persist]"
---

Add JDBC persistence using $ARGUMENTS. Apply all rules below strictly; `/bce`, `/java-conventions`, and the stack skill hold wherever this skill is silent.

## Principle

Data-Oriented Programming applied to the relational store: the record **is** the row shape, the sealed interface **is** the closed set of variants, and the control is a set of functions that move data between the two representations. Nothing is hidden — no proxies, no lazy loading, no dirty checking, no generated SQL. What the code reads is what the database executes.

- model the data, the whole data, nothing but the data — one record per row shape, one record per projection; no record carries columns a query does not return
- data is immutable and transparent — a change is a new record (`with`-style copy through the canonical constructor), then an `UPDATE`
- make illegal states unrepresentable — closed sets as sealed interfaces or enums; the compiler, not a `default` branch, proves the column mapping complete
- separate operations from data — SQL and binding live in the control; records carry components, invariants, domain behavior, and their own creation from a row (`fromResultSet`), the read-side twin of `toJSON()`; `java.sql` enters a record only through that factory

## Placement (BCE)

- `entity` — the records and sealed interfaces; free of annotations and SQL strings; the only `java.sql` types a record sees are the `ResultSet` and `SQLException` of its `fromResultSet` factory
- `control` — classes named after their responsibilities (`Customers`, `OrderReports`), never `*Repository`, `*Dao`, `*Mapper`, `*Store`; a control holds the SQL, binding, and row collection for the queries it answers and may read or write several tables of its BC in one method; one table is written by exactly one control
- `boundary` — owns the unit of work: one connection, one transaction, several control calls; never contains SQL
  - `/microprofile-server`: `@Transactional` on the boundary class, never on single methods, and `@Inject` of the control; the boundary never sees a `DataSource` or `Connection` — the container opens the transaction, the control's connections enlist in it, the container commits. When the project defines a `@Boundary` stereotype (a CDI `@Stereotype` carrying `@ApplicationScoped` and `@Transactional`), use it instead of an explicit `@Transactional`, likewise on the class; introduce the stereotype only when the project already has one or asks for it
  - `/java-cli-app`: the boundary opens the connection (`DriverManager.getConnection(url, user, password)` or a driver `DataSource`), sets `setAutoCommit(false)`, passes the connection to the controls, commits; a thrown exception rolls back in a `catch` and rethrows. A read-only query may run with auto-commit; a boundary method that writes never does
- connection coordinates come from configuration, never from literals: `quarkus.datasource.*` / MicroProfile Config on the server, `/zcfg` keys (`db.url`, `db.user`, `db.password`) in a CLI; the password never lands in a source file
- the schema belongs to the BC: each BC ships its own DDL; a table is owned by exactly one BC
- a reference to another BC is an `id` component in the referencing record, backed by a real `FOREIGN KEY` (see Schema); never a nested record of the other BC's entity; a join across BC boundaries is confined to read-only projections and is a coupling signal

## Control Functions

- `/microprofile-server`: the control is an `@ApplicationScoped` CDI bean with `@Inject DataSource dataSource;`; each method opens its own connection in a try-with-resources — `try (var connection = this.dataSource.getConnection())` — and the pool hands back the connection enlisted in the boundary's JTA transaction, so several control calls in one boundary method share one transaction without passing anything around
- `/java-cli-app`: the control is an interface with `static` methods and the `Connection` is the first parameter — `static Order save(Connection connection, Order order)`
- every method does one statement, or one logical unit of statements; it never commits or rolls back
- SQL lives in `static final String` text blocks next to the function that executes it; explicit column lists, never `SELECT *`
- parameters are bound with `?` only; a dynamic clause (sort column, filter name) comes from an enum constant or a `switch` over a sealed type, never from an interpolated argument
- a `ResultSet` never leaves the function that opened it: a `List<T>` result is collected in one `while (rows.next())` loop inside a single row-collector method that calls `Order.fromResultSet(rows)` per row — the one place where a loop beats a stream
- bulk writes use `addBatch`/`executeBatch`
- generated keys: `RETURNING id` (PostgreSQL) or `getGeneratedKeys`; the function returns the record **with** the id — the caller never mutates anything
- a lookup by id returns `Optional<T>`; `List` results are empty, never `null`

## Row Mapping

- assume a persisted record has five or more columns, so the constructor call that reads them is a function of its own and belongs next to the components it fills: every persisted record and projection creates itself through `static Order fromResultSet(ResultSet rows) throws SQLException` on the record, reading columns by **name** and funnelling through the canonical constructor, so the compact constructor validates persisted data on the way in. The factory reads the current row only — it never calls `next()` — and the control's row collector is the sole caller; no `toOrder` in the control
- writes stay in the control: one `static void bind(PreparedStatement statement, Order order)` per record, shared by `save` and `update`; a record never imports `PreparedStatement`
- a column is the `snake_case` form of its component name (`featureNumber` ↔ `feature_number`), so the column-to-component mapping is derivable and never documented as a list; the one place a column cannot follow the convention (legacy schema, reserved word) gets a one-line `///` comment on the component naming the column — no column table in a doc comment, it is a fifth copy of the list the checklist already keeps in agreement
- for a wide record, `fromResultSet` may read each column into a local named like the component before the constructor call (`var customer = rows.getString("customer");`), so the reader sees which component a column feeds without counting constructor positions; it is the readable shape, not a requirement
- columns map to the most specific Java SE type through `getObject(column, Type.class)`: `Instant`/`OffsetDateTime`/`LocalDate` for temporal columns, `BigDecimal` for money, `UUID` where the DBMS supports it; never `java.sql.Timestamp` or `Date` in a record
- enums are stored by `name()` and read with `Enum.valueOf`; never by ordinal
- a sealed interface maps to a `NOT NULL` discriminator column plus one typed column per variant detail (or one `JSONB` column for wide variants); the DDL `CHECK` on the discriminator, the write `switch`, and the read `switch` enumerate the same set. Writing is an exhaustive `switch` over the sealed type with no `default` in the control's `bind`, so a new variant fails compilation until bound; reading is `static Status fromResultSet(ResultSet rows)` on the sealed interface, called from the owning record's `fromResultSet`, a `switch` over the discriminator whose only `default` throws — it guards against rows written by a newer deployment, not against a missing case
- a nullable column is a modeling signal: prefer a variant (`Shipped(on)` vs `Open()`) or a separate table over a nullable component; a variant's detail column is nullable by nature, every other nullable column exists only where the record documents an optional component, read with `getObject(column, Type.class)`, and `null` never passes on into a collection
- projections (a report row, a count with a name, a dropdown entry) get their own small record in `entity`; never a half-filled aggregate record
- a parent with children (`Customer` with `List<Address>`) is loaded by one join grouped in the row collector, or by two queries when the child list is large; never by one query per parent
- no reflection, no annotation-driven mapping, no mapping library; a record with twenty columns has a twenty-line `fromResultSet` and that is fine

## Exceptions

- control functions translate `SQLException` at the point of execution, directly into the exception the stack already understands, so no layer maps it a second time; the `SQLException` travels as the cause
  - `/microprofile-server`: SQLState class `23` (integrity violation) becomes `ClientErrorException(constraintName, Response.Status.CONFLICT, cause)`; everything else becomes `InternalServerErrorException(sql, cause)`. Both are `WebApplicationException`s (the `/microprofile-server` exception rule), so the JAX-RS runtime answers 409 or 500 and `@Transactional` rolls back; the boundary catches nothing and no `ExceptionMapper` is written
  - `/java-cli-app`: class `23` becomes an `IllegalArgumentException` whose message names the constraint; everything else an `IllegalStateException` carrying the SQL statement; the boundary prints the message and exits non-zero
- where the driver reports the SQLState subclass (PostgreSQL, H2), be more concrete and split class `23` by fault: `23505` (unique) and `23503` (foreign key) are the client's — only the database can detect a duplicate or an unknown id — and stay the conflict above, with the message naming the kind and the constraint (`duplicate: uq_customer_email`); `23502` (not null) and `23514` (check) mean the compact constructor and the DDL disagree, a bug, so they become the internal error like any other failure. MySQL and MariaDB report only `23000` and distinguish by `getErrorCode()`; SQLite's driver reports vendor codes, so there class `23` stays undivided
- the message passes the driver's `getMessage()` through: PostgreSQL and H2 name the violated constraint in it, SQLite names `table.column` instead; a bare constraint name is a one-line regex in the control, never a parsing library and never a cast to the driver's exception type
- the translation is one `static` function per BC — a `switch` over `getSQLState()` that the control methods call from their `catch` — so the mapping is written once and the methods stay one statement each
- a constraint violation is never caught to steer logic: create-or-update is `INSERT ... ON CONFLICT` (PostgreSQL, SQLite) or `MERGE` (H2), one statement, no exception

## Schema

- the agent derives the **first** migration from the records with the mapping below; from then on the DDL is the source of truth — a schema change starts in SQL and the record follows, never the other way round (no `ddl-auto`, no schema derived at runtime)
- DDL is hand-written SQL, one file per BC per change, named for the BC with a timestamp version so BCs never collide: `db/migration/V<yyyyMMddHHmm>__<bc>_<change>.sql`
- table and column names are `snake_case`, singular table per record (`order_line`), primary key `id`
- every constraint and index is named — `pk_<table>`, `uq_<table>_<column>`, `fk_<table>_<referenced_table>`, `ck_<table>_<rule>`, `ix_<table>_<column>` — because the integrity-violation message (see Exceptions) carries exactly that name
- DDL targets the named DBMS (PostgreSQL by default); no portable lowest-common-denominator SQL, since tests run against the production engine; in a CLI on H2 or SQLite use that engine's identity syntax and check its `IF NOT EXISTS` support
- `/microprofile-server`: apply through Flyway (`quarkus-flyway`) at startup — ask before adding the dependency to `pom.xml`
- `/java-cli-app`: a `Schema` control in the owning BC executes idempotent `CREATE TABLE IF NOT EXISTS` statements from a text block at startup; add a `schema_version` table only when destructive changes appear

### Type Mapping

The inverse of the `getObject(column, Type.class)` rule; one column per record component:

- `long id` → `BIGINT GENERATED ALWAYS AS IDENTITY`
- `String` → `VARCHAR(n)` when the compact constructor bounds the length, else `TEXT`
- `BigDecimal` → `NUMERIC(p, s)` with explicit precision and scale
- `Instant` / `OffsetDateTime` → `TIMESTAMPTZ`; `LocalDate` → `DATE`; no `LocalDateTime` column (it has no zone)
- `UUID` → `UUID`; `boolean` → `BOOLEAN`; `int` → `INTEGER`; `long` → `BIGINT`
- enum → `VARCHAR` plus `CHECK (column IN ('A', 'B'))` listing the `name()` values
- sealed interface → discriminator `VARCHAR NOT NULL` with a `CHECK` listing the variants, plus the variants' detail columns typed by this table (see Row Mapping)

### Constraints and Indexes

- constraints mirror the compact constructor: every invariant it checks and SQL can express becomes `NOT NULL`, `CHECK`, or `UNIQUE`
- every column a control's `WHERE` or `ORDER BY` uses beyond the primary key gets an index, declared in the same migration as the query that needs it, or a one-line comment saying why not; every foreign key column is indexed
- a reference inside the BC is a `FOREIGN KEY` to the owning table; a reference to another BC is also a real `FOREIGN KEY`, declared in the referencing BC's migration, which is versioned after the referenced BC's `CREATE TABLE`; that FK is the only DDL that names another BC's table

## Drivers

- `/microprofile-server`: the Quarkus JDBC extension of the target DBMS (`quarkus-jdbc-postgresql` and friends) brings the driver and the Agroal pool; `@Inject DataSource` is the only API the code sees
- `/java-cli-app`: `zb` bundles no binary dependencies, so the driver JAR lives in `lib/` and is appended at launch — `java -cp zbo/app.jar:lib/* Main`; put that line in the `/java-cli-script` launcher; prefer a single-JAR, pure-Java driver (PostgreSQL, H2, SQLite) that needs no native setup
- `DriverManager` auto-discovers drivers from `META-INF/services`; never call `Class.forName`

## Health

- a database-backed BC needs a readiness check that proves the pool can hand out a working connection; a server that answers ready while its `DataSource` is down fails requests with 500 instead of being taken out of rotation
- this skill writes no check: the concrete MicroProfile runtime provides it (Quarkus registers a `DataSource` readiness check through `quarkus-smallrye-health` when the Agroal extension is present) and `/microprofile-server` owns any custom `@Readiness` check, placed in the boundary package; a control never exposes `isValid` or a `SELECT 1` for that purpose
- `/java-cli-app`: no readiness probe exists; the boundary's first `getConnection` is the check, and its failure is reported as in Exceptions

## Testing

- control functions are tested against a real connection of the **same DBMS** as production; no in-memory substitute unless production is that engine
- `/microprofile-server`: `@QuarkusTest` with Dev Services starts the container; the test injects the control, calls it, and asserts on returned records with AssertJ; each test seeds and cleans its own rows
- `/java-cli-app`: a `/zunit` test opens a connection to a disposable database (temporary H2/SQLite file, or a local PostgreSQL named in `app.properties`), runs `Schema`, exercises the control, asserts with `assert`
- the mapping is tested through a round trip — `save` then `findById` yields an equal record; records make equality exact, so one assertion covers every column
- system tests stay black box: they exercise the boundary (HTTP, CLI) and never read the tables (`/system-tests`)

## Checklist Before Finishing

- the column lists of the `INSERT`, the control's `bind`, the record's `fromResultSet`, and the DDL agree
- a round-trip test exists for every record that is persisted
- every compact-constructor invariant SQL can express is a named constraint

## Composition

- `/sbce`: a persisted record and its control are converged like any other code — the spec names the boundary operation, this skill shapes the storage behind it
- `/memory-persistence` is the collection-backed sibling with the same control contract; a BC that started there migrates by replacing the control class, adding `fromResultSet` to its records, and adding `@Transactional` (or the `@Boundary` stereotype) to the boundary
