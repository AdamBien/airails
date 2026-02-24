---
name: zcfg
description: Integrate zcfg (Zero Dependency Configuration Utility) into Java applications. Use when adding configuration loading, reading properties files, setting up application configuration, or integrating zcfg into a Java project. Triggers on "zcfg", "add configuration", "load properties", "application configuration with zcfg", or requests to use the zcfg library for configuration management.
---

Integrate zcfg into a Java application using $ARGUMENTS. Apply all rules below.

## What is zcfg

zcfg is a zero-dependency, single-class Java configuration loader. It reads standard Java properties files from multiple sources with defined precedence and provides type-safe access to configuration values.

- Source: https://github.com/AdamBien/zcfg
- Requires: Java 21+

## Source Integration

zcfg is integrated by copying the source file directly into the target project — no Maven dependency required.

1. Determine the target project's base package (e.g., `com.example.myapp`)
2. Create the file `ZCfg.java` in a `zcfg` sub-package under the base package (e.g., `src/main/java/com/example/myapp/zcfg/ZCfg.java` or `src/com/example/myapp/zcfg/ZCfg.java`)
3. Write the following source with the `package` declaration adjusted to match the target location:

```java
package <base-package>.zcfg;

import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.util.List;
import java.util.Properties;
import java.util.stream.Stream;

public class ZCfg {

    static final String PROPERTIES_FILE = "app.properties";
    static Properties CACHE;

    public static void load(String appName) {
        CACHE = loadProperties(appName);
    }

    static Properties loadProperties(String appName) {
        var properties = new Properties();
        var userHome = System.getProperty("user.home");
        var globalConfig = Path.of(userHome, "." + appName, PROPERTIES_FILE);
        if (Files.exists(globalConfig)) {
            loadFromFile(globalConfig, properties);
        }
        var localConfig = Path.of(PROPERTIES_FILE);
        if (Files.exists(localConfig)) {
            loadFromFile(localConfig, properties);
        }
        properties.putAll(System.getProperties());
        return properties;
    }

    static void loadFromFile(Path file, Properties properties) {
        try (var is = Files.newBufferedReader(file)) {
            properties.load(is);
        } catch (IOException e) {
            throw new IllegalStateException("Cannot load properties from: " + file, e);
        }
    }

    public static String string(String key) {
        if (CACHE == null)
            throw new IllegalStateException("Call ZCfg.load(appName) first");
        return CACHE.getProperty(key);
    }

    public static String string(String key, String defaultValue) {
        if (CACHE == null)
            throw new IllegalStateException("Call ZCfg.load(appName) first");
        return CACHE.getProperty(key, defaultValue);
    }

    public static int integer(String key, int defaultValue) {
        if (CACHE == null)
            throw new IllegalStateException("Call ZCfg.load(appName) first");
        var value = CACHE.getProperty(key);
        return value != null ? Integer.parseInt(value) : defaultValue;
    }

    public static boolean bool(String key, boolean defaultValue) {
        if (CACHE == null)
            throw new IllegalStateException("Call ZCfg.load(appName) first");
        var value = CACHE.getProperty(key);
        return value != null ? Boolean.parseBoolean(value) : defaultValue;
    }

    public static List<String> strings(String key) {
        if (CACHE == null)
            throw new IllegalStateException("Call ZCfg.load(appName) first");
        var value = CACHE.getProperty(key);
        if (value == null)
            return List.of();
        return split(value);
    }

    static List<String> split(String value) {
        var values = value.split(",");
        return Stream.of(values)
                .map(String::trim)
                .toList();
    }
}
```

Replace `<base-package>` with the actual target project package. The import in consuming classes becomes `import <base-package>.zcfg.ZCfg;`.

## Configuration Loading Order

`ZCfg.load("myapp")` merges properties from three sources, each overwriting the previous:

1. `$HOME/.myapp/app.properties` — global user configuration (`user.home` system property)
2. `./app.properties` — local project configuration (current working directory)
3. System properties via `-D` flags, e.g., `-Dserver.port=9090`

The app name passed to `load()` determines the global directory name: `load("myapp")` reads from `$HOME/.myapp/`.

## Properties File Format

Standard Java properties format:

```properties
server.port=8080
db.url=localhost:5432
db.timeout=30
debug.enabled=true
tags=web,api,backend
```

## API

### Initialization

Call once at application startup before accessing any values:

```java
ZCfg.load("myapp");
```

### Typed Accessors

| Method | Returns | Behavior |
|--------|---------|----------|
| `ZCfg.string(key)` | `String` | Returns value or `null` |
| `ZCfg.string(key, default)` | `String` | Returns value or default |
| `ZCfg.integer(key, default)` | `int` | Parses with `Integer.parseInt` |
| `ZCfg.bool(key, default)` | `boolean` | Parses with `Boolean.parseBoolean` |
| `ZCfg.strings(key)` | `List<String>` | Splits comma-separated values, trims whitespace |

All methods throw `IllegalStateException` if called before `load()`.

## Integration Example

```java
public class App {
    public static void main(String[] args) {
        ZCfg.load("myapp");

        var port = ZCfg.integer("server.port", 8080);
        var dbUrl = ZCfg.string("db.url", "localhost:5432");
        var debug = ZCfg.bool("debug.enabled", false);
        var tags = ZCfg.strings("tags");

        // use configuration values
    }
}
```

Override at runtime without changing files:

```bash
java -Dserver.port=9090 -Ddb.url=prod-db:5432 -jar myapp.jar
```

## Integration Rules

1. Copy `ZCfg.java` into a `zcfg` sub-package of the target project's base package — never add it as a Maven dependency
2. Adjust the `package` declaration to match the target location
3. Call `ZCfg.load()` exactly once, early in the application lifecycle (main method, `@Initialized(ApplicationScoped.class)` observer, or servlet context listener)
4. Use typed accessors with defaults for resilience — avoid `string(key)` without a default unless the key is mandatory
5. The app name should match the project/application name in kebab-case or lowercase (e.g., `"myapp"`, `"order-service"`)
6. For list values, use comma-separated format: `tags=web,api,backend`
7. Do not call `load()` multiple times — the configuration is cached statically
