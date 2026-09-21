---
name: zhtmldb
description: Use zhtmldb, a zero-dependency single-file Java 25 CLI, as an agent's persistent datastore — notes, memory, task lists, logs, configuration, research findings, any key-value records that must survive the session. Records are XHTML pages in a folder, so the data is browsable, diffable with git and parseable as XML without a client library. Use whenever an agent needs to remember, store, look up, append to or track structured records across runs, when the user says "zhtmldb", "store this", "remember that", "log this", "keep a list of", "track these", "look up what we stored", or when a task needs lightweight persistence without a database server, JSON files or SQLite. Also use when a dedicated table CLI (a symlink such as `talks`, `notes`, `todos`) is on PATH.
argument-hint: "[what to store or look up, and in which table]"
---

Persist and query agent data with zhtmldb using $ARGUMENTS. Apply all rules below.

## What is zhtmldb

zhtmldb is a key-value datastore whose storage format is also its UI: every record is a valid XHTML page (`<table>/<key>.html`) holding its fields as `<dt>`/`<dd>` pairs, and every table folder plus the root carries a generated `index.html`. The database is a folder that a browser renders, `git diff` tracks per record and an XML parser reads. One executable script, no server, no libraries.

- Source: https://github.com/AdamBien/zhtmldb
- Requires: Java 25+ on PATH (`java -version`)

## Setup

Check for the tool before the first command; install it when missing:

```bash
command -v zhtmldb || {
  curl -O https://raw.githubusercontent.com/AdamBien/zhtmldb/main/zhtmldb
  chmod +x zhtmldb
  sudo cp zhtmldb /usr/local/bin/
}
zhtmldb -help
```

The database root is the current directory unless `db.dir` is set. Decide the root deliberately before the first write, so records do not land in whatever directory the shell happens to be in:

- Project data: run from the project root, or put `db.dir=<path>` into `./app.properties`.
- Agent memory across projects: `~/.zhtmldb/app.properties` with `db.dir=/path/to/memory`.
- One-off: `java -Ddb.dir=<path> --source 25 /usr/local/bin/zhtmldb <table> <command> ...`

Precedence is global file, then local file, then `-D`; the last one wins.

## Command Contract

```
zhtmldb <table> set <key> <field>=<value>...   store fields, merged into the record
zhtmldb <table> set <key> <field>+=<value>     append, newline separated
zhtmldb <table> set <key> <field>              value from stdin (<field>+ appends)
zhtmldb <table> set <key> "<v1,v2,...>"        positional by declared columns
zhtmldb <table> add <value>...                 new record under a timestamp key
zhtmldb <table> columns <name,name,...>        declare column order (once per table)
zhtmldb <table> get <key> [field]              field<TAB>value lines, or one value
zhtmldb <table> rm <key> [field]               remove a record, or one field
zhtmldb <table> keys                           sorted keys
zhtmldb <table> find <term>...                 key<TAB>label, for piping
zhtmldb <table> list [<term>...]               aligned table, for reading
zhtmldb tables
```

Terms: `<text>` matches any field value or the key, `<field>=<text>` one field, `<field>=` records having the field. All terms are case-insensitive substrings and every term has to match.

Names for tables, keys and fields: letters, digits, `_`, `-`. `index` is reserved. Values are arbitrary text, including newlines.

## Reading Output Correctly

- stdout carries data only. The version banner, `stored:`/`added:`/`removed:` confirmations and errors go to stderr, ANSI colored. Capture stdout for data and read stderr for the outcome.
- Exit code 0 on success, 1 on a missing table or key, no match, ambiguous key, or usage error. Check it rather than parsing error text.
- `keys`, `get` and `find` are tab separated: `cut -f1` gives bare keys, `cut -f2-` the value. `list` pads columns and cuts long values at 40 characters, so use it to read, never to parse.
- A `find` or `list` with terms that match nothing exits 1. An unfiltered `list` of an empty table exits 0 with no output.

## Rules

1. **Declare columns before `add`.** `add` needs a column order and fails without one. Run `zhtmldb <table> columns title,body` once; the order also drives positional `set` and how fields appear in `get`, `list` and the index label.
2. **Keys are abbreviated on `set`, `get` and `rm`.** An exact key wins; otherwise the argument is a substring of stored keys and has to match exactly one. Several matches exit 1 listing the candidates. On `set`, a new key that is a substring of an existing one updates that record instead of creating one, so check `keys` before choosing short new keys in a table with timestamp records.
3. **Append with `+=` for logs.** `field+=value` joins with a newline and works on a missing field too, so a `log` or `history` field grows without a read-modify-write.
4. **Pass long or multi-line values on stdin.** `echo "$text" | zhtmldb notes set n1 body` avoids shell quoting problems. `body+` appends.
5. **Values containing commas go through `add` or `field=value`,** never through positional CSV `set`, which splits on every comma.
6. **Look up with `find`, not `list`.** `find` prints `key<TAB>label` where the label is the first column's value, so a pipeline sees what a hit is about. `cut -f1` and feed the key to `get`.
7. **Read the pages directly when it is cheaper.** `<table>/<key>.html` is well-formed XML with the record in `<dl>`; `index.html` links every record by label. `grep -l` across a table folder is a valid query.
8. **Do not write pages by hand.** The CLI keeps every page well-formed, escapes values and regenerates both indexes atomically. A hand edit that breaks the XML makes the whole table unreadable to the tool.
9. **Do not store secrets.** Pages are plain text meant to be committed and browsed.

## Dedicated CLIs

A symlink named after a table is a preconfigured command for that table. It reads its own `~/.<name>/app.properties`, so each persona can own a database:

```bash
sudo ln -s /usr/local/bin/zhtmldb /usr/local/bin/notes
mkdir -p ~/.notes && echo "db.dir=$HOME/notes" > ~/.notes/app.properties
notes columns title,body
notes add "zhtmldb persona" "one symlink, one table, one database"
notes find persona | cut -f1
```

Prefer a persona whenever an agent works with one table repeatedly: the commands read as domain vocabulary and the database location travels with the name. Put the symlink on PATH, never inside the database root, where it would collide with the table folder of the same name.

## Recipes

Task tracking:

```bash
zhtmldb todos columns title,status,notes
zhtmldb todos add "Migrate config loading" open
zhtmldb todos list status=open
zhtmldb todos set 142122 status=done notes+="finished in commit a1b2c3"   # abbreviated timestamp key
```

Session memory that survives restarts:

```bash
zhtmldb memory set project-x stack="Java 25, zb" owner="platform team"
zhtmldb memory set project-x decisions+="2026-09-21: keep single-file layout"
zhtmldb memory get project-x decisions
```

Research findings with a searchable label:

```bash
zhtmldb findings columns claim,source,confidence
zhtmldb findings add "Virtual threads pin on synchronized" "JEP 444" high
zhtmldb findings find confidence=high | cut -f2-
```

## Verification

After writing, confirm the record from the tool's point of view, not by trusting the confirmation line:

```bash
zhtmldb <table> get <key>
xmllint --noout <table>/<key>.html      # optional: well-formedness
```

During experiments use a throwaway root (`cd "$(mktemp -d)"` or `-Ddb.dir`) so the user's data and the working tree stay untouched.
