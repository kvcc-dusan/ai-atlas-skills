# ai-atlas-skills

A curated collection of [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skills maintained by INOVA IT.

A skill is a set of instructions that changes how Claude Code approaches a specific class of task — enforcing a procedure, a constraint, or domain knowledge it would otherwise not apply consistently. Every skill in this repo is tested against a fixed evaluation process before it ships (see [docs/testing-methodology.md](docs/testing-methodology.md)).

## Skills

| Skill | Version | Description |
|---|---|---|
| [`figma-grounding`](skills/figma-grounding/SKILL.md) | 1.0.0 | Forces `get_metadata`/`get_variable_defs` extraction before writing any size, spacing, or color value from a Figma design — flags unresolved tokens instead of guessing from a screenshot. Invoked via the `/scrapedesign` command. |

## Repository structure

```
.claude-plugin/plugin.json   Plugin manifest — declares the skill set for future plugin-based install
skills/<name>/SKILL.md       One directory per skill; references/ holds supporting material
skills/_template/            Scaffold every new skill starts from
commands/                    Slash commands that invoke skills
docs/testing-methodology.md  Mandatory validation process for every skill
CHANGELOG.md                 Per-skill version history (semver)
```

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) (CLI, desktop app, or IDE extension)
- Skill-specific dependencies are listed in each skill's `SKILL.md` — `figma-grounding` requires the Figma MCP server to be connected.

## Installation

Installation is currently manual. Copy the skill directory and its matching slash command into your local Claude Code configuration:

```bash
cp -r skills/figma-grounding ~/.claude/skills/figma-grounding
cp commands/scrapedesign.md ~/.claude/commands/scrapedesign.md
```

Restart Claude Code (or start a new session) to pick up the skill and command.

The repository is already structured as a Claude Code plugin (`.claude-plugin/plugin.json`), so it can move to plugin-based installation later without restructuring.

## Versioning

Skills are versioned independently using [semver](https://semver.org/), tracked in [CHANGELOG.md](CHANGELOG.md). A breaking behavior change is a major bump, a new capability is minor, a wording or bug fix is patch.

## Contributing

New skills must follow the template, pass the full testing methodology, and document what was tested. See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

Not yet licensed — see [LICENSE](LICENSE). Internal use only until a license is decided. Do not redistribute.
