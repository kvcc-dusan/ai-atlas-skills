---
name: figma-to-swiftui
description: >
  Converts a Figma frame or screen into production-ready SwiftUI View + ViewModel files,
  mapped to the target project's actual design system components, Code Connect
  mappings, tokens, concurrency model, and localization conventions — not generic
  SwiftUI primitives. Use whenever the user shares a Figma link, node-id, selection, or
  screenshot and wants iOS/SwiftUI code: "turn this Figma screen into SwiftUI", "build
  this frame as a View", "implement this design", "code this screen", or when a Figma
  MCP server is connected and the user points at a specific frame. Also trigger on a
  pasted Figma URL containing a node-id, or when a designer/dev says a screen was
  "marked" or "shared" for implementation. Assumes `figma-grounding` has already gated
  the value extraction — this skill owns everything downstream: component resolution,
  layout translation, assets, concurrency-safe state, localization, and validation
  against the actual Figma screenshot before calling anything done.
---

# Figma → SwiftUI

Turn a single Figma frame into a complete, drop-in SwiftUI `View` — matching this
project's actual design system, architecture, and concurrency model, not a generic
SwiftUI rewrite that looks right and gets quietly redone by hand in review.

## Why this exists

Figma describes what a screen looks like. It never fully describes what a screen does,
which design-system component a shape is standing in for, what architecture a project
already uses, or whether the codebase runs Swift 6 strict concurrency. A model that
fills those gaps by guessing produces code that *looks* production-ready and isn't: it
invents a component that doesn't exist, assumes a state-management pattern the project
doesn't use, hardcodes pixel values that break on other devices, or ships a ViewModel
that won't compile under strict concurrency checking. Every one of those failures is
invisible in a quick glance at the generated code — they surface later, in review or in
CI, which is the expensive place to catch them. Real teams running this exact workflow
report the same root cause repeatedly: the model has no visibility into what it doesn't
know about the target codebase, so it guesses, and the guess looks confident. This
skill's job is to make it ask the few things it genuinely can't know and refuse to
guess the rest.

## Skill boundaries

- Use this skill when the deliverable is **SwiftUI code in the project's repository**.
- If the user wants to create/edit/delete nodes inside Figma itself, that's a different
  capability (e.g. `use_figma`/`figma-use`), not this skill.
- If the user wants only Code Connect mappings authored, that's a narrower task —
  produce the mapping (see `references/code-connect-swiftui.md`) without generating a
  full screen.
- If the user wants a project rules file authored from scratch with no screen to
  implement yet, that's what Pre-flight step 1 already does inline — run that instead
  of skipping straight to screen generation.
- This skill does not run without `figma-grounding` having gated the value extraction
  first (see below) — it is not a replacement for that gate.

## The rule

