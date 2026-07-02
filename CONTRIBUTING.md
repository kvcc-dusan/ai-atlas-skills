# Contributing

This document covers how to add a new skill or modify an existing one. The bar is deliberately high: a skill that fires unreliably or produces confident guesses is worse than no skill, because users stop double-checking output they believe is grounded.

## Adding a new skill

### 1. Start from the template

Copy `skills/_template/` to `skills/<your-skill-name>/` and fill in every section of `SKILL.md`:

- `## Why this exists`
- `## The rule` (the core constraint)
- `## Procedure`
- `## Failure awareness`
- `## What this skill does NOT do`

Don't skip sections — if one genuinely doesn't apply, state that explicitly rather than deleting it. Keep `SKILL.md` under ~500 lines; move reference tables, failure catalogs, and anything consulted only when something has already gone wrong into `references/`.

### 2. Write a triggering-oriented description

The frontmatter `description` is the only thing Claude sees when deciding whether to invoke the skill. It must name the exact situation that should trigger it — not a category label.

- **Bad:** "Helps with Figma design work."
- **Good:** "Invoked via `/scrapedesign` before any task that touches a Figma design through MCP — implementing a component, fixing spacing/color drift, matching a frame. Forces real value extraction before writing a size, spacing, or color, instead of estimating from a screenshot."

A vague description means the skill never fires, or fires on the wrong task. This is a hard requirement, not a style preference.

### 3. Test it — no exceptions

Every skill must pass the full process in [docs/testing-methodology.md](docs/testing-methodology.md) before merging. "It seemed fine in one test" is not a pass. If you haven't run the complete eval set, the skill is not ready for a PR.

### 4. Register the skill

A new skill touches four places. All four are required:

1. `skills/<name>/` — the skill itself
2. `.claude-plugin/plugin.json` — add the path to the `skills` array
3. `README.md` — add a row to the skills table
4. `CHANGELOG.md` — add an entry for `<name> 1.0.0`

If the skill has a companion slash command, add it under `commands/` and mention it in the skill's README row.

## Modifying an existing skill

Behavior changes require re-running the relevant parts of the testing methodology — at minimum the eval cases the change could affect. Bump the version according to the rules below and add a CHANGELOG entry.

## Versioning

Skills are versioned independently, tracked in [CHANGELOG.md](CHANGELOG.md), using [semver](https://semver.org/):

- **Major** — breaking behavior change: the skill now does something meaningfully different than before
- **Minor** — new capability added; existing behavior unchanged
- **Patch** — wording fix, typo, or small correction with no behavior change

## Pull requests

The PR description must state what was tested and against how many cases — for example: "5-case eval set, 3 runs each, dependency-failure check done, 2 external testers." A PR that says only "tested it" will be sent back.

### Checklist

- [ ] `SKILL.md` follows the template structure, all sections present
- [ ] Frontmatter `description` is specific and triggering-oriented
- [ ] Full testing methodology completed and results logged
- [ ] Registered in `plugin.json`, `README.md`, and `CHANGELOG.md`
- [ ] Version bump matches the semver rules above
