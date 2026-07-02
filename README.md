# ai-atlas-skills

A curated collection of [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skills, built and maintained by [Dušan Kovačić](https://github.com/kvcc-dusan) for INOVA IT workflows.

A skill is a set of instructions that changes how Claude Code approaches a specific class of task — enforcing a procedure, a constraint, or domain knowledge it would otherwise not apply consistently. Every skill in this repo is tested against a fixed evaluation process before it ships (see [docs/testing-methodology.md](docs/testing-methodology.md)).

## Skills

| Skill | Version | Description |
|---|---|---|
| [`figma-grounding`](skills/figma-grounding/SKILL.md) | 1.0.0 | Forces `get_metadata`/`get_variable_defs` extraction before writing any size, spacing, or color value from a Figma design — flags unresolved tokens instead of guessing from a screenshot. Invoked via the `/scrapedesign` command. |

## Repository structure

```
.claude-plugin/plugin.json       Plugin manifest — name/version/metadata
.claude-plugin/marketplace.json  Marketplace catalog — makes this repo installable via /plugin install
skills/<name>/SKILL.md           One directory per skill; references/ holds supporting material
skills/_template/                Scaffold every new skill starts from
commands/                        Slash commands that invoke skills — Claude Code auto-discovers these,
                                  same as skills/, no manifest entry needed
docs/testing-methodology.md      Mandatory validation process for every skill
CHANGELOG.md                     Per-skill version history (semver)
```

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) (CLI, desktop app, or IDE extension)
- Skill-specific dependencies are listed in each skill's `SKILL.md` — `figma-grounding` requires the Figma MCP server to be connected.

## Installation

Two ways to get skills from this repo into your own Claude Code setup. Both work; pick whichever fits.

### Option A — Plugin install (recommended)

```
/plugin marketplace add kvcc-dusan/ai-atlas-skills
/plugin install ai-atlas-skills@kvcc-dusan
```

Installs every skill and its matching command together — no risk of copying a skill folder and forgetting its command, or vice versa. To pick up new skills added later, re-run `/plugin install`.

### Option B — Manual copy-paste

Copy the skill directory and its matching slash command (if it has one — check the skill's `SKILL.md` frontmatter) into your local Claude Code configuration:

```bash
cp -r skills/figma-grounding ~/.claude/skills/figma-grounding
cp commands/scrapedesign.md ~/.claude/commands/scrapedesign.md
```

Restart Claude Code (or start a new session) to pick up the skill and command. You're responsible for grabbing both files and for pulling updates yourself with this method.

## Versioning

Skills are versioned independently using [semver](https://semver.org/), tracked in [CHANGELOG.md](CHANGELOG.md). A breaking behavior change is a major bump, a new capability is minor, a wording or bug fix is patch.

## Contributing

New skills must follow the template, pass the full testing methodology, and document what was tested. See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

Not yet licensed — see [LICENSE](LICENSE). Internal use only until a license is decided. Do not redistribute.