No size/spacing/color value gets written without going through `figma-grounding`
first. No design-system component gets invented — every component maps to something
real in the project (confirmed via Code Connect, the project's documented catalogue, or
an actual search of the codebase — never from what a component "should probably be
called"). No architecture or concurrency model gets assumed silently — both are read
from the project's own context file, and if there isn't one, the skill offers to
generate one before proceeding rather than defaulting to whatever pattern the model has
seen most in training. No behavior gets invented — Figma has no behavior layer, so
missing interaction gets a placeholder and a specific `// TODO:`, never a guessed
implementation. And nothing gets marked done until it's checked against the actual
Figma screenshot, not just against how plausible the code looks on its own.

## Pre-flight — project context (blocking)

INOVA works across multiple client codebases, not one house app, so this reads the
*current* project's own context rather than assuming a fixed answer every time.

1. **Look for an existing rules file** — `CLAUDE.md`, `AGENTS.md`, or
   `.cursor/rules/*.mdc` at the repo root or nearest parent. It should define: the
   architecture/state-management pattern in use, the design-system catalogue and real
   Swift APIs (or confirmation that Code Connect is set up, which makes a manual
   catalogue mostly redundant), the token namespace, localization paths and key
   convention, the minimum iOS/Swift version, and whether the project runs Swift 6
   strict concurrency (this changes how generated ViewModels must be isolated — see
   `references/concurrency-and-modern-swift.md`).

2. **If no rules file exists, offer to generate one before proceeding.** If the
   connected Figma MCP server exposes a `create_design_system_rules`-style tool, run it
   with the project's real language/framework values (Swift, SwiftUI) and use its
   output — cross-referenced against an actual scan of the codebase — as the starting
   rules file, following the template in `references/PROJECT_CONTEXT.template.md`.
   This mirrors Figma's own recommended workflow for other frameworks; there's no
   reason SwiftUI projects should skip straight to guessing when the same tooling
   applies.

3. **If that tool isn't available, or the user wants to skip straight to one screen,**
   ask once, grouped, and proceed on the stated default if there's no strong opinion:

   ```
   No project context file found. Before I generate this screen, I need to know:

   1. Architecture / state-management pattern this project uses
   2. Design system — component library present? Is Code Connect set up for it?
   3. Token namespace for spacing/color/type (or: raw SwiftUI values are fine)
   4. Localization convention, if this screen has text
   5. Minimum iOS version, and: does this project run Swift 6 strict concurrency?

   No strong opinion? I'll default to: plain MVVM, raw SwiftUI values with inline TODOs
   marking where a token would go, no localization (flagged inline), iOS 17+, and
   Swift 6-safe (@MainActor ViewModels). I'll state explicitly that's what I did.
   ```

4. **If a file exists but is missing one item**, ask only for that piece.

This replaces silently defaulting to any one architecture or concurrency model — the
point isn't that MVVM, TCA, or anything else is wrong, it's that guessing which one a
given client's codebase uses is exactly the kind of silent assumption this skill exists
to eliminate.

## Procedure

### Phase A — Ground and plan (produces an inspectable plan, not code yet)

This mirrors how more mature implementations of this workflow are actually built:
separate *building a structured, reviewable plan* from *emitting code*, so a wrong
component mapping gets caught by looking at a table, not by reading generated Swift.

**1. Ground the design data.** Apply `figma-grounding` against the target node —
`get_metadata`, `get_variable_defs`, `get_code_connect_map`. Don't re-derive that logic
here; if grounding surfaced unresolved values, carry those flags forward. Keep the call
scoped to this one frame, not a whole page.

**2. Pull structural/behavioral context, explicitly targeting iOS.** Call
`get_design_context` and explicitly state the target platform in the request (e.g.
"generate this selection for iOS/SwiftUI") — it defaults to React + Tailwind otherwise,
which is a lossy detour for something you're about to translate again. Treat its output
as a structural hint only, per `figma-grounding`'s rule — never as ground truth for the
values already pulled in step 1. This is a deliberate divergence from Figma's own
generic implementation-skill ordering (which treats `get_design_context` as primary);
see `references/design-rationale.md` for why value-grounding stays first for this
project.

**3. Capture the screenshot and keep it.** `get_screenshot` on the same node. This is
the source of truth for the final validation pass in Phase C — don't discard it once
you've eyeballed the layout once.

**4. Resolve component identity.** Walk the layer tree for structure, repetition, and
component identity, in that order:
   - Structure → top-level `VStack`/`ZStack` boundaries.
   - Repetition → its own subview + `ForEach`/`List`/`LazyVStack`, never inlined copies.
   - Component identity, priority order: Code Connect mapped (use exactly as given —
     see `references/code-connect-swiftui.md`) → DS organism → DS molecule → DS atom →
     **no match**. Confirm DS matches against an actual search of the codebase
     (`grep`/project search for the component name), not against memory of what the
     context file described — codebases drift from their own docs.
   - **No-match case:** stop, name it clearly, ask the developer to choose: compose
     from DS primitives with an inline gap comment, add a `// TODO:` placeholder, or
     flag it as a new component candidate.
   - **Variant/property gap check:** compare each mapped component's Figma variant
     properties against its real Swift API. Any Figma property with no matching case is
     a gap — list all gaps in one message, don't silently pick the closest case.
   - **Content vs. chrome:** separate what's real content (belongs as a parameter/model
     input) from what's decorative (safe to hardcode in the view) — this is what keeps
     the top-level `View` "immediately callable" per the output structure in step 9,
     rather than a one-off hardcoded to this exact frame's copy.
   - **Multi-style text runs:** check whether a text block that reads as one paragraph
     is actually several differently-styled spans — different colors, weights, or
     opacities on different words or lines. Don't collapse it into a single flat style;
     reproduce it with `Text` concatenation (`Text("a").foregroundColor(x) +
     Text("b").foregroundColor(y)`) so the emphasis survives, and note it in assumptions
     if its purpose (e.g. a partial-reveal or highlight state) isn't obvious from the
     static frame alone.
   - Note any affordance that implies behavior (tappable, swipeable, stateful) even
     without a Figma click-handler to confirm it. When a control's purpose isn't obvious
     from its icon alone, infer from context — what it's positioned next to, whether the
     same element recurs elsewhere in the file — rather than guessing from shape alone.

