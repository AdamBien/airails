---
name: jdbc-persistence
description: Persist and query data with plain JDBC and Data-Oriented Programming (DOP) — records as row shapes, sealed interfaces for variants, exhaustive pattern matching for the column mapping, SQL in text blocks, and controls that execute the statements — CDI beans with an injected `DataSource` under a `@Transactional` boundary on the server, stateless functions over a boundary-owned `Connection` in a CLI. No ORM, no JPA annotations, no reflection-based mapping. Composes with `/bce` for placement, `/java-conventions` for style, and the stack skill — `/microprofile-server` (CDI, `@Inject DataSource`, Dev Services) or `/java-cli-app` (driver JAR on the classpath, explicit commit). Use when adding persistence, a table, a query, a repository, or a database-backed BC; when mapping `ResultSet` rows to records; or when asked for "JDBC", "plain SQL", "data-oriented persistence", "persist records", "query records", "DataSource", "PreparedStatement", "ResultSet mapping", "schema", "migration SQL", "no JPA", "no Hibernate". Not for JPA/Hibernate projects and not for NoSQL stores.
argument-hint: "[BC, record, or query to persist]"
---

Add JDBC persistence using $ARGUMENTS. Apply all rules below strictly; `/bce`, `/java-conventions`, and the stack skill hold wherever this skill is silent.

## Principle

Data-Oriented Programming applied to the relational store: the record **is** the row shape, the sealed interface **is** the closed set of variants, and the control is a set of functions that move data between the two representations. Nothing is hidden — no proxies, no lazy loading, no dirty checking, no generated SQL. What the code reads is what the database executes.

- model the data, the whole data, nothing but the data — one record per row shape, one record per projection; no record carries columns a query does not return
- data is immutable and transparent — records only; a change is a new record (`with`-style copy through the canonical constructor), then an `UPDATE`
- make illegal states unrepresentable — closed sets as sealed interfaces or enums; the compiler, not a `default` branch, proves the column mapping complete
- separate operations from data — SQL, binding, and row mapping live in the control; records carry components, invariants, and domain behavior, never `java.sql` imports

## Placement (BCE)

- `entity` — the records and sealed interfaces; free of `java.sql`, annotations, and SQL strings
- `control` — one class per aggregate root holding its SQL, binding, and row mapping; name it after the responsibility in plural (`Orders`, `Speakers`), never `*Repository`, `*Dao`, `*Mapper`, `*Store`
- `boundary` — owns the unit of work: on the server it carries `@Transactional` and injects the control; in a CLI it opens the connection, passes it to the control functions, and commits; never contains SQL
- the schema belongs to the BC: each BC ships its own DDL; a table is owned by exactly one BC
- a foreign key across BCs is an `id` component in the referencing record (`/bce`: the schema is the source of truth); never a nested record of the other BC's entity

## Control Functions

- `/microprofile-server`: the control is an `@ApplicationScoped` CDI bean with `@Inject DataSource dataSource;` (field injection, `/java-conventions`); each method opens its own connection in a try-with-resources — `try (var connection = this.dataSource.getConnection())` — and the pool hands back the connection enlisted in the boundary's JTA transaction, so several control calls in one boundary method share one transaction without passing anything around
- `/java-cli-app`: the control is an interface with `static` methods and the `Connection` is the first parameter — `static Order save(Connection connection, Order order)`; there is no container to scope a transaction, so the boundary passes the connection it owns
- every method does one statement, or one logical unit of statements; it never commits or rolls back
- SQL lives in `static final String` text blocks next to the function that executes it; explicit column lists, never `SELECT *`
- parameters are bound with `?` only; a dynamic clause (sort column, filter name) comes from an enum constant or a `switch` over a sealed type, never from an interpolated argument
- `PreparedStatement` and `ResultSet` are closed by try-with-resources inside the function
- a `List<T>` result is collected in one `while (rows.next())` loop inside a single `rows(ResultSet)`-style method — the one place where a loop beats a stream; callers see a `List`, never a `ResultSet`
- bulk writes use `addBatch`/`executeBatch`
- generated keys: `RETURNING id` (PostgreSQL) or `getGeneratedKeys`; the function returns the record **with** the id — the caller never mutates anything

