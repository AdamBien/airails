---
name: zllm-scripts
description: Use the zllm-scripts commands (files, search, show, outline, imports, clip, paste, parseFrontmatter, quickValidate) instead of OS shell commands (find, ls, grep, rg, cat, head, tail, sed -n, wc -l, grep for Markdown headings, pbcopy, pbpaste, xclip, grep for import statements) whenever files are listed, searched, or read, or the clipboard is used, through a shell. The scripts are zero-dependency single-file Java 25 executables. They are read-only, stay inside the project root, refuse secrets, and cap their output. Use before running any of those OS commands, when another skill says to search or read files safely, when it composes /zllm-scripts, or when the user mentions "zllm", "zllm-scripts", "safe search", "clip" or "paste". Not for writing new scripts — use java-cli-script; not for writing files, git, builds, or network access.
argument-hint: "[the file, search, or clipboard step to perform]"
---

Perform the file and clipboard steps of $ARGUMENTS with the zllm-scripts commands instead of OS shell commands. Apply all rules below.

- Source: https://github.com/AdamBien/zllm-scripts
- Requires: Java 25+ on PATH

## Why

OS commands have no limits: `cat ~/.ssh/id_rsa`, `grep -r token /`, or `find /` with megabytes of output. The zllm-scripts can't do any of that:

- **read-only:** no writes, no network, no spawned processes
- **confined to a root:** the nearest directory containing `.git`, otherwise the current directory; paths and symlinks leaving it are refused
- **secrets refused:** `.env`, `*.pem`, `*.key`, `.npmrc`, `id_rsa*` and `.git/` are never read
- **bounded output:** results are capped, long lines are cut, and stderr reports what was left out and how to continue
- **one contract:** the same exit codes and options in every script, so one permission rule per script (`Bash(search:*)`) replaces many per-command approvals

## Setup

Check once per session; install when missing:

```bash
command -v files search show outline imports clip paste || {
  git clone https://github.com/AdamBien/zllm-scripts && cd zllm-scripts
  chmod +x filesystem/* javascript/* markdown/* clipboard/* skills/*
  sudo cp filesystem/* javascript/* markdown/* clipboard/* skills/* /usr/local/bin/
}
```

When installation is not possible, use the OS command and state that the scripts are missing.

## Command Mapping

| Instead of | Use |
|---|---|
| `find . -name '*.js'`, `ls -R`, `fd` | `files '*.js'` |
| `find . -type d` | `files -type d` |
| `ls -l`, `wc -l` on many files | `files -long '*.java'` |
| `find … \| wc -l` | `files -count '*.ts'` |
| `grep -rn pattern`, `rg pattern` | `search pattern` |
| `grep -rn --include='*.ts' pattern src` | `search pattern src -glob '*.ts'` |
| `grep -rl`, `grep -c`, `grep -i`, `grep -F`, `grep -C 3` | `search -files`, `-count`, `-i`, `-fixed`, `-context 3` |
| `cat file`, `head -50 file` | `show file`, `show file -max 50` |
| `sed -n '120,180p' file`, `tail -n +120 file`, `sed -n '47,48p;60,70p' file` | `show file -lines 120-180`, `show file -lines 120-`, `show file -lines 47-48,60-70` |
| `grep -n '^#' file`, `grep -n '^[0-9]*\. \*\*' file \| sed \| cut` for a table of contents | `outline file` (headings), `outline file -list` (plus titled list items), `outline file -depth 2` |
| `grep -rn pattern <directory outside the project>` | `cd <directory> && search pattern` — the root becomes that directory or its repository |
| `grep -rn "import .* from"`, `grep -rho "from '[^']*'"`, `perl -0ne` over `import … from`, `require(` | `imports`, `imports -external -count -package` (external libraries with import and file counts), `imports -names` (imported names, import-map targets) |
| `pbcopy`, `xclip -selection clipboard` | `clip` |
| `pbpaste`, `xclip -o` | `paste` |
| parsing `SKILL.md` frontmatter with `sed`/`awk` | `parseFrontmatter <skill-dir>`, `quickValidate <skill-dir>` |

Globs without `/` match the file name (`'*.{ts,tsx}'`, `package.json`). Globs with `/` match the root-relative path (`'src/**/test/*.js'`). Quote globs so the shell does not expand them.

## Reading Output Correctly

- **First stdout line:** `<name> <version> root=<path>`. Skip it when parsing or piping.
- **Exit codes** (`files`, `search`, `show`, `outline`): `0` found, `1` nothing found, `2` usage error, `3` denied. Exit 1 is an answer, not a failure.
- **Truncation:** read stderr. `show` names the next command (`show file -lines 181-`); `files` and `search` report `output stopped at -max N`. Narrow the query, or raise `-max` deliberately.
- **Skips:** stderr counts skipped entries — ignored directories (`node_modules`, `dist`, `target`, ...), denied files, binaries, files over 1 MiB. Repeat with `-all` when the answer may be in an ignored directory.

## Rules

- **A denial is final.** Exit 3 means the path is outside the root or may contain secrets. Never retry it with `cat`, `grep`, or the harness's read tool. Report the denial and let the user decide.
- **Use the scripts for one-off lookups too.** Consistent output and one allowlist entry per script outweigh familiarity with the OS command.
- **Fall back to OS commands only where no script covers the need:** writing or moving files, `git`, builds and tests, process or network inspection, and searching outside the root at the user's explicit request. State the reason for the fallback.
- **Shell commands only.** Where the harness provides dedicated read or search tools, those remain valid unless the composing skill says otherwise.

## Composition

- **java-cli-script:** every zllm-script is a java-cli-script; use it to write a missing one.
- **Composing skills** reference this skill instead of repeating the rules, e.g. `Search and read files per /zllm-scripts.` The composing skill keeps its workflow; this skill only decides which command performs each file or clipboard step.
