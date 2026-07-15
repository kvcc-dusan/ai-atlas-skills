# figma-to-react — known failure signatures

v1 — this skill inherits every signature in
`figma-grounding/references/failure-modes.md`; that table isn't repeated here. This
file covers failures specific to the React/web codegen layer, drawn from reported field
experience (a LogRocket worked example of Figma-MCP React codegen, monday.com's
design-system-MCP writeup, Figma's own skill docs) plus the shared catalogue in
`figma-to-swiftui/references/failure-modes.md` where the failure is
framework-independent.

## Silent quality failures

| Signature | Cause | What it means |
|---|---|---|
| Generated code imports a component (`PrimaryButton`, `Card`, `TextField`) that doesn't exist anywhere in the target project | Model pattern-matched a plausible DS component name instead of confirming via `get_code_connect_map` or a real codebase search | The single most commonly reported failure across every team writeup — treat any unconfirmed component name as invented until proven otherwise |
| Tailwind utility classes in a project that uses CSS Modules / styled-components / vanilla CSS | `get_design_context` defaults to React + Tailwind output, and that output got pasted through instead of re-expressed in the project's actual styling approach | Pre-flight's styling question is blocking for exactly this reason; the tool's fluent default output is the trap |
| Grounded color emitted in the wrong token representation — raw `bg-[#0047AB]` where the project mandates `bg-primary-500`, or vice versa | Token representation is project-specific and wasn't pinned in pre-flight | Same value, three spellings, only one canonical per project — see `responsive-and-styling.md` |
| Responsive behavior written from a single fixed-width frame with no flag — e.g. an element that should be hidden on mobile stays visible | The frame describes one viewport; the model invented the rest. This exact miss is documented in LogRocket's own worked example — a real, reported failure, not a hypothetical | Responsive decisions without a grounded breakpoint frame are assumptions and must be listed as such |
| Two breakpoint frames exist but only one was grounded; the other's values were guessed as a delta | Grounding ran once "for the screen" instead of per-frame | Each breakpoint frame is grounded separately, always |
| A literal canvas width/height hardcoded where grounding said Fill or Hug | Sizing mode grounded but ignored during translation | Cross-check every literal `w-`/`h-`/`width:` against the grounded sizing mode |
| Figma frame height translated into a CSS `height` on a content container | Canvas dimension mistaken for a design decision | Web content flows; fixed heights are for genuinely fixed elements only |
| Component variant silently mapped to the "closest" defined prop value | Figma variant combination has no matching prop/union member; nearest picked instead of surfacing the gap | Same discipline as `figma-grounding`: never treat proximity as a match |
| A state library (Redux, Zustand, React Query) appears in generated code that pre-flight never confirmed | Model defaulted to a commonly-seen pattern | Local `useState`/`useReducer` is the ceiling for an undesigned screen |
| Asset import compiles-looking (`import bg from './hero.png'`) but the file was never fetched or confirmed to exist | Asset was *named* from the layer, no download tool was attempted, gap unstated | Always state which assets were fetched vs. still need manual export |
| **Framelink output standing in for grounded values** | The connected server is `figma-developer-mcp` (tools: `get_figma_data`, `download_figma_images` — no `get_variable_defs`, no Code Connect), and its JSON got treated as if it were the official grounding output | Grounding as this repo defines it cannot run against Framelink. The tool absence is loud; the danger is the model quietly substituting `get_figma_data` and *looking* grounded. Pre-flight step 0 exists for this — if official tool names are missing, say so and stop treating values as grounded |
| **Code presented as done without ever being checked against the screenshot** | Phase C skipped or treated as implicit | Highest-severity entry in this file — "typechecks and looks plausible" and "matches the design" are different claims |

## Loud-but-walk-into-able traps

| Signature | Cause | What it means |
|---|---|---|
| Runtime warning or broken render from a prop named `key`, `ref`, `children`, `style`, or `className` | A Figma layer literally named "Children", "Style", etc. mapped straight to a prop name | Check every Figma-derived identifier against React/DOM-reserved props and JS reserved words before using it |
| Every list item re-mounts on data change; console warns about keys | `.map()` emitted with index keys or none | Stable keys from the data, not the index |
| Next.js build error: hook/browser API in a server component | `"use client"` decision never made consciously | Framework flavor is a pre-flight item; add the directive only where state/effects genuinely require it, and say why |
| Nested groups with visibly different spacing flattened into one `gap` value | Gaps read once per screen instead of per sibling-pair per nesting level | Each distinct gap gets its own container — a compromise value is wrong for both sections it averaged |
| Multi-style text run rendered as one flat-styled paragraph | Differently styled spans collapsed to the dominant style | Nested spans / DS text primitives preserve per-span styling |
| Interactive element built as a clickable `div` | Layer tree has no semantics; model mirrored the div soup | Real `<button>`/`<a>`, keyboard-reachable, visible focus |

## Screen-complexity failure mode

Very large or deeply nested frames (many variants, mixed scroll behavior, heavy
nesting) can overload one MCP call, produce an unreadably long mapping table, or need a
second generation pass. If a frame is outgrowing one pass: split into logical sections
(header, list, footer) and run the skill per section. Same guidance as the SwiftUI
sibling — this is a workflow property, not a framework one.

## Debugging order

1. Confirm which MCP server is connected (official vs. Framelink) — pre-flight step 0.
2. Confirm `figma-grounding` actually ran and its values are in use, not re-guessed.
3. Confirm the project context file was found or generated — if pre-flight was skipped,
   every styling/state/component assumption downstream is suspect.
4. Check component imports against the real codebase before trusting they resolve.
5. Confirm Phase C actually happened against the screenshot, per breakpoint frame.
6. If a screen is complex, split it into sections rather than debugging one giant pass.
