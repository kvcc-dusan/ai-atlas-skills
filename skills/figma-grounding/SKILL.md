---
name: figma-grounding
description: >
  Invoked via /scrapedesign before any task that touches a Figma design through MCP —
  implementing a component, fixing spacing/color drift, matching a frame, whatever the
  actual ask is. Forces Claude to pull real values (get_metadata, get_variable_defs,
  get_code_connect_map) before writing a single size, spacing, color, or layout-sizing
  value, instead of estimating from a screenshot. This is a precondition that runs inline
  before the real task, not a standalone report — don't stop here, don't output a report,
  just get grounded and then do the work.
---

# Figma Grounding

Screenshots are for layout sanity-checking and content, not values. Get the real numbers
first, then do the actual task. This skill has no output of its own — it's a gate the
real task passes through.

## Why this exists

A screenshot tells you what something looks like at one moment, at one zoom level, on
one screen. It does not tell you the bound variable name, the exact spacing token, or
whether a frame is set to hug its content or fill its parent — it just shows you the
pixel result of whatever that value currently resolves to. A model reading only the
screenshot rounds to "close enough," and — this is the actual danger — presents that
rounded guess with the same confidence as a verified value. Per this repo's own testing
bar, a confidently wrong answer is worse than a visibly wrong one: the visible kind gets
caught, the confident kind doesn't. This skill exists to make "I don't know, this didn't
resolve" an always-available, always-visible output instead of something a fluent guess
quietly papers over.

## The rule

Before writing any size, spacing, color, radius, shadow, type, or **layout-sizing**
value in code: it came from `get_metadata` or `get_variable_defs`, or it's explicitly
marked as unresolved. Never from "what it looks like." This includes values people don't
normally think to check by hand: whether a frame is set to hug/fill/fixed, and its
alignment mode (`space-between` vs. centered vs. min/max) — these are metadata fields,
not visual impressions, and screenshots make them *easy* to eyeball wrong because two
very different sizing modes can render identically at one arbitrary frame size. If you
catch yourself about to write a hex code, a px value, or a `.frame(width:)` because it
looked like that in the screenshot, stop — go pull it properly instead.

Component *identity* is a separate grounding concern from component *values*. Knowing a
button's fill color is `#0047AB` doesn't tell you it should become `PrimaryButton(...)`
in code rather than a raw shape — that mapping comes from Code Connect, not from
variables. Ground both, don't let one stand in for the other.

## Procedure

1. **Identify the node.** URL node-id, or current Figma selection confirmed via a quick
   `get_metadata` call. Don't proceed on an assumed selection — and note that
   **selection-based prompting only works on the desktop MCP server**; the remote server
   requires an explicit node-id in the URL and cannot read "whatever's currently
   selected." Confirm which server you're on before assuming selection will work at all.

2. **`get_metadata`** on the target — ground truth for padding, gaps, dimensions,
   radius, position, hierarchy, and layout-sizing mode (hug / fill / fixed) + axis
   alignment. Pull it before you touch spacing, sizing, or layout behavior in code.

3. **`get_variable_defs`** on the target — ground truth for bound colors, typography,
   spacing tokens, shadows. Pull it before you touch color or type in code.

