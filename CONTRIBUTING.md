# Contributing a skill

## Structure

Every skill needs a `SKILL.md` following the structure in [`skills/_template/SKILL.md`](skills/_template/SKILL.md). Don't skip sections — if one genuinely doesn't apply, say so explicitly rather than deleting it.

## Description quality

Frontmatter `description` must be specific and triggering-oriented — the exact situation that should cause Claude to invoke it — not a generic category label.

- **Bad:** "Helps with Figma design work."
- **Good:** "Invoked via `/scrapedesign` before any task that touches a Figma design through MCP — implementing a component, fixing spacing/color drift, matching a frame. Forces real value extraction before writing a size, spacing, or color, instead of estimating from a screenshot."

If the description is vague, the skill either never fires or fires on the wrong task. Treat this as a hard requirement, not a nice-to-have.

## Testing — no exceptions

Every skill must pass the process in [`docs/testing-methodology.md`](docs/testing-methodology.md) before merging. "It seemed fine in one test" is not a pass. If you haven't run the full eval set, the skill isn't ready for a PR.

## Versioning

Skills are versioned independently and tracked in [`CHANGELOG.md`](CHANGELOG.md), semver:

- **Major** — breaking behavior change (the skill now does something meaningfully different than before)
- **Minor** — new capability added, existing behavior unchanged
- **Patch** — wording fix, typo, small correction with no behavior change

## PR description

State what was tested and against how many cases — e.g. "5-case eval set, 3 runs each, 2 external testers." Not "tested it."
