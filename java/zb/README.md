# zb

Agent instructions for building Java 21+ projects with [zb](https://github.com/AdamBien/zb) — a zero-dependency Java build tool.

## Usage

### AGENTS.md — per-project context

Download `AGENTS.md` into your Java project root. AI coding agents pick it up automatically and learn how to build the project with zb. Use this when you want every agent session in that project to know about zb without explicit prompting.

### SKILL.md — reusable skill across projects

Install `SKILL.md` into your IDE's skills directory. It registers a `/zb` slash command available across all projects. Use this when you want to invoke zb instructions on demand from any workspace.
