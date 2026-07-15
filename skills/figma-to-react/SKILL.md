---
name: figma-to-react
description: >
  Converts a Figma frame or screen into production-ready React + TypeScript components,
  mapped to the target project's actual design system, Code Connect mappings, tokens,
  styling approach (Tailwind / CSS Modules / styled-components / vanilla CSS — read from
  the project, never assumed), and responsive-breakpoint reality — not generic JSX. Use
  whenever the user shares a Figma link, node-id, selection, or screenshot and wants
  web/React code: "turn this Figma screen into React", "build this frame as a component",
  "implement this design", "code this screen for web", or when a Figma MCP server is
  connected and the user points at a specific frame with a web/React target. Also trigger
  on a pasted Figma URL containing a node-id when the project is a React codebase.
  Assumes `figma-grounding` has already gated the value extraction — this skill owns
  everything downstream: component resolution, AutoLayout → flexbox/grid translation,
  styling-token representation, responsive behavior, assets, accessibility, and
  validation against the actual Figma screenshot before calling anything done.
---

# Figma → React

Turn a single Figma frame into a complete, drop-in React component — matching this
project's actual design system, styling approach, and conventions, not a generic
React+Tailwind rewrite that looks right and gets quietly redone by hand in review.

## Why this exists

Figma describes what a screen looks like. It never fully describes what a screen does,
which design-system component a shape is standing in for, how a project expresses its
tokens in code, or how a fixed-width frame is supposed to behave at other viewport
widths. A model that fills those gaps by guessing produces code that *looks*
production-ready and isn't: it invents a component that doesn't exist, writes Tailwind
classes into a CSS Modules codebase, hardcodes a hex value where the project mandates a
token class, or silently decides how a desktop frame collapses on mobile. Every one of
those failures is invisible in a quick glance at the generated code — they surface in
review or in the browser, the expensive places to catch them. The web adds a trap the
SwiftUI sibling of this skill never had: because Figma's own tooling defaults to
React + Tailwind output, generated code for a React target looks *especially* plausible
— fluent, idiomatic, and still wrong about everything project-specific. This skill's job
is to make the model ask the few things it genuinely can't know and refuse to guess the
rest.

## Skill boundaries

- Use this skill when the deliverable is **React + TypeScript code in the project's
  repository**.
- If the user wants to create/edit/delete nodes inside Figma itself, that's a different
  capability (e.g. `use_figma`/`figma-use`), not this skill.
- If the user wants only Code Connect mappings authored, that's a narrower task —
  produce the mapping (see `references/code-connect-react.md`) without generating a
  full screen.
- If the user wants a project rules file authored from scratch with no screen to
  implement yet, that's what Pre-flight step 2 already does inline — run that instead
  of skipping straight to screen generation.
- If the target is SwiftUI/iOS, that's `figma-to-swiftui`, not this skill.
- This skill does not run without `figma-grounding` having gated the value extraction
  first (see below) — it is not a replacement for that gate.

## The rule

No size/spacing/color value gets written without going through `figma-grounding` first.
No design-system component gets invented — every component maps to something real in the
project (confirmed via Code Connect, the project's documented catalogue, or an actual
search of the codebase — never from what a component "should probably be called"). No
styling approach or token representation gets assumed silently — both are read from the
project's own context file, and if there isn't one, the skill offers to generate one
before proceeding rather than defaulting to whatever stack the model has seen most in
training. No responsive behavior gets invented — a single fixed-width frame describes
one viewport, and anything written for other viewports is a flagged assumption, never a
silent decision. No behavior gets invented — Figma has no behavior layer, so missing
interaction gets a placeholder and a specific `// TODO:`, never a guessed
implementation. And nothing gets marked done until it's checked against the actual
Figma screenshot, not just against how plausible the code looks on its own.

## Pre-flight — project context (blocking)

INOVA works across multiple client codebases, not one house app, so this reads the
*current* project's own context rather than assuming a fixed answer every time.

0. **Confirm which Figma MCP server is actually connected.** This skill and
   `figma-grounding` are written against the official Figma MCP tool names
   (`get_metadata`, `get_variable_defs`, `get_code_connect_map`, `get_design_context`,
   `get_screenshot`). A second server is common in React shops because it removes the
   Dev-seat cost barrier: Framelink (`figma-developer-mcp`), which exposes entirely
   different tools (`get_figma_data`, `download_figma_images`) and has **no Code Connect
   or variable-defs support at all**. If the official tool names aren't in the connected
   tool list, say so explicitly: grounding as this repo defines it cannot run against
   Framelink, and its `get_figma_data` output must not be silently substituted as if it
   were grounded values. See `references/failure-modes.md`.

