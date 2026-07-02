# ai-atlas-skills

Claude Code skill collection for INOVA IT. Manual copy-paste install today; structured to be plugin-installable later without restructuring (`.claude-plugin/plugin.json` already declares the skill set).

## Skills

| Skill | Description |
|---|---|
| [`figma-grounding`](skills/figma-grounding/SKILL.md) | Forces `get_metadata`/`get_variable_defs` extraction before writing any size, spacing, or color value from a Figma design — flags unresolved tokens instead of guessing from a screenshot. |

## Install (manual, current method)

Copy the skill and its matching slash command into your local Claude Code config:

```bash
cp -r skills/figma-grounding ~/.claude/skills/figma-grounding
cp commands/scrapedesign.md ~/.claude/commands/scrapedesign.md
```

Restart Claude Code (or start a new session) to pick up the new skill and command.

## Adding a skill

See [CONTRIBUTING.md](CONTRIBUTING.md).
