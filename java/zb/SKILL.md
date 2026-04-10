---
name: zb
description: Build Java 21+ projects with zb (Zero Dependencies Builder). Use when compiling, building, packaging, or running Java projects that have no external dependencies. Triggers on "build with zb", "zb build", "compile and package", "create executable JAR", or when a .zb configuration file is present in the project. Not for projects requiring Maven/Gradle dependency management.
---

# zb - Zero Dependencies Builder

Build single-module Java 21+ projects into executable JARs without Maven or Gradle.

## How to Build

### Step 1: Check for zb.sh script

Search for a `zb.sh` wrapper script in this order — stop at the first match:

1. `zb.sh` in the project root
2. Glob for `**/zb.sh` within the project directory tree
3. Glob for `**/zb.sh` one level above the project (sibling projects)
4. Check `~/bin/zb.sh` or `~/.local/bin/zb.sh`

**If `zb.sh` is found**, skip to Step 3 (zb.sh) — the script handles jar location and execution internally.

**If `zb.sh` is not found**, continue to Step 2 to locate `zb.jar` manually.

### Step 2: Find zb.jar (only if zb.sh not found)

Locate `zb.jar` in this order — stop at the first match:

1. `zb.jar` in the project root
2. Glob for `**/zb.jar` within the project directory tree
3. Glob for `**/zb.jar` one level above the project (sibling projects)
4. Check `~/bin/zb.jar` or `~/.local/bin/zb.jar`

If neither `zb.sh` nor `zb.jar` is found, tell the user to download from [github.com/AdamBien/zb](https://github.com/AdamBien/zb).

### Step 3: Find the build directory

zb must run from the directory containing the `.zb` config file (or where one should be generated). In multi-module repos, this is the module subdirectory, not the repo root.

- Glob for `.zb` files in the project
- `cd` into the directory containing the `.zb` file before running zb
- If no `.zb` exists, run from the directory that contains `src/main/java`

### Step 4: Run the build

**With zb.sh:**

```bash
cd <build-directory> && <path-to-zb.sh>
```

**With zb.jar (fallback):**

```bash
cd <build-directory> && java -jar <path-to-zb.jar>
```

### Step 5: Verify

Check exit code and output. A successful build prints the number of compiled files. Common errors:
- **"Multiple main classes found"** — you are in the wrong directory (likely the repo root instead of a module subdirectory)
- **Compilation errors** — fix the source and rebuild

## Source Directory Convention

zb auto-detects sources in order:

1. `src/main/java` (Maven convention, preferred)
2. `src/`
3. Current directory `.`

Resources auto-detected from `src/main/resources` or current directory.

## Entry Point Requirement

The project must have exactly one class with a `void main(` method (Java 21+ unnamed main).

## Configuration (.zb)

Optional `.zb` properties file in the build directory (auto-generated on first run):

```properties
sources.dir=<discovered by zb>
resources.dir=<discovered by zb>
classes.dir=<temp.dir>
jar.dir=zbo/
jar.file.name=app.jar
```

- `<discovered by zb>` — auto-detect directory
- `<temp.dir>` — use temporary directory, cleaned after build

## CLI Arguments (Positional)

```bash
java -jar zb.jar [sources] [classes] [jar_dir] [jar_file]
```

Precedence: CLI args > `.zb` file > defaults.

## Output

Default: `zbo/app.jar` — executable JAR with Main-Class manifest entry.

Run with:

```bash
java -jar zbo/app.jar
```

## Constraints

- Java 21+ required
- No external dependency support (no Maven/Gradle dependency resolution)
- Single-module projects only
- Exactly one main class per project

## After a Successful Build

If the project has a `test/` directory containing `*Test.java` files, ask the user if they want to run tests with `/zunit`.
