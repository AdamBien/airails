# Python to Java 25 Idiom Mapping

## Argument Parsing

```python
# Python: argparse
parser = argparse.ArgumentParser(description="...")
parser.add_argument("--name", required=True)
parser.add_argument("--verbose", action="store_true")
args = parser.parse_args()
```

```java
// Java: switch loop (for <6 flags, otherwise use /zargs)
String name = null;
boolean verbose = false;
for (int i = 0; i < args.length; i++) {
    switch (args[i]) {
        case "--name" -> name = args[++i];
        case "--verbose" -> verbose = true;
    }
}
```

## File I/O

| Python | Java |
|---|---|
| `Path("file.txt")` | `Path.of("file.txt")` |
| `path.read_text()` | `Files.readString(path)` |
| `path.write_text(s)` | `Files.writeString(path, s)` |
| `path.exists()` | `Files.exists(path)` |
| `path.is_dir()` | `Files.isDirectory(path)` |
| `path.mkdir(parents=True, exist_ok=True)` | `Files.createDirectories(path)` |
| `path.rglob("*.json")` | `Files.walk(path).filter(p -> p.toString().endsWith(".json"))` |
| `path.parent` | `path.getParent()` |
| `path.name` | `path.getFileName().toString()` |
| `path / "sub"` | `path.resolve("sub")` |
| `path.relative_to(base)` | `base.relativize(path)` |
| `path.resolve()` | `path.toAbsolutePath()` |

## Data Structures

| Python | Java |
|---|---|
| `dict` | `LinkedHashMap<>` (preserves order) |
| `list` | `ArrayList<>` or `List.of()` |
| `set` | `Set.of()` or `HashSet<>` |
| `tuple` | `record` |
| `@dataclass` | `record` |
| `NamedTuple` | `record` |
| `dict.get(key, default)` | `map.getOrDefault(key, default)` |
| `key in dict` | `map.containsKey(key)` |
| `[x for x in list if cond]` | `list.stream().filter(cond).toList()` |

## String Operations

| Python | Java |
|---|---|
| `f"text {var}"` | `"text %s".formatted(var)` |
| `s.strip()` | `s.strip()` |
| `s.startswith("x")` | `s.startsWith("x")` |
| `s.split("\n")` | `s.split("\n")` |
| `"\n".join(list)` | `String.join("\n", list)` |
| `re.match(pattern, s)` | `Pattern.compile(pattern).matcher(s).find()` |
| `re.search(r"<tag>(.*?)</tag>", s, re.DOTALL)` | `Pattern.compile("<tag>(.*?)</tag>", Pattern.DOTALL).matcher(s)` |
| `html.escape(s)` | `s.replace("&","&amp;").replace("<","&lt;").replace(">","&gt;")` |

## Subprocess / Process Management

```python
# Python: subprocess.run
result = subprocess.run(["cmd", "arg"], capture_output=True, text=True, timeout=30)
result.stdout
result.returncode
```

```java
// Java: ProcessBuilder
var process = new ProcessBuilder("cmd", "arg").start();
var stdout = new String(process.getInputStream().readAllBytes());
int exitCode = process.waitFor();
```

```python
# Python: subprocess.Popen with streaming
process = subprocess.Popen(cmd, stdout=subprocess.PIPE)
for line in process.stdout:
    ...
```

```java
// Java: ProcessBuilder with BufferedReader
var process = new ProcessBuilder(cmd).start();
var reader = new BufferedReader(new InputStreamReader(process.getInputStream()));
String line;
while ((line = reader.readLine()) != null) { ... }
```

```python
# Python: environment filtering
env = {k: v for k, v in os.environ.items() if k != "KEY"}
```

```java
// Java: environment filtering
var pb = new ProcessBuilder(cmd);
pb.environment().remove("KEY");
```

## Concurrency

```python
# Python: ProcessPoolExecutor
with ProcessPoolExecutor(max_workers=10) as executor:
    futures = {executor.submit(func, arg): arg for arg in items}
    for future in as_completed(futures):
        result = future.result()
```

```java
// Java: virtual threads + fixed thread pool
try (var executor = Executors.newFixedThreadPool(numWorkers)) {
    var futures = new LinkedHashMap<Future<Boolean>, String>();
    for (var item : items) {
        futures.put(executor.submit(() -> func(item)), item);
    }
    for (var entry : futures.entrySet()) {
        var result = entry.getKey().get();
    }
}
```

For I/O-bound work with timeouts:

```java
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    var future = executor.submit(() -> doWork());
    var result = future.get(timeout, TimeUnit.SECONDS);
}
```

## Zip Files

```python
# Python
with zipfile.ZipFile(name, 'w', zipfile.ZIP_DEFLATED) as zipf:
    zipf.write(file_path, arcname)
```

```java
// Java
try (var zos = new ZipOutputStream(Files.newOutputStream(path))) {
    zos.putNextEntry(new ZipEntry(arcName));
    Files.copy(filePath, zos);
    zos.closeEntry();
}
```

## Browser / Desktop

```python
webbrowser.open(url)
```

```java
// Use fully qualified name to avoid java.awt.List conflict
java.awt.Desktop.getDesktop().browse(URI.create(url));
```

## Temporary Files

```python
import tempfile
Path(tempfile.gettempdir()) / "file.txt"
```

```java
Files.createTempFile("prefix", ".txt")
```

## Printing

| Python | Java |
|---|---|
| `print(msg)` | `IO.println(msg)` |
| `print(msg, file=sys.stderr)` | `System.err.println(msg)` |
| `sys.exit(1)` | `System.exit(1)` |
| `sys.stdin.read()` | `new String(System.in.readAllBytes())` |

## Common Patterns

| Python | Java |
|---|---|
| `if __name__ == "__main__":` | `void main(String... args)` |
| `from pathlib import Path` | (automatic in java.base) |
| `from typing import Optional` | (not needed — use nullable types) |
| `isinstance(x, str)` | `x instanceof String s` (pattern matching) |
| `match x: case ...` | `switch (x) { case ... -> }` |
| `lambda x: x + 1` | `x -> x + 1` |
| `sorted(list, key=func)` | `list.stream().sorted(Comparator.comparing(func)).toList()` |
| `max(list, key=func)` | `list.stream().max(Comparator.comparing(func))` |
| `sum(x for x in list)` | `list.stream().mapToInt(...).sum()` |
| `any(cond for x in list)` | `list.stream().anyMatch(cond)` |
| `random.shuffle(list)` | `Collections.shuffle(list, rng)` |
| `uuid.uuid4().hex[:8]` | `UUID.randomUUID().toString().substring(0, 8)` |
| `datetime.now(timezone.utc)` | `java.time.Instant.now()` |
| `time.time()` | `System.currentTimeMillis() / 1000.0` |