4. **`get_code_connect_map`** (or `get_code_connect_suggestions` if no mapping exists
   yet) — ground truth for *which real code component* this node represents, when one
   exists. Treat an unmapped component as a data point ("no Code Connect mapping for
   this node"), not as license to invent a component name.

5. **`get_design_context`** if useful for structure — treat as a hint only, never as
   ground truth for the values in steps 2–4. Cross-check anything it hands you against
   what you already pulled. This is also the tool most prone to a specific silent
   failure: on accounts without Code Connect set up, or selections with designer
   annotations, it can fall back to image-guessing internally and still return
   plausible-looking output. See `references/failure-modes.md`.

   Note the tool naming: `get_code` doesn't exist in the current Figma MCP tool list —
   the design-to-code tool is `get_design_context`, and it defaults to React + Tailwind
   output unless you tell it otherwise (e.g. "generate my Figma selection in iOS"). If a
   client you're using still exposes something called `get_code`, treat it as this
   tool's predecessor, not a different capability.

6. **Screenshot last, and only for what has no structured source** — icon choice,
   imagery, general layout sanity check. Not for a color, a pixel number, or a
   hug-vs-fill decision that steps 2–4 should have already given you.

7. **If a value doesn't resolve** — no matching variable, no clean number, no Code
   Connect mapping — say so in one line as you go ("no bound variable for this fill,
   using raw #XXXXXX" / "no Code Connect mapping, treating as a generic frame"), and
   keep moving. Don't silently invent a token, don't silently invent a component name,
   don't silently hardcode without flagging it. One line, not a report.

8. **Keep calls scoped to the node you're actually working on.** Don't fetch a whole
   page or a root frame "to be thorough" — large selections are the single biggest
   cause of slow/incomplete responses, and free/Collab seats are capped around 6 tool
   calls a month with paid Dev seats rate-limited per-minute same as Figma's Tier 1 REST
   API. Grounding a multi-screen file happens per-screen, not in one call.

9. **Now do the actual task** the user asked for, using what you just pulled. This
   isn't a separate step for the user to wait on — grounding happens, then the real
   work happens, same turn.

## Failure awareness

Two categories matter here, and they fail differently:

**Loud failures** (connection-layer, auth, 500s, tools not loading) are annoying but
safe — you can't proceed, so you can't proceed wrong. Full signature table in
`references/failure-modes.md`.

**Silent failures** are the dangerous category — no error thrown, output looks
complete, and it's simply wrong:

- `get_metadata`/`get_variable_defs` come back empty or clearly wrong on a frame that
  obviously has structure/tokens — that's a tool-failure signal (wrong node, unshared
  variable library, selection scoped wrong), not "this frame has no tokens." Don't paper
  over it by switching to the screenshot.
- `get_design_context` "succeeds" but the content is generic/plausible rather than
  matching the frame — likely a silent screenshot-fallback because Code Connect isn't
  set up for this account tier. Don't trust it; lean entirely on `get_metadata` /
  `get_variable_defs`.
- **Wrong tool got called.** You asked for grounding, but tool auto-routing picked
  `get_design_context` instead of `get_variable_defs` and handed back code that never
  touched the variable API — it will still look grounded. This is a documented,
  acknowledged behavior (Figma's own docs note the agent "chooses which tool to call
  based on your prompt... sometimes it guesses wrong"). The fix is being explicit about
  the exact tool name in the procedure, not trusting auto-routing to infer intent from a
  vague ask.
- A value "looks about right" but isn't an exact match to any variable — rounding to the
  nearest familiar value instead of the real one. Never treat proximity as a match; flag
  it as unresolved.

Full signature tables (connection errors, rate limits, debugging order) live in
`references/failure-modes.md` — consult it on sight of anything in that list rather than
re-deriving the diagnosis from scratch.

## What this skill does NOT do

- **Not a report generator.** It doesn't produce a summary of what it found — it grounds
  inline and moves on.
- **Not an implementation checklist beyond "get the values right first."** It doesn't
  tell you how to structure the component, what architecture pattern to use, or how to
  translate auto-layout into a specific framework's layout primitives — that's the
  downstream task's job (see `figma-to-swiftui` for the SwiftUI-specific version of that
  translation).
- **Not a verification/diff step after the fact.** It doesn't check already-written code
  against the design after the fact — that's a separate concern if you want it built as
  its own skill; this is purely about not writing wrong numbers in the first place.
- **Not a decision-maker for remote vs. desktop MCP server.** It assumes you already
  know which one is configured — confirm that before assuming selection-based prompting
  will work.
