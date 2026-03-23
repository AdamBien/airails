# continuous-testing

An [AIrails.dev](https://airails.dev) skill that enforces a full verification loop after every code change in MicroProfile server projects.

## Scope

Applies on top of `/microprofile-server`. After each change, the loop executes:

1. Build & Unit Tests (`mvn test`)
2. Integration Tests (`mvn verify`)
3. Start the server
4. System Tests (`mvn verify` in the `-st` module)
5. Stop the server

Failures at any step halt the loop until fixed.

## Usage

Activate alongside a MicroProfile server project:

```
/continuous-testing create coffee BC
```