## Row Mapping

- one `static Order toOrder(ResultSet rows)` per record in the control, reading columns by **name** and funnelling through the canonical constructor, so the compact constructor validates persisted data on the way in
- one `static void bind(PreparedStatement statement, Order order)` per record for writes; `save` and `update` share it
- columns map to the most specific Java SE type through `getObject(column, Type.class)`: `Instant`/`OffsetDateTime`/`LocalDate` for temporal columns, `BigDecimal` for money, `UUID` where the DBMS supports it; never `java.sql.Timestamp` or `Date` in a record
- enums are stored by `name()` and read with `Enum.valueOf`; never by ordinal
- a sealed interface maps to a discriminator column plus the variant's detail columns (or one JSON column for wide variants); writing is an exhaustive `switch` over the sealed type with no `default`, so a new variant fails compilation until bound; reading is a `switch` over the discriminator whose only `default` throws — it guards against rows written by a newer deployment, not against a missing case
- a nullable column is a modeling signal: prefer a variant (`Shipped(on)` vs `Open()`) or a separate table over a nullable component; when the column is genuinely optional, read it with `getObject(column, Type.class)`, document the invariant in the compact constructor, and never pass `null` on into a collection
- projections (a report row, a count with a name, a dropdown entry) get their own small record in `entity`; never a half-filled aggregate record
- no reflection, no annotation-driven mapping, no mapping library; a record with twenty columns has a twenty-line mapper and that is fine

## Transactions and Connections

- the boundary owns the unit of work: one connection, one transaction, several control calls
- `/microprofile-server`: `@Transactional` on the boundary method and `@Inject` of the control; the boundary never sees a `DataSource` or `Connection` — the container opens the transaction, the control's connections enlist in it, the container commits
- `/java-cli-app`: the boundary opens the connection (`DriverManager.getConnection(url, user, password)` or a driver `DataSource`), sets `setAutoCommit(false)`, calls the controls, commits; a thrown exception rolls back in a `catch` and rethrows
- a read-only query may run with auto-commit; a boundary method that writes never does
- connection coordinates come from configuration, never from literals: `quarkus.datasource.*` / MicroProfile Config on the server; `/zcfg` keys (`db.url`, `db.user`, `db.password`) in a CLI app; the password never lands in a source file

## Exceptions

- control functions translate `SQLException` at the point of execution; checked exceptions never cross a layer
- SQLState class `23` (integrity violation) becomes an `IllegalArgumentException` whose message names the constraint; everything else becomes an `IllegalStateException` carrying the SQL statement
- the boundary maps those to the protocol: `/microprofile-server` throws `ClientErrorException(Response.Status.CONFLICT)` for the integrity case and lets the rest surface as 500; `/java-cli-app` prints the message and exits non-zero
- a lookup by id returns `Optional<T>`; `List` results are empty, never `null`

## Schema

- DDL is hand-written SQL in text, one file per BC, named for the BC: `db/migration/V<n>__<bc>_<change>.sql`
- table and column names are `snake_case`, singular table per record (`order_line`), primary key `id`
- `/microprofile-server`: apply through Flyway (`quarkus-flyway`) at startup — ask before adding the dependency to `pom.xml`
- `/java-cli-app`: a `Schema` control in the owning BC executes idempotent `CREATE TABLE IF NOT EXISTS` statements from a text block at startup; add a `schema_version` table only when destructive changes appear
- never let the application derive the schema from the records (no `ddl-auto`); the records follow the schema, not the other way round

## Drivers

