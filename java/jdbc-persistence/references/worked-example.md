# Worked Example: `orders` BC

One BC, two stacks. The `entity` and `control` layers are identical in both; only the boundary differs.

## Schema — `db/migration/V1__orders_create.sql`

```sql
CREATE TABLE purchase_order (
    id            BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    customer      VARCHAR(200)   NOT NULL,
    total         NUMERIC(12, 2) NOT NULL,
    status        VARCHAR(20)    NOT NULL,
    status_detail VARCHAR(400),
    placed_at     TIMESTAMP WITH TIME ZONE NOT NULL
);
```

## Entity — `airhacks.shop.orders.entity`

```java
public record Order(long id, String customer, BigDecimal total, Status status, Instant placedAt) {

    public Order {
        if (total.signum() < 0) {
            throw new IllegalArgumentException("total must not be negative: " + total);
        }
    }

    public static Order placed(String customer, BigDecimal total) {
        return new Order(0, customer, total, new Open(), Instant.now());
    }

    public Order withId(long id) {
        return new Order(id, this.customer, this.total, this.status, this.placedAt);
    }

    public Order shipped(LocalDate on) {
        return new Order(this.id, this.customer, this.total, new Shipped(on), this.placedAt);
    }
}
```

```java
public sealed interface Status permits Open, Shipped, Cancelled {}
public record Open() implements Status {}
public record Shipped(LocalDate on) implements Status {}
public record Cancelled(String reason) implements Status {}
```

```java
public record OrderSummary(String customer, long count, BigDecimal total) {}
```

## Control — `airhacks.shop.orders.control.Orders`

```java
public interface Orders {

    String INSERT = """
        INSERT INTO purchase_order (customer, total, status, status_detail, placed_at)
        VALUES (?, ?, ?, ?, ?)
        RETURNING id
        """;

    String UPDATE = """
        UPDATE purchase_order
        SET customer = ?, total = ?, status = ?, status_detail = ?, placed_at = ?
        WHERE id = ?
        """;

    String SELECT = """
        SELECT id, customer, total, status, status_detail, placed_at
        FROM purchase_order
        """;

    String BY_ID = SELECT + " WHERE id = ?";

    String SUMMARY = """
        SELECT customer, COUNT(*) AS count, SUM(total) AS total
        FROM purchase_order
        GROUP BY customer
        ORDER BY total DESC
        """;

    static Order save(Connection connection, Order order) {
        try (var statement = connection.prepareStatement(INSERT)) {
            bind(statement, order);
            try (var keys = statement.executeQuery()) {
                keys.next();
                return order.withId(keys.getLong("id"));
            }
        } catch (SQLException e) {
            throw translate(e, INSERT);
        }
    }

    static void update(Connection connection, Order order) {
        try (var statement = connection.prepareStatement(UPDATE)) {
            bind(statement, order);
            statement.setLong(6, order.id());
            statement.executeUpdate();
        } catch (SQLException e) {
            throw translate(e, UPDATE);
        }
    }

    static Optional<Order> findById(Connection connection, long id) {
        try (var statement = connection.prepareStatement(BY_ID)) {
            statement.setLong(1, id);
            try (var rows = statement.executeQuery()) {
                return rows.next() ? Optional.of(toOrder(rows)) : Optional.empty();
            }
        } catch (SQLException e) {
            throw translate(e, BY_ID);
        }
    }

    static List<Order> all(Connection connection) {
        try (var statement = connection.prepareStatement(SELECT);
             var rows = statement.executeQuery()) {
            return orders(rows);
        } catch (SQLException e) {
            throw translate(e, SELECT);
        }
    }

    static List<OrderSummary> summaries(Connection connection) {
        try (var statement = connection.prepareStatement(SUMMARY);
             var rows = statement.executeQuery()) {
            var summaries = new ArrayList<OrderSummary>();
            while (rows.next()) {
                summaries.add(new OrderSummary(
                    rows.getString("customer"),
                    rows.getLong("count"),
                    rows.getBigDecimal("total")));
            }
            return summaries;
        } catch (SQLException e) {
            throw translate(e, SUMMARY);
        }
    }

    static List<Order> orders(ResultSet rows) throws SQLException {
        var orders = new ArrayList<Order>();
        while (rows.next()) {
            orders.add(toOrder(rows));
        }
        return orders;
    }

    static Order toOrder(ResultSet rows) throws SQLException {
        return new Order(
            rows.getLong("id"),
            rows.getString("customer"),
            rows.getBigDecimal("total"),
            toStatus(rows.getString("status"), rows.getString("status_detail")),
            rows.getObject("placed_at", OffsetDateTime.class).toInstant());
    }

    static Status toStatus(String discriminator, String detail) {
        return switch (discriminator) {
            case "OPEN" -> new Open();
            case "SHIPPED" -> new Shipped(LocalDate.parse(detail));
            case "CANCELLED" -> new Cancelled(detail);
            default -> throw new IllegalStateException("unknown order status: " + discriminator);
        };
    }

    static void bind(PreparedStatement statement, Order order) throws SQLException {
        statement.setString(1, order.customer());
        statement.setBigDecimal(2, order.total());
        statement.setString(3, discriminator(order.status()));
        statement.setString(4, detail(order.status()));
        statement.setObject(5, order.placedAt().atOffset(ZoneOffset.UTC));
    }

    static String discriminator(Status status) {
        return switch (status) {
            case Open _ -> "OPEN";
            case Shipped _ -> "SHIPPED";
            case Cancelled _ -> "CANCELLED";
        };
    }

    static String detail(Status status) {
        return switch (status) {
            case Open _ -> null;
            case Shipped s -> s.on().toString();
            case Cancelled c -> c.reason();
        };
    }

    static RuntimeException translate(SQLException e, String sql) {
        var integrityViolation = e.getSQLState() != null && e.getSQLState().startsWith("23");
        if (integrityViolation) {
            return new IllegalArgumentException("constraint violated: " + e.getMessage(), e);
        }
        return new IllegalStateException("statement failed: " + sql, e);
    }
}
```