1. **Look for an existing rules file** — `CLAUDE.md`, `AGENTS.md`, or
   `.cursor/rules/*.mdc` at the repo root or nearest parent. It should define: the
   styling approach (Tailwind / CSS Modules / styled-components / vanilla CSS custom
   properties) and — if Tailwind — the canonical token representation (config class
   `bg-primary-500` vs. arbitrary value `bg-[#0047AB]` vs. custom property
   `bg-[var(--color-primary)]`; see `references/responsive-and-styling.md`), the
   design-system catalogue and real component APIs (or confirmation that Code Connect
   is set up, which makes a manual catalogue mostly redundant), the state-management
   pattern in use, the breakpoint set, the framework flavor (Vite SPA / Next.js —
   server vs. client components), and any i18n convention.

2. **If no rules file exists, offer to generate one before proceeding.** If the
   connected Figma MCP server exposes a `create_design_system_rules`-style tool, run it
   with the project's real values (`clientLanguages="typescript"`,
   `clientFrameworks="react"`) and use its output — cross-referenced against an actual
   scan of the codebase (`package.json` dependencies, `tailwind.config.*`, existing
   component directories) — as the starting rules file, following the template in
   `references/PROJECT_CONTEXT.template.md`. This is Figma's own recommended workflow,
   and React is the case it was primarily built for.

3. **If that tool isn't available, or the user wants to skip straight to one screen,**
   ask once, grouped, and proceed on the stated default if there's no strong opinion:

   ```
   No project context file found. Before I generate this screen, I need to know:

   1. Styling approach — Tailwind, CSS Modules, styled-components, or vanilla CSS?
      If Tailwind: token classes, arbitrary values, or CSS custom properties?
   2. Design system — component library present? Is Code Connect set up for it?
   3. State management pattern, if this screen needs state
   4. Framework flavor — Vite SPA, Next.js (App Router?), other?
   5. Breakpoints — does this design have per-breakpoint frames, or one width?

   No strong opinion? I'll default to: React + TypeScript + Tailwind + Vite, config-based
   token classes with inline TODOs where no token matches, local state only, single
   breakpoint implemented as designed with responsive behavior flagged as open. That's a
   stated default — my author's usual stack — not a discovered fact about this project,
   and I'll say explicitly that's what I did.
   ```

4. **If a file exists but is missing one item**, ask only for that piece.

This replaces silently defaulting to any one styling approach or framework flavor — the
point isn't that Tailwind, CSS Modules, or anything else is wrong, it's that guessing
which one a given client's codebase uses is exactly the kind of silent assumption this
skill exists to eliminate.

## Procedure

### Phase A — Ground and plan (produces an inspectable plan, not code yet)

This mirrors how more mature implementations of this workflow are actually built:
separate *building a structured, reviewable plan* from *emitting code*, so a wrong
component mapping gets caught by looking at a table, not by reading generated JSX. (See
`references/design-rationale.md` — the reasoning is borrowed deliberately from a
production design-system-MCP architecture.)

**1. Ground the design data.** Apply `figma-grounding` against the target node —
`get_metadata`, `get_variable_defs`, `get_code_connect_map`. Don't re-derive that logic
here; if grounding surfaced unresolved values, carry those flags forward. Keep the call
scoped to this one frame, not a whole page.

**2. Identify the breakpoint situation before anything else structural.** Check whether
the file contains sibling frames for other viewport widths (common names: `Desktop`,
`Tablet`, `Mobile`, or width-suffixed duplicates of the same screen). If yes, **ground
each breakpoint frame separately** — never diff two screenshots and guess the delta. If
only one frame exists, the design describes exactly one viewport; implement that, and
record "responsive behavior undesigned" as an assumption for step 6. Full decision table
in `references/responsive-and-styling.md`.

**3. Pull structural context and keep the screenshot.** Call `get_design_context` —
its React + Tailwind default output is, for once, the aligned case, so no platform
override is needed — but treat it as a structural hint only, per `figma-grounding`'s
rule: never as ground truth for the values already pulled in step 1, and never as
license to keep its Tailwind classes if pre-flight said this project doesn't use
Tailwind. Then `get_screenshot` on the same node — this is the source of truth for the
final validation pass in Phase C; don't discard it after one glance.

