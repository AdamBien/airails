# bce

An [AIrails.dev](https://airails.dev) skill defining the generic, composable [Boundary-Control-Entity (BCE/ECB)](https://bce.design) architecture pattern.

## Scope

- Core BCE/ECB layering rules
- Business component (BC) concept and composition
- Package / directory structure
- Cross-BC relationships and refactoring signals
- Naming conventions for BCs and classes

Technology-neutral. No language, framework, build, testing, or protocol rules — those come from composed skills.

## Composition

This skill is designed to be combined with stack-specific skills, e.g.:

- `microprofile-server` — MicroProfile / Jakarta EE servers
- `java-cli-app` — Java CLI applications
- `web-components` — frontend single-page applications
- `aws-cdk` — AWS CDK infrastructure stacks

Composed skills specialize BCE rules; they never contradict them.

## Usage

Activated automatically when scaffolding, writing, or reviewing code organized as business components with boundary, control, and entity layers. The rules in [SKILL.md](SKILL.md) guide structural decisions.
