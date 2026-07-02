---
name: figma-grounding
description: Invoked via /scrapedesign before any task that touches a Figma design through MCP — implementing a component, fixing spacing/color drift, matching a frame, whatever the actual ask is. Forces Claude to pull real values (get_metadata, get_variable_defs) before writing a single size, spacing, or color, instead of estimating from a screenshot. This is a precondition that runs inline before the real task, not a standalone report — don't stop here, don't output a report, just get grounded and then do the work.
---

# Figma Grounding

Screenshots are for layout and content, not values. Get the real numbers first, then do the actual task. This skill has no output of its own — it's a gate the real task passes through.

## The rule

Before writing any size, spacing, color, radius, shadow, or type value in code: it came from `get_metadata` or `get_variable_defs`, or it's explicitly marked as unresolved. Never from "what it looks like." If you catch yourself about to write a hex code or a px value because it looked like that in the screenshot, stop — go pull it properly instead.

## What to actually do, in order

1. **Identify the node.** URL node-id, or current Figma selection confirmed via a quick `get_metadata` call. Don't proceed on an assumed selection.
2. **`get_metadata`** on the target — this is ground truth for padding, gaps, dimensions, radius, position, hierarchy. Pull it before you touch spacing in code.
3. **`get_variable_defs`** on the target — ground truth for bound colors, typography, spacing tokens, shadows. Pull it before you touch color or type in code.
4. **`get_design_context`/`get_code` if useful** — treat as a structural hint only. Cross-check any value it hands you against steps 2–3; don't take it at face value, it's the tool most prone to silently falling back to image-guessing when Code Connect isn't set up or the selection has annotations (see `references/failure-modes.md`).
5. **Screenshot last, and only for what has no structured source** — icon choice, imagery, general layout sanity check. Not for a color or a pixel number that steps 2–3 should have already given you.
6. **If a value doesn't resolve** — no matching variable, no clean number — say so in one line as you go ("no bound variable for this fill, using raw #XXXXXX") and keep moving. Don't silently invent a token, don't silently hardcode without flagging it. One line, not a report.
7. **Now do the actual task** the user asked for, using what you just pulled. This isn't a separate step for the user to wait on — grounding happens, then the real work happens, same turn.

## Failure awareness

If `get_metadata`/`get_variable_defs` come back empty or clearly wrong on a frame that obviously has structure/tokens, that's a signal of tool failure, not "this frame has no tokens." Don't paper over it by switching to the screenshot — surface it and figure out why (wrong node, unshared variable library, selection scoped wrong) before proceeding. `references/failure-modes.md` has the specific signatures worth recognizing on sight.

## What this skill is not

Not a report generator. Not an implementation checklist beyond "get the values right first." Not a verification/diff step after the fact — that's a separate concern if you want it, this is purely about not writing wrong numbers in the first place.
