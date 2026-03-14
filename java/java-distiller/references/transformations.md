# Transformation Catalog

## 1. Syntax Modernization

### var for local variables

```java
// before
Map<String, List<Integer>> grouped = items.stream().collect(groupingBy(Item::category));

// after
var grouped = items.stream().collect(groupingBy(Item::category));
```

### Text blocks for multi-line strings

```java
// before
String json = "{\n" +
    "  \"name\": \"" + name + "\",\n" +
    "  \"age\": " + age + "\n" +
    "}";

// after
var json = """
    {"name": "%s", "age": %d}
    """.formatted(name, age);
```

### Switch expressions over if-else chains

```java
// before
String label;
if (status == 1) {
    label = "active";
} else if (status == 2) {
    label = "inactive";
} else {
    label = "unknown";
}

// after
var label = switch (status) {
    case 1 -> "active";
    case 2 -> "inactive";
    default -> "unknown";
};
```

### Module imports over individual imports

```java
// before
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;

// after
import module java.net.http;
```

### Instance main

```java
// before
public class App {
    public static void main(String[] args) {
        System.out.println("Hello");
    }
}

// after
void main() {
    IO.println("Hello");
}
```

## 2. API Upgrades

### java.time over Date/Calendar

```java
// before
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
Date date = sdf.parse("2025-01-15");
Calendar cal = Calendar.getInstance();
cal.setTime(date);
cal.add(Calendar.DAY_OF_MONTH, 7);
Date result = cal.getTime();

// after
var date = LocalDate.parse("2025-01-15");
var result = date.plusDays(7);
```

### java.nio.file over java.io.File

```java
// before
BufferedReader br = new BufferedReader(new FileReader("data.txt"));
StringBuilder sb = new StringBuilder();
String line;
while ((line = br.readLine()) != null) {
    sb.append(line).append("\n");
}
br.close();
String content = sb.toString();

// after
var content = Files.readString(Path.of("data.txt"));
```

### HttpClient over HttpURLConnection

```java
// before
URL url = new URL("https://api.example.com/data");
HttpURLConnection conn = (HttpURLConnection) url.openConnection();
conn.setRequestMethod("GET");
BufferedReader reader = new BufferedReader(new InputStreamReader(conn.getInputStream()));
String body = reader.lines().collect(Collectors.joining("\n"));
reader.close();

// after
var client = HttpClient.newHttpClient();
var request = HttpRequest.newBuilder(URI.create("https://api.example.com/data")).build();
var body = client.send(request, HttpResponse.BodyHandlers.ofString()).body();
```

### Immutable collection factories

```java
// before
List<String> items = new ArrayList<>();
items.add("a");
items.add("b");
items.add("c");
List<String> unmodifiable = Collections.unmodifiableList(items);

// after
var items = List.of("a", "b", "c");
```

### String.formatted over String.format

```java
// before
String msg = String.format("Hello %s, you have %d items", name, count);

// after
var msg = "Hello %s, you have %d items".formatted(name, count);
```

## 3. Pattern Adoption

### Records over boilerplate classes

```java
// before
public class Point {
    private final int x;
    private final int y;
    public Point(int x, int y) { this.x = x; this.y = y; }
    public int getX() { return x; }
    public int getY() { return y; }
    @Override public boolean equals(Object o) { ... }
    @Override public int hashCode() { return Objects.hash(x, y); }
    @Override public String toString() { return "Point[x=" + x + ", y=" + y + "]"; }
}

// after
record Point(int x, int y) {}
```

### Sealed interfaces over abstract class hierarchies

```java
// before
abstract class Shape { abstract double area(); }
class Circle extends Shape { ... }
class Rectangle extends Shape { ... }

// after
sealed interface Shape {
    record Circle(double radius) implements Shape {}
    record Rectangle(double width, double height) implements Shape {}
}
```

### Pattern matching instanceof

```java
// before
if (obj instanceof String) {
    String s = (String) obj;
    process(s.toLowerCase());
}

// after
if (obj instanceof String s) {
    process(s.toLowerCase());
}
```

### Pattern matching in switch

```java
// before
if (shape instanceof Circle) {
    Circle c = (Circle) shape;
    return Math.PI * c.radius() * c.radius();
} else if (shape instanceof Rectangle) {
    Rectangle r = (Rectangle) shape;
    return r.width() * r.height();
} else {
    throw new IllegalArgumentException("Unknown shape");
}

// after
return switch (shape) {
    case Circle c -> Math.PI * c.radius() * c.radius();
    case Rectangle r -> r.width() * r.height();
};
```

