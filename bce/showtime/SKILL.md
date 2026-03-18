---
name: showtime
description: Live coding mode — generates code without running tests, system tests, or builds. Applies on top of any project skill (microprofile-server, java-cli-app, java-cli-script, etc.). Use during live demos, workshops, or rapid prototyping sessions where speed matters over verification. Triggers on "showtime", "live coding", "demo mode", "skip tests", or requests to generate code quickly without test execution.
---

Live coding mode using $ARGUMENTS. Apply all rules from the active project skill, but skip all verification steps.

## What to Skip

- do not run unit tests
- do not run integration tests
- do not run system tests
- do not run builds (Maven, zb, or other)
- do not execute system tests after changes
- do not suggest running tests or builds

## What to Do

- generate code following the active project skill's architecture and coding rules
- create test files when asked, but do not execute them
- focus on fast, continuous code generation
- keep responses short — show code, not explanations
- on opening existing projects, load AGENTS.md (if present) before making changes
- always ask before changing pom.xml
