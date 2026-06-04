# java-cli-script

A Skill for creating zero-dependency, executable Java 25 scripts for system-wide use via PATH — no build tool, no `.java` extension, just a shebang and `chmod +x`.

Scripts are installed in a PATH directory (e.g., `/usr/local/bin`) and invoked like any shell command, leveraging [JEP 458: Launch Multi-File Source-Code Programs](https://openjdk.org/jeps/458).

Composes with [`/java-conventions`](../java-conventions) for modern Java 25 code style, naming, and structure rules.
