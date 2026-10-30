---
name: memory-persistence
description: In-memory persistence for Java records in plain `java.util` collections, following Data-Oriented Programming — the record is the stored shape, a `ConcurrentHashMap` keyed by id is the table, queries are streams over its values. Same control contract as `/jdbc-persistence`, so a BC starts in memory and moves to JDBC by replacing one control class. Composes with `/bce` for placement, `/java-conventions` for style, and the stack skill — `/microprofile-server` (`@ApplicationScoped` CDI bean) or `/java-cli-app` (one instance created at startup). Use for prototypes, demos, the first `/sbce` convergence, test doubles for other BCs, or whenever asked for "in-memory", "memory persistence", "store in a map", "no database yet", "collection-backed", "fake repository", "stub persistence". Not for durable storage — use `/jdbc-persistence`.
argument-hint: "[BC or record to store in memory]"
---

Add in-memory persistence using $ARGUMENTS. Apply all rules below strictly; `/bce`, `/java-conventions`, and the stack skill hold wherever this skill is silent.

## Principle

The store is a collection of immutable records. No entity lifecycle, no identity map, no change tracking: a write replaces a record, a read returns the record, a query is a stream.

- records only — the stored value is the entity record itself, never a mutable copy or a wrapper
- the control is the sole holder of state; no other class references the collection
- the control's public contract is identical to the `/jdbc-persistence` control for the same BC: `save` returns the record with its id, `findById` returns `Optional`, `all` and queries return `List`; a BC migrates by swapping the control class and adding `@Transactional` (or the project's `@Boundary` stereotype) to the boundary
- state lives for the process lifetime and is lost on restart; say so in the BC's `package-info.java`, not in the code

## Placement (BCE)

- `entity` — the records and sealed interfaces, unchanged from `/jdbc-persistence`
- `control` — classes named after their responsibilities (`Customers`, `OrderReports`), never `*Repository`, `*Store`, `*InMemory*`; the storage technology does not appear in the name because the boundary must not know it; a control may hold more than one map when it owns more than one record type
- `boundary` — injects or receives the control; no collection, no stream over stored data

## Control

- storage: `Map<Long, Order> orders = new ConcurrentHashMap<>();` keyed by id; a `Set` or `List` only when the record has no identity and is never updated
- ids: `AtomicLong sequence = new AtomicLong();` — `save` assigns `sequence.incrementAndGet()` when the id is `0`, returns `order.withId(id)`, and stores the returned record
- update: `orders.replace(order.id(), order)`; a missing id is a `NoSuchElementException` from the control, never a silent insert
- delete: `orders.remove(id)` returning whether something was removed
- queries are pure streams over `orders.values()`: `.filter(...)`, `.sorted(...)`, `.toList()`; every predicate and comparator is a named method, never an inline lambda (`/java-conventions`)
- `all` returns `List.copyOf(orders.values())`; no caller sees the live collection or an `Iterator` over it
- constraints the schema would enforce are enforced here explicitly: a uniqueness rule is a `putIfAbsent` or a pre-check that throws `IllegalArgumentException` naming the rule, matching the `/jdbc-persistence` exception contract
- concurrency: rely on `ConcurrentHashMap` atomics (`compute`, `putIfAbsent`, `replace`); no `synchronized`, no `Collections.synchronizedMap`, no locks
- no eviction, no size cap, no TTL unless the spec asks; then prefer the rule in the control over a caching library
- `/microprofile-server`: the control is an `@ApplicationScoped` CDI bean; the collection is an instance field, so one store per application
- `/java-cli-app`: the control is a class instantiated once in the entry point and handed to the boundaries; never a mutable `static` field (`/java-conventions`)

## Snapshots (opt-in)

- durability across restarts is not this skill's job; when a demo needs it, ask first, then write the collection as JSON on shutdown and read it on startup through the records' `toJSON`/`fromJSON` (`/microprofile-server` JSON-P rule, or `/zjson` in a CLI) into a single file under the application's data directory
- a snapshot is a convenience, not a database: no partial writes, no concurrent readers, no schema evolution beyond what `fromJSON` tolerates

## Testing

- the control is a plain object: `new Orders()` in a unit test, no container, no `@QuarkusTest`
- `/microprofile-server`: JUnit 5 with AssertJ; `/java-cli-app`: `/zunit`
- the round trip is one assertion — `save` then `findById` yields an equal record
- a query test seeds three or four records and asserts the exact list with `containsExactly`
- other BCs that depend on this one use the real control in their tests instead of a mock; it is already in memory

## Checklist Before Finishing

- the collection is referenced in exactly one class
- no mutable `static` field
- no `synchronized`, no lock
- every returned collection is a copy or an immutable list
- the control's method signatures match what `/jdbc-persistence` would generate for the same records

## Shape (`/microprofile-server`)

```java
@ApplicationScoped
public class Orders {

    Map<Long, Order> orders = new ConcurrentHashMap<>();
    AtomicLong sequence = new AtomicLong();

    public Order save(Order order) {
        var saved = order.withId(this.sequence.incrementAndGet());
        this.orders.put(saved.id(), saved);
        return saved;
    }

    public void update(Order order) {
        if (this.orders.replace(order.id(), order) == null) {
            throw new NoSuchElementException("no order with id " + order.id());
        }
    }

    public Optional<Order> findById(long id) {
        return Optional.ofNullable(this.orders.get(id));
    }

    public List<Order> all() {
        return List.copyOf(this.orders.values());
    }

    public List<Order> openOrders() {
        return this.orders.values().stream()
            .filter(Orders::isOpen)
            .sorted(Comparator.comparing(Order::placedAt))
            .toList();
    }

    static boolean isOpen(Order order) {
        return order.status() instanceof Open;
    }
}
```

In a `/java-cli-app` the class loses the annotation and is created once in the entry point; everything else is identical.

## Composition

- `/bce` owns BC carving and layer placement; this skill only fixes where the collection lives
- `/java-conventions` owns syntax and style, including the ban on mutable statics and inline lambdas that shapes the control above
- `/jdbc-persistence` is the durable sibling; the two share the entity layer and the control contract, and this skill never introduces a method the JDBC control could not implement
- `/microprofile-server` and `/java-cli-app` own lifecycle and the test loop; `/showtime` and the first `/sbce apply` iteration are the usual callers
