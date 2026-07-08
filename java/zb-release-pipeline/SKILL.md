---
name: zb-release-pipeline
description: Generate a GitHub Actions pipeline that builds a zb (Zero Dependencies Builder) project and publishes a GitHub Release with the produced JAR. Use whenever the user wants CI/CD, a build pipeline, a release workflow, or GitHub Actions for a zb-based Java project — phrases like "set up GitHub Actions for this zb project", "add a zb release pipeline", "create a build workflow", "automate the zb build/release", or when a project has a .zb config and needs continuous builds/releases. Trigger even if the user doesn't say "zb" explicitly, as long as the project is a zb project (a .zb file is present and there is no Maven/Gradle build).
---

# zb GitHub Actions Pipeline

Generate `.github/workflows/release.yml` for a zb project. The pipeline checks out the
repo, installs Java, reads the project version, downloads the latest `zb.jar`, builds the
project, and creates a GitHub Release with the produced JAR attached and tagged
`v<version>.<run_number>`.

This skill produces a workflow equivalent to the reference below — only with the
project-specific paths and names auto-detected, nothing hardcoded:

```yaml
- name: Build
  working-directory: <module>
  run: java -jar ../zb.jar
- name: Create Release
  run: gh release create "v${{ steps.version.outputs.version }}.${{ github.run_number }}" ...
```

## Procedure

### 1. Locate the build directory

The build directory is where zb runs from — the directory that contains the project's
`.zb` config alongside the sources (`src/main/java` or `src/`).

- Glob for `.zb` files in the repository.
- If a repo has both a root `.zb` and a `.zb` in a module subdirectory, the **build
  directory is the one that also contains `version.txt` and the actual `src/` tree**.
  A multi-module-style layout (e.g. `zsmith/zsmith/`) builds from the inner module.
- Record `BUILD_DIR` as the path relative to the repository root. If the build directory
  *is* the repo root, `BUILD_DIR` is `.`.

### 2. Read the `.zb` config in the build directory

From the build directory's `.zb`, extract:

- `jar.file.name` (e.g. `zsmith.jar`) → this is the artifact file name.
- `jar.dir` (e.g. `zbo/`) → defaults to `zbo/` if absent.

Derive:

- `ARTIFACT_NAME` = `jar.file.name` without the `.jar` suffix (used in the release title).
- `ARTIFACT_PATH` = `<BUILD_DIR>/<jar.dir><jar.file.name>`, normalized (no `./` prefix,
  collapse `//`). Example: `zsmith/zbo/zsmith.jar`.

### 3. Resolve the version file

The version step does `cat <VERSION_FILE>` at the repo root.

- Look for `version.txt` in the build directory first, then the repo root.
- `VERSION_FILE` = that file's path relative to the repo root (e.g. `zsmith/version.txt`
  or `version.txt`).
- If no `version.txt` exists anywhere, tell the user one is required for the release tag
  and offer to create one (single line, e.g. `1.0.0`) in the build directory. Do not
  invent a different versioning scheme.

### 4. Compute the zb.jar reference

`zb.jar` is downloaded to the repository root. The Build step runs with
`working-directory: <BUILD_DIR>`, so the `java -jar` argument is the relative path from
the build directory back up to the root:

- `BUILD_DIR` is `.` (repo root) → `ZB_JAR_REF` = `zb.jar`
- `BUILD_DIR` is one level deep (e.g. `zsmith`) → `ZB_JAR_REF` = `../zb.jar`
- `n` levels deep → `n` repetitions of `../` then `zb.jar`

### 5. Java version

Default to `25` (matches the zb / Java 21+ requirement and the reference workflow). If the
project's `.zb` or build config pins a different release/source level, use that instead.
If unsure, ask the user; do not silently downgrade.

### 6. Generate the workflow

Read `assets/release.yml` (the template) and substitute:

| Placeholder         | Value                                              |
|---------------------|----------------------------------------------------|
| `__JAVA_VERSION__`  | resolved Java version (e.g. `25`)                  |
| `__VERSION_FILE__`  | `VERSION_FILE`                                      |
| `__BUILD_DIR__`     | `BUILD_DIR`                                         |
| `__ZB_JAR_REF__`    | `ZB_JAR_REF`                                        |
| `__ARTIFACT_PATH__` | `ARTIFACT_PATH`                                     |
| `__ARTIFACT_NAME__` | `ARTIFACT_NAME`                                     |

If `BUILD_DIR` is `.`, drop the `working-directory:` line entirely (a `working-directory`
of `.` is noise) rather than emitting `working-directory: .`.

Write the result to `.github/workflows/release.yml` in the repository root. Create
`.github/workflows/` if it does not exist. If `release.yml` already exists, show the diff
and confirm before overwriting.

### 7. Report

Summarize for the user:

- Detected build directory, artifact path, version file, Java version.
- The trigger: every push to `main` and manual `workflow_dispatch`.
- The release tag scheme: `v<version>.<github.run_number>`.
- Reminder that the workflow needs no extra secrets — it uses the built-in
  `GITHUB_TOKEN`, and `permissions: contents: write` (already in the template) is what
  lets it create releases.

Then offer to generate a launcher with the `/java-cli-script` skill: a single-file,
PATH-installed Java script that downloads the latest released JAR from the GitHub
Release and runs it, so the app is invoked by name instead of `java -jar <artifact>`.

## Notes

- Keep the workflow minimal. Do not add caching, matrices, or test steps unless the user
  asks — zb projects are zero-dependency and build in seconds, so the overhead is not
  worth it. Tests, if any, run via the zb post-build hook already.
- The pipeline always builds from the latest `zb.jar` release on purpose, so projects
  track zb improvements without manual bumps. Do not pin a zb version unless asked.
- This skill only generates the workflow; it does not run the build locally. To verify a
  zb build locally, use the `zb` skill.
