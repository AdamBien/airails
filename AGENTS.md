# Installing airails.dev Skills

Instructions for AI agents asked to install, list, or remove the skills in this repository.

A skill is a directory containing a `SKILL.md`, organized under `bce/`, `java/`, `web/`, and `documentation/`. Installation means copying the skill directory — preserving its folder name — into the skills directory of the target agent, either globally (home directory) or project-locally (project root, checked in with the project).

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

### 1. Interactive terminal available (user is driving)

Ask the user to run `./installSkills` from the repository root, or `./installSkills -p <project-root>` for a project-local install. Do **not** run it from an agent shell — it requires an interactive terminal (`System.console()`) and exits with `interactive terminal required` when stdin is piped or non-TTY.

### 2. Agent shell (no TTY)

Copy the skill directories directly. From the repository root:

```
mkdir -p ~/.claude/skills
find . -name SKILL.md -not -path './target/*' | while read f; do
    cp -R "$(dirname "$f")" ~/.claude/skills/
done
```

Replace `~/.claude/skills` with the target agent's directory from the table above. Copying overwrites an existing skill of the same name — acceptable, since this repository is the source of truth for these skills.

With network access, `downloadSkills` installs a bundle subset instead — it needs no TTY and extracts straight into the skills directory given with `-d` (default `~/.claude/skills`). From a clone, or piped without one:

```
./downloadSkills web sbce -d ~/.claude/skills
curl -fsSL https://raw.githubusercontent.com/AdamBien/airails/main/downloadSkills | java --source 25 /dev/stdin web sbce -d ~/.claude/skills
```

Bundles: `skills` (every skill), `java-server`, `java-cli`, `web`, `migrations`, `sbce`, `documentation`. `./downloadSkills -h` lists them.

### 3. No JDK, no git

Download the per-agent zip from the rolling `skills` release and unzip into the home directory:

```
curl -fsSLO https://github.com/AdamBien/airails/releases/download/skills/airails-claude.zip
unzip -o airails-claude.zip -d ~
```

Each zip contains all skills with the agent's skills directory baked into the entry paths (`airails-claude.zip`, `airails-vibe.zip`, `airails-kiro.zip`, `airails-copilot.zip`, `airails-codex.zip`, `airails-goose.zip`).

Agent-neutral bundles hold a per-stack subset under a plain `skills/` prefix and unzip into the agent's configuration directory (the parent of its skills directory) instead of the home directory: `airails-java-server.zip`, `airails-java-cli.zip`, `airails-web.zip`, `airails-migrations.zip`, plus the add-ons `airails-sbce.zip` (spec-driven workflow) and `airails-documentation.zip`. Prefer the bundle matching the project when one does, since it keeps unrelated skills out of the agent's context:

```
curl -fsSLO https://github.com/AdamBien/airails/releases/download/skills/airails-web.zip
unzip -o airails-web.zip -d ~/.claude
```

## Project-Local Installation

When the user asks to install skills into a project rather than for their user account, place them in the project-local directory of the agent from the table above, e.g. `.claude/skills` in the project root for Claude Code. A project skill takes precedence over a user-level skill of the same name, and a checked-in skills directory applies to everyone working on the repository. With a terminal, the user runs `./installSkills -p <project-root>`, which prompts per agent directory like the global install. From an agent shell, either copy single skill directories from this repository:

```
mkdir -p <project-root>/.claude/skills
cp -R web/web-conventions <project-root>/.claude/skills/
```

Or install a bundle into the project with `downloadSkills` (JDK) or `unzip` (no JDK):

```
cd <project-root>
./path/to/airails/downloadSkills web -d .claude/skills
```

```
cd <project-root>
curl -fsSLO https://github.com/AdamBien/airails/releases/download/skills/airails-web.zip
unzip -o airails-web.zip -d .claude
```

Verify with the same checks as below, substituting the project-local directory for `~/.claude/skills`; `./installSkills -l -p <project-root>` lists what a project has installed and needs no TTY.

## Listing and Removing

`./installSkills -l` needs no TTY — safe to run from an agent shell. It lists all skills available in the repository and all skills installed per agent directory (requires a JDK 25+); with `-p <project-root>` it lists the project's agent directories instead.

`./installSkills -d <name>` (optionally with `-p <project-root>`) is interactive; from an agent shell remove the skill directory directly instead:

```
rm -r ~/.claude/skills/<name>
```

## Verifying

After installation, confirm the skill directories exist and each contains a `SKILL.md`:

```
ls ~/.claude/skills/*/SKILL.md
```

Skills that bundle executables in `scripts/` (e.g. `web-conventions/scripts/zws`) must keep the exec bit; `installSkills`, the zips, and `cp -R` all preserve it:

```
ls -l ~/.claude/skills/*/scripts/
```
