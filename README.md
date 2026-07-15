# airails.dev

Home of Spec-Driven BCE ([**sbce.space**](https://sbce.space)) — development conventions, [SKILLS.md](https://agentskills.io/specification), and [AGENTS.md](https://agents.md) for building modern Java and web applications.

## Installation

Download the repository without `git`, then run the installer:

```
curl -fsSL https://raw.githubusercontent.com/AdamBien/airails/main/downloadSkills | java --source 25 /dev/stdin
cd airails
./installSkills
```

Or clone it with `git` instead of the first step:

```
git clone https://github.com/AdamBien/airails
cd airails
./installSkills
```

The installer finds all skills and prompts before copying to each supported agent directory (`~/.claude/skills`, `~/.vibe/skills`, `~/.kiro/skills`, `~/.copilot/skills`, `~/.agents/skills`, `~/.config/goose/skills`).

Without a JDK or `git`, download the prepacked zip for your agent from the rolling [skills release](https://github.com/AdamBien/airails/releases/tag/skills) and unzip it into the home directory:

```
curl -fsSLO https://github.com/AdamBien/airails/releases/download/skills/airails-claude.zip
unzip -o airails-claude.zip -d ~
```

Each zip contains all skills with the agent's skills directory baked into the entry paths: `airails-claude.zip`, `airails-vibe.zip`, `airails-kiro.zip`, `airails-copilot.zip`, `airails-codex.zip`, `airails-goose.zip`.

AI agents installing the skills themselves should follow [AGENTS.md](AGENTS.md) — `installSkills` requires an interactive terminal.

Manage installed skills:

```
./installSkills -l           # list available and installed skills
./installSkills -d <name>    # delete a skill by name
./installSkills -h           # show help
```

## What's Inside

### Spec-Driven BCE (SBCE)

One capability spec equals one business component, and the spec is the boundary contract ([sbce.space](https://sbce.space)). Declare the capability, then converge the code against the stack's own test loop.

- [**sbce**](bce/sbce) — The workflow: `new` decomposes a feature into capability specs, `apply` converges code to spec; composes with bce and a stack skill (java-cli-app, microprofile-server, web-components)
- [**ears-tests**](bce/ears-tests) — Parameterized (table-driven) tests generated from the EARS requirement statements in a capability spec, one labeled row per requirement id

### Java

- [**java-conventions**](java/java-conventions) — Composable Java 25 code conventions: modern syntax, naming, visibility, streams, and exceptions across all Java contexts
- [**java-cli-script**](java/java-cli-script) — Zero-dependency, single-file executable Java scripts for system-wide use via PATH
- [**java-distiller**](java/java-distiller) — Simplifies, modernizes, and refactors existing code into idiomatic Java 25
- [**python-to-java**](java/python-to-java) — Converts Python scripts to zero-dependency Java 25 CLI programs
- [**enterprisifier**](java/enterprisifier) — Deliberately overengineers code with maximum patterns, indirections, and abstractions, for comedic or educational purposes
- [**zb**](java/zb) — Zero Dependencies Builder for compiling, building, and packaging Java 21+ projects
- [**zb-release-pipeline**](java/zb-release-pipeline) — GitHub Actions pipeline that builds a zb project and publishes the JAR as a GitHub Release
- [**zargs**](java/zargs) — Zero-dependency, enum-based argument parsing for Java CLI applications
- [**zcfg**](java/zcfg) — Zero Dependency Configuration Utility for loading properties and application configuration
- [**zcl**](java/zcl) — Zero-dependency Colour Logger for colored terminal output in Java applications
- [**zjson**](java/zjson) — JSON parsing and generation by copying the org.json source into the project, no Maven/Gradle dependency
- [**zunit**](java/zunit) — Generates and runs zunit tests for java-cli-app projects

### Web

- [**web-conventions**](web/web-conventions) — Composable baseline for all web frontends: semantic HTML, accessibility, design tokens, and Baseline browser-support policy
- [**web-static**](web/web-static) — Modern static websites using semantic HTML and CSS without external dependencies or build systems; composes web-conventions
- [**web-components**](web/web-components) — Single-page applications using web components, BCE layering, lit-html, Redux Toolkit, and client-side routing; composes web-conventions
- [**web-latest**](web/web-latest) — Modifier for experiments and PoCs that lifts the Baseline browser-support policy: newest web platform features without fallbacks, with a declared support floor; composes on top of web-static or web-components
- [**web-performance-reviewer**](web/web-performance-reviewer) — Opt-in performance review of the rendered site through Chrome DevTools MCP: throttled traces, Core Web Vitals thresholds, network waterfall, heap-snapshot leak checks for SPAs; never part of the verification loop; composes on top of web-static or web-components

### BCE

- [**bce**](bce/bce) — Composable, technology-neutral architecture rules for the Boundary-Control-Entity pattern: business components, layer responsibilities, and package structure
- [**java-cli-app**](bce/java-cli-app) — Multi-file Java 25 CLI applications packaged as executable JARs with zb
- [**microprofile-server**](bce/microprofile-server) — Architecture and coding conventions for long-running MicroProfile/Jakarta EE server applications using BCE pattern
- [**continuous-testing**](bce/continuous-testing) — Test-driven loop on top of microprofile-server: builds, starts the server, and runs Unit, Integration, and System Tests after every change
- [**showtime**](bce/showtime) — Live coding mode on top of any project skill: generates code without running tests or builds, for demos and workshops

### Documentation

- [**bce-diagrams**](documentation/bce-diagrams) — High-level overview diagrams showing interactions between business components, subsystems, and services
- [**drawio**](documentation/drawio) — Draw.io overview diagrams with BCE shape mapping and consistent visual style
- [**mermaid**](documentation/mermaid) — Mermaid overview diagrams for architecture and component visualization
- [**readme**](documentation/readme) — Conventions for writing README.md files targeting advanced developers

### Quickstarters

- [**quarkus-microprofile**](https://github.com/AdamBien/quarkus-microprofile) — Quarkus MicroProfile template with BCE architecture, REST endpoints, CDI, and System Tests
- [**bce.design**](https://github.com/AdamBien/bce.design) — Web Components starter with lit-html, Redux Toolkit, and standards-based routing (Navigation API + URLPattern)
- [**ebank**](https://github.com/AdamBien/ebank) — CRUD with Quarkus, JPA, and PostgreSQL
- [**java-cli-app**](https://github.com/AdamBien/java-cli-app) — Template for building zero-dependency Java CLI applications with zb
- [**zeeds**](https://github.com/AdamBien/zeeds) — Zero-dependency Java 25+ source files as starting points, run directly with `java filename.java`