**5. Present the plan and wait for confirmation.** One message containing: the
component mapping table (Figma layer → DS component → key props), any variant gaps,
any no-match components, and the `contains_images`/`contains_animations` flags. Do not
write Swift until this is confirmed — catching a wrong mapping here costs seconds,
catching it after generation costs a review cycle.

### Phase B — Generate

**6. State assumptions.** Figma has no behavior layer — name what you're inferring in
a short `Assumptions:` list before code. Keep it to what actually matters for this
screen.

**7. Handle images and animations** (skip whichever flag is false).
   - Search the project for existing assets (`find . -name "*.xcassets" -type d`) before
     asking anything.
   - If the connected Figma MCP exposes an asset-download tool (e.g. `download_assets`)
     or serves a fetchable asset URL for the node, use it to actually pull the image
     bytes rather than only naming the asset and leaving export as pure manual work —
     push automation as far as the available tooling allows, and say plainly which
     images still needed a manual export because no download path was available.
   - Ask once, for all images at once, how each should load: local asset, remote URL
     (async), or reuse of a found existing asset.
   - Animations: ask for the Lottie JSON filename, default to `.looping()` unless the
     design clearly shows a one-shot animation.

**8. Translate AutoLayout to SwiftUI**, using the grounded sizing mode — not the canvas
dimensions — to decide behavior:

   | Figma AutoLayout | SwiftUI |
   |---|---|
   | Direction: vertical / horizontal | `VStack(spacing:)` / `HStack(spacing:)` |
   | Wrap | `LazyVGrid`/`LazyHGrid` |
   | Overlapping, no auto-layout | `ZStack` |
   | Sizing: **Fill container** | `.frame(maxWidth: .infinity)` / `Spacer()` — never a literal canvas width |
   | Sizing: **Hug contents** | No explicit frame |
   | Sizing: **Fixed** | `.frame(width:height:)` from the grounded value — the *only* case a literal dimension is correct |
   | Alignment: space-between | `Spacer()` between children, or per-child `.frame(maxWidth: .infinity, alignment:)` |
   | Padding/inset | `.padding(<token>)` from the grounded token |

   **Don't flatten nested gaps into one compromise value.** Figma's own generated
   output mixes two different kinds of numbers: incidental absolute `x`/`y` canvas
   position (ignore for layout) and actual spacing decisions (`gap`/`padding`, however
   they're expressed). Read every gap between siblings at *every* nesting level and
   reproduce each one as its own `spacing:`/`.padding()` — don't average several
   different gap sizes into a single value on an outer container. Tight spacing inside
   a sub-group and looser spacing between groups needs matching nested stacks, not one
   stack with a single compromise value.

   Avoid `GeometryReader` unless the layout genuinely needs the parent's size. Avoid
   nesting past ~3-4 stack levels — split into subviews instead.

**9. Handle missing interaction and write the code.**
   - Placeholder closures (`onTap: () -> Void = {}`) and specific `// TODO:`s for
     anything Figma can't express — never guessed business logic.
   - If a state clearly must exist (loading/empty/error, disabled-on-invalid-input) but
     Figma only shows the happy path, name it in assumptions rather than picking one.
   - **Concurrency:** if the project runs Swift 6 strict concurrency (confirmed in
     pre-flight), mark ViewModels `@MainActor`, wrap async work in `Task { }`, and use
     `nonisolated` only for genuine background work — never a manual
     `DispatchQueue.main.async`. Full pattern in
     `references/concurrency-and-modern-swift.md`.
   - Standard SwiftUI hygiene: native layout primitives over reimplementation, Dynamic
     Type support, `.accessibilityLabel`/`.accessibilityHint` on icon-only controls,
     `LazyVStack`/`List` for anything that can grow long, `@State` scoped tight, no
     force unwraps/casts, `body` under ~40 lines, Swift API Design Guidelines naming.
   - **Check every Figma-derived identifier for a collision before using it.** Before
     naming any stored property, parameter, or method after a Figma layer name, check
     it against every other member already declared on that type — including ones
     inherited from a protocol it conforms to, like `View`'s `body` or `Identifiable`'s
     `id`. A name collision is a compile error, not a style issue, and it's an easy trap
     when a layer literally named "Body," "Content," or "Frame" gets mapped straight to
     a Swift identifier without this check. Re-scan every generated type for this right
     before presenting the code, not just while first naming things.
   - Add localization keys for every user-facing string, using the path and convention
     confirmed in pre-flight — never guess a `Localizable.strings` path.
   - One primary `View` named after the Figma frame (UpperCamelCase); extract genuinely
     reusable sections into their own subviews; always include an `#Preview` with mock
     data. Check whether target files already exist before writing — ask before
     overwriting.

### Phase C — Validate (don't skip this — it's the step most drafts of this workflow omit)

**10. Check the generated code against the screenshot from step 3**, not just against
how plausible it reads on its own — a model has no innate visibility into its own
rendered output, so this has to be a deliberate pass, not an assumption. Use the
checklist in `references/validation-checklist.md`: layout/spacing/alignment, typography,
colors, interactive states, accessibility, and — critically — that every asset
reference actually resolves to something that will exist in the built app. If SwiftLint
is configured, run it and fix violations; if a build/compile check is available in the
environment, prefer catching a compile error here over leaving it for the developer.

**11. Close with a completion summary** — files written, localization keys added (or
"none"), which images still need manual export, and concrete next steps (wire the
ViewModel into DI, register the navigation route). Nothing left implicit.

## Output format

1. **Plan** (Phase A, step 5) — mapping table, gaps, flags. Skip if already confirmed
   earlier in the conversation.
2. **Assumptions** (step 6).
3. **The code** — full SwiftUI file(s), file names as headers if split.
4. **Validation notes** (step 10) — anything the checklist caught, or confirmation it's
   clean.
5. **Completion summary** (step 11).

Don't pad the response with a restated design description — the plan, code, and
assumptions speak for themselves.

## Failure awareness

Full tables in `references/failure-modes.md` (this skill's own) and
`figma-grounding/references/failure-modes.md` (inherited, not repeated here). The
highest-severity ones to recognize on sight: a component name that sounds plausible but
doesn't exist in the real codebase; an architecture or concurrency assumption made
because pre-flight was skipped; a literal canvas dimension hardcoded where grounding
said the node was Fill or Hug; and code that was never actually checked against the
screenshot from step 3 before being called done.

## What this skill does NOT do

- **Does not fully automate image/Lottie export.** It pushes automation as far as the
  connected MCP's asset tools allow, but whatever isn't fetchable through those tools
  remains a manual step — and it says so explicitly rather than implying otherwise.
- **Does not decide the project's architecture or concurrency model for it.** Reads
  both from the project's context file, or generates one via pre-flight — never
  defaults silently.
- **Does not replace a real build.** SwiftLint and, where available, a compile check are
  run — a full test suite or manual QA pass is still the developer's job.
- **Does not replace `figma-grounding`.** Assumes grounding already ran for the values
  in play; if invoked standalone, treat every size/color/spacing value as unverified.
