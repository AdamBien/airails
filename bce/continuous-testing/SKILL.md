---
name: continuous-testing
description: Continuous test-driven development loop — after every code change, builds the project, starts the server, and runs Unit Tests, Integration Tests, and System Tests. Applies on top of microprofile-server skill. Use during development when you want full verification after each change. Triggers on "continuous testing", "continuous-testing", "test loop", "st-loop", or requests to run all tests after every change.
---

Continuous test-driven development loop using $ARGUMENTS. Apply all rules from `/microprofile-server`, plus the verification loop below.

## Server Lifecycle

- start Quarkus in dev mode (`mvn quarkus:dev`) once at the beginning of the session
- keep the server running across changes — Quarkus dev mode handles live reload automatically
- only restart the server on major errors (e.g., server crash, port conflict, unrecoverable startup failure)
- do not stop and restart the server between changes

## Verification Loop

After every code change, execute the following steps in order:

1. **Build & Unit Tests** — run `mvn test` in the service module to compile and execute unit tests
2. **Integration Tests** — run `mvn verify` in the service module to execute integration tests (failsafe plugin)
3. **System Tests** — before running, verify that the `-st` module's `pom.xml` includes the `maven-failsafe-plugin` configuration. Then run `mvn verify` in the `-st` module against the running server (already running in dev mode)

## Rules

- do not skip any step — every change triggers the full loop
- if unit tests fail, stop and fix before proceeding to integration tests
- if integration tests fail, stop and fix before proceeding to system tests
- if system tests fail, stop and fix before continuing with the next change
- report the outcome of each step before proceeding to the next
- treat compilation errors as a failed step — fix before continuing
- keep the loop running until all steps pass or the user intervenes
