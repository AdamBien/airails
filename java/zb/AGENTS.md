# Building Java Projects with zb

Use zb to build Java 21+ projects without external dependencies.

## When to Use

All three conditions must apply:

- Java project with exactly one `void main()` entry point
- No Maven/Gradle dependencies required
- Single-module application

## Build Procedure

### 1. Locate zb.jar

Search in this order — stop at the first match:

1. `zb.jar` in the current project root
2. Glob `**/zb.jar` within the project tree (prefer one inside a `zbo/` or `target/` directory)
3. Glob `**/zb.jar` one level above the project (sibling project directories)
4. `~/bin/zb.jar` or `~/.local/bin/zb.jar`

If none found, inform the user that zb.jar is missing and point them to https://github.com/AdamBien/zb

### 2. Find the correct build directory

zb must run from the directory that contains the `.zb` config file. In multi-module repos, each module has its own `.zb` file.

- Glob for `.zb` files in the project
- The build directory is the one containing `.zb` whose `sources.dir` points to your target sources
- If no `.zb` exists, use the directory containing `src/main/java`
- **Never run from the repo root of a multi-module project** — this causes "Multiple main classes found"

### 3. Run the build

```bash
cd <build-directory> && java -jar <absolute-path-to-zb.jar>
```

Always use an absolute path to `zb.jar` since you are cd-ing into the build directory.

### 4. Check the result

- Success: output shows "compiled N files" and exit code 0
- "Multiple main classes found": wrong directory — find the right module subdirectory
- Compilation errors: fix source code and rebuild

## Output

Executable JAR: `zbo/<jar-name>.jar` (jar name configured in `.zb` as `jar.file.name`)

## Configuration

A `.zb` file is auto-generated in the build directory on first run:

```properties
sources.dir=<discovered by zb>
resources.dir=<discovered by zb>
classes.dir=<temp.dir>
jar.dir=zbo/
jar.file.name=app.jar
```

- `<discovered by zb>` — zb uses its auto-detection logic
- `<temp.dir>` — temporary directory for compilation, deleted after JAR is built
