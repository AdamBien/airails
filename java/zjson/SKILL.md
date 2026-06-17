---
name: zjson
description: Use JSON in zero-dependency Java by copying the org.json source (JSONObject, JSONArray, JSONTokener) directly into the project instead of adding a Maven/Gradle dependency. Use when adding JSON parsing or generation to a Java CLI script, zb project, or any project that must stay dependency-free. Triggers on "JSON in Java", "parse JSON", "generate JSON", "org.json", "zjson", "zjsoncp", "JSONObject", "JSONArray", or requests to read/write JSON without external libraries.
---

# zjson — JSON via Source Copy

Add JSON parsing and generation to a Java project by copying the `org.json` source files in — no Maven or Gradle dependency.

## What is zjson

A Java 25, zero-dependency fork of the `org.json` reference implementation, trimmed to three classes:

- `JSONObject` — JSON object: key/value map with typed accessors
- `JSONArray` — JSON array: ordered values, `Iterable<Object>`
- `JSONTokener` — the parser both classes delegate to

- Package: `org.json`
- Source: https://github.com/AdamBien/z-JSON-java (`src/main/java/org/json`)
- Requires: Java 21+ (developed against Java 25)

Not included in this fork: `JSONStringer`, `JSONWriter`, XML/CDL/HTTP/Cookie conversion. Use `JSONObject` and `JSONArray` for building output.

## Source Integration

Copy the `org/json` package into the project's source root — keep the `package org.json;` declarations unchanged. Unlike single-class utilities, the package is not repackaged; `org.json` stays as-is and is imported with `import org.json.*;`.

Copy the three files from GitHub, preserving the `org/json` path. This is the canonical method — it needs only `curl` and works on any machine, with no local checkout or extra tooling:

```bash
for f in JSONObject JSONArray JSONTokener; do
  curl -sf "https://raw.githubusercontent.com/AdamBien/z-JSON-java/main/src/main/java/org/json/$f.java" \
    -o "src/main/java/org/json/$f.java" --create-dirs
done
```

For a single-file zb/CLI project whose source root is the project root, drop the `src/main/java/` prefix and copy into `org/json/` directly. After copying, build normally (e.g. `/zb`). The three files compile alongside application code with no further configuration.

Optional local shortcut: if a `z-JSON-java` checkout and the `zjsoncp` script are present on the machine, running `zjsoncp` from the source root copies the same `org/json` package locally. This is a convenience only — the `curl` method above is the portable default.

## API

### Parsing

```java
import org.json.*;

JSONObject obj = new JSONObject("""
    {"name":"Duke","age":29,"tags":["java","jvm"],"active":true}
    """);

String name   = obj.getString("name");          // "Duke"
int age        = obj.getInt("age");              // 29
boolean active = obj.getBoolean("active");       // true
JSONArray tags = obj.getJSONArray("tags");       // ["java","jvm"]

JSONArray arr = new JSONArray("[1,2,3]");
int first = arr.getInt(0);                        // 1
```

### Building

`put` returns the same instance, so calls chain. Nest `JSONObject` and `JSONArray` freely:

```java
JSONObject speaker = new JSONObject()
        .put("name", "Duke")
        .put("age", 29)
        .put("active", true)
        .put("tags", new JSONArray().put("java").put("jvm"))
        .put("address", new JSONObject().put("city", "Helsinki"));

String compact = speaker.toString();        // single line
String pretty  = speaker.toString(2);       // indented by 2 spaces
```

### Typed Accessors

`get*` throws `JSONException` when the key/index is missing or the type mismatches. `opt*` returns `null`/`0`/`false` or a supplied default instead of throwing — prefer `opt*` with a default for optional fields.

| `JSONObject` (by `String` key) | `JSONArray` (by `int` index) | Returns |
|---|---|---|
| `get(key)` / `opt(key)` | `get(i)` / `opt(i)` | `Object` |
| `getString` / `optString(key[, def])` | `getString` / `optString(i[, def])` | `String` |
| `getInt` / `optInt(key[, def])` | `getInt` / `optInt(i[, def])` | `int` |
| `getLong` / `optLong` | `getLong` / `optLong` | `long` |
| `getDouble` / `optDouble` | `getDouble` / `optDouble` | `double` |
| `getBoolean` / `optBoolean` | `getBoolean` / `optBoolean` | `boolean` |
| `getJSONObject` / `optJSONObject` | `getJSONObject` / `optJSONObject` | `JSONObject` |
| `getJSONArray` / `optJSONArray` | `getJSONArray` / `optJSONArray` | `JSONArray` |

### Inspection & Conversion

```java
boolean has   = obj.has("name");
Set<String> keys = obj.keySet();
int size      = arr.length();

Map<String, Object> map  = obj.toMap();    // deep, nested JSON → Map/List
List<Object> list        = arr.toList();   // deep, nested JSON → List/Map

for (Object value : arr) {                  // JSONArray is Iterable
    // ...
}
```

### Constructors

```java
new JSONObject();                  // empty
new JSONObject(String json);       // parse
new JSONObject(Map<?,?> map);      // from a Java Map

new JSONArray();                   // empty
new JSONArray(String json);        // parse
new JSONArray(Collection<?> c);    // from a Java Collection
```

## Integration Rules

1. Copy the `org/json` package into the source root — never add a Maven/Gradle dependency.
2. Keep the `package org.json;` declarations unchanged; import with `import org.json.*;`.
3. Treat the copied files as vendored source — do not edit them; re-copy to update.
4. Use `opt*` accessors with defaults for optional fields; reserve `get*` for mandatory keys where a missing value should fail fast.
5. Build JSON with chained `put(...)` on `JSONObject`/`JSONArray`; use `toString(indentFactor)` for human-readable output.
6. In `microprofile-server` projects, prefer the platform JSON-P (`jakarta.json`) instead — zjson targets zero-dependency CLI scripts and zb projects.
