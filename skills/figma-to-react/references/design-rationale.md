# Design rationale — where this skill follows Figma's own workflow, and where it doesn't

Documented per this repo's `CONTRIBUTING.md` guidance: when a rule looks arbitrary,
explain the reasoning, so a future maintainer doesn't "fix" a deliberate choice back
into the more common but weaker default. Read alongside
`figma-to-swiftui/references/design-rationale.md` — the plan-before-code and
asset-honesty arguments there apply verbatim and aren't repeated in full.

## The aligned case: React + Tailwind is `get_design_context`'s native output

Figma's official "Implement Design" and "Create Design System Rules" skills are
React + Tailwind-flavored by default — `get_design_context` emits React + Tailwind
unless told otherwise. The SwiftUI skill had to explicitly divert (request iOS output,
treat the default as a lossy detour). This skill is the case Figma's tooling was built
for, so:

- No platform override is needed on the `get_design_context` call (Procedure step 3).
- The `create_design_system_rules` bootstrap in pre-flight is used exactly as Figma
  recommends, with `clientLanguages="typescript"`, `clientFrameworks="react"`.

Being the aligned case is also this skill's distinctive trap: the tool's default output
is *fluent* React + Tailwind, which makes it maximally tempting to paste through. Fluent
is not project-correct — the output knows nothing about this client's styling approach,
token representation, or component library. Hence the blocking styling question in
pre-flight and the "no second styling system" line in the validation checklist.

## The deliberate divergence: grounding still runs first

Figma's generic skill ordering treats `get_design_context` as primary
(`get_metadata` only on truncation). This repo's stance — values come from
`get_metadata`/`get_variable_defs`/`get_code_connect_map` first, `get_design_context`
is a structural hint — stands unchanged for React, because the reason is
framework-independent: `get_design_context` has a documented silent-failure mode
(accounts without Code Connect, or selections with annotations, can fall back to
image-guessing internally and still return plausible output). Framework alignment fixes
the lossy-detour half of the SwiftUI argument; it does nothing for the
silent-fallback half. Cost: one or two extra tool calls per screen. Worth it.

## Why the plan/confirm gate before code (borrowed, not invented)

Same reasoning as the SwiftUI skill, from monday.com's published design-system-MCP
architecture: return a structured, reviewable plan before generating code, because a
wrong component mapping caught in a table costs seconds and caught in generated JSX
costs a review cycle. See the SwiftUI rationale file for the full argument.

## Why responsive behavior is treated as design input, not codegen output

A Figma frame is one viewport. Tailwind prefixes are several. Nothing in the frame
licenses the model to decide how the layout collapses — that's a design decision, and
the one reported first-pass failure in the field example we could find (LogRocket's
worked writeup) was exactly this: an element that should have been hidden on mobile
wasn't. So: per-breakpoint frames get grounded per-frame; a single frame gets
implemented as designed with responsive gaps flagged as assumptions. The skill would
rather ship a correct fixed-width component plus an explicit question than a plausible
responsive one that quietly invented the mobile design.

## Why pre-flight refuses a stack default (and what the fallback default is)

INOVA is an agency: one client repo is Tailwind, the next is CSS Modules, the next is
styled-components. Any hardcoded default would be right sometimes and *silently* wrong
the rest of the time — the exact failure class this repo exists to prevent. The
fallback default when the user explicitly has no opinion (React + TypeScript +
Tailwind + Vite) is the maintainer's own daily stack, and the skill is required to
label it as a stated default, not a discovered fact about the project.

## Why pre-flight checks which MCP server is connected

Framelink (`figma-developer-mcp`) is popular in web/React shops specifically because it
needs no Figma Dev seat — which makes it likelier to show up in exactly the client
contexts this skill targets. It exposes different tools (`get_figma_data`,
`download_figma_images`) and no Code Connect or variable defs. `figma-grounding`'s
procedure can't run against it — loudly, since the tool names don't exist — but the
dangerous path is the model quietly substituting `get_figma_data` JSON and looking
grounded. Cheaper to check the tool list once in pre-flight than to detect the
substitution after the fact.
