# Design rationale — why this skill diverges from a couple of "obvious" defaults

Documented per this repo's own `CONTRIBUTING.md` guidance: when a rule looks arbitrary,
explain the reasoning, so a future maintainer doesn't "fix" a deliberate choice back
into the more common but weaker default.

## Why value-grounding runs before `get_design_context`, not after

Figma's own published generic skill ("Skill: Implement Design",
developers.figma.com/docs/figma-mcp-server/skill-figma-implement-design) uses this
order: `get_design_context` first, `get_metadata` only if the response is truncated,
`get_screenshot` for visual reference, then implement. That's a reasonable default for
their target audience (general web implementation), but two things make it a worse fit
here:

1. `get_design_context` has a documented silent-failure mode: on accounts without Code
   Connect configured, or on selections with designer annotations, it can fall back to
   image-guessing internally and still return plausible-looking output. Treating it as
   the primary source means that failure is invisible until someone notices a value is
   wrong.
2. It defaults to React + Tailwind output. For a SwiftUI target, that's a lossy detour
   — translating React-shaped output into SwiftUI shapes is an extra layer of
   interpretation on top of whatever the tool already interpreted from the design.

So this skill's ordering (via `figma-grounding`) treats `get_metadata` +
`get_variable_defs` + `get_code_connect_map` as the ground-truth layer, and
`get_design_context` as a structural/behavioral hint only — cross-checked, never
trusted at face value. This costs one or two extra tool calls per screen. That's a
reasonable price for not silently trusting a tool with a known plausible-but-wrong
failure mode.

## Why component mapping happens before any code is written, not interleaved with it

A production Figma-to-code system built at scale (monday.com's design-system MCP +
agent) made a deliberate architectural choice worth naming explicitly: their agent
returns a structured *context/plan* object, not code, and a separate step formats that
plan into the target repository's actual conventions. The reasoning: across many
codebases with different versions and patterns, forcing a single generation pass to get
both "what should this be" and "how should it look in Swift" right simultaneously means
a wrong component guess gets baked into syntax it's more effort to unwind.

This skill can't run a multi-node agent graph inside a markdown file, but it borrows the
same principle at a smaller scale: Phase A produces a complete, human-reviewable plan
(mapping table, gaps, flags) and stops for confirmation before Phase B touches Swift
syntax at all. Catching "that's not our DS component for that" in a table takes
seconds; catching it after it's threaded through a `View` and a `ViewModel` takes a
review cycle.

## Why asset handling tries to automate, but doesn't promise full automation

Every real-world writeup of this workflow we found (Halodoc's public case study, the
patterns in Figma's own official skill docs) reports the same thing: design-system and
component mapping automate cleanly, but image/asset export into the final binary format
remains at least partially manual, because it depends on what asset tooling the
connected MCP server actually exposes in a given session — that varies by server and by
account tier. Rather than either overclaiming full automation (which erodes trust the
first time it doesn't work) or giving up and treating it as 100% manual (when some
servers do expose a fetchable asset endpoint), this skill tries the available tooling
first and states plainly what it could and couldn't do. That's a narrower, more honest
claim than either extreme.
