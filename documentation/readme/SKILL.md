---
name: readme
description: >
  Write and refine README.md files for software projects targeting advanced developers.
  Use when asked to create, write, generate, update, refine, or review a README or README.md file.
  Triggers on "create a README", "write README", "update README", "review README",
  "generate README.md", or when a README.md is being created or edited as part of a project.
  Not for API documentation, changelogs, or non-README markdown files.
---

# README Writing

## Style

- Target audience: advanced developers. Assume familiarity with tools, frameworks, and patterns.
- Use precise, concise language. Every sentence must convey information.
- Use imperative form for instructions (e.g., "Run the build" not "You can run the build").
- Avoid generic adjectives: "simple", "lightweight", "powerful", "easy-to-use", "robust".
- Avoid the term "Orchestrates" — use more specific alternatives (e.g., "coordinates", "delegates to", "calls").

## Structure

- Keep READMEs brief and to the point — favor clarity over completeness.
- Do NOT include detailed project structure (file/folder tree listings). High-level module descriptions are acceptable.
- Do NOT list REST resources or API endpoints in READMEs.
- If modules are listed, provide links to their directories or documentation.
- Use Mermaid for diagrams. GitHub renders Mermaid natively in markdown.

## Content

- Lead with what the project does in one or two sentences.
- Include only essential sections: purpose, prerequisites, build/run instructions, configuration (if non-obvious).
- Omit sections that add no value (e.g., "Contributing", "License" boilerplate) unless explicitly requested.

## Example skeleton

```markdown
# Project Name

One or two sentences: what it does and why.

## Prerequisites

Java 25+, Docker

## Build and Run

\`\`\`
mvn clean package
java -jar target/app.jar
\`\`\`

## Configuration

Only if non-obvious.
```