The `default` branch in `toStatus` is the single permitted one: it guards against a row written by a newer deployment, not against a missing case in the sealed type. `discriminator` and `detail` have no `default`; a fourth `Status` fails compilation there.

## Boundary — `/microprofile-server`

```java
@Path("orders")
@Produces(MediaType.APPLICATION_JSON)
@Consumes(MediaType.APPLICATION_JSON)
public class OrdersResource {

    @Inject
    DataSource dataSource;

    @POST
    @Transactional
    public Response place(JsonObject json) {
        var order = Order.placed(json.getString("customer"), new BigDecimal(json.getString("total")));
        try (var connection = this.dataSource.getConnection()) {
            var saved = Orders.save(connection, order);
            return Response.created(URI.create("orders/" + saved.id())).build();
        } catch (SQLException e) {
            throw new InternalServerErrorException(e);
        } catch (IllegalArgumentException e) {
            throw new ClientErrorException(e.getMessage(), Response.Status.CONFLICT);
        }
    }

    @GET
    @Path("{id}")
    public Response find(@PathParam("id") long id) {
        try (var connection = this.dataSource.getConnection()) {
            return Orders.findById(connection, id)
                .map(Order::toJSON)
                .map(Response::ok)
                .orElseThrow(NotFoundException::new)
                .build();
        } catch (SQLException e) {
            throw new InternalServerErrorException(e);
        }
    }
}
```

`getConnection()` under `@Transactional` returns a connection enlisted in the JTA transaction; closing it returns it to the pool, the container commits. `toJSON` follows `/microprofile-server`'s JSON-P rule and lives on the record.

Configuration — `application.properties`:

```properties
quarkus.datasource.db-kind=postgresql
quarkus.flyway.migrate-at-start=true
```

Dev Services supplies the container in dev and test mode; production sets `quarkus.datasource.jdbc.url`, `username`, `password` through the environment.

## Boundary — `/java-cli-app`

```java
public interface OrdersCommand {

    static void place(String customer, BigDecimal total) {
        try (var connection = Database.connect()) {
            connection.setAutoCommit(false);
            try {
                var saved = Orders.save(connection, Order.placed(customer, total));
                connection.commit();
                IO.println("placed order " + saved.id());
            } catch (RuntimeException e) {
                connection.rollback();
                throw e;
            }
        } catch (SQLException e) {
            throw new IllegalStateException("database unavailable", e);
        }
    }
}
```

```java
public interface Database {

    static Connection connect() throws SQLException {
        return DriverManager.getConnection(
            ZCfg.string("db.url", "jdbc:h2:./zbo/shop"),
            ZCfg.string("db.user", "sa"),
            ZCfg.string("db.password", ""));
    }
}
```

`Database` is root-package plumbing in the `/bce` sense: one class, no business semantics. The `orders` BC applies its own DDL at startup:

```java
public interface Schema {

    String CREATE = """
        CREATE TABLE IF NOT EXISTS purchase_order (
            id            BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
            customer      VARCHAR(200)   NOT NULL,
            total         NUMERIC(12, 2) NOT NULL,
            status        VARCHAR(20)    NOT NULL,
            status_detail VARCHAR(400),
            placed_at     TIMESTAMP WITH TIME ZONE NOT NULL
        )
        """;

    static void create(Connection connection) {
        try (var statement = connection.createStatement()) {
            statement.execute(CREATE);
        } catch (SQLException e) {
            throw new IllegalStateException("schema creation failed", e);
        }
    }
}
```

Launch with the driver on the classpath:

```bash
java -cp "zbo/app.jar:lib/*" Main place Duke 42.00
```

## Test — `/zunit`

```java
import module java.sql;

void main() throws SQLException {
    try (var connection = DriverManager.getConnection("jdbc:h2:mem:orders;DB_CLOSE_DELAY=-1")) {
        Schema.create(connection);
        var saved = Orders.save(connection, Order.placed("Duke", new BigDecimal("42.00")));

        var found = Orders.findById(connection, saved.id()).orElseThrow();

        assert saved.equals(found) : "expected %s but got %s".formatted(saved, found);
    }
}
```

## Test — `@QuarkusTest`

```java
@QuarkusTest
class OrdersTest {

    @Inject
    DataSource dataSource;

    @Test
    void roundTripShippedOrder() throws SQLException {
        try (var connection = this.dataSource.getConnection()) {
            var saved = Orders.save(connection, Order.placed("Duke", new BigDecimal("42.00")));
            var shipped = saved.shipped(LocalDate.of(2026, 10, 30));
            Orders.update(connection, shipped);

            var found = Orders.findById(connection, saved.id()).orElseThrow();

            assertThat(found).isEqualTo(shipped);
        }
    }
}
```

One `isEqualTo` on the record verifies every column, the discriminator, and the detail round trip.