- `/microprofile-server`: the Quarkus JDBC extension of the target DBMS (`quarkus-jdbc-postgresql` and friends) brings the driver and the Agroal pool; `@Inject DataSource` is the only API the code sees
- `/java-cli-app`: `zb` bundles no binary dependencies, so the driver JAR lives in `lib/` and is appended at launch — `java -cp zbo/app.jar:lib/* Main`; put that line in the `/java-cli-script` launcher; prefer a single-JAR, pure-Java driver (PostgreSQL, H2, SQLite) that needs no native setup
- `DriverManager` auto-discovers drivers from `META-INF/services`; never call `Class.forName`

## Testing

- control functions are tested against a real connection of the **same DBMS** as production; no in-memory substitute unless production is that engine
- `/microprofile-server`: `@QuarkusTest` with Dev Services starts the container; the test injects the control, calls it, and asserts on returned records with AssertJ; each test seeds and cleans its own rows
- `/java-cli-app`: a `/zunit` test opens a connection to a disposable database (temporary H2/SQLite file, or a local PostgreSQL named in `app.properties`), runs `Schema`, exercises the control, asserts with `assert`
- the mapping is tested through a round trip — `save` then `findById` yields an equal record; records make equality exact, so one assertion covers every column
- system tests stay black box: they exercise the boundary (HTTP, CLI) and never read the tables (`/system-tests`)

## Checklist Before Finishing

- no `java.sql` import in `entity`
- no SQL string, `DataSource`, or `Connection` in a server `boundary`
- no `SELECT *`, no string-built SQL
- every `switch` over a sealed type is exhaustive without `default`; the discriminator switch's `default` throws
- every `Connection`, `PreparedStatement`, `ResultSet` is in a try-with-resources
- a round-trip test exists for every record that is persisted

## Shape (`/microprofile-server`)

```java
@ApplicationScoped
public class Orders {

    static final String INSERT = """
        INSERT INTO purchase_order (customer, total, status, status_detail)
        VALUES (?, ?, ?, ?)
        RETURNING id
        """;

    @Inject
    DataSource dataSource;

    public Order save(Order order) {
        try (var connection = this.dataSource.getConnection();
             var statement = connection.prepareStatement(INSERT)) {
            statement.setString(1, order.customer());
            statement.setBigDecimal(2, order.total());
            statement.setString(3, discriminator(order.status()));
            statement.setString(4, detail(order.status()));
            try (var keys = statement.executeQuery()) {
                keys.next();
                return order.withId(keys.getLong("id"));
            }
        } catch (SQLException e) {
            throw translate(e, INSERT);
        }
    }

    static Order toOrder(ResultSet rows) throws SQLException {
        return new Order(
            rows.getLong("id"),
            rows.getString("customer"),
            rows.getBigDecimal("total"),
            toStatus(rows.getString("status"), rows.getString("status_detail")));
    }

    static String discriminator(Status status) {
        return switch (status) {
            case Open _ -> "OPEN";
            case Shipped _ -> "SHIPPED";
            case Cancelled _ -> "CANCELLED";
        };
    }

    static Status toStatus(String discriminator, String detail) {
        return switch (discriminator) {
            case "OPEN" -> new Open();
            case "SHIPPED" -> new Shipped(LocalDate.parse(detail));
            case "CANCELLED" -> new Cancelled(detail);
            default -> throw new IllegalStateException("unknown order status: " + discriminator);
        };
    }
}
```

In a `/java-cli-app` the same class is an interface, the methods are `static`, and `Connection connection` is the first parameter in place of the injected `DataSource`.

## Composition

- `/bce` owns BC carving and layer placement; this skill only says which persistence artifact lands in which layer
- `/java-conventions` owns syntax and style; the only deviation here is the single `while (rows.next())` loop inside the row collector
- `/microprofile-server` and `/java-cli-app` own build, configuration, lifecycle, and the test loop; this skill specializes how they reach the database and never contradicts them
- `/sbce`: a persisted record and its control are converged like any other code — the spec names the boundary operation, this skill shapes the storage behind it
