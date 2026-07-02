---
name: skill-name-here
# Description must contain SPECIFIC TRIGGERING LANGUAGE, not a vague category.
# Bad:  "Helps with Figma design work."
# Good (see skills/figma-grounding/SKILL.md): names the exact trigger point
# ("before any task that touches a Figma design through MCP"), what it forces
# the model to do differently, and that it's a gate, not a standalone report.
# Claude decides whether to invoke a skill from this line alone — vague
# descriptions mean it never fires, or fires on the wrong thing.
description: one-line description with specific triggering language
---

# Skill Name

<!--
  Guideline: keep this file under ~500 lines. If a section is growing into
  a reference table, failure catalog, or anything you'd only consult when
  something's already gone wrong, move it to references/ and link it —
  don't let the main file balloon.
-->

<!--
  Standing principle for anyone authoring a skill in this repo:
  a skill that produces a confident-looking guess instead of flagging
  uncertainty is worse than no skill at all. No skill leaves a human
  to notice they're unsure. A bad skill hides that uncertainty behind
  a plausible-looking answer. Design every rule and procedure below so
  that "I don't know" or "this didn't resolve" is always an allowed,
  visible output — never silently papered over.
-->

## Why this exists

<!-- The problem this skill prevents, and what happens without it. -->

## The rule

<!-- The core constraint — the one thing that must always be true. -->

## Procedure

<!-- Numbered steps, in order, for what to actually do. -->

## Failure awareness

<!-- Known failure signatures and how to recognize them on sight.
     Link to references/failure-modes.md if this grows long. -->

## What this skill does NOT do

<!-- Explicit scope boundary — what a reader might assume this covers but doesn't. -->