### Record patterns for deconstruction

```java
// before
if (obj instanceof Point p) {
    int x = p.x();
    int y = p.y();
    return x + y;
}

// after
if (obj instanceof Point(var x, var y)) {
    return x + y;
}
```

## 4. Functional Style

### Streams over loops

```java
// before
List<String> result = new ArrayList<>();
for (String name : names) {
    if (name.length() > 3) {
        result.add(name.toUpperCase());
    }
}

// after
var result = names.stream()
    .filter(name -> name.length() > 3)
    .map(String::toUpperCase)
    .toList();
```

### Method references over lambdas

```java
// before
list.forEach(item -> System.out.println(item));
list.stream().map(s -> s.toLowerCase()).toList();

// after
list.forEach(IO::println);
list.stream().map(String::toLowerCase).toList();
```

### Lambdas over anonymous inner classes

```java
// before
Comparator<String> comp = new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        return a.length() - b.length();
    }
};

// after
Comparator<String> comp = Comparator.comparingInt(String::length);
```

### Stream collectors

```java
// before
Map<String, List<Employee>> byDept = new HashMap<>();
for (Employee e : employees) {
    byDept.computeIfAbsent(e.department(), k -> new ArrayList<>()).add(e);
}

// after
var byDept = employees.stream().collect(groupingBy(Employee::department));
```

### Effectively final capture — hoist computation out of lambdas

```java
// before — "." + extension is recomputed for every element
files.stream()
    .filter(name -> name.endsWith("." + extension))
    .toList();

// after — computed once, captured as effectively final
var suffix = "." + extension;
files.stream()
    .filter(name -> name.endsWith(suffix))
    .toList();
```

### Optional over null checks

```java
// before
String value = map.get(key);
if (value != null) {
    return value.toUpperCase();
} else {
    return "DEFAULT";
}

// after
return Optional.ofNullable(map.get(key))
    .map(String::toUpperCase)
    .orElse("DEFAULT");
```

## 5. Concurrency Modernization

### Virtual threads over platform threads

```java
// before
ExecutorService executor = Executors.newFixedThreadPool(10);
for (String url : urls) {
    executor.submit(() -> fetch(url));
}
executor.shutdown();

// after
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    urls.forEach(url -> executor.submit(() -> fetch(url)));
}
```

### Structured concurrency

```java
// before
Future<String> user = executor.submit(() -> findUser(id));
Future<String> order = executor.submit(() -> fetchOrder(id));
try {
    return new Response(user.get(), order.get());
} catch (Exception e) {
    // cleanup is manual and error-prone
}

// after
try (var scope = StructuredTaskScope.open()) {
    var user = scope.fork(() -> findUser(id));
    var order = scope.fork(() -> fetchOrder(id));
    scope.join();
    return new Response(user.get(), order.get());
}
```

### Scoped values over thread-local

```java
// before
static final ThreadLocal<User> CURRENT_USER = new ThreadLocal<>();
CURRENT_USER.set(user);
try {
    process();
} finally {
    CURRENT_USER.remove();
}

// after
static final ScopedValue<User> CURRENT_USER = ScopedValue.newInstance();
ScopedValue.where(CURRENT_USER, user).run(() -> process());
```

## 6. Structural Simplification

### try-with-resources

```java
// before
InputStream in = null;
try {
    in = new FileInputStream("data.bin");
    // use in
} finally {
    if (in != null) {
        in.close();
    }
}

// after
try (var in = Files.newInputStream(Path.of("data.bin"))) {
    // use in
}
```

### Remove redundant modifiers

```java
// before
public interface Service {
    public abstract void process();
    public static final String NAME = "svc";
}

// after
interface Service {
    void process();
    String NAME = "svc";
}
```

### Inline trivial getters/setters when records fit

When a class is purely a data carrier with no mutable state or custom behavior — replace with a record. Do not force records onto classes with mutable state or complex logic.

### Remove unnecessary boxing

```java
// before
Integer count = Integer.valueOf(42);
if (count.intValue() > 0) { ... }

// after
int count = 42;
if (count > 0) { ... }
```

### Sequenced collections

```java
// before
var first = list.get(0);
var last = list.get(list.size() - 1);

// after
var first = list.getFirst();
var last = list.getLast();
```
