# Installing airails.dev Skills

Instructions for AI agents asked to install, list, or remove the skills in this repository.

A skill is a directory containing a `SKILL.md`, organized under `bce/`, `java/`, `web/`, and `documentation/`. Installation means copying the skill directory — preserving its folder name — into the skills directory of the target agent.

## Skill Directories per Agent

| Agent | Skills directory |
|---|---|
| claude | `~/.claude/skills` |
| vibe | `~/.vibe/skills` |
| kiro | `~/.kiro/skills` |
| copilot | `~/.copilot/skills` |
| codex | `~/.agents/skills` |
| goose | `~/.config/goose/skills` |

## How to Install

Pick the first applicable option:

### 1. Interactive terminal available (user is driving)

Ask the user to run `./installSkills` from the repository root. Do **not** run it from an agent shell — it requires an interactive terminal (`System.console()`) and exits with `interactive terminal required` when stdin is piped or non-TTY.

### 2. Agent shell (no TTY)

Copy the skill directories directly. From the repository root:

```
mkdir -p ~/.claude/skills
find . -name SKILL.md -not -path './target/*' | while read f; do
    cp -R "$(dirname "$f")" ~/.claude/skills/
done
```

Replace `~/.claude/skills` with the target agent's directory from the table above. Copying overwrites an existing skill of the same name — acceptable, since this repository is the source of truth for these skills.

### 3. No JDK, no git

Download the per-agent zip from the rolling `skills` release and unzip into the home directory:

```
curl -fsSLO https://github.com/AdamBien/airails/releases/download/skills/airails-claude.zip
unzip -o airails-claude.zip -d ~
```

Each zip contains all skills with the agent's skills directory baked into the entry paths (`airails-claude.zip`, `airails-vibe.zip`, `airails-kiro.zip`, `airails-copilot.zip`, `airails-codex.zip`, `airails-goose.zip`).

## Listing and Removing

`./installSkills -l` needs no TTY — safe to run from an agent shell. It lists all skills available in the repository and all skills installed per agent directory (requires a JDK 25+).

`./installSkills -d <name>` is interactive; from an agent shell remove the skill directory directly instead:

```
rm -r ~/.claude/skills/<name>
```

## Verifying

After installation, confirm the skill directories exist and each contains a `SKILL.md`:

```
ls ~/.claude/skills/*/SKILL.md
```
