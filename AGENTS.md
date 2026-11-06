# Installing airails.dev Skills

Instructions for AI agents asked to install, list, or remove the skills in this repository.

A skill is a directory containing a `SKILL.md`, organized under `bce/`, `java/`, `web/`, `migrations/`, and `documentation/`. Installation means placing the skill directory — preserving its folder name — into the skills directory of the target agent, either globally (home directory) or project-locally (project root, checked in with the project).

## Skill Directories per Agent

| Agent | Global | Project-local |
|---|---|---|
| claude | `~/.claude/skills` | `.claude/skills` |
| vibe | `~/.vibe/skills` | `.vibe/skills` |
| kiro | `~/.kiro/skills` | `.kiro/skills` |
| copilot | `~/.copilot/skills` | `.github/skills` |
| codex | `~/.agents/skills` | `.agents/skills` |
| goose | `~/.config/goose/skills` | `.agents/skills` |

Project-local paths are relative to the project root. Note that Copilot's project directory is `.github/skills`, not `.copilot/skills`, and that Codex and Goose share `.agents/skills`.

## How to Install

Pick the first applicable option:

### 1. Released bundles with the wizard (JDK 25+, network access)

`airails` installs released bundles and needs no TTY when every answer is given as a flag. Pipe it from GitHub, or run `./airails` from a clone:

```
curl -fsSL https://raw.githubusercontent.com/AdamBien/airails/main/airails | java --source 25 /dev/stdin -project <project-root> -agents claude -bundles web,sbce -yes
curl -fsSL https://raw.githubusercontent.com/AdamBien/airails/main/airails | java --source 25 /dev/stdin -global -agents claude,codex -bundles skills -yes
```

- `-global` installs globally, `-project <project-root>` into a project; exactly one of them. Values take either form, `-project <root>` or `-project:<root>`.
- `-agents` takes comma-separated agent ids from the table above.
- `-bundles` takes comma-separated bundle names: `java-server`, `java-cli`, `web`, `migrations`, `sbce`, `documentation`, or `skills` for every skill. `airails -list` lists what the release currently offers.
- `-yes` skips the confirmation.

Prefer the bundle matching the project, since it keeps unrelated skills out of the agent's context. Without flags the wizard asks its questions on `/dev/tty`; in a shell without a terminal it exits with `no terminal for questions`. It installs the last release, not the working tree — for an unreleased change use option 2 or 3.

### 2. Working tree, interactive terminal available (user is driving)

Ask the user to run `./installSkills` from the repository root, or `./installSkills -p <project-root>` for a project-local install. It copies every skill and prompts per agent directory. Do **not** run it from an agent shell — it requires an interactive terminal (`System.console()`) and exits with `interactive terminal required` when stdin is piped or non-TTY.

### 3. Working tree, agent shell (no TTY)

Copy the skill directories directly. From the repository root:

```
mkdir -p ~/.claude/skills
find . -name SKILL.md -not -path './target/*' | while read f; do
    cp -R "$(dirname "$f")" ~/.claude/skills/
done
```

Replace `~/.claude/skills` with the target directory from the table above — global or project-local. Copying overwrites an existing skill of the same name — acceptable, since this repository is the source of truth for these skills.

### 4. No JDK

Download the per-agent zip from the rolling `skills` release and unzip into the home directory:

```
curl -fsSLO https://github.com/AdamBien/airails/releases/download/skills/airails-claude.zip
unzip -o airails-claude.zip -d ~
```

Each zip contains all skills with the agent's skills directory baked into the entry paths (`airails-claude.zip`, `airails-vibe.zip`, `airails-kiro.zip`, `airails-copilot.zip`, `airails-codex.zip`, `airails-goose.zip`).

The bundle zips (`airails-java-server.zip`, `airails-java-cli.zip`, `airails-web.zip`, `airails-migrations.zip`, `airails-sbce.zip`, `airails-documentation.zip`, `airails-skills.zip`) carry a plain `skills/` prefix and unzip into the agent's configuration directory, the parent of its skills directory — globally or in the project:

```
curl -fsSLO https://github.com/AdamBien/airails/releases/download/skills/airails-web.zip
unzip -o airails-web.zip -d ~/.claude
unzip -o airails-web.zip -d <project-root>/.claude
```

## Listing and Removing

`./installSkills -l` needs no TTY — safe to run from an agent shell. It lists all skills available in the repository and all skills installed per agent directory (requires a JDK 25+); with `-p <project-root>` it lists the project's agent directories instead.

`./installSkills -d <name>` (optionally with `-p <project-root>`) is interactive; from an agent shell remove the skill directory directly instead:

```
rm -r ~/.claude/skills/<name>
```

## Verifying

After installation, confirm the skill directories exist and each contains a `SKILL.md`, substituting the project-local directory for a project install:

```
ls ~/.claude/skills/*/SKILL.md
```

Skills that bundle executables in `scripts/` (e.g. `web-conventions/scripts/zws`) must keep the exec bit; `airails`, `installSkills`, the zips, and `cp -R` all preserve it:

```
ls -l ~/.claude/skills/*/scripts/
```