**4. Resolve component identity.** Walk the layer tree for structure, repetition, and
component identity, in that order:
   - Structure → top-level semantic containers (`header`/`main`/`section`/`nav` where
     the content warrants it, `div` otherwise) with flex/grid layout.
   - Repetition → its own component + `.map()` over typed data, never inlined copies —
     with a stable `key` from the data, not the array index, unless the list is truly
     static.
   - Component identity, priority order: Code Connect mapped (use exactly as given —
     see `references/code-connect-react.md`) → DS organism → DS molecule → DS atom →
     **no match**. Confirm DS matches against an actual search of the codebase
     (`grep`/project search for the component's export), not against memory of what the
     context file described — codebases drift from their own docs.
   - **No-match case:** stop, name it clearly, ask the developer to choose: compose
     from DS primitives with an inline gap comment, add a `// TODO:` placeholder, or
     flag it as a new component candidate. Never silently invent one.
   - **Variant/prop gap check:** compare each mapped component's Figma variant
     properties against its real TypeScript props interface. Any Figma property with no
     matching prop or union member is a gap — list all gaps in one message, don't
     silently pick the closest value.
   - **Content vs. chrome:** separate real content (belongs as typed props/model input)
     from decoration (safe to hardcode in the component) — this keeps the top-level
     component immediately callable rather than hardcoded to this exact frame's copy.
   - **Multi-style text runs:** a block that reads as one paragraph may be several
     differently styled spans. Don't flatten it — reproduce with nested `<span>`s (or
     the DS's text primitives) so the emphasis survives, and note it in assumptions if
     its purpose isn't obvious from the static frame.
   - Note any affordance that implies behavior (clickable, hoverable, stateful) even
     without a Figma prototype wire to confirm it; infer a control's purpose from
     context (position, recurrence elsewhere in the file), not from its shape alone.

**5. Present the plan and wait for confirmation.** One message containing: the
component mapping table (Figma layer → DS component → key props), any variant/prop
gaps, any no-match components, the breakpoint situation from step 2, and the
`contains_images`/`contains_animations` flags. Do not write TSX until this is confirmed
— catching a wrong mapping here costs seconds, catching it after generation costs a
review cycle.

### Phase B — Generate

**6. State assumptions.** Figma has no behavior layer and (usually) no responsive
layer — name what you're inferring in a short `Assumptions:` list before code:
interaction behavior, missing states, and every responsive decision not backed by a
grounded breakpoint frame. Keep it to what actually matters for this screen.

**7. Handle images, icons, and animations** (skip whichever flag is false).
   - Search the project for existing assets (`src/assets`, `public/`, an existing icon
     component set) before asking anything.
   - If the connected Figma MCP exposes an asset-download tool (e.g. `download_assets`)
     or serves a fetchable asset URL for the node, actually pull the bytes rather than
     only naming the asset — push automation as far as the tooling allows, and say
     plainly which assets still need manual export because no download path existed.
   - Ask once, for all images at once, how each should load: static import, `public/`
     URL, or reuse of a found existing asset. Every `<img>` gets real `alt` text (or
     `alt=""` if genuinely decorative) and `width`/`height` from grounded dimensions.
   - Icons: prefer the project's existing icon system (component set, sprite, library)
     over inlining fresh SVG — inline only when no system exists, and say so.
   - Animations: ask for the Lottie/rive filename or motion spec; default to a plain
     CSS transition only where the design clearly shows a simple state change.

**8. Translate AutoLayout to flexbox/grid**, using the grounded sizing mode — not the
canvas dimensions — to decide behavior. (Utility classes shown for the Tailwind case;
express the same properties in the project's actual styling approach per pre-flight.)

   | Figma AutoLayout | CSS / Tailwind |
   |---|---|
   | Direction: vertical / horizontal | `flex flex-col` / `flex flex-row` |
   | Wrap | `flex-wrap`, or CSS grid if the design is genuinely a grid |
   | Overlapping, no auto-layout | `relative` parent + `absolute` children (last resort — check whether it's really a grid first) |
   | Gap | `gap-<token>` from the grounded value — per nesting level, see below |
   | Sizing: **Fill container** | `flex-1` / `w-full` / `self-stretch` — never a literal canvas width |
   | Sizing: **Hug contents** | No explicit size — intrinsic (`w-fit` only if needed to override) |
   | Sizing: **Fixed** | Literal `w-`/`h-` from the grounded value — the *only* case a literal dimension is correct |
   | Alignment: space-between | `justify-between` |
   | Padding/inset | `p-`/`px-`/`py-` from the grounded token |

   **Don't flatten nested gaps into one compromise value.** Figma output mixes
   incidental absolute `x`/`y` canvas positions (ignore for layout) with actual spacing
   decisions (`gap`/`padding`). Read every gap between siblings at *every* nesting level
   and reproduce each as its own container's `gap`/padding — tight spacing inside a
   sub-group and looser spacing between groups needs matching nested containers, not one
   flex parent with an averaged value.

   Avoid absolute positioning for anything auto-layout already expresses. Avoid
   translating Figma's fixed frame height into a CSS `height` on content containers —
   web content flows; a grounded fixed height is for genuinely fixed elements (an
   avatar, an icon button), not a screen.

**9. Handle missing interaction and write the code.**
   - Typed props: an explicit `interface`/`type` per component, no `any`, optional
     handler props with safe defaults (`onClick?: () => void`) and a specific
     `// TODO:` for anything Figma can't express — never guessed business logic.
   - If a state clearly must exist (loading/empty/error, disabled-on-invalid-input) but
     Figma only shows the happy path, name it in assumptions rather than picking one.
   - No state library gets introduced that pre-flight didn't confirm — local
     `useState`/`useReducer` is the ceiling for an undesigned screen.
   - Next.js projects: components stay server components unless they genuinely need
     state/effects/browser APIs — add `"use client"` only then, and say why.
   - Standard React/web hygiene: semantic HTML over div soup, real `<button>`/`<a>` for
     interactive elements (never a clickable `div`), visible focus states, labels tied
     to inputs, `aria-*` only where semantics don't already cover it, stable `key`s,
     functional components with explicit return types, 2-space indent.
   - **Check every Figma-derived identifier before using it.** Layer names become prop
     and component names — check them against JS reserved words, existing exports in the
     target directory, and React/DOM-reserved props (`key`, `ref`, `className`,
     `children`, `style`). A layer literally named "Children" or "Style" mapped straight
     to a prop is a runtime bug, not a style issue. Re-scan generated code for this
     right before presenting it.
   - Every user-facing string goes through the project's i18n convention if pre-flight
     confirmed one — never guess a translation-file path; if there's no i18n, plain
     strings are fine and say so.
   - One primary component named after the Figma frame (PascalCase); extract genuinely
     reusable sections into their own components; keep files focused. Check whether
     target files already exist before writing — ask before overwriting.

### Phase C — Validate (don't skip this — it's the step most drafts of this workflow omit)

**10. Check the generated code against the screenshot from step 3**, not just against
how plausible it reads on its own — a model has no innate visibility into its own
rendered output, so this has to be a deliberate pass, not an assumption. Use the
checklist in `references/validation-checklist.md`: layout/spacing/alignment, typography,
colors traced to grounded tokens in the project's canonical representation, interactive
states, accessibility, responsive claims vs. grounded breakpoint frames, and —
critically — that every asset import actually resolves. If the environment allows it,
prefer real checks over reading: `tsc --noEmit`, the project's linter, and — best of
all — actually rendering the component in the dev server and comparing against the
screenshot. A type error caught here beats one caught by the developer.

**11. Close with a completion summary** — files written, i18n keys added (or "none"),
which assets still need manual export, which responsive decisions remain assumptions,
and concrete next steps (wire data in, register the route). Nothing left implicit.

## Output format

1. **Plan** (Phase A, step 5) — mapping table, gaps, breakpoint situation, flags. Skip
   if already confirmed earlier in the conversation.
2. **Assumptions** (step 6).
3. **The code** — full file(s), file paths as headers if split.
4. **Validation notes** (step 10) — anything the checklist caught, or confirmation it's
   clean.
5. **Completion summary** (step 11).

Don't pad the response with a restated design description — the plan, code, and
assumptions speak for themselves.

## Failure awareness

Full tables in `references/failure-modes.md` (this skill's own) and
`figma-grounding/references/failure-modes.md` (inherited, not repeated here). The
highest-severity ones to recognize on sight: a component name that sounds plausible but
doesn't exist in the real codebase; Tailwind classes written into a non-Tailwind project
because `get_design_context`'s default output was pasted through; a literal canvas
dimension hardcoded where grounding said the node was Fill or Hug; responsive behavior
invented from a single fixed-width frame without a flag; Framelink output silently
standing in for grounded values; and code that was never actually checked against the
screenshot from step 3 before being called done.

## What this skill does NOT do

- **Does not fully automate asset export.** It pushes automation as far as the
  connected MCP's asset tools allow; whatever isn't fetchable remains a manual step —
  and it says so explicitly rather than implying otherwise.
- **Does not decide the project's styling approach, token representation, or state
  management for it.** Reads all three from the project's context file, or generates
  one via pre-flight — never defaults silently.
- **Does not design responsive behavior.** It implements grounded breakpoint frames
  when they exist and flags the gap when they don't — inventing a mobile layout is a
  design decision, not a codegen step.
- **Does not replace a real build or browser check.** Typecheck/lint and, where
  available, a rendered comparison are run — full test coverage and cross-browser QA
  remain the developer's job.
- **Does not replace `figma-grounding`.** Assumes grounding already ran for the values
  in play; if invoked standalone, treat every size/color/spacing value as unverified.